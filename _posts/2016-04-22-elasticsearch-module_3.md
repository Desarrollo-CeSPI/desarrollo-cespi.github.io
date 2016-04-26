---
layout: post
title: Elasticsearch, base de datos distribuida - Módulo 3
author: Maira Diaz
categories: elasticsearch cluster node shard replica
---

##MODULO 3

Una de las características que poseen las BBDD NoSQL es que son **BBDD distribuidas**.
Como se va  a manejar grandes volúmenes de datos, necesitaremos dividir el volumen de 
datos en varias partes. En DDBMS(1) existen diferentes técnicas para dividir y luego almacenar esos datos:

1. Fragmentación/partición
2. Réplica

Una  **Partición**  divide de forma lógica a una base de datos, reubicandola en diferentes 
entidades físicas. El particionamiento mejora el rendimiento, manejabilidad y disponibilidad de los datos, y
ayuda a reducir el coste total de propiedad para almacenar grandes volúmenes de datos.

La partición puede ser horizontal o vertical:

![Partición Horizontal vs Vertical](/assets/images/elasticsearch-modules/horizontal_vs_vertical_split_DDBMS.gif)

Ejemplo con la base de datos del módulo 2

![Contact DB](/assets/images/elasticsearch-modules/contact_db.png){: style="height: 300px;width: 350px;"}

{% highlight bash %}
detalle: anotar el id de "aula cisco" y de "guaraní" con el  id que corresponda

+---+-------------+---------------------------+---------------+---------+
|id |     name    |         email             |     type      |parent_id|
+---+-------------+---------------------------+---------------+---------+
| 1 | UNLP        | email@unlp.edu.ar         | Institution   |         |
+---+-------------+---------------------------+---------------+---------+
| 2 | CeSPI       | cespi@unlp.edu.ar         | Dependency    |    1    |
+---+-------------+---------------------------+---------------+---------+
| 3 | Aula Cisco  | cisco@cespi.unlp.edu.ar   | Office        |    2    |
+---+-------------+---------------------------+---------------+---------+
| 4 | Guaraní     | guarani@cespi.unlp.edu.ar | Office        |    2    |
+---+-------------+---------------------------+---------------+---------+
{% endhighlight %}

Fragmentación horizontal
--------------------
Fragmento 1
{% highlight bash %}
+---+-------------+---------------------------+---------------+---------+
|id |     name    |         email             |     type      |parent_id|
+---+-------------+---------------------------+---------------+---------+
| 1 | UNLP        | email@unlp.edu.ar         | Institution   |         |
+---+-------------+---------------------------+---------------+---------+
| 2 | CeSPI       | cespi@unlp.edu.ar         | Dependency    |    1    |
+---+-------------+---------------------------+---------------+---------+
{% endhighlight %}

Fragmento 2
{% highlight bash %}
+---+-------------+---------------------------+---------------+---------+
|id |     name    |         email             |     type      |parent_id|
+---+-------------+---------------------------+---------------+---------+
| 3 | Aula Cisco  | cisco@cespi.unlp.edu.ar   | Office        |    2    |
+---+-------------+---------------------------+---------------+---------+
| 4 | Guaraní     | guarani@cespi.unlp.edu.ar | Office        |    2    |
+---+-------------+---------------------------+---------------+---------+
{% endhighlight %}


Fragmentación vertical
--------------------
Fragmento 1
{% highlight bash %}
+---+-------------+---------------------------+
|id |     name    |         email             |
+---+-------------+---------------------------+
| 1 | UNLP        | email@unlp.edu.ar         |
+---+-------------+---------------------------+
| 2 | CeSPI       | cespi@unlp.edu.ar         |
+---+-------------+---------------------------+
| 3 | Aula Cisco  | cisco@cespi.unlp.edu.ar   |
+---+-------------+---------------------------+
| 4 | Guaraní     | guarani@cespi.unlp.edu.ar |
+---+-------------+---------------------------+
{% endhighlight %}

Fragmento 2
{% highlight bash %}
+---+---------------+---------+
|id |     type      |parent_id|
+---+---------------+---------+
| 1 | Institution   |         |
+---+---------------+---------+
| 2 | Dependency    |    1    |
+---+---------------+---------+
| 3 | Office        |    2    |
+---+---------------+---------+
| 4 | Office        |    2    |
+---+---------------+---------+
{% endhighlight %}

Elasticsearch utiliza **partición horizontal** o **sharding**.Cuando se particiona la base
de datos, se crean réplica o copias de cada partición.


##Ejemplo Shards
Se poseen un conjunto de datos que se los dividie y distribuye de la siguiente manera:

1. Se dividen en 4 partes o **shards**: *Shard 1*, *Shard 2*, *Shard 3*, *Shard 4*.

2. Por cada shard, se hace una copia o **replica** de cada parte: 
*Replica 1*, *Replica 2*, *Replica 3*, *Replica 4*.


{% highlight bash %}
BASE DE DATOS
+_________________________+           +_________________________+
|            _|       |   |           |            _|       |   |
|SHARD 1  __|        _|   |           |REPLICA 1__|        _|   |
|________|         _|     |           |________|         _|     |
|                _|       |  ------\  |                _|       |
|  SHARD 2    __|         |  ------/  |  REPLICA 2  __|         |
|____________|  | SHARD 4 |  copia    |____________|  |REPLICA 4|
|               |         |           |               |         |
|  SHARD 3      |         |           |  REPLICA 3    |         |
+_______________|_________+           +_______________|_________+

{% endhighlight %}

{% highlight bash %}
                            __
                 _________ |__|_____|--  SHARD 1
                |         /___/     |--  SHARD 2
                |                   |--  REPLICA 3
                |
                |
CLUSTER --------|           __
Elasticsearch   |_________ |__|_____|-- SHARD 3
                |         /___/     |-- REPLICA 1
                |                   |-- REPLICA 4
                |
                |           __
                |_________ |__|_____|-- SHARD 4
                          /___/     |-- REPLICA 2

{% endhighlight %}

Que ventajas tiene esta forma de distribuir los datos? 
Esta forma de organización ayuda a distribuir y **paralelizar** las operaciones a través de los
fragmentos o *shards* aumentando de esta forma el rendimiento.


## de lo práctico a la teórico

Elasticsearch se va a organizar en **CLUSTERS**, conjunto de 1 o más **nodos**. 
Los *n* nodos contienen la totalidad de los datos. Cada **nodo** almacena datos y participa 
en la indexación y búsqueda de datos. 

Como se mencionó en el módulo 1, un índice es una colección de documentos que tienen
características similares.

Seguir definiendo: index, type, document

relacionar index - shard,replica


------------------------------------------------------
(1)DDBMS: Distributed Database Management Systems
