# Proceso PokeDex

La configuración del sistema junto con cierta información de estado estará almacenada de manera centralizada en el proceso PokeDex, siguiendo la estructura de un árbol de directorios de un filesystem. Todos los procesos del sistema accederán a dicha información mediante un punto de montaje y la actualizarán cuando corresponda utilizando su instancia del "Cliente del PokeDex".

Al ser el PokeDex un filesystem distribuído, este componente estará compuesto por un proceso servidor que atiende las peticiones de manera concurrente y mantiene las estructuras administrativas, y un cliente que permitirá montar dicho filesystem en los diversos equipos remotos.

![Arquitectura PokeDex](/assets/image01.png)

## Servidor PokeDex

Este proceso gestionará un Filesystem en formato Otro Sistema Académico de Archivos (OSADA) (ver [Anexo I - Otro Sistema Académico de Archivos](anexo-i---otro-sistema-académico-de-archivos-osada.md)) almacenado en un archivo binario y se lo presentará a los diversos procesos PokeDex Cliente que se conecten.

La implementación deberá permitir que se pueda:

* Leer archivos
* Crear archivos
* Escribir y modificar archivos
* Borrar archivos
* Crear directorios y subdirectorios.
* Borrar directorios vacíos
* Renombrar archivos
Se deberá tener especial cuidado en proteger con semáforos las estructuras administrativas del sistema de archivos ya que se pretende que el proceso pueda recibir solicitudes de manera concurrente de varios procesos Pokedex Cliente.

## Proceso Cliente del PokeDex

Este proceso funcionará como interfaz entre las solicitudes que un proceso o el Sistema Operativo quieran realizar sobre el punto de montaje y el PokeDex.

Al ser iniciado obtendrá la IP y Puerto del PokeDex por variable de entorno, se conectará y quedará montado en el directorio recibido por argumento.

Todas aquellas solicitudes que se realicen sobre ese punto de montaje, por ejemplo listar archivos, las reenviará al Pokedex para que este le devuelva el resultado correspondiente, de manera análoga al cliente de un filesystem basado en la nube como Dropbox o Google Drive, teniendo en cuenta que no se almacenará ningún contenido en la computadora que ejecute el cliente.

Para simplificar el desarrollo de la interacción con el sistema operativo el alumno deberá implementar el cliente utilizando la biblioteca FUSE la cual ejecutará en modo single thread.

### Comandos recomendados
* `md5sum`: devuelve la reducción criptográfica MD5 del contenido de un archivo. Si este fuera modificado aunque sea un bit, el valor se modificaría.
* `touch`: crea un archivo vacío (`create`)
