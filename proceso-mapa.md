# Proceso Mapa

Cada instancia del proceso Mapa será encargado de gestionar la interacción entre los diversos procesos Entrenador y los Pokémons disponibles en el Mapa.

Para lograrlo, será indispensable el uso de estructuras que reflejen qué Entrenadores están listos para moverse, cuáles se encuentran esperando capturar un Pokémon y cuáles han finalizado anormalmente (víctimas de interbloqueos, finalizados por muerte, etc).

Los diversos Entrenadores ingresarán al mapa en busca de determinados Pokémons, los cuales se encuentran en las diversas PokeNest. El Entrenador solicitará la ubicación de la PokeNest que tiene el Pokémon que está buscando y se dirigirá al mismo. Al llegar, solicitará una instancia del Pokémon al Mapa el cual le será otorgado siempre y cuando hubiera disponibles. Cada PokeNest tendrá Pokémons de una única especie.

El Mapa tendrá un único punto de acceso por socket y, luego de realizar un intercambio de mensajes inicial (_handshake_), delegará cada conexión al hilo responsable.

El proceso Mapa al ser ejecutado **obtendrá de la línea de comandos su nombre y la ruta del Pokédex**. En el directorio `/Mapas` del Pokédex existirá un subdirectorio con el nombre de cada Mapa, por lo que **el proceso deberá leer su archivo de metadata y los diversos subdirectorios dentro del directorio PokeNests para así poder conocer los recursos que tiene disponibles y crearlos**.

## Hilo Planificador

