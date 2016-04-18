---
layout: post
title: Intro Elasticsearch integrado con Ruby on Rails 
author: Maira Diaz
categories: elasticsearch
---

Esto es una introducción a elasticsearch. Tendrá n módulos, y cada uno de los n_i módulos
tienen una duración de 4 horas de aprendizaje.
Se calcula que este post se aprendería en n días

### MÓDULO 1


## Elasticsearch

Página principal: [https://www.elastic.co/](https://www.elastic.co/)


Elasticsearch es un servidor de búsqueda basado en Lucene.
Provee un motor de búsqueda de texto completo, distribuido y con capacidad de multi-tenencia con una interfaz web RESTful y con documentos JSON. 
Elasticsearch está desarrollado en Java y está publicado como código abierto bajo las condiciones de la licencia Apache.


## Instalando Elasticsearch

{% highlight bash  %}
wget https://download.elastic.co/elasticsearch/elasticsearch/elasticsearch-1.7.2.deb
sudo dpkg -i elasticsearch-1.7.2.deb
{% endhighlight %}

iniciando el servicio:

{% highlight bash  %}
sudo service elasticsearch start
{% endhighlight %}

Para probar si elasticsearch está funcionando: 

[http://localhost:9200/](http://localhost:9200/)


## Primer ejemplo de búsqueda
1. Baja el proyecto desde gitlab:   **Agenda Telefónica Backend**
2. Inicia el servicio de lasticsearch
{% highlight bash  %}
sudo service elasticsearch start
{% endhighlight %}
3. Installa las gemas
{% highlight bash  %}
bundle
{% endhighlight %}
4. Crea y carga la base de datos con los ejemplos.
{% highlight bash  %}
bundle exec rake phonebook:init_all
{% endhighlight %}
5. Visualize la base de datos. Esto crea un pdf con nombre **unlp_agenda_telefonica_db.pdf**
{% highlight bash  %}
bundle exec erd  --attributes=foreign_key,content,inheritance --inheritance --polymorphism --filename=unlp_agenda_telefonica_db.pdf
{% endhighlight %}
6. Instala plugin para visualizar los datos indexados
{% highlight bash  %}
elasticsearch/bin/plugin -install mobz/elasticsearch-head
{% endhighlight %}

8. [Abrir plugin elasticsearch](http://localhost:9200/_plugin/head/)

7. Dentro del plugin, ir a **Browser**, elegir el índice **contact_index**


Que fue lo que paso? 

**Los datos de la base de datos fueron indexados, y ahora están en un documento de texto con formato JSON**

Porque? 

**Porque la búsqueda no se realizará con consultas sql a la base de datos; la búsqueda se realizará sobre el documento json que generó elasticsearch: contact_index**


### MODULO 2

## Entendiendo el mapeo: de base de datos a documento indexado

Lo que se tiene que realizar es el **mapeo** de la base de datos al documento que se genera para la busqueda.

La siguiente imagen es un ejemplo de mapeo

![Mapeo de Base de datos a Indice elasticsearch](/assets/images/elasticsearch/db_to_elasticsearch_index_mapping.png)


# En la base de datos

Oficina Infraestructura y Redes

![Oficina Infraestructura y Redes](/assets/images/elasticsearch/office_id_8_db.png)

Teléfonos de la oficina

![Teléfonos de la Oficina](/assets/images/elasticsearch/office_id_8_telephones_db.png)

Ancestros: CeSPI y UNLP

![Dependencia Cespi](/assets/images/elasticsearch/dependency_id_1_db.png)

![Institución UNLP](/assets/images/elasticsearch/institution_id_2_db.png)


Para poder visualizarlo desde la consola ejecute lo siguiente
{% highlight bash %}
$ cd agenda_telefonica_backend
$ bundle exec rails console
> Office.find(8)
> Office.find(8).telephones
> Dependency.find(1)
> Institution.find(2)
{% endhighlight %}

###En elasticsearch
Ejemplo de mapeo **contact_index** : 
![indice: contact_index , name: Infraestructura y redes, id: 8](/assets/images/elasticsearch/elasticsearch_office_8.png)

Para poder visualizarlo mejor dirijase a:
[http://localhost:9200/contact_index/_search?q=_id:8](http://localhost:9200/contact_index/_search?q=_id:8)







++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
El cluster va a tener un conjunto de computadoras. Cada computadoras es un nodo.
![Ejemplo Cluster Elasticsearch](/assets/images/elasticsearch/elasticsearch_cluster.svg)

La *computadora 1* contiene una parte del documento donde se tiene que indexar: *shard 1*, *shard 3*.
La *computadora 2* contiene la otra parte del documento a indexar: *shard 2* , *shard 4*.
Para mayor seguridad, se poseen replicas o copias, pero no en el mismo nodo, si no que en otro nodo.

shard: subdivide your index into multiple pieces called shards. Each shard is in itself a fully-functional and independent "index" that can be hosted on any node in the cluster


![Componentes de Elasticsearch](/assets/images/elasticsearch_components.svg)

indices: An index is a collection of documents that have somewhat similar characteristics.
For example, you can have an index for customer data, another index for a product catalog, and yet another index for order data

*******************************************************************************************
proximo: explicar que es un shard, una replica.
https://www.elastic.co/guide/en/elasticsearch/guide/current/replica-shards.html
https://www.elastic.co/guide/en/elasticsearch/reference/current/_basic_concepts.html
mostrar un ejemplo desde una db, indexarlo, mostrar el doc indexado
http://www.elasticsearchtutorial.com/elasticsearch-in-5-minutes.html
*******************************************************************************************



## Elasticsearch  con Ruby on Rails
Se puede integrar elasticsearch con activerecord y ruby on rails
https://github.com/elastic/elasticsearch-rails



def self.es_find(ids)
      $es_client.search index: index_name, type: type_name, body: {
        query: {
          ids: {
            values: ids
          }
        }
      }
    end
