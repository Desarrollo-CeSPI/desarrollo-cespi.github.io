---
layout: post
title: Elasticsearch, base de datos distribuida
author: Maira Diaz
categories: elasticsearch DDBMS
tags: [elasticsearch, cluster, node, shard, replica, DDBMS]
---



##MODULO 2

Una de las características que poseen las BBDD NoSQL es que son **BBDD distribuidas**.
Como se va  a manejar grandes volúmenes de datos, necesitaremos particionar el conjunto de
datos.
El  **Particionamiento**  divide de forma lógica a una base de datos, reubicandola en diferentes 
entidades físicas. El particionamiento mejora el *rendimiento*, *manejabilidad* y 
*disponibilidad* de los datos, y ayuda a reducir el coste total de propiedad para almacenar
grandes volúmenes de datos.

### Partición horizontal y partición vertical

En DDBMS(1) existen diferentes técnicas para particionar y luego almacenar los datos:

![Partición Horizontal vs Vertical](/assets/images/elasticsearch-modules/horizontal_vs_vertical_split_DDBMS.gif)


Veamos un ejemplo de **partición horizontal** y **partición vertical**, haciendo referencia 
al ejemplo mencionado en el módulo 2.

####1.Diagrama de clases 

![Contact DB](/assets/images/elasticsearch-modules/contact_db.png){: style="height: 300px;width: 350px;"}

id  |     name    |         email             |     type      | parent_id
--- | ----------- | ------------------------- | ------------- | --------
1   | UNLP        | email@unlp.edu.ar         | Institution   |
2   | CeSPI       | cespi@unlp.edu.ar         | Dependency    |    1
3   | Aula Cisco  | cisco@cespi.unlp.edu.ar   | Office        |    2
4   | Guaraní     | guarani@cespi.unlp.edu.ar | Office        |    2



####2.Fragmentación horizontal

2.a Fragmento 1

id  |     name    |         email             |     type      |parent_id
--- | ----------- | ------------------------- | ------------- | --------
1   | UNLP        | email@unlp.edu.ar         | Institution   |
2   | CeSPI       | cespi@unlp.edu.ar         | Dependency    |    1

2.b Fragmento 2

id  |     name    |         email             |     type  |parent_id
--- | ----------- | ------------------------- | --------- | ---------
3   | Aula Cisco  | cisco@cespi.unlp.edu.ar   | Office    |    2
4   | Guaraní     | guarani@cespi.unlp.edu.ar | Office    |    2



####3.Fragmentación vertical

3.a Fragmento 1


id|     name    |         email
--|-------------|--------------------------
1 | UNLP        | email@unlp.edu.ar
2 | CeSPI       | cespi@unlp.edu.ar
3 | Aula Cisco  | cisco@cespi.unlp.edu.ar
4 | Guaraní     | guarani@cespi.unlp.edu.ar

3.b Fragmento 2


id  |     type      | parent_id
--- | ------------- | --------
1   | Institution   |
2   | Dependency    |    1
3   | Office        |    2
4   | Office        |    2

Elasticsearch utiliza **partición horizontal**. Este tipo de partición se denomina **Sharding**.
Es útil y recomendable contar con mecanismos de conmutación por error en caso de que un 
nodo o shard fallen o se desconecten por cualquier razón. Elasticsearch permite realizar 
una o más copias de cada *shard*. A cada copia se la denomina **Replica**.

### Shards y replicas

Cree un índice como se detalla a continuación: 

{% highlight bash  %}
$ curl -XPUT 'http://localhost:9200/articles/' -d '
index :
    number_of_shards : 4
    number_of_replicas : 1
'
{% endhighlight  %}

Para visualizarlo acceda a:

