---
layout: post
title: Elasticsearch - Módulo n
author: Rosario Santa Marina
usernames: [rosariosm]
tags: [elasticsearch, bulk API, search API]
---

En este módulo veremos cómo crear un índice, agregarle un conjunto de datos provenientes de un JSON válido y realizar diferentes consultas mediante la API de búsqueda.


Título crear índice
-------------------
Antes de comenzar a trabajar con los datos, necesitamos un lugar donde almacenarlos. Para esto creamos un índice. Como vamos a trabajar con artículos de un diario, lo llamaremos **articles**.

{% highlight bash %}
curl -XPUT http://localhost:9200/articles
{% endhighlight %}

Cargando los datos
------------------

Cuando se quieren ejecutar múltiples operaciones sobre documentos la mejor opción es utilizar la API bulk ya que permite ejecutarlas en un solo llamado, incrementando de forma notable la velocidad de indexado.

El cuerpo del llamado espera una estructura de JSON particular con la siguiente forma:

{% highlight bash%}
{ action: { metadata }}\n
{ request body        }\n
{ action: { metadata }}\n
{ request body        }\n
...
{% endhighlight %}

Donde *action* define qué tipo de operación se va a realizar, por ejemplo: crear, borrar o actualizar un documento. La *metadata* incluye información del índice, el tipo y el id del documento al cual se le va a ejecutar la operación.
¿Por qué se necesita esta estructura? Como cada documento referenciado puede estar alocado en cualquier nodo del cluster, cada accion definida en el JSON debe ser redirigida al shard correcto del nodo al que pertenece. Elasticsearch utiliza cada caracter de nueva línea para identificar el par action/metadata de forma tal que pueda manejar cada petición, leyendo los datos del *request body* directamente. 

Para continuar con los ejemplos, se utilizará el siguiente JSON válido que tiene información sobre los artículos del diario. 
