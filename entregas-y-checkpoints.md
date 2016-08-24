# Entregas y checkpoints

A modo de guía para los alumnos se proponen una serie de puntos de control, cada uno con un conjunto de objetivos a desarrollar y un plazo en el que deberían estar finalizados.

## Primer Checkpoint - Entrega Obligatoria

**Tiempo estimado**: 2 semanas

**Fecha**: 10 de Septiembre

#### Hitos
* Estar familiarizado con el setup de las VMs, repositorio y el entorno de desarrollo
* Implementar la commons-library para el manejo de archivos de configuración, listas/colas y logs
* Desarrollar un proceso servidor de conexiones que soporte múltiples clientes (posteriormente hilo Planificador)
* Desarrollar un proceso cliente liviano que permita recibir conexiones de diversos clientes y transmitir mensajes paquetizados (Mapa)
* Implementar la biblioteca `libnivel.so` y desarrollar un proceso que permita dibujar elementos en pantalla (Mapa) investigando el código de ejemplo
* Desarrollar un código capaz de reconocer las estructuras administrativas de un filesystem OSADA

#### Recursos necesarios
* Configuración de un repositorio GitHub en Eclipse - [Link](https://youtu.be/nj_n02fz1lI)
* Video debug con Eclipse - [Link](https://youtu.be/XsefDXRfA9k)
* Video GIT Basic ([link](https://youtu.be/8O4LFYPY-Ww)) y GIT desde Eclipse ([link](https://youtu.be/s-r_-o8nxDw))
* Video creación una Shared Library - [Link](https://youtu.be/s5ac8CPDkMg)
* Guia Beej de Programación en redes - [Link](http://books.openlibra.com/pdf/Beej-Programacion-en-Redes.pdf)

## Segundo checkpoint

**Tiempo estimado**: 3 semanas

**Fecha**: 01 de Octubre

#### Hitos
* Comprender la lógica de FUSE y trabajar con el ejemplo.
* Desarrollar las funciones para leer el contenido del árbol de directorios del filesystem OSADA.
* Desarrollar la lógica del proceso Entrenador conectándose a los diversos procesos Mapa e interactuando con los mismos. Implementar los algoritmos de planificación.
* Desarrollar las estructuras del Mapa para manejar los Pokemones disponibles y asignados y los diversos estados de un Entrenador.
* Crear y representar mediante `libnivel.so` los elementos y personajes del Mapa.
* Activar rutinas en los procesos mediante el uso de señales.

#### Recursos necesarios
* SO FUSE Example - [Link](https://github.com/sisoputnfrba/so-fuse_example)
* Guia de FUSE por Joseph J. Pfeiffer (en inglés) - [Link](http://www.cs.nmsu.edu/~pfeiffer/fuse-tutorial/)
* Linux POSIX Threads (pthread) - [Link](http://www.yolinux.com/TUTORIALS/LinuxTutorialPosixThreads.html)
* Gcc, Makefile - [Link parte 1](https://youtu.be/DIY8O0vayUo) y [Link parte 2](https://youtu.be/C93u4HpZ5qI)

## Tercer checkpoint

**Tiempo estimado**: 2 semanas
**Fecha**: 15 de Octubre

#### Hitos
* Desarrollar el hilo que detecta interbloqueos (deadlocks) en el Mapa
* Desarrollar las operaciones de escritura y lectura de archivos de OSADA
* Integrar `readdir()` y `getattr()` en PokeDex
* Crear el proceso Cliente de Pokedex que permita realizar por un socket `readdir()` y `getattr()` sobre el PokeDex mediante mensajes paquetizados
* Desarrollar la lógica de muerte de un Entrenador y reiniciado de un Mapa

## Cuarto checkpoint

**Tiempo Estimado**: 2 semanas

**Fecha**: 29 de Octubre

#### Hitos

* Iniciar fase de testing de integración y stress
* Integrar las operaciones de lectura y escritura de archivos al Pokedex
* Permitir realizar operaciones de lectura y escritura desde el cliente de PokeDex

## Entregas

**Entrega final**: 19 de Noviembre

**Primer recuperatorio**: 3 de Diciembre

**Segundo recuperatorio**: 17 de Diciembre

> _Todas las fechas son estimativas y pueden ser modificadas con su debida notificación_