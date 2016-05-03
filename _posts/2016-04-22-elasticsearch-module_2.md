---
layout: post
title: Elasticsearch, base de datos distribuida
author: Maira Diaz
categories: elasticsearch DDBMS
tags: [elasticsearch, cluster, node, shard, replica, DDBMS]
---


##MODULO 2

Una de las características que poseen las **BBDD NoSQL** es que son **BBDD distribuidas**.
Como se van a manejar grandes volúmenes de datos, necesitaremos particionar el conjunto de
datos.
El  **Particionamiento**  divide de forma lógica a una base de datos, reubicandola en diferentes 
entidades físicas. El particionamiento mejora el *rendimiento*, *manejabilidad* y 
*disponibilidad* de los datos, y ayuda a reducir el coste total de propiedad para almacenar
grandes volúmenes de datos.

### Partición horizontal y partición vertical

En DDBMS(1) existen diferentes técnicas para particionar y luego almacenar los datos:

![Partición Horizontal vs Vertical](/assets/images/elasticsearch-modules/horizontal_vs_vertical_split_DDBMS.gif)


Veamos un ejemplo de **partición horizontal** y **partición vertical**


id  |     name    |         email             |     type      | parent_id
--- | ----------- | ------------------------- | ------------- | --------
1   | UNLP        | email@unlp.edu.ar         | Institution   |
2   | CeSPI       | cespi@unlp.edu.ar         | Dependency    |    1
3   | Aula Cisco  | cisco@cespi.unlp.edu.ar   | Office        |    2
4   | Guaraní     | guarani@cespi.unlp.edu.ar | Office        |    2


####1.Fragmentación horizontal

Fragmento 1


id  |     name    |         email             |     type      | parent_id
--- | ----------- | ------------------------- | ------------- | --------
1   | UNLP        | email@unlp.edu.ar         | Institution   |
2   | CeSPI       | cespi@unlp.edu.ar         | Dependency    |    1

Fragmento 2

id  |     name    |         email             |     type  |parent_id
--- | ----------- | ------------------------- | --------- | ---------
3   | Aula Cisco  | cisco@cespi.unlp.edu.ar   | Office    |    2
4   | Guaraní     | guarani@cespi.unlp.edu.ar | Office    |    2


####2.Fragmentación vertical

Fragmento 1

id  |     name     |         email
--- | ------------ | --------------------------
1   | UNLP         | email@unlp.edu.ar
2   | CeSPI        | cespi@unlp.edu.ar
3   | Aula Cisco   | cisco@cespi.unlp.edu.ar
4   | Guaraní      | guarani@cespi.unlp.edu.ar

Fragmento 2

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

Veamos un ejemplo creando un índice y configurando sus shards y replicas.
Escriba en la consola:

{% highlight bash  %}
$ curl -XPUT 'http://localhost:9200/articles/?pretty' -d '
index :
    number_of_shards : 4
    number_of_replicas : 1
'
{% endhighlight  %}

Se ha creado un índice con los siguientes datos:

* nombre del índice: **article**
* cantidad de shards: 4
* cantidad de replicas: 1 (1 replica  por shard)


Para chequear si se ha creado correctamente, escriba en la consola:

{% highlight bash  %}
$ curl -XGET 'http://localhost:9200/articles/_settings?pretty'
{% endhighlight  %}

El siguiente dibujo representa un bosquejo de como se podrían particionar los datos respecto
del ejemplo arriba mencionado.

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


La finalidad de las particiones es que se distribuyan en diferentes nodos, para incrementar 
la eficiencia en la búsqueda y asegurar la escalabilidad.

Si, por ejemplo, nuestro cluster contara con 3 nodos, la distribución de shards y replicas 
se podrían realizar de la siguiente forma:

{% highlight bash %}
                           pc_1
                            __
                 _________ |__|______|--  SHARD 0
                |         /___/      |--  SHARD 1
                |  NODO "Oesterheld" |--  REPLICA 2
                |
                |
                |
                |
                |          pc_2
CLUSTER --------|           __
"Elasticsearch" |_________ |__|_______|-- SHARD 2
                |         /___/       |-- REPLICA 0
                | NODO "Solano López" |-- REPLICA 3
                |
                |
                |
                |          pc_3
                |           __
                |_________ |__|_____|-- SHARD 3
                          /___/     |-- REPLICA 1
                   NODO  "Walsh"

{% endhighlight %}


