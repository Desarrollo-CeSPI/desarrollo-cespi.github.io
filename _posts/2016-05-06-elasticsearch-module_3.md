---
layout: post
title: Elasticsearch - Módulo n
author: Rosario Santa Marina
usernames: [rosariosm maira1001001]
tags: [elasticsearch, bulk API, search API]
---

En este módulo veremos cómo crear un índice, agregarle un conjunto de datos provenientes de un JSON válido y realizar una búsqueda mediante la API de búsqueda.



## Creando un índice

En este módulo trabajaremos con ejemplos de artículos de diario. Antes de comenzar a trabajar con los datos, necesitamos crear un índice para identificar a los artículos del diario. Para ello creamos el índice **article_index** como se detalla a continuación.

{% highlight bash %}
$ curl -XPUT 'http://localhost:9200/article_index'
{% endhighlight %}

Para visualizar el índice creado, escriba en consola la siguiente consulta:

{% highlight bash %}
$ curl -XGET 'http://localhost:9200/article_index?pretty'
{% endhighlight %}

Esta consulta dará como resultado lo siguiente:

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

1. **"aliases"**:
2. **"mappings"**:
3. **"settings"**: define la cantidad de shards y réplicas. Esta configuración se detalla en el [módulo 2](http://www.desarrollo.unlp.edu.ar/elasticsearch/ddbms/2016/04/22/elasticsearch-module_2.html)
4. **"warmers"**:

## Cargando la BBDD con datos existentes

Los artículos se encuentran almacenados en un JSON. El archivo se puede descargar aquí.
(incluir el link de descarga al JSON)

### bulk API

Para cargar los datos del archivo JSON a Elasticsearch utilizaremos la API bulk. Con esta API es posible indexar los elementos con un solo llamado. 

El cuerpo del llamado espera una estructura de JSON particular, como la que se detalla a continuación:

{% highlight bash%}
{ action: { metadata }}\n
{ request body        }\n
{ action: { metadata }}\n
{ request body        }\n
...
{% endhighlight %}


Donde **action** define qué tipo de operación se va a realizar. Las operaciones son: **create**, **index**, **delete** y **update**.
La **metadata** incluye información del **índice**, el **tipo** y el **id** del documento al cual se le va a ejecutar la operación.
¿Por qué se necesita esta estructura? Como cada documento referenciado puede estar alocado en cualquier nodo del cluster, cada accion definida en el JSON debe ser redirigida al shard correcto del nodo al que pertenece. Elasticsearch utiliza cada caracter de nueva línea para identificar el par action/metadata de forma tal que pueda manejar cada petición, leyendo los datos del **request body** directamente. 


Para continuar con los ejemplos, se utilizará el siguiente JSON válido que tiene información sobre los artículos del diario. El archivo **articles.json** contiene **x** elementos. La operación que se realizará a cada elemento es **index** y se incluirá elmetadato **id**, para identificar univocamente a cada artículo.

{% highlight bash  %}
{"index":{"_id":"1"}}
{
  "slug":  "kicillof-un-40-de-inflacion-para-este-ano-es-un-dibujo-25393",
  "is_visible": true,
  "title": "Kicillof: “Un 40% de inflación para este año es un dibujo”",
  "lead": "Volvió a criticar las mediciones privadas y afirmó que cerrará en un 24%. Rechazo desde la oposición",
  "body": "<div><p class=“texto principal”><a href=“/edis/20141201/fotos_g/graf5.gif” target=“U_blan0....”"
}
{% endhighlight %}

Cada elemento tiene los atributos: 
1. **"slug"**:  string que identifica al artículo
2. **"is_visible"**: boolean que indica si el articulo debe mostrarse o no.
3. **"title"**: string que se corresponde con el título del artículo
4. **"lead"**: string que corresponde con el copete del artículo
5. **"body"**: text que se corresponde con el cuerpo del artículo


Para indexar los datos al índice **article_index**, escriba en consola la siguiente consulta:

{% highlight bash %}
curl -XPUT 'http://localhost:9200/articles/politics/_bulk?pretty' --data-binary @articles.json
{% endhighlight %}

## Realizando la primera búsqueda

Antes de comenzar con la búsqueda, haremos una pequeña introducción respecto del formato de consulta con curl con que iremos a realizar una búsqueda.

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

Un tipo de consulta personalizada tiene la siguiente estructura:

{% highlight bash %}
{
    QUERY_NAME: {
        ARGUMENT: VALUE,
        ARGUMENT: VALUE,...
    }
}{% endhighlight %}


Si hacemos referencia a un campo en particular, la consulta tiene la siguiente estructura: 

{% highlight bash %}
{
    QUERY_NAME: {
        FIELD_NAME: {
            ARGUMENT: VALUE,
            ARGUMENT: VALUE,...
        }
    }
}
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

Si deseamos buscar los artículos cuyo **title** o título *coincida exactamente* con el string "Mujica propone a Argentina recuperar las Malvinas “con las mujeres”"

{% highlight bash %}
$ curl -XGET 'localhost:9200/article_index/politics/_search?pretty' -d '
{
  "query": {
    "term": {
      "title": "Mujica propone a Argentina recuperar las Malvinas “con las mujeres”"
    }
  }
}
'
{% endhighlight %}

## Analizando la estructura de los datos cargados






