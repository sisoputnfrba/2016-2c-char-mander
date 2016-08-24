# Descripción General

![Esquema del sistema](/assets/image00.png)

El sistema consiste en una simulación de un juego de Pokémon, mediante la cual representaremos un pequeño sistema operativo capaz de planificar procesos asignándole recursos, administrar datos mediante un sistema de archivos, y comunicar sus componentes a través de una conexión de red.

Contaremos con distintos tipos de procesos para plantear la analogía: Entrenadores, Mapas y PokéDex. El proceso Entrenador se encargará de cumplir una Hoja de Viaje, que contendrá la información de distintos Mapas que deberá recorrer, junto con distintos Pokémons que deberá atrapar en cada uno, para cumplir sus objetivos.

Por su parte, distintos Entrenadores podrán jugar un mismo Mapa de forma concurrente. Será responsabilidad de cada Mapa administrar y planificar las operaciones de cada uno de los procesos Entrenador, implementando los distintos algoritmos de planificación que vemos en la teoría.

El proceso PokeDex estará encargado de administrar los datos del sistema. Tendrá información de cada uno de los Pokémon disponibles y la información de cada uno de los Entrenadores.

Este proceso implementará un Sistema de Archivos para poder realizar el almacenamiento.
Al tener varios Entrenadores jugando en un Mapa, podrían darse situaciones donde un Entrenador no pueda cumplir su objetivo, debido a que la instancia requerida esté retenida por otro Entrenador, generando una situación de interbloqueo (deadlock). En ese caso se deberá resolver el conflicto con una _Batalla Pokémon_.