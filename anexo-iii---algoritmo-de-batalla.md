# Anexo III - Algoritmo de Batalla

Para determinar el ganador del Encuentro de una Batalla se diseñó un algoritmo simplificado, dada la complejidad de que tienen las batallas Pokémon. Dicho algoritmo consistirá en:

> Vence el Pokémon que tenga mayor en nivel, más un factor que dependerá de la efectividad del ataque ante el Pokémon contrario.

Dicho factor de efectividad será un valor entero correspondiente a los siguientes:

* `5`: para ataques super efectivos.
* `0`: para ataques comunes.
* `-5`: para ataques poco efectivos.
* `-20`: para ataques que no causen daño (ej, eléctrico a tierra).

Se deberá tener en cuenta lo siguiente:

* El tipo de los ataques de un Pokémon serán del tipo principal del Pokémon, a fines de simplificar el algoritmo.
* Si el Pokémon contrario tiene 2 tipos, se realizará la suma del factor de efectividad de cada tipo, con la sola excepción de que algún factor sea por _no causar daño_. En dicho caso _el factor de efectividad total será de no causar daño_, es decir, -20.

En caso de empate, ganará el Pokémon de mayor nivel (y en caso de empate, el ganador será el primer Pokémon, dado que empezó a atacar primero).

Por ejemplo:

> Pikachu [Electrico] Nivel: 30 Vs Rhyhorn [Tierra/Roca] Nivel: 6
>
> **Factor de Efectividad de Pikachu**: 30 + -20 (Sin Daño al Tierra y Daño Normal a Roca) = **10**
>
> **Factor de Efectividad de Rhyhorn**: 6 + 5 (Daño Doble) = **11**
>>
>
> Gana **Rhyhorn**

La cátedra implementó dicho algoritmo en una biblioteca compartida, que además de permitir efectuar una batalla, permite la creación de Pokémons a modo de simplificar aún más el funcionamiento de la biblioteca.

Dicha biblioteca, junto con un programa de ejemplo, se encuentran en el siguiente repositorio:

> https://github.com/sisoputnfrba/so-nivel-pkmn-utils.git

También se provee un breve tutorial en la wiki de dicho repositorio.