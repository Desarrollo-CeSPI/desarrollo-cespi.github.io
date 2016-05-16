---
layout: post
title: 3. Elasticsearch, realizando la primera búsqueda
usernames: [maira1001001, rosariosm]
tags: [elasticsearch, bulk API, search API]
---

En este módulo veremos cómo crear un índice, agregarle un conjunto de datos provenientes de un JSON válido  <!-- more --> y realizar una búsqueda simple mediante la API de búsqueda.


## Creando un índice

En este módulo trabajaremos con ejemplos de artículos de diario. Antes de comenzar necesitamos crear el índice **article_index** para almacernarlos. 

{% highlight bash %}
$ curl -XPUT 'http://localhost:9200/article_index'
{% endhighlight %}

Para visualizar el índice creado, escriba en consola la siguiente consulta:

{% highlight bash %}
$ curl -XGET 'http://localhost:9200/article_index?pretty'
{% endhighlight %}

El resultado será lo siguiente:

{% highlight bash %}
{
  "article_index" : {
    "aliases" : { },
    "mappings" : { },
    "settings" : {
      "index" : {
        "creation_date" : "1462807670938",
        "uuid" : "E84HLb0kSDC5QzFlni8R8g",
        "number_of_replicas" : "1",
        "number_of_shards" : "5",
        "version" : {
          "created" : "1040599"
        }
      }
    },
    "warmers" : { }
  }
}
{% endhighlight %}

Como podrá observar, el índice se creo con algunas características por defecto y otras por configurar.

* **"aliases"**: Define un conjunto de aliases para el índice.
* **"mappings"**: Define cómo el documento y sus campos son guardados e indexados.  
* **"settings"**: Define la cantidad de shards y réplicas. Esta configuración se detalla en el [módulo 2](http://www.desarrollo.unlp.edu.ar/elasticsearch/ddbms/2016/04/22/elasticsearch-module_2.html)
* **"warmers"**: A partir de la version 2.3.0 estan obsoletos. Permiten preparar al índice para que pueda responder de forma más eficiente a requerimientos que contengan grandes manejos de datos.  

## bulk API: Cargando la BBDD con datos existentes

Para cargar los datos del archivo JSON a Elasticsearch utilizaremos la API bulk. Con esta API es posible indexar los elementos con un solo llamado. 

El cuerpo del llamado espera la siguiente estructura JSON:

{% highlight bash%}
{ action: { metadata }}\n
{ request body        }\n
{ action: { metadata }}\n
{ request body        }\n
...
{% endhighlight %}


Donde **action** define qué tipo de operación se va a realizar, siendo estas: *create*, *index*, *delete* y *update*.
La **metadata** incluye información del *_index*, el *_type* y el *_id* del documento al cual se le va a ejecutar la operación.
¿Por qué se necesita esta estructura? Como cada documento referenciado puede estar alocado en cualquier nodo del cluster, cada acción definida en el JSON debe ser redirigida al shard correcto del nodo al que pertenece. Elasticsearch utiliza cada caracter de nueva línea para identificar el par *action/metadata* de forma tal que pueda manejar cada petición, leyendo los datos del **request body** directamente. 



Para continuar con los ejemplos, se utilizará el siguiente [JSON](/assets/data/articles.json) válido que tiene información sobre los artículos del diario. Este archivo  contiene **40** elementos cuya estructura se detalla a continuación:


{% highlight json  %}
{"index":{"_id":"1"}}
{
  "slug":  "kicillof-un-40-de-inflacion-para-este-ano-es-un-dibujo-25393",
  "is_visible": true,
  "created_at": "2014-12-01T04:41:31.000-03:00",
  "title": "Kicillof: “Un 40% de inflación para este año es un dibujo”",
  "lead": "Volvió a criticar las mediciones privadas y afirmó que cerrará en un 24%. Rechazo desde la oposición",
  "body": "<div><p class=“texto principal”><a href=“/edis/20141201/fotos_g/graf5.gif” target=“U_blan0....”"
}
{% endhighlight %}

La *action* u operación a realizar por cada elemento es **index**, la cual permite crear nuevos documentos o reemplazarlos si ya existen. Cada action tendrá como *metadato* el **_id** correspondiente a cada artículo para poder identificarlo univocamente. Si este atributo no se especifica, Elasticsearch lo creará automáticamente. 
Cada elemento del *body_request* está compuesto por los siguientes atributos:

1. **"slug"**:  string que identifica al artículo.
2. **"is_visible"**: boolean que indica si el articulo debe mostrarse o no.
3. **"created_at"**: date que identifica la fecha de creación del artículo.
3. **"title"**: string que se corresponde con el título del artículo.
4. **"lead"**: string que corresponde con el copete del artículo.
5. **"body"**: text que se corresponde con el cuerpo del artículo.

Para indexar los datos se deberá realizar el siguiente llamado a la bulk API:

{% highlight bash %}
curl -XPUT 'http://localhost:9200/article_index/politics/_bulk?pretty' --data-binary @articles.json
{% endhighlight %}

Como en el ejemplo estamos utilizando curl, debemos utilzar el flag --data-binary. 


Los endpoints a esta api son `/_bulk`, `/{index}/_bulk` y `/{index}/{type}/_bulk`. En este caso, se agregarán los datos en el índice **article_index** y serán de tipo **politics**.

## Realizando la primer búsqueda

Antes de comenzar haremos una pequeña introducción respecto del formato de consulta con el que iremos a realizar la búsqueda.

### Búsqueda mediante la URI: un enfoque más simple 

Una consulta simple tiene el siguiente formato:


{% highlight bash %}
$ curl -XGET '<host>:<port>/<index>/<type>/_search?<parameters>'
{% endhighlight %}


Para realizar una búsqueda simple, debemos enviar el parámetro **q** seguido de un par *clave:valor*. Por ejemplo, si deseamos buscar artículos cuyo atributo **slug** sea  exactamente igual a "cristina-felicito-a-vazquez-por-la-victoria-electoral-25380" la consulta sería la siguiente:

{% highlight bash %}
$ curl -XGET 'localhost:9200/article_index/politics/_search?pretty&q=slug:cristina-felicito-a-vazquez-por-la-victoria-electoral-25380'
{% endhighlight %}


### Búsqueda mediante un JSON estructurado

Elasticsearch provee una muy completa Query DSL basada en JSON para escribir las consultas.
Dentro del body va la consulta o [query](https://www.elastic.co/guide/en/elasticsearch/guide/current/query-dsl-intro.html). Una consulta típica tiene la siguiente estructura: 


{% highlight bash %}
curl -XGET '<host>:<port>/<index>/<type>/_search?' -d '
{
  "query": <consulta_personalizada>
}
'
{% endhighlight %}


Por ejemplo, si deseamos buscar los artículos cuyos **title** o título contengan la palabra **"Cristina"** (no distingue mayúsculas ni minúsculas):

{% highlight bash %}
$ curl -XGET 'localhost:9200/article_index/politics/_search?pretty' -d '
{
  "query": {
    "match": {
      "title": "Cristina"
    }
  }
}
'
{% endhighlight %}

Si deseamos buscar entre el 1° de Enero de 2014 y la fecha actual, realizamos la siguiente consulta:

{% highlight bash %}
curl -XGET 'localhost:9200/article_index/politics/_search?pretty' -d '
{
  "query" : {
    "range" : {
      "created_at" : {
        "gt" : "2014-01-01T00:00:00",
        "lt" : "now"
      }
    }
  }
}'

{% endhighlight %}

