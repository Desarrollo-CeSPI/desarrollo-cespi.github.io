---
layout: post
title: 1. Intro Elasticsearch
author: Maira Diaz
categories: elasticsearch NoSQL
tags: [elasticsearch, NoSQL, document, SQL]
usernames: [ maira1001001 ]
---


# Modulo 1


## Definición

[Elasticsearch](https://www.elastic.co/) es un servidor de búsqueda basado en Lucene.
Provee un motor de búsqueda de texto completo, distribuido y con capacidad de multi-tenencia con una interfaz web RESTful y con documentos JSON Elasticsearch está desarrollado en Java y está publicado como código abierto bajo las condiciones de la licencia Apache.


## Instalando Elasticsearch

{% highlight bash  %}
wget https://download.elastic.co/elasticsearch/elasticsearch/elasticsearch-1.7.2.deb
sudo dpkg -i elasticsearch-1.7.2.deb
{% endhighlight %}

iniciando el servicio:

{% highlight bash  %}
sudo service elasticsearch start
{% endhighlight %}

Para probar si elasticsearch está funcionando: [http://localhost:9200/](http://localhost:9200/)

## Primer ejemplo con Elasticsearch y curl

> Dependencias :  *curl*

1. Creando el índice **twitter** desde la consola con **curl**

{% highlight bash %}
curl -XPOST 'http://localhost:9200/twitter/'
{% endhighlight %}

Lo puede verificar aquí: [http://localhost:9200/twitter/](http://localhost:9200/twitter/)

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

Lo puede verificar aquí: [http://localhost:9200/contact/](http://localhost:9200/contact/)

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

**MySQL** utiliza **SQL** como mecanismo de almacenamiento de datos. 
En cambio **Elasticsearch** utiliza **NoSQL** ("Not Only SQL").
Y donde está la diferencia? Tanto SQL como  NoSQL  almacenan datos, pero 
SQL utiliza sistema de gestion  de base de datos relacionales.
NoSQL NO utiliza modelo de datos relacional. 

Un ejemplo de instancia de cada uno de ellos es el siguiente:

1. table SQL

{% highlight bash %}

+------------+--------------+------------------------+
| name       | address      | email                  |
+------------+--------------+------------------------+
| Secretaria | 7 e/ 47 y 48 | secretaria@unlp.edu.ar |
+------------+--------------+------------------------+

{% endhighlight %}

2. document  NoSQL

{% highlight bash %}
{
  "name": "Secretaria",
  "address": "7 e/ 47 y 48",
  "email": "secretaria@unlp.edu.ar"
}
{% endhighlight %}

El diseño de una tabla SQL es *rígido*. No se puede usar la misma tabla  para almacenar, por ejemplo, un string en vez de un número. Las bases de datos NoSQL son como archivos **JSON** con su respectivo **clave-valor**. Al no ser tan rígido, podriamos agregar un nuevo campo sin que la BBDD NoSQL se queje

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

Pero que sucede si no es conveniente almacenar datos en tablas SQL? Y si se tiene otro tipo de relacion entre los registros y se quiere acceder de forma rápida a los datos? Otro problema con el que se encontraron las bases de datos relacionales fue que las consultas SQL no eran muy adecuadas para las estructuras de datos orientados a objetos. La solución emergente fue NoSQL.

Las bases de datos **SQL** han sido uno de los principales mecanismos de almacenamiento de datos durante las ultimas 4 décadas. Su uso explotó la década de los 90 con el auge de las aplicaciones webs y las opciones de código abierto como **MySQL**, **PostgreSQL** y **SQLite**. Las bases de datos **NoSQL** han existido desde los 60, pero han ido ganando terreno con opciones populares como **MongoDB**, **CouchDB**, **Redis** y * Apache Cassandra**

Otro problemas con el que se presentan las bases de datos relacionales se relaciona un aumento exponencial en los volúmenes de datos. Las operaciones de consulta SQL estándar no responden en tiempos aceptables, dificultando de esta forma el uso de bases de datos relacionales. NoSQL es una buena opción para bases de datos que manipulan grandes conjuntos de datos. 



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

