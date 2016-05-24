---
layout: post
title: 1. Elasticsearch, intro
categories: elasticsearch NoSQL
tags: [elasticsearch, NoSQL, document, SQL]
usernames: [ maira1001001 ]
---


## Definición

[Elasticsearch](https://www.elastic.co/) es un servidor de búsqueda basado en Lucene.
Provee un motor de búsqueda de texto completo, distribuido y con capacidad de multi-tenencia con una interfaz web RESTful y con documentos JSON. Elasticsearch está desarrollado en Java y está publicado como código abierto bajo las condiciones de la licencia Apache.




## Instalando Elasticsearch

{% highlight bash  %}
wget https://download.elastic.co/elasticsearch/elasticsearch/elasticsearch-1.7.2.deb
sudo dpkg -i elasticsearch-1.7.2.deb
{% endhighlight %}

Iniciando el servicio:

{% highlight bash  %}
$ sudo service elasticsearch start
{% endhighlight %}

Para probar si elasticsearch está funcionando, escriba en consola la siguiente consulta: 
{% highlight bash %}
$ curl -XGET 'http://localhost:9200/'
{% endhighlight %}


## Primer ejemplo con Elasticsearch y curl

> Dependencias :  *curl*

1. Creando el índice **twitter** desde la consola con **curl**

{% highlight bash %}
curl -XPOST 'http://localhost:9200/twitter/'
{% endhighlight %}

Lo puede verificar escribiendo en consola la siguiente consulta:

{% highlight bash %}
$ curl -XGET 'http://localhost:9200/twitter?pretty'
{% endhighlight %}

2. Creando el índice **contact** con el atributo **office**

{% highlight bash %}
curl -XPOST localhost:9200/contact -d '{
    "mappings" : {
        "office" : {
            "properties" : {
                "id"      : { "type": "string"  },
                "name"    : { "type" : "string" },
                "address" : { "type" : "string" },
                "email"   : { "type" : "string" }
            }
        }
    }
}'
{% endhighlight %}

Lo puede verificar escribiendo en consola la siguiente consulta:

{% highlight bash %}
$ curl -XGET 'http://localhost:9200/contact?pretty'
{% endhighlight %}

Un **índice** es un espacio de nombre lógico, es decir, es una forma de organizar los datos.

## Elasticsearch y MySQL

Se puede, a grandes rasgos, pensar que un índice es como una base de datos y realizar la siguiente comparación:

> MySQL => Databases => Tables => Columns/Rows
>
> Elasticsearch => Indices => Types => Documents with Properties

Podriamos reemplazar el índice creado arriba por una base de datos relacional

{% highlight bash %}
CREATE DATABASE contact;
{% endhighlight %}

{% highlight bash %}
CREATE TABLE office ( 
  id INT(6)  PRIMARY KEY,
  name VARCHAR(30),
  address VARCHAR(30),
  email VARCHAR(50) 
);
{% endhighlight %}

**MySQL** utiliza **SQL** como mecanismo de almacenamiento de datos. En cambio **Elasticsearch** utiliza **NoSQL** ("Not Only SQL"). ¿Dónde está la diferencia? Tanto SQL como  NoSQL  almacenan datos, pero SQL utiliza sistema de gestión de base de datos relacionales. NoSQL NO utiliza modelo de datos relacional. 

Un ejemplo de instancia de cada uno de ellos es el siguiente:

+ Tabla SQL

{% highlight bash %}

+------------+--------------+------------------------+
| name       | address      | email                  |
+------------+--------------+------------------------+
| Secretaria | 7 e/ 47 y 48 | secretaria@unlp.edu.ar |
+------------+--------------+------------------------+

{% endhighlight %}

+ Documento  NoSQL

{% highlight bash %}
{
  "name": "Secretaria",
  "address": "7 e/ 47 y 48",
  "email": "secretaria@unlp.edu.ar"
}
{% endhighlight %}

El diseño de una tabla SQL es *rígido*: las columnas de las tablas tienen definido un tipo de dato determinado, por lo tanto, no se puede usar la misma columna para almacenar, por ejemplo, un string en lugar de un entero. En cambio, en una base de datos NoSQL generalmente los ítems se almacenan como pares **clave-valor**. No tienen un esquema determinado, por lo que podriamos agregar un nuevo campo o utilizar otro tipo de dato sin problemas.   


{% highlight bash %}
{
  "name": "Secretaria",
  "address": "7 e/ 47 y 48",
  "email": "secretaria@unlp.edu.ar",
  "geolocation": [{
    "position": {
      "lng": -57.92656692734454,
      "lat": -34.95091819492699
    },
    "title": "altos de san lorenzo"
  }]
}
{% endhighlight %}

Dependiendo de las necesidades del problema, existen diferentes mecanismos para almacenar datos.

¿Qué sucede si no es conveniente almacenar datos en tablas SQL o si se tiene otro tipo de relación entre los registros y se quiere acceder de forma rápida a los datos? Otro problema con el que se encontraron las bases de datos relacionales fue que las consultas SQL no eran muy adecuadas para las estructuras de datos orientados a objetos. La solución emergente fue NoSQL.

Las bases de datos **SQL** han sido uno de los principales mecanismos de almacenamiento de datos durante las ultimas cuatro décadas. Su uso explotó la década del 90 con el auge de las aplicaciones webs y las opciones de código abierto como **MySQL**, **PostgreSQL** y **SQLite**. Las bases de datos **NoSQL** han existido desde los 60, pero han ido ganando terreno con opciones populares como **MongoDB**, **CouchDB**, **Redis** y **Apache Cassandra**

Otro problema que presentan las bases de datos relacionales es el aumento exponencial en los volúmenes de datos. Las operaciones de consulta SQL estándar no responden en tiempos aceptables, dificultando de esta forma el uso de bases de datos relacionales. NoSQL es una buena opción para bases de datos que manipulan grandes conjuntos de datos. 



## Características de las bases de datos NoSQL

* Modelo de datos no relacional y schema-less
* Baja latencia y alto rendimiento
* Altamente escalable
* Se ejecutan mejor en clusters. Las base de datos relacionales no fueron diseñadas para correr eficientemente en clusters


## Clasificación de bases de datos NoSQL

* Key-Value (K-V) Stores
* Document Store
* Column-Oriented Stores
* Graph Databases

Elasticsearch utiliza almacenamiento de tipo **Document**. 
Cada documento es un objeto **JSON**.


