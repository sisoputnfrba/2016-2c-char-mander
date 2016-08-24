# Proceso Entrenador

Como vimos anteriormente el objetivo del proceso Entrenador es el de completar su Hoja de Viaje. Existirá una instancia de este proceso por cada entrenador que se encuentre participando del sistema. **No hay un límite de entrenadores ni un momento definido donde el entrenador puede ingresar o abandonar la simulación.**

Además de tener como atributos un nombre, un identificador y una cantidad inicial de vidas, cada entrenador tendrá asociada su _Hoja de Viaje_. La misma será una lista de los mapas que deberá completar de manera **secuencial y ordenada**. Tendrá definido también los _objetivos por mapa_: una **lista ordenada** de los Pokémons que deberán ser obtenidos para poder completar dicho Mapa.

![Configuracion de un Entrenador](/assets/image02.png)

El proceso Entrenador obtendrá de la línea de comandos su nombre y la ruta del Pokédex, por ejemplo: `./entrenador Ash /mnt/pokedex`

En el Pokédex existirá un subdirectorio con el nombre de cada Entrenador, por lo que el proceso deberá leer su archivo de metadata del Pokédex y así conocer su Hoja de Viaje.

La Hoja de Viaje especificará la lista de Mapas que el Entrenador deberá recorrer. Para comenzar su aventura Pokémon, el Entrenador obtendrá la dirección IP y puerto del primer Mapa leyendo el correspondiente _archivo de metadata_. Una vez establecida la conexión, el Entrenador quedará esperando a cada aviso del Mapa de que es su turno de realizar una acción. En este momento, el Entrenador, según el estado en que se encuentre:

1. **Solicitará** al mapa **la ubicación** de la PokeNest del próximo Pokémon que desea obtener, en caso de aún no conocerla.
1. **Avanzará una posición** hacia la siguiente PokeNest[^1], informando el movimiento al Mapa, en caso de aún no haber llegado a ella.
1. Al llegar a las coordenadas de una PokeNest, **solicitará** al mapa **atrapar un Pokémon**. El Entrenador quedará bloqueado hasta que el Mapa le confirme la captura del Pokémon. En ese momento, el Entrenador evaluará (e informará al Mapa) si debe continuar atrapando Pokémons en este Mapa, o si ya cumplió sus objetivos en el mismo.

Tras informarle al Mapa que cumplió sus objetivos, el Entrenador **copiará la medalla del Mapa a su directorio**, se desconectará del mismo, y procederá a repetir toda la operatoria con el siguiente Mapa de su Hoja de Viaje.

Una vez cumplidos los objetivos del último Mapa de su Hoja de Viaje, el Entrenador se habrá convertido en un _Maestro Pokémon_. El proceso Entrenador informará el logro por pantalla, indicando el Tiempo Total que le tomó toda la aventura, cuánto tiempo pasó bloqueado en las PokeNests, en cuántos Deadlocks estuvo involucrado, y cuántas veces Murió durante la hazaña (ver apartado siguiente).

## Muerte del Entrenador y vidas

Un Entrenador puede ser elegido como víctima en el caso de generarse un interbloqueo (más información en la sección “Mapa”). En este caso, el Entrenador mostrará por pantalla el motivo de su muerte, borrará todos los archivos en su Directorio de Bill, se desconectará del Mapa y este le expropiará todos los Pokémons que había capturado, pudiendo estos ser otorgados a algún otro Entrenador en espera.

En caso de tener vidas disponibles, el Entrenador se descontará una vida y volverá a conectarse al Mapa para reiniciar su objetivo.
Además, el Entrenador puede **recibir vidas** a través del envío de la señal `SIGUSR1` al proceso, y puede **perder vidas** mediante la señal `SIGTERM`. En estos casos, el desempeño del Entrenador en cada uno de los Mapas no se verá afectado, exceptuando el caso en que no tuviera vidas disponibles.

**Si no le quedaran vidas disponibles**, el Entrenador deberá mostrar un mensaje preguntando al usuario si desea reiniciar el juego, informando también la cantidad de reintentos que ya se realizaron. De aceptar, el Entrenador incrementará su contador de reintentos y reiniciará su Hoja de Viaje, **borrando también todas las medallas y Pokémons conseguidas hasta el momento**. En caso negativo, el Entrenador se cerrará, abandonando el juego.
Adicionalmente, el proceso Entrenador podría ser interrumpido por el administrador (`kill`, CTRL+C, etc) o por una situación anómala. En este caso, **el sistema deberá reaccionar de forma favorable** asumiendo que el Entrenador abandonó el juego.

## Archivos y directorios del Entrenador

### Medallas
* Nombre: `medallas`
* Tipo: Directorio
* Descripción: Ubicación donde el Entrenador copiará la medalla correspondiente a cada Mapa que concluya
* Ruta: `/Entrenadores/[nombre]/medallas`

### Directorio de Bill
* Nombre: `Dir de Bill`
* Tipo: Directorio
* Descripción: Ubicación donde el Entrenador copiará el archivo de metadata de cada Pokémon que capture
* Ruta: `/Entrenadores/[nombre]/Dir de Bill/`

### Metadata

* Nombre: `metadata`
* Tipo: Archivo
* Descripción: configuración del Entrenador, donde está almacenada información como su Hoja de Viaje
* Ruta: `/Entrenadores/[nombre]/metadata`

El archivo de metadata del proceso Entrenador contendrá al menos los siguientes parámetros:

* **Símbolo**: caracter que representará visualmente al Entrenador en el Mapa
* **Hoja de Viaje**: lista de los Mapas que debe completar el Entrenador
* **Objetivos por Mapa**: por cada Mapa de su Hoja de Viaje, una lista de los Pokémons que deberá obtener de forma ordenada para completarlo
* **Vidas**: cantidad de vidas restantes
* **Reintentos realizados**: cantidad de veces que el Entrenador reintentó el juego

#### Restricciones del archivo de Metadata
* No hay una cantidad definida de Mapas, de objetivos por Mapa o de Pokémons por Mapa y por juego
* Los identificadores de Pokémon pueden reutilizarse en niveles diferentes
* Un Pokémon ubicado fuera de los márgenes del área de juego se considera un error
* Las PokeNests deben estar espaciadas al menos por dos posiciones en cada eje
* Dos Pokémons del mismo tipo de manera consecutiva en el objetivo de un Mapa se considera un error de sintaxis

> Ejemplo:
> 
> `obj[Mapa8]=[P,P,F]` (error!)
>
> `obj[Mapa8]=[P,F,P,F,P,F]` (ok!)

#### Ejemplo de archivo de Metadata de un Entrenador
```
nombre=Red
simbolo=@
hojaDeViaje=[PuebloPaleta,CiudadVerde,CiudadPlateada]
obj[PuebloPaleta]=[P,B,G]
obj[CiudadVerde]=[C,Z,C]
obj[CiudadPlateada]=[P,M,P,M,S]
vidas=5
reintentos=0
```

---

[^1] El movimiento del personaje deberá ser, cuando fuera posible, alternando eje X y eje Y (uno y uno) de a una unidad por vez. No existe posibilidad de realizar movimientos en diagonal.