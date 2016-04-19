---
layout: post
title: Intro Elasticsearch - Módulo 1
author: Maira Diaz
categories: elasticsearch NoSQL  document  SQL
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


## Primer ejemplo con Elasticsearch y curl

Dependencias :  *curl*

1. Creando el índice **twitter** desde la consola con **curl**

{% highlight bash %}
curl -XPOST 'http://localhost:9200/twitter/'
{% endhighlight %}

Lo puede verificar aquí: [http://localhost:9200/twitter/](http://localhost:9200/twitter/)

2. Creando el indice **contact** con el atributo **office**

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

Se puede comparar con la creación de una base de datos en sql? 

Podriamos hacer  

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

La diferencia? 

SQL y NoSQL  ALMACENAN DATOS, pero...

**SQL** utiliza sistema de gestion  de base de datos relacionales **RDBMS**. 

**NoSQL** ("Not Only SQL") NO utiliza modelo de datos relacional.


### Pero entonces, dependiendo de las necesidades del problema, existen diferentes mecanismos para almacenar datos?

### SI, y uno de ellos es NoSQL

Algunas características de estos lenguajes son:

1. no utiliza modelos relacionales
2. schema-less
3. se ejecutan mejor en clusters. Base de datos relacionales no fueron diseñadas para correr eficientemente en clusters

Un ejemplo de instancia de cada uno de ellos es el siguiente:

1. tablas SQL

{% highlight bash %}

+------------+--------------+------------------------+
| name       | address      | email                  |
+------------+--------------+------------------------+
| Secretaria | 7 e/ 47 y 48 | secretaria@unlp.edu.ar |
+------------+--------------+------------------------+

{% endhighlight %}

2. documentos  NoSQL

{% highlight bash %}
{
 name:    "Secretaria",
 address: "7 e/ 47 y 48",
 email:   "secretaria@unlp.edu.ar"
}
{% endhighlight %}

Pero... y entonces....

El diseño de una tabla es *rígido*. No se puede usar la misma tabla  para almacenar, por ejemplo, un string en vez de un número.

Las bases de datos NoSQL son como archivos **JSON** con su respectivo **clave-valor**

### Al no ser tan rígido, podriamos agregar un nuevo campo sin que la BBDD NoSQL se queje

{% highlight bash %}
{
  name:        "Secretaria",
  address:     "7 e/ 47 y 48",
  email:       "secretaria@unlp.edu.ar",
  geolocation: [
    {
      position: {
        lng: -57.92656692734454,
        lat: -34.95091819492699
      },
      title: "altos de san lorenzo"
    }
  ]
}
{% endhighlight %}


Las bases de datos **SQL** han sido uno de los principales mecanismos de almacenamiento de datos durante las ultimas 4 décadas. Su uso explotó la década de los 90 con el auge de las aplicaciones webs y las opciones de código abierto como *MySQL*, *PostgreSQL* y *SQLite*.

Las bases de datos **NoSQL** han existido desde los 60, pero han ido ganando espacio con opciones populares como **MongoDB**, **CouchDB**, **Redis** y **Apache Cassan**

### Clasificacion NoSQL
Existen 4 tipos de bases de datos NoSQL:

1. Key-Value 
2. Graph
3. Document
4. Big Table

[LINK A BASES DE DATOS NOSQL http://nosql-database.org/](http://nosql-database.org/)

### Bueno, pero donde ubicamos a Elasticsearch?

Elasticsearch utiliza la clasificación  **Documents**

El almacenamiento y recuperación de los datos puede darse en forma de **XML**, **JSON**, **BSON**, etc
