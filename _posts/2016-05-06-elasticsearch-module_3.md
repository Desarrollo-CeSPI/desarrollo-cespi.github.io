---
layout: post
title: 3. Elasticsearch, realizando la primera búsqueda
usernames: [maira1001001, rosariosm]
tags: [elasticsearch, bulk API, search API]
---

En este módulo veremos cómo crear un índice, agregarle un conjunto de datos provenientes de un JSON válido  <!-- more --> y realizar una búsqueda simple mediante la API de búsqueda.


## Creando un índice

En este módulo trabajaremos con ejemplos de artículos de diario. Antes de comenzar necesitamos crear el índice  `article_index` para almacernarlos. 

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

Como se podrá observar, el índice se creo con 4 características configurables:

* **"aliases"**: Define un conjunto de aliases para el índice.
* **"mappings"**: Define cómo el documento y sus campos son guardados e indexados.  
* **"settings"**: Define la cantidad de shards y réplicas. Esta configuración se detalla en el [módulo 2](http://www.desarrollo.unlp.edu.ar/elasticsearch/ddbms/2016/04/22/elasticsearch-module_2.html)
* **"warmers"**: A partir de la version 2.3.0 estan obsoletos. Permiten preparar al índice para que pueda responder de forma más eficiente a requerimientos que contengan grandes manejos de datos.  

`article_index` se creo con la característica `settings` por defecto. En cambio `aliases`, `mappings` y `warmers`  faltan definir.


## bulk API: Cargando la BBDD con datos existentes

*bulk API* permite realizar operaciones, de forma muy simple, sobre la base de datos de Elasticsearch. En este módulo utilizaremos *bulk API* para importar datos desde un archivo e indexarlos a `article_index`.

El archivo a importar debe tener la siguiente estructura:

{% highlight bash%}
{ action: { metadata }}\n
{ request body        }\n
{ action: { metadata }}\n
{ request body        }\n
...
{% endhighlight %}

> NOTA
> La línea final debe terminar con \n (enter)

El archivo va a estar conformado por *n* líneas. Cada línea es un documento JSON, y por lo tanto acepta una estructura JSON válida. Una línea puede ser un **request** o un **action**. ¿Qué relación existe entre un **action** y un **request**? Por cada **request** existe un **action** correspondiente. 

Un **action** define qué tipo de operación se va a realizar, siendo estas `create`, `index`, `delete` o `update`.
La **metadata** incluye información del `_index`, el `_type` y el `_id` del documento al cual se le va a ejecutar la operación.

¿Por qué se necesita esta estructura?  Cada par *action/request* opera sobre distintos shards. Elasticsearch utiliza la **metadata** para redireccionar cada petición al shard correspondiente.


El **request body** contiene los datos con los que se van a operar. Para nuestro ejemplo, cada request body contiene información de los artículos periodísticos.

Como queremos indexar los artículos a `article_index`, se definen las siguiente características:
1. el **action** a utilizar erá `index`. Esta operación permite crear nuevos documentos o reemplazarlos si ya existen.
2. la **metadata** que utilizaremos será `id` para poder identificar cada artículo univocamente.
3. el **request body** contendrá los siguientes datos:

- **"slug"**:  string que identifica al artículo.
- **"is_visible"**: boolean que indica si el articulo debe mostrarse o no.
- **"created_at"**: date que identifica la fecha de creación del artículo.
- **"title"**: string que se corresponde con el título del artículo.
- **"lead"**: string que corresponde con el copete del artículo.
- **"body"**: text que se corresponde con el cuerpo del artículo.

De esta forma, cada par *action/request* queda definido de la siguiente forma:

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

Para indexar los datos se utilizará el archivo válido que puede descargar [aquí](/assets/data/articles.txt). Este archivo  contiene **40** artículos periodísticos.

Para indexar los datos deberá realizar el siguiente llamado a la *bulk API*:

{% highlight bash %}
curl -XPUT 'http://localhost:9200/article_index/politics/_bulk?pretty' --data-binary @articles.txt
{% endhighlight %}

Como en el ejemplo estamos utilizando curl, debemos utilzar el flag --data-binary. 


Los endpoints a esta api son `/_bulk`, `/{index}/_bulk` y `/{index}/{type}/_bulk`. En este caso, se agregarán los datos en el índice `article_index` y serán de tipo `politics`.

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

Las consultas pueden ser simples o compuestas. Las consultas simples buscan en un valor en particular en un determinado campo. En cambio las comsultas más complejas están compuestas de consultas simples pudiendo alterar el comportamiento del documento (?????)

++++++++++++
Leaf query clauses look for a particular value in a particular field, such as the match, term or range queries. These queries can be used by themselves.
Compound query clauses wrap other leaf or compound queries and are used to combine multiple queries in a logical fashion (such as the bool or dis_max query), or to alter their behaviour (such as the not or constant_score query).

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



