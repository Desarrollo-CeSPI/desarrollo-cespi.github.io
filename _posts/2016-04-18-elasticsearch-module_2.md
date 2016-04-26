---
layout: post
title: Probando Elasticsearch - Módulo 2
author: Maira Diaz
categories: elasticsearch
---
##  MODULO 2

### descargar proyecto en docker

### Entendiendo el mapping de BBDD sql a documento elasticsearch

Una vez descargado el proyecto y levantado el servicio, podrá acceder a la base de datos 

Diagrama de la Base de Datos Relacional

![Diagrama Base de datos Relacional](/assets/images/elasticsearch-modules/phonebook-db.png)

{% highlight bash %}
$ mysql -u root -p
{% endhighlight %}

{% highlight bash %}
mysql> use agenda_telefonica_backend (cambiar nombre de DDBB);
mysql> desc contacts;
+------------+--------------+
| Field      | Type         |
+------------+--------------+
| id         | int(11)      |
| name       | varchar(255) |
| email      | varchar(255) |
| address    | varchar(255) |
| geolocation| varchar(255) |
| type       | varchar(255) |
| parent_id  | varchar(255) |
+------------+--------------+
mysql> desc telephones;
+-------------------+---------+
| Field             | Type    |
+-------------------+---------+
| id                | int(11) |
| number            | text    |
| contact_id        | int(11) |
| telephone_type_id | int(11) |
+-------------------+---------+
mysql> desc telephone_types;
+-------------------+--------------+
| Field             | Type         |
+-------------------+--------------+
| id                | int(11)      |
| description       | text         |
| type              | varchar(255) |
+-------------------+--------------+
{% endhighlight %}



