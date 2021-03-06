---
layout: post
title: 2. Elasticsearch, base de datos distribuida
usernames: [ maira1001001, rosariosm]
categories: elasticsearch DDBMS
tags: [elasticsearch, cluster, node, shard, replica, DDBMS]
---

Una de las características que poseen las **BBDD NoSQL** es que son **distribuidas**.
<!-- more -->
Como se van a manejar grandes volúmenes de datos, necesitaremos particionar el conjunto de datos.
El  **particionamiento**  divide de forma lógica a una base de datos, reubicándola en diferentes entidades físicas. Este proceso mejora el *rendimiento*, *manejabilidad* y *disponibilidad* de los datos, además ayuda a reducir el costo total de propiedad para almacenar grandes volúmenes de datos.



## Partición horizontal y vertical

En DDBMS(1) existen diferentes técnicas para particionar y luego almacenar los datos:

![Partición Horizontal vs Vertical](/assets/images/elasticsearch-modules/horizontal_vs_vertical_split_DDBMS.gif)


Veamos un ejemplo de **partición horizontal** y **partición vertical**


id  |     name    |         email             |     type      | parent_id
--- | ----------- | ------------------------- | ------------- | --------
1   | UNLP        | email@unlp.edu.ar         | Institution   |
2   | CeSPI       | cespi@unlp.edu.ar         | Dependency    |    1
3   | Aula Cisco  | cisco@cespi.unlp.edu.ar   | Office        |    2
4   | Guaraní     | guarani@cespi.unlp.edu.ar | Office        |    2




### 1.Fragmentación horizontal

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


### 2.Fragmentación vertical

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

Elasticsearch utiliza **partición horizontal**. Este tipo de partición se denomina **Sharding**. Es útil y recomendable contar con mecanismos de conmutación por error en caso de que un nodo o shard fallen o se desconecten por cualquier razón. Elasticsearch permite realizar una o más copias de cada *shard*. A cada copia se la denomina **réplica**.

## Shards y réplicas

Veamos un ejemplo de creación de un índice donde se configurarán sus *shards* y *réplicas*.
Escriba en la consola:

{% highlight bash  %}
$ curl -XPUT 'http://localhost:9200/contacts/?pretty' -d '
index :
    number_of_shards : 4
    number_of_replicas : 1
'
{% endhighlight  %}

El índice creado contará con los siguientes datos:

* Nombre: **contacts**
* Número de shards: 4
* Número de réplicas: 1 (una por cada shard)


Para chequear si se ha creado correctamente, escriba en la consola:

{% highlight bash  %}
$ curl -XGET 'http://localhost:9200/contacts/_settings?pretty'
{% endhighlight  %}

El siguiente dibujo representa un bosquejo de cómo se podrían particionar los datos del índice creado: 

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


La finalidad de las particiones es que se distribuyan en diferentes nodos para incrementar la eficiencia en la búsqueda y asegurar la escalabilidad.

Si, por ejemplo, nuestro cluster contara con 3 nodos, la distribución de shards y réplicas se podría realizar de la siguiente forma:

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


Tanto la réplica como su respectivo shard deben residir en diferentes nodos. ¿Por qué ocurre esto? Si el shard y su réplica se ubicaran en el mismo nodo, y este fallara o se desconectara, entonces el shard y su réplica se perderían. De esta forma, la réplica no cumpliría con una de sus funciones principales: la de servir como copia al shard ante un fallo.

La **distribución de los shards** ocurre cuando se inicializa el servicio, cuando se agrega o se elimina un nodo, durante la locación de las réplcias o durante un rebalanceo.



### _cat APIs: shards

El conjunto de **_cat** APIs permite visualizar diferentes relaciones que se encuentran en los datos. En particular, el parámetro *shard* permite ver, de forma detallada, qué nodos contienen qué shards, si son réplicas o shards primarios, los bytes que ocupan en el disco y dónde se encuentra el nodo que los contiene.
Para visualizar en el índice **contacts** esta información, escriba en la consola:

{% highlight bash  %}
$ curl -XGET 'http://localhost:9200/_cat/shards/contacts?v'
{% endhighlight  %}

Volviendo al ejemplo arriba mencionado, donde teníamos tres nodos en el cluster "Elasticsearch", podriamos imaginar el siguiente esquema:

{% highlight bash  %}

index    shard prirep state   docs store ip        node
contacts 0     p      STARTED    0   79b 127.0.1.1 Oesterheld
contacts 0     r      STARTED    0   79b 127.0.1.1 Solano López
contacts 1     p      STARTED    0   79b 127.0.1.1 Oesterheld
contacts 1     r      STARTED    0   79b 127.0.1.1 Walsh
contacts 2     p      STARTED    0  115b 127.0.1.1 Solano López
contacts 2     r      STARTED    0   79b 127.0.1.1 Oesterheld
contacts 3     p      STARTED    0  115b 127.0.1.1 Walsh
contacts 3     r      STARTED    0   79b 127.0.1.1 Solano López
{% endhighlight  %}


