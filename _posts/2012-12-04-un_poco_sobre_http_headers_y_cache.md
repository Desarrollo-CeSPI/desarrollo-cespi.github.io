---
layout: post
title: Un poco sobre http headers y cache
author: Álvaro F. Lara
author_email: alvarola@gmail.com
---

## {{ page.title }}

Desde hace unos dias vengo investigando tecnologias para poder brindar un
acceso mas eficiente al contenido que se entrega desde el servidor (en mi caso
una aplicación sinatra) y evitar la transferencia inncesaria de contenido.

En el trayecto me fui encontrando con varios articulos y documentos pero en un
intento de empezar desde lo mas basico busque un poco sobre como funciona el
browser y que posibilidades de manejar cache tenemos.

### Cache

Podemos decir que entre las posibilidades de cache que tenemos, teniendo en
cuenta la cache del browser y no tecnologias como memcache o varnish, podemos
encontrar 4 maneras manejar el contenido que reside en cache manipulando
distintos headers http.

#### Last Modified

Este header determinar cuando fue la ultima fecha de modificación del
contenido. La interacción browser - servidor para determinar si el contenido
debe ser actualizado o no tiene es la siguiente:

* El browser hace una solicitud a un recurso, digamos "/favicon.png"
* Nuevamente el browser hace quiere hacer una solicitud al recurso
  "/favicon.png". Antes de que esto suceda, se envia solicitud especial al
  mismo recurso para verificar si fue modificado (usando la fecha del recurso
  del browser y la actual del server).
* Si el recurso no cambio, se notifica al browser con un codigo "304 - Not
  Modified"
* El browser consume la imagen desde cache y evita descargarla.

***Pro***: Nos ahorramos tener que descargar la imagen si no fue modificada.

***Contra***

* Tenemos la necesidad de realizar un request al servidor.
* Podemos tener problemas si el reloj del servidor cambia, dando fallos de
  cache inncesesarios.

#### ETag:

Similar a Last Modified, en este caso lo que se utiliza es un parametro al
estilo de un hash generador a partir del contenido del recurso. Con lo cual si
es recurso se ve modificado, el tag cambia.

El mecanismo de verificación es igual al anterior, pero en lugar de verificar
por Last Modified, se chequea que el Etag sea el mismo.

***Pro***: evitamos tener problemas con relojes
***Contra***: seguimos teniendo la necesidad de hacer un request al servidor.

#### Expires: 

Como nos damos cuenta que la leche se puso fea? Por la fecha de vencimiento.
Este mecanismo sigue la misma idea: cuando pedimos un recurso el servidor nos
informa cuando expira. Cada vez que necesitamos utilizarlo verificamos contra
nuestra hora actual y determinamos si es momento o no de pedir una nueva copia.

El mecanismo de verificación ocurre solamente del lado del cliente, con lo
cual es una mejor sobre los otro dos.

***Pro***: no necesitamos hacer requests extra al servidor.
***Contra***: hay que calcular la fecha de hoy para poder compararla.

#### Max-age:

Por ultimo, una variante del expires, es en lugar de decir la fecha de
vencimiento, decir cuanto tiempo a partir de la solicitud este recurso es
valido.

Tiene como mejor sobre Expires que es mas simple del lado del servidor decir
cuando vence que calcular la fecha.

### Bonus Extra: Cache-control.

Ademas de determinar cuando un recurso debe ser actualizado o no, podemos decir
que recursos deben ser almacenados en cache y que visibilidad hay sobre esos
datos.

***Cache-control: Public***

En este caso, tanto el browser, los proxies y cualquier intermediario por el
cual pase el request puede almacenarlo y dejarlo publico por si otros llegan a
necesitar ese recurso. Es muy util para cuando tenemos recurso generales
compartidos por todos los usuarios.

***Cache-control: Private***

En caso como puede ser la pagina principal de un usuario, varia de usuario a
usuario, pero también seria bueno poder almacenarla en cache. Esta opción le
permite al browser hacer esto, pero esta información no es accesible por otros
usuario ni intermediarios. 

***Cache-control: no-cache***

Hay contenido que directamente no debe ser almacenado en ninguna cache y esta
es la forma de indicarlo.

Un recurso interesante que user para ecribir este articulo fue [How To Optimize Your Site With HTTP Caching](http://betterexplained.com/articles/how-to-optimize-your-site-with-http-caching/).