El hilo Planificador será el encargado de contestar las peticiones de los Entrenadores que actúan dentro del Mapa, ordenándolos **según un algoritmo de planificación de corto plazo Round Robin o SRDF** (ver [Algoritmos de Planificación](#algoritmos-de-planificaci-n)).

Gestionará una cola de Listos y una cola de Bloqueados. Ante cada modificación de estas colas, el Planificador **deberá informar por archivo de log la modificación realizada, y la lista de los Entrenadores en cada una de las colas**.

Mientras haya Entrenadores listos, el Planificador seleccionará a cuál atenderá según el algoritmo de planificación activo. Una vez seleccionado, el Planificador contestará una por una las peticiones que el Entrenador le haga, hasta que este solicite capturar un Pokemon, o hasta que el algoritmo de planificación indique expropiarlo.

Las posibles operaciones a atender son:

1. **Conocer la ubicación de una PokeNest** - El Mapa contestará al Entrenador las coordenadas en que se encuentra la PokeNest de un Pokémon determinado
1. **Moverse en alguna dirección** - El Mapa registrará dicho movimiento y contabilizará el uso de una unidad de tiempo al Entrenador
1. **Capturar un Pokémon** - El Planificador moverá a dicho personaje a la cola de bloqueados. Descartará, si quedara, el quantum de tiempo restante del Entrenador y planificará al siguiente que se encuentre listo.

Eventualmente, el Planificador podrá detectar que un Entrenador se desconectó (cumplió los objetivos del Mapa, o murió). Cuando esto ocurra, deberá liberar todos los Pokémons que éste había capturado y, si correspondiera, otorgarlos a los Entrenadores bloqueados por ellos, desbloqueándolos. Al mismo tiempo, el Planificador eliminará al Entrenador de sus listas de Planificación.

El hilo Planificador tendrá un **algoritmo de planificación, valor de quantum[^3] y tiempo de retardo entre turnos**[^4] que será notificado por el Mapa al iniciar.Cuando reciba la señal `SIGUSR2`, el proceso Mapa deberá releer su archivo de Metadata, actualizando estos parámetros durante la ejecución. Junto con los parámetros del Planificador mencionados, el Mapa deberá actualizar, también, el **tiempo de chequeo de interbloqueo**, descrito a continuación.

### Algoritmos de Planificación

El Planificador podrá ir alternando el algoritmo de planificación entre Round Robin (con quantum configurable) y Shortest Remaining Distance First (SRDF) a medida que se relea su archivo de configuración. Es válido esperar a finalizar la ráfaga de ejecución actual antes de efectivizar el cambio de algoritmo.

El SRDF es un algoritmo que sigue dos reglas:

1. Atiende una única operación del primer Entrenador Listo que no conozca su distancia a la próxima PokeNest (ya sea por recién haberse conectado, o por haber recién capturado un Pokemon); o, si no hubiera ninguno,
1. Atiende todas las operaciones que necesite el Entrenador Listo con menor distancia a su PokeNest destino, hasta que éste se bloquee por solicitar capturar un Pokemon.

Cualquiera de estas dos opciones cuenta como una ráfaga de ejecución, por lo que, para poder volver a ejecutar una siguiente ráfaga, el Entrenador tendrá primero que pasar por el final de la cola de Listos. En otras palabras, si el algoritmo de planificación cambiara de SRDF a RR mientras un Entrenador está consultando la ubicación de su próxima PokeNest al haber sido elegido por la regla 1 del SRDF, ese Entrenador irá al final de la cola de Listos, afectando su orden según Round Robin.

La distancia de un Entrenador a una PokeNest se mide en cantidad de movimientos que debería realizar el Entrenador - es decir, sin contar pasos en diagonal.

## Detección de deadlock

Cada instancia del proceso Mapa deberá analizar periódicamente la existencia de un interbloqueo mediante una frecuencia de chequeo llamada **tiempo de chequeo de interbloqueo**. En caso de detectarlo, informará por archivo de log dicha situación, **indicando los Entrenadores involucrados en el interbloqueo y las tablas utilizadas para su detección**.

Adicionalmente, si se encontrara activado por archivo de configuración el modo _batalla_, seleccionará y mostrará por archivo de log el Entrenador que fue seleccionado como víctima para resolver dicha situación.

Es importante aclarar que **la ejecución del algoritmo de interbloqueo no deberá interrumpir el funcionamiento habitual del Mapa**, excepto en aquellos casos en que ambos requieran acceder a los mismos recursos compartidos.

## Resolución del deadlock

En caso de detectar un deadlock, el Mapa de manera ordenada según el tiempo de ingreso al mapa les notificará a los Entrenadores involucrados en el interbloqueo que deben elegir su Pokémon más fuerte, es decir, al Pokémon de mayor nivel, para la batalla.

Luego, el mapa deberá efectuar la simulación de una Batalla Pokémon enfrentando entre sí a los Pokémons, de a dos, dado el orden según el tiempo de ingreso del Entrenador.

El Pokémon perdedor de la batalla, enfrentará en un Encuentro al Pokémon del Entrenador siguiente. Finalmente, el Entrenador cuyo Pokémon haya perdido en la última batalla deberá ser elegido como víctima para solucionar el deadlock. Cada vez que se efectúe un Encuentro, el mapa deberá notificar a los Entrenadores el resultado de dicho encuentro.

> Ejemplo:
>
> Entrenadores ordenados según tiempo de ingreso:
>
> &nbsp;
>
> Entrenador 1: Pikachu Nivel 10
>
> Entrenador 2: Squirtle Nivel 5
>
> Entrenador 3: Rhyhorn Nivel: 100
>
> &nbsp;
>
> Pikachu Vs Squirtle => Perdedor: Squirtle
>
> Squirtle Vs Rhyhorn => Perdedor: Squirtle
>
> &nbsp;
>
> => El Entrenador 2 será elegido como víctima.

Para simplificar el algoritmo de batalla Pokémon, **la cátedra proveerá una biblioteca[^5] para la realización del Encuentro entre 2 Pokémons** y evitar el cálculo innecesario de los factores de efectividad de los ataques. Un ejemplo práctico de aplicación de la misma se detalla en el [Anexo III - Algoritmo de Batalla](anexo-iii---algoritmo-de-batalla.md).

## Atrapar un Pokémon

Cada PokeNest tendrá una cantidad limitada de Pokémons de una determinada especie. Estos se representarán como archivos en un directorio.

Cuando un entrenador atrapa un Pokémon, copiará como prueba de dicha operación el archivo de metadata a su Directorio de Bill.

## Dibujado del Mapa

Para mostrar el mapa, **la cátedra proveerá una biblioteca encargada de dibujar en pantalla el Mapa, los Entrenadores y las PokeNests**. La misma se encuentra descrita en el [Anexo II - Interfaz Gráfica](anexo-ii---interfaz-gráfica.md).

## Archivos y directorios del Mapa

### Metadata

* **Nombre**: `metadata`
* **Tipo:**: Archivo
* **Descripción**: archivo donde están almacenados los parámetros del mapa
* **Ruta**: `/Mapas/[nombre]/metadata`
* **Ejemplo de Ruta: `/Mapas/Ciudad Paleta/metadata`
* **Contenido**:
    * Tiempo de chequeo de interbloqueo (en ms)
    * Batalla (on/off)
    * Algoritmo de planificación
    * Quantum
    * Retardo entre quantums (en ms)
    * IP/Puerto en que recibirá conexiones de Entrenadores
* **Ejemplo**:

```
TiempoChequeoDeadlock=10000
Batalla=1
algoritmo=RR
quantum=3
retardo=500
IP=127.0.0.1
Puerto=5001
```

### Medalla
* **Nombre**: Medalla del Mapa
* **Tipo:**: Archivo
* **Descripción**: medalla del Mapa, que se otorgará a cada Entrenador que lo concluya
* **Ruta**: `/Mapas/[nombre]/medalla-[nombre].jpg`
* **Ejemplo de Ruta: `/Mapas/Ciudad Paleta/medalla-Ciudad Paleta.jpg`
* **Contenido**: no determinado

### PokeNest
* **Nombre**: PokeNest
* **Tipo:**: Directorio
* **Descripción**: directorio que representa y contiene la información de una PokeNest. Puede haber más de uno por Mapa.
* **Ruta**: `/Mapas/[nombre]/PokeNests/[nombre-de-PokeNest]/`
* **Ejemplo de Ruta: `/Mapas/Ciudad Paleta/PokeNests/Pikachu/`

### Metadata de un PokeNest
* **Nombre**: `metadata`
* **Tipo:**: Archivo
* **Descripción**: información de la PokeNest
* **Ruta**: `/Mapas/[nombre]/PokeNests/[nombre-de-PokeNest]/metadata`
* **Ejemplo de Ruta**: `/Mapas/Ciudad Paleta/PokeNests/Pikachu/metadata`
* **Contenido**:
    * Tipo de Pokemon
    * Posición X;Y de la PokeNest
    * Caracter identificador
* **Ejemplo de contenido**:
```
Tipo=Electrico
Posicion=23;18
Identificador=P
```

### Metadata de un Pokemon
* **Nombre**: Metadata de un Pokemon
* **Tipo:**: Archivo
* **Descripción**: información de un Pokemon dentro de la PokeNest
* **Ruta**: `/Mapas/[nombre]/PokeNests/[PokeNest]/[PokeNest]NNN.dat`
* **Ejemplos de Ruta**:
    * `/Mapas/Ciudad Paleta/PokeNests/Pikachu/Pikachu001.dat`
    * `/Mapas/Ciudad Paleta/PokeNests/Pikachu/Pikachu002.dat`
    * `/Mapas/Ciudad Paleta/PokeNests/Bulbasaur/Bulbasaur001.dat`
* **Contenido**:
    * Nivel del Pokemon
    * Imagen del Pokemon en Ascii Art
* **Ejemplo de Contenido**:
```
Nivel=33
[Ascii Art]
```

## Restricciones del Mapa

* No hay una cantidad definida de PokeNests. El proceso Mapa al iniciar debe recorrer el árbol y crear las instancias correspondientes.
* No se instanciarán dos Mapas con el mismo nombre en el sistema.
* Las PokeNests deberán estar dentro de los márgenes del Nivel y no podrán superponerse.
* Los archivos necesarios para ejecutar una simulación serán provistos siempre en su totalidad y sin errores de sintaxis o semántica. Es correcto considerar un error abortivo la carencia o error de alguno de ellos.

---
[^3] El valor de quantum aplica únicamente para el algoritmo de RR

[^4] Valor en milisegundos que cada planificador de manera independiente deberá esperar entre cada asignación de turno

[^5] https://github.com/sisoputnfrba/so-pkmn-utils
