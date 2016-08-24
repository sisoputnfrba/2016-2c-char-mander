# Anexo I - Otro Sistema Académico de Archivos (OSADA)

El OSADA es un filesystem creado con propósitos académicos para que el alumno se interiorice y comprenda el funcionamiento básico de la gestión de archivos en un sistema operativo asemejando características de sistemas de archivos modernos.

* Tamaño máximo de disco soportado: 4GB
* Tamaño máximo de archivo: 4GB
* Soporte de directorios y subdirectorios
* Tamaño máximo de nombre de archivo: 17 caracteres
* Cantidad máxima de archivos/directorios: 2048

## Características técnicas

* Tamaño de bloque: 64 bytes
* Tamaño de la tabla de archivos: 1024 bloques
* Tabla de bloques libres: Bitmap
* Estructura de gestión: Tabla de asignación. Bloques enlazados.

## Arquitectura

Cada bloque en el sistema de archivos estará direccionado por un puntero de 4 bytes `ptrOBloque` permitiendo así un máximo de 2^32 bloques.

Para un disco de tamaño T [bytes] y el BLOCK_SIZE de 64 las estructuras se definen de esta manera:

![Arquitectura OSADA](/assets/image05.png)

> Por ejemplo: si tengo un filesystem de 1,584 KB tengo 25344 bloques de 64B… ¿Cuánto ocuparía el bitmap?
>
> Hagamos cuentas: Para representar 25344 bloques necesito 25344 bits. Esa cantidad termina resultando en 3168 bytes (Si 8 bits = 1 byte, entonces 25344 bits = 25344/8 = 3168 bytes).
>
> 3168 bytes resultan en 49.5 bloques. Como no podemos ocupar “49 bloques y medio” el bitmap ocupará 50 bloques.

### Header

El encabezado del sistema de archivos estará almacenado en el primer bloque. Contará con los siguientes campos:

| Campo | Tamaño | Valor |
|-------|--------|-------|
| Identificador | 7 bytes | `OsadaFS` (sin `\n`) |
| Versión | 1 byte | 1 |
| Tamaño del FS [bloques] | 4 bytes | T / BLOCK_SIZE |
| Tamaño del Bitmap [bloques] | 4 bytes | N: Tamaño del FS [bloques] / 8 / BLOCK_SIZE |
| Inicio Tabla Asignaciones [bloque] | 4 bytes | 1 + N + 1024 |
| Tamaño Datos [bloques] | 4 bytes | F - 1 - N - 1024 - A |
| Relleno | 40 bytes | [no definido] |

### Bitmap

El Bitmap, también conocido como bitvector, es un array de bits en el que cada bit identifica si el bloque ubicado en su posición se encuentra ocupado (1) o no (0).

Por ejemplo, un bitmap con este valor 000001010100 nos indicaría que los bloques 5, 7 y 9 están ocupados y los restantes libres (recordar que la primer ubicación es la 0).

Cuando necesite localizar un bloque libre, el filesystem utiliza esta estructura para encontrar uno, lo marca como ocupado y lo utiliza.

Es importante recalcar que los bloques utilizados por las estructuras administrativas deben siempre estar marcados como utilizados.

### Tabla de Archivos

La tabla de archivos es un array de tamaño fijo de 2048 posiciones de estructuras de tipo `osadaFile`.
Cada estructura `osadaFile` consta de los siguientes campos:

| Campo | Tamaño | Valor |
|-------|--------|-------|
| Estado | 1 byte | 0: Borrado, 1: Ocupado, 2: Directorio |
| Nombre de archivo | 17 bytes | Nombre del archivo |
| Bloque padre | 2 bytes | Posición del array de la tabla de archivos donde está almacenado el directorio padre o `0xFFFF` si está en el directorio raíz |
| Tamaño del archivo | 4 bytes | Tamaño total del archivo en bytes |
| Fecha última modificación | 4 bytes | Timestamp de la última modificación[^6] |
| Bloque inicial | 4 bytes | Ubicación del primer elemento en la tabla de asignaciones |

### Tabla de Asignaciones

La tabla de asignaciones es una estructura que permite identificar los bloques que componen cada archivo.

Es un array de enteros donde cada posición representa un bloque de datos y el valor almacenado en dicha posición indica el próximo bloque.

El bloque de inicio de cada archivo es un campo de la estructura `osadaFile` de la tabla de archivos.
Supongamos que sabemos que un archivo empieza en el bloque 8, vamos a la posición 8 de la tabla de asignaciones y ahí nos dirá el próximo bloque (supongamos el 12) y sucesivamente.

Se ve en el diagrama a continuación que nuestro archivo está compuesto por los bloques de datos 8, 12, 13 y 15. _Recuerde que los arrays comienzan en cero._

También a modo de ejemplo, si se nos indicara que un archivo empieza en el bloque 17, podríamos inferir que el archivo se compone de los bloques de datos, 17, 4 y 2

Ejemplo:

| | | `FFFFFFFF` | | `2` |
|-|-|-|-|-|
| | | | `12` | |
| | | `13` | `15` | |
| `FFFFFFFF` | | `4` | | |

## Osada Utils - Herramientas y estructuras

Se le proveerán al alumno dos herramientas para simplificar el desarrollo del trabajo práctico:
* `osada-format`: Recibe como parámetro un archivo e inicializa un filesystem OSADA dentro del mismo.
* `osada-dump`: Imprime por pantalla el estado de las estructuras administrativas de un filesystem OSADA.

Además, se publicarán, también, las definiciones de las estructuras básicas de un filesystem OSADA, a modo de cumplimentar la especificación.
Para acceder a estos, consultar el repositorio https://github.com/sisoputnfrba/osada-utils


---
[^6] Ver función `time()`