> NOTA
> "prirep": hace referencia a "primary/replica", identificando con la letra "p" a los nodos "primary", y con "r" a los nodos "replica".

¿Qué ventaja tiene esta forma de distribuir los datos? Nos permite **paralelizar** las operaciones a través de los shards aumentando de esta forma el rendimiento.

¿Qué ventaja proveen las replicas? Proporcionan **disponibilidad** de los datos en caso de en caso de que un fragmento falle.


## Cluster y nodos

Elasticsearch opera en un ambiente distribuido y corre en un [cluster](https://www.elastic.co/guide/en/elasticsearch/reference/current/_basic_concepts.html#_cluster) definido como una colección de uno o más [nodos](https://www.elastic.co/guide/en/elasticsearch/reference/current/_basic_concepts.html#_node) conectados.

Para visualizar el cluster al que usted pertenece, escriba en la consola:

{% highlight bash  %}
$ curl -XGET 'http://localhost:9200'
{% endhighlight  %}

Cada nodo dentro de un cluster, sirve para uno o más de los siguientes propósitos:

1. **Data node**: contienen los datos y ejecutan operaciones (CRUD, búsqueda, agregaciones). Por defecto un nodo es considerado como *data node*
2. **Client node**: son balanceadores de carga. No son *data node*  ni *master node*. Pueden redireccionar operaciones a los nodos que contienen los datos relevantes sin tener que preguntar a todos los nodos.
3. **Master node**: posee el control del cluster. Es responsable de, por ejemplo, crear o eliminar índices, identificar que nodos son parte del cluster, reubicar los shards dentro de los nodos.
4. **Tribe node**: es un tipo de *client node*, que se conecta con múltiples cluster. Permite realizar búsqueda y operaciones de lectura/escritura sobre los clusters conectados.

### _cat APIs: nodes

Para visualizar los nodos de su cluster, usando [cat nodes API](https://www.elastic.co/guide/en/elasticsearch/reference/current/cat-nodes.html), escriba en su consola:

{% highlight bash  %}

$ curl -XGET 'http://localhost:9200/_cat/nodes?v&h=host,ip,port,nodeRole,master,name'
{% endhighlight  %}

Si volvemos al ejemplo mencionado anteriormente, donde teníamos tres nodos en el cluster "Elasticsearch", podriamos imaginar el siguiente esquema:

{% highlight bash  %}
 host  ip         port  nodeRole master name
 pc_1  127.0.1.1  9200  d           *      Oesterheld
 pc_2  127.0.1.1  9200  d           *      Solano López
 pc_3  127.0.1.1  9200  d           m      Walsh
{% endhighlight  %}

> NOTA
> "nodeRole": los nodos pueden ser "Data node" o "Client node". Si es "Data node" se identifica con la letra "d". Si es "Client node" se identifica con la letra "c".

## Índices, tipos y documentos

Como se mencionó en el módulo 1, un [índice](https://www.elastic.co/guide/en/elasticsearch/reference/current/_basic_concepts.html#_index) es una colección de documentos que poseen características similares. Dentro de un cluster, se pueden definir la cantidad de índices que se deseen.

Por cada índice, se pueden definir uno o más [tipos](https://www.elastic.co/guide/en/elasticsearch/reference/current/_basic_concepts.html#_type). Un tipo es una categorización o partición semántica.

Un [documento](https://www.elastic.co/guide/en/elasticsearch/reference/current/_basic_concepts.html#_document) es una unidad básica de información que puede ser indexado. Los documentos se representan con el formato [JSON](http://json.org/). Dentro de un índice se pueden almacenar tantos documentos como se desee. La búsqueda se realiza sobre los documentos.

Elasticsearch provee la característica de subdividir un índice en múltiples partes o [shards](https://www.elastic.co/guide/en/elasticsearch/reference/current/_basic_concepts.html#_shards_amp_replicas). Cada shard es en sí mismo un índice y se puede alojar en cualquier nodo del cluster.

Para visualizar los *índices* con sus respectivos shards y réplicas que residen en el cluster, escriba en consola:

{% highlight bash %}
$ curl -XGET 'http://localhost:9200/_settings?pretty'
{% endhighlight %}

En el siguiente módulo, se explicará como crear un índice, cómo se debe realizar una búsqueda, y de esta forma se comprenderá con mayor profundidad los términos *tipos* y *documento*.



------------------------------------------------------


(1)*DDBMS*: Distributed Database Management Systems

