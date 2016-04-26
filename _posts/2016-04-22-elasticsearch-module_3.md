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

1. Fragmentación/partición (2)
2. Réplica

El  **Particionamiento**  divide de forma lógica a una base de datos, reubicandola en diferentes 
entidades físicas. El particionamiento mejora el rendimiento, manejabilidad y disponibilidad de los datos, y
ayuda a reducir el coste total de propiedad para almacenar grandes volúmenes de datos.

La partición puede ser horizontal o vertical:

![Partición Horizontal vs Vertical](/assets/images/elasticsearch-modules/horizontal_vs_vertical_split_DDBMS.gif)


Veamos un ejemplo de **partición horizontal** y **partición vertical**, haciendo referencia 
al ejempolo mencionado en el módulo 2.

####1.Diagrama de clases 

![Contact DB](/assets/images/elasticsearch-modules/contact_db.png){: style="height: 300px;width: 350px;"}

{% highlight bash %}
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


####2.Fragmentación horizontal

2.a Fragmento 1
{% highlight bash %}
+---+-------------+---------------------------+---------------+---------+
|id |     name    |         email             |     type      |parent_id|
+---+-------------+---------------------------+---------------+---------+
| 1 | UNLP        | email@unlp.edu.ar         | Institution   |         |
+---+-------------+---------------------------+---------------+---------+
| 2 | CeSPI       | cespi@unlp.edu.ar         | Dependency    |    1    |
+---+-------------+---------------------------+---------------+---------+
{% endhighlight %}

2.b Fragmento 2
{% highlight bash %}
+---+-------------+---------------------------+---------------+---------+
|id |     name    |         email             |     type      |parent_id|
+---+-------------+---------------------------+---------------+---------+
| 3 | Aula Cisco  | cisco@cespi.unlp.edu.ar   | Office        |    2    |
+---+-------------+---------------------------+---------------+---------+
| 4 | Guaraní     | guarani@cespi.unlp.edu.ar | Office        |    2    |
+---+-------------+---------------------------+---------------+---------+
{% endhighlight %}


####3.Fragmentación vertical

3.a Fragmento 1
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

3.b Fragmento 2
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

Elasticsearch utiliza **partición horizontal**. Este tipo de partición se denomina **Sharding**.
Cuando la base de datos se particiona en  *n* fragmento, por cada fragmento  se crea una 
copia.  o **réplica**


## Shards & Replicas

Veamos un ejemplo donde se tienen un conjunto de datos y se particiona de la siguiente manera:

1. Se dividen en 4 partes o **shards**: *Shard_1*, *Shard_ 2*, *Shard_3*, *Shard_4*.

2. Por cada shard, se hace una copia o **replica** de cada parte: 
*Replica_1*, *Replica_2*, *Replica_3*, *Replica_4*.


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
                |       NODO pc_1   |--  REPLICA 3
                |
                |
CLUSTER --------|           __
Elasticsearch   |_________ |__|_____|-- SHARD 3
                |         /___/     |-- REPLICA 1
                |      NODO pc_2    |-- REPLICA 4
                |
                |           __
                |_________ |__|_____|-- SHARD 4
                          /___/     |-- REPLICA 2
                       NODO pc_3
{% endhighlight %}

Que ventaja tiene esta forma de distribuir los datos? Nos permite **paralelizar**
las operaciones a través de los *shards* aumentando de esta forma el rendimiento.

Que ventaja proveen las *replicas*? Las réplicas proporcionan **disponibilidad** de 
los datos en caso de en caso de que un fragmento falle.

## Cluster y nodos

Elasticsearch se va a organizar en 
[clusters](https://www.elastic.co/guide/en/elasticsearch/reference/current/_basic_concepts.html#_clustero){: style="color:#6383d7;font-weight:bold;"} ,
conjunto de 1 o más 
[nodos](https://www.elastic.co/guide/en/elasticsearch/reference/current/_basic_concepts.html#_node){: style="color:#6383d7;font-weight:bold;"}
Los *n nodos* contienen la totalidad de los datos. Cada *nodo* almacena un subconjunto 
de esos datos y participa en la indexación y búsqueda.

## Índices, tipos y documentos

Como se mencionó en el módulo 1, un 
[índice](https://www.elastic.co/guide/en/elasticsearch/reference/current/_basic_concepts.html#_index){: style="color:#6383d7;font-weight:bold;"} 
es una colección de documentos que tienen características similares. Dentro de un *cluster*, 
se pueden definir la cantidad de *índices* que se deseen.

Por cada *índice*, se pueden definir 1 o más 
[tipos](https://www.elastic.co/guide/en/elasticsearch/reference/current/_basic_concepts.html#_type){: style="color:#6383d7;font-weight:bold;"}
Un tipo es una categorización o partición semántica.

Un [documento](https://www.elastic.co/guide/en/elasticsearch/reference/current/_basic_concepts.html#_document){: style="color:#6383d7;font-weight:bold;"}
es una unidad básica de información que puede ser indexado. Los *documentos* se representan 
con el formato [JSON](http://json.org/). Dentro de un *índice* se pueden almacenar tantos
*documentos* como se desee. La búsqueda se realiza sobre los *documentos*.

En el siguiente módulo, se explicará como crear un *índice*, como se realizar una búsqueda, y de 
esta forma se comprenderá  con mayor profundidad los términos *índice*, *tipos* y *documento*

Elasticsearch provee la característica de subdividir un *índice* en múltiples partes o
[shards](https://www.elastic.co/guide/en/elasticsearch/reference/current/_basic_concepts.html#_shards_amp_replicas){: style="color:#6383d7;font-weight:bold;"}.
Cada *shard* es en sí mismo un *índice* y se puede alojar en cualquier *nodo* del *cluster*.




------------------------------------------------------

(1)*DDBMS*: Distributed Database Management Systems

(2)*Fragmentación/partición*: ambos términos serán utilizados indistintamente