Dado un shard que reside en un nodo, su réplica no puede estar en el mismo nodo. 
¿Porqué no puede ocurrir esto? 
Si el shard y su réplica residieran en el mismo nodo, y el nodo fallara o se desconectara,
entonces el shard y su réplica se perderían.
De esta forma, la réplica no cumpliría con una de sus funciones originales, que es servir
como copia al shard ante un fallo.

La **distribución de los shards** ocurre cuando se inicializa el servicio, cuando se agrega o 
se elimina un nodo, durante la locación de las réplcias o durante un rebalanceo.


####cat Shards API

Para visualizar la distribución de los shards y replicas del índice **articles**, escriba 
en la consola:

{% highlight bash  %}
$ curl -XGET 'http://localhost:9200/_cat/shards/articles?v'
{% endhighlight  %}

Volviendo al ejemplo arriba mencionado, donde teníamos 3 nodos en el cluster "Elasticsearch",
podriamos imaginar el siguiente esquema:

{% highlight bash  %}

index    shard prirep(*) state   docs store ip        node
articles 0     p         STARTED    0   79b 127.0.1.1 Oesterheld
articles 0     r         STARTED    0   79b 127.0.1.1 Solano López
articles 1     p         STARTED    0   79b 127.0.1.1 Oesterheld
articles 1     r         STARTED    0   79b 127.0.1.1 Walsh
articles 2     p         STARTED    0  115b 127.0.1.1 Solano López
articles 2     r         STARTED    0   79b 127.0.1.1 Oesterheld
articles 3     p         STARTED    0  115b 127.0.1.1 Walsh
articles 3     r         STARTED    0   79b 127.0.1.1 Solano López
{% endhighlight  %}

(*)prirep: primary/replica

¿Qué ventaja tiene esta forma de distribuir los datos? Nos permite **paralelizar**
las operaciones a través de los shards aumentando de esta forma el rendimiento.

¿Qué ventaja proveen las replicas? Proporcionan **disponibilidad** de los datos en caso 
de en caso de que un fragmento falle.



### Cluster y nodos

Elasticsearch opera en un ambiente distribuido, y corre un
[clusters](https://www.elastic.co/guide/en/elasticsearch/reference/current/_basic_concepts.html#_cluster),
definido como una colleción de 1 o más
[nodos](https://www.elastic.co/guide/en/elasticsearch/reference/current/_basic_concepts.html#_node)
conectados.

Para visualizar el cluster al que usted pertenece, escriba en la consola:

{% highlight bash  %}
$ curl -XGET 'http://localhost:9200'
{% endhighlight  %}


Cada nodo conoce a los otros nodos del cluster. Pero, ¿Qué rol tienen los 
[nodos](https://www.elastic.co/guide/en/elasticsearch/reference/2.3/modules-node.html)
dentro del cluster? Cada nodo sirve para uno o más  de los  propósitos, como se detalla a continuación:

1. **Data node**: contienen los datos y ejecutan operaciones (CRUD, búsqueda, agregaciones).
Por defecto un nodo es considerado como *data node*
2. **Client node**: son balanceadores de carga. No son *data node*  ni *master node*.
Pueden redireccionar operaciones a los nodos que contienen los datos relevantes sin tener que
preguntar a todos los nodos.
3. **Master node**: posee el control del cluster. Es responsable de, por ejemplo, crear o
eliminar índices, identificar que nodos son parte del cluster, reubicar los shards dentro 
de los nodos.
4. **Tribe node**: es un tipo de *client node*, que se conecta con múltiples cluster. 
Permite realizar búsqueda y operaciones de lectura/escritura sobre los clusters
conectados.

#### Cat nodes API

Para visualizar los nodos de su cluster, usando
[cat nodes API](https://www.elastic.co/guide/en/elasticsearch/reference/current/cat-nodes.html),
escriba en su consola:

{% highlight bash  %}
$ curl -XGET 'http://localhost:9200/_cat/nodes?v&h=host,ip,port,nodeRole,master,name'
{% endhighlight  %}

Si volvemos al ejemplo arriba mencionado, donde teníamos 3 nodos en el cluster "Elasticsearch",
podriamos imaginar el siguiente esquema:

{% highlight bash  %}
 host  ip         port  nodeRole(*) master name
 pc_1  127.0.1.1  9200  d           *      Oesterheld
 pc_2  127.0.1.1  9200  d           *      Solano López
 pc_3  127.0.1.1  9200  d           m      Walsh
{% endhighlight  %}

(*)nodeRole: Data node (d); Client node (c)



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