> [http://localhost:9200/articles/_settings](http://localhost:9200/articles/_settings)

Se creó un *índice* con los siguientes datos:

* nombre del índice: **articles**
* cantidad de shards: 4
* cantidad de replicas: 1 (1 replica  por shard)

El siguiente dibujo representa un bosquejo de como se particionarían los datos

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

Para visualizar en que nodo se distribuyeron los shards y replicas del índice 
**articles**, acceda a:

> [http://localhost:9200/_cat/shards/articles](http://localhost:9200/_cat/shards/articles)

Si, por ejemplo, nuestro cluster contara con 3 nodos, la distribución de shards y 
replicas, se podrían realizar de la siguiente forma:

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

¿Qué ventaja tiene esta forma de distribuir los datos? Nos permite **paralelizar**
las operaciones a través de los shards aumentando de esta forma el rendimiento.

Cada replica es una copia de un shard, y por lo tanto cada replica contiene información
redundante.

¿Qué ventaja proveen las replicas? Proporcionan **disponibilidad** de los datos en caso 
de en caso de que un fragmento falle.

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
EXPLICAR COMO SE DISTRIBUYEN y acceden a LOS DATOS de cada shard. COMO ES EL MECANISMO DE PARTICIONAMIENTO 
https://www.elastic.co/guide/en/elasticsearch/guide/current/distrib-write.html
The node uses the document’s _id to determine that the document belongs to shard 0.
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

### Cluster y nodos

Elasticsearch se va a organizar en 
[clusters](https://www.elastic.co/guide/en/elasticsearch/reference/current/_basic_concepts.html#_cluster),
conjunto de 1 o más
[nodos](https://www.elastic.co/guide/en/elasticsearch/reference/current/_basic_concepts.html#_node)

Para visualizar el cluster al que usted pertenece:

>[http://localhost:9200/](http://localhost:9200/)

Los *n* nodos contienen la totalidad de los datos. Cada nodo almacena un subconjunto 
de esos datos y participa en la indexación y búsqueda.

Para visualizar los nodos de su cluster acceda a: 

> [http://localhost:9200/_cat/nodes](http://localhost:9200/_cat/nodes)

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
EXPLICAR SOBRE NODOS MASTER O DEDICADOS ....
https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html
https://www.elastic.co/guide/en/elasticsearch/reference/1.4/modules-node.html
By default, each node is considered to be a data node, and it can be turned off by setting node.data to false.
This is a powerful setting allowing to create 2 types of non-data nodes: dedicated master nodes and client nodes.
1. client nodes
2. dedicated nodes
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++


### Índices, tipos y documentos

Como se mencionó en el módulo 1, un 
[índice](https://www.elastic.co/guide/en/elasticsearch/reference/current/_basic_concepts.html#_index)
es una colección de documentos que poseen características similares. Dentro de un cluster, 
se pueden definir la cantidad de índices que se deseen.

Por cada índice, se pueden definir 1 o más 
[tipos](https://www.elastic.co/guide/en/elasticsearch/reference/current/_basic_concepts.html#_type)
Un tipo es una categorización o partición semántica.

Un [documento](https://www.elastic.co/guide/en/elasticsearch/reference/current/_basic_concepts.html#_document)
es una unidad básica de información que puede ser indexado. Los documentos se representan 
con el formato [JSON](http://json.org/). Dentro de un índice se pueden almacenar tantos
documentos como se desee. La búsqueda se realiza sobre los documentos.

Elasticsearch provee la característica de subdividir un índice en múltiples partes o
[shards](https://www.elastic.co/guide/en/elasticsearch/reference/current/_basic_concepts.html#_shards_amp_replicas).
Cada shard es en sí mismo un índice y se puede alojar en cualquier nodo del cluster.

Para visualizar los *índice* con sus respectivos shards y replicas que residen en el cluster
acceda a: [http://localhost:9200/_settings](http://localhost:9200/_settings) 

En el siguiente módulo, se explicará como crear un índice, como se realizar una búsqueda, y de 
esta forma se comprenderá  con mayor profundidad los términos tipos y documento.


------------------------------------------------------

(1)*DDBMS*: Distributed Database Management Systems


