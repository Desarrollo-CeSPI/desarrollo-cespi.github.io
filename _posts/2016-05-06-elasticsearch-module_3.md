---
layout: post
title: 3. Elasticsearch, realizando la primera búsqueda
usernames: [maira1001001, rosariosm]
tags: [elasticsearch, bulk API, search API]
---

En este módulo veremos cómo crear un índice, agregarle un conjunto de datos provenientes de un archivo  <!-- more --> y realizar una búsqueda simple mediante una API de búsqueda.


## Creando un índice

En este módulo trabajaremos con ejemplos de artículos de diario. Primero se debe crear índice  `article_index` para almacernarlos. Para ello escriba en consola la siguiente consulta:

{% highlight bash %}
$ curl -XPUT 'http://localhost:9200/article_index'
{% endhighlight %}

Para visualizar el índice creado, escriba en consola:

{% highlight bash %}
$ curl -XGET 'http://localhost:9200/article_index?pretty'
{% endhighlight %}

El resultado será el siguiente:

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


Un índice tiene las siguientes características: 

* `aliases`: Define un conjunto de aliases para el índice.
* `mappings`: Define cómo el documento y sus campos son guardados e indexados.  
* `settings`: Define la cantidad de shards y réplicas. Esta configuración se detalla en el [módulo 2](http://www.desarrollo.unlp.edu.ar/elasticsearch/ddbms/2016/04/22/elasticsearch-module_2.html)
* `warmers`: A partir de la version 2.3.0 estan obsoletos. Permiten preparar al índice para que pueda responder de forma más eficiente a requerimientos que contengan grandes manejos de datos.  

Un índice se crea con ciertas características por defecto: la fecha de creación, un identificador único, el número de réplicas y de shards, y la versión. El resto de las características pueden ser configuradas a medida que se requieran.

Para configurar ciertos valores en la creación del índice, como por ejemplo, el número de réplicas, puede enviarse un cuerpo JSON en la consulta como se muestra a continuación:

{% highlight bash %}
$ curl -XPUT 'http://localhost:9200/article_index?pretty' \ 
       -d '{"index":{ "number_of_replicas":"2" }}'
{% endhighlight%}



## bulk API: Cargando la BBDD con datos existentes

*bulk API* permite realizar operaciones de forma muy simple sobre la base de datos de Elasticsearch. En este módulo utilizaremos *bulk API* para importar datos desde un archivo e indexarlos a `article_index`.

El archivo a importar debe tener la siguiente estructura:

{% highlight bash%}
{ action: { metadata }}\n
{ request body        }\n
{ action: { metadata }}\n
{ request body        }\n
...
{% endhighlight %}

> NOTA
> La última línea debe terminar con un salto de línea

El archivo va a estar conformado por *n* líneas. Cada línea está compuesta por un objeto JSON. El primer objeto representa la operación que se va a realizar y el segundo los datos que se utilizarán para ejecutar dicha operación. Una *action* u operación puede ser cualquiera de las siguientes:  `create`, `index`, `delete` o `update`. La *metadata* incluye información del `_index`, el `_type` y el `_id` del documento al cual se le va a ejecutar la operación.

¿Por qué se necesita esta estructura?  Cada par *action/request* opera sobre distintos shards. Elasticsearch utiliza la `metadata` para redireccionar cada petición al shard correspondiente, utilizando los datos del *request body* inmediato.


Para nuestro ejemplo, cada *request body* contiene información sobre artículos periodísticos. Como queremos indexar los artículos a `article_index`, se redefinen las siguiente características:

1. el **action** a utilizar será `index`. Esta operación permite crear nuevos documentos o reemplazarlos si ya existen.
2. la **metadata** que utilizaremos será `id` para poder identificar cada artículo univocamente.
3. el **request body** contendrá los siguientes datos:
- `"slug"`:  *string* que identifica al artículo.
- `"is_visible"`: *boolean* que indica si el articulo debe mostrarse o no.
- `"created_at"`: *date* que identifica la fecha de creación del artículo.
- `"title"`: *string* que se corresponde con el título del artículo.
- `"lead"`: *string* que corresponde con el copete del artículo.
- `"body"`: *text* que se corresponde con el cuerpo del artículo.

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
$ curl -XPUT 'http://localhost:9200/article_index/politics/_bulk?pretty' --data-binary @articles.txt
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

Donde `<host>:<port>` se reemplazan por el host y el puerto correspondientes a la instancia activa de Elasticsearch, `index`  por el nombre del índice, `type` por el tipo de documento, `_search` es el *endpoint* a la API de búsqueda y el contenido de `<parameters>` dependerá de qué se quiera buscar. 

Por ejemplo, si deseamos buscar artículos en el índice `article_index` de tipo `politics`, cuyo campo `slug` sea  *exactamente igual* a "cristina-felicito-a-vazquez-por-la-victoria-electoral-25380" se debe realizar la siguiente consulta:

{% highlight bash %}
$ curl -XGET 'localhost:9200/article_index/politics/_search?pretty&q=slug:cristina-felicito-a-vazquez-por-la-victoria-electoral-25380'
{% endhighlight %}

En este caso enviamos el parámetro `q` (*query string*) seguido de un par clave:valor definido en los documentos indexados. 

### Búsqueda mediante un JSON estructurado

Elasticsearch provee una estructura especial basada en JSON para definir las consultas llamada **Query DSL**. El endpoint sigue siendo `_search` con la diferencia que se enviará como parámetro el cuerpo JSON especial. Si bien esta estructura es muy completa, antes de complejizarla, mostraremos un ejemplo simple de su funcionamiento.

Una consulta típica posee la siguiente estructura: 

{% highlight bash %}
curl -XGET '<host>:<port>/<index>/<type>/_search?pretty' \
     -d ' { "query": <consulta_personalizada> }'
{% endhighlight %}

Donde `"query"` es el comienzo de la consulta, y `<consulta_personalizada>` define la estructura de la consulta con el formato JSON. Estas consultas personalizadas se componen de uno o varios elementos hoja. Un elemento hoja es la unidad más simple de búsqueda, generalmente buscan un valor particular en un campo particular. Contienen clausulas como `match`, `term` o `range`. Para realizar consultas más completas, es posible combinar varios de estos elementos de una forma lógica.
En este módulo nos centraremos en mostrar una consulta hoja para ejemplificar su funcionamiento, en los siguientes módulos iremos agregando nuevos conceptos para poder obtener resultados más correctos.

Si deseamos buscar los artículos cuyos `title` o título contengan la palabra **"Cristina"**, se realiza la siguiente consulta:

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

La cláusula `match` permite buscar un valor particular en un campo en particular. En este caso busca en el campo `title` los títulos que contengan la palabra **Cristina**, sin distinguir entre mayúsculas y minúsculas.

