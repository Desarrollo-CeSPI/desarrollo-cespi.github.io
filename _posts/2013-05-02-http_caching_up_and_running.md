---
date: 3013-05-02 09:00:00 -3000
layout: post
title: Http Caching
excerpt: Análisis en profundidad del funcionamiento de cache en http
author: Alvaro Lara & Fernando Martínez
author_email: patriciomacadden@gmail.com
published: false
---

#Http caching up and running
Todos en algun momento nos topamos con la necesidad de tener que optimizar nuestro
sistema para que se aproveche las ventaja de usar cache, pero, como funciona una cache
y que parametros son los que tenemos que tener en cuenta. De eso se va a tratar este
post y esperamos pueda quedar claro que es lo que nos brinda HTTP para lograrlo.

##HTTP - Niveles de cache
Las cache HTTP no son mas que un repositorio donde se almacena paginas en base
a algun criterio. Estos almacenes de paginas pueden aparecer en distintos lugares
entre el Cliente y el Servidor. Cuando implementamos una cache debemos se concientes
de cada uno de los niveles de cache que tenemos para poder hacer un uso efectivo de
la misma.

A grandes razgos tenemos 3 niveles de cache:

Cliente --> Browser Cache ---> [Gateway Cache]* ---> [Reverse Proxy Cache]* ---> Server

En este caso vamos a hablar sobre Browser Caches y Reverse Proxy Caches, ya que son
estas las que podemos llegar a controlar de manera mas precisa.

Ambos tipos de cache funcionan de manera similar: alguien hace un requerimiento, verifican
si poseen una copia aceptable, en caso afirmativo lo devuelven sin acceder al servidor
y de lo contrario, consultan al servidor y (a veces) almacenan la respuesta.

##HTTP - Expiración y Validación
Como dijimos anteriormente, cuando alguien hace un requerimiento HTTP,
queda en la cache determinar si lo que esta almacenado puede ser utilizado o
si necesitamos consultar el servidor por una nueva copia.

El protocolo posee dos mecanismos: Validación y Expiración.

La Expiración es un mecanismo que apunta a disminuir la cantidad de accesos al
servidor, mediante la utilización de información que nos permita saber si un
requerimiento esta "fresco" o si se venció. Si tenemos una representación "fresca"
del recurso en cache, no tenemos necesidad de ir al servidor.
  Por el contrario, si la representación del recurso esta vencida, vamos a necesitar
consultar el servidor, actualizar la copia local y retornarla al cliente.

La Validación es otro mecanismo que apunta a disminuir el ancho de banda utilizado.
Permite determinar si una representación de un recurso se modifico respecto a la que
poseemos en cache. La verificación suele hacerse utilizando la fecha en la cual fue
modifica la representación o bien con un valor generado a partir del contenido de
la misma.

###Expiración
Los mecanismos de expiración varian segun la version del protocolo HTTP que
estemos utilizando

###Validación

### Cache Control Headers

| Request                             |   Response                    |
| ------------------------------------|-------------------------------|
| "no-cache"                          | "public"                      |
| "no-store"                          | "private" [="field-name"]     |
| "max-age" "=delta-seconds           | "no-cache" [="field-name"]    |
| "max-stale" [=delta-seconds ]       | "no-store"                    |
| "min-fresh" "=" delta-seconds       | "no-transform"                |
| "no-transform"                      | "must-revalidate"             |
| "only-if-cached"                    | "proxy-revalidate"            |
| cache-extension                     | "max-age" "=" delta-seconds   |
|                                     |  "s-maxage" "=" delta-seconds |




### ¿Qué es cacheable?

A menos que un header de Cache-Control especifique otra cosa, una cache siempre podría almacenar una **respuesta exitosa** (status = 200 OK),
podría servirla sin validar si está **fresca** (no venció) o bien, luego de una **validación exitosa** contra el servidor de origen (304 Not Modified | 200 OK)

Si la respuesta no lleva información de expiración ni de validación, se espera que la cache no almacene dicha respuesta. Decimos se espera ya que en ciertos casos, este tipo de respuestas puede ser almacenada y devuelta (por ejemplo, en situaciones de baja o nula conectividad)

---- esto podría pasarse  a otra sección...
El tráfico que viaja encriptado (https) no puede ser almacenado en cache, justamente debido a que el contenido no es visible para las caches intermedias (Existen de todas formas soluciones para estos casos que serán comentadas mas adelante...)

Cookies y Authorization headers: Las secciones privadas de una página (es decir que pertenecen a un usuario específico), no pueden ser almacenadas en caches compartidas, ya que justamente al ser compartidas, la información almacenada dejaría de ser privada.

 

### Cache Control Headers

| Request                             |   Response                    |
| ------------------------------------|-------------------------------|
| "no-cache"                          | "public"                      | 
| "no-store"                          | "private" [="field-name"]     | 
| "max-age" "=delta-seconds           | "no-cache" [="field-name"]    | 
| "max-stale" [=delta-seconds ]       | "no-store"                    | 
| "min-fresh" "=" delta-seconds       | "no-transform"                | 
| "no-transform"                      | "must-revalidate"             | 
| "only-if-cached"                    | "proxy-revalidate"            | 
| cache-extension                     | "max-age" "=" delta-seconds   | 
|                                     |  "s-maxage" "=" delta-seconds | 
--------------------------------------|--------------------------------


Las directivas de control de cache se pueden dividir en las siguientes categorías:

  - Restrictiones acerca de qué puede ser **cacheado** (sólo disponibles para el servidor de origen)

  - Restricciones acerca de qué puede ser **almacenado** en una cache (servidor | user agent)

  - Modificaciones de los mecanismos báscios de **expiración***. (servidor | user agent)

  - Controles sobre la **revalidación** y **actualización** de cache (user agent)

  - Control sobre la transformación de entidades

  - Extensiones al sistema de cache.



### Controles sobre qué es almacenable en cache
#### public

**Response:**
 
 Indica que la respuesta podría ser almacenada por **cualquier** cache, incluso si normalmente no sería almacenable, o sólo cacheable por una cache no compartida.


#### private

**Response:** 

  Indica que la respuesta o parte de la respuesta está destinada sólo un usuario, por lo tanto no debe ser almacenada por caches compartidas (ya que no serviría como respúesta a requerimientos de otros usuarios). 
Sí podría almacenarse en una cache privada.

#### no-cache

Esta directiva puede llevar opcionalmente un valor, que debe ser le nombrede un header.

**Request:**
  Esta directiva indica que el cliente está buscando una respuesta que no provenga de cache sin antes haber sido validada con el servidor de  origen.

**Response:**

  Esta directiva indica que la cache no debe utilizar esta respuesta para satisfacer una petición subsiguiente sin antes validarla exitosamente con el servidor de origen. Esto permite al servidor de origen evitar que las caches devuelvan respuestas almacenadas vencidas, a pesar de que hayan sido configuradas para hacerlo de esta manera.

Atención: Esta directiva no implica que las caches **no almacenen** la respuesta.


### Control sobre qué puede ser guardado por una cache
#### no-store

 Usado tanto en la respuesta como el la petición, no-store indica a las caches, compartidas o privadas, que ni la respuesta ni la petición deben ser almacenadas en almacenamiento no volátil y que deben intentar eliminarlas lo antes posible luego de utilizarlas. Esto se utiliza con el fin de prevenir el almacenamiento o publicación inadvertidas de información sensible.


### Control sobre mecanismo de expiración

#### max-age
**Request**
  Indica que el cliente está solicita una respuesta cuya edad no es mayor al valor especificado. A menos que el request posea también un valor para max-stale, significa que el cliente no acpetará una respuesta vencida.

**Response**
  Indica a las caches la edad para la cual el recurso debe dejar de considerarse fresco.

**Nota:** Si tanto el request como response incluyen la directiva max-age, la menor de las dos es la que se tiene en cuenta para determinar la frescura de la entrada en la cache.


#### s-maxage

**Response**
  Esta directiva afecta **sólo** a caches compartidas (shared max age). Tiene la misma semántica que **max-age + proxy-revalidate** (es decir, no permite a la cache reenviar una respuesta vencida sin antes revalidarla con el servidor de origen). En las caches compartidas este control tiene precedencia tanto sobre **max-age** como sobre **expires**.


#### min-fresh

**Request**
  Indica que el cliente quiere una respuesta que seguirá siendo válida al menos durante la cantidad de segundos especificados

#### max-stale
**Request**
  Indica que el cliente busca respuestas que están vencidas. Si se asigna un valor, indica que aceptará respuestas cuya edad excedan a la edad de vencimiento a lo sumo por dicho valor. Si no se asigna valor, indica que el cliente aceptará respuestas de cualquier edad.


### Control de revalidación y actualización de caches


End-to-end reload
The request includes a "no-cache" cache-control directive or, for compatibility with HTTP/1.0 clients, "Pragma: no-cache". Field names MUST NOT be included with the no-cache directive in a request. The server MUST NOT use a cached copy when responding to such a request.
Specific end-to-end revalidation
The request includes a "max-age=0" cache-control directive, which forces each cache along the path to the origin server to revalidate its own entry, if any, with the next cache or server. The initial request includes a cache-validating conditional with the client's current validator.
Unspecified end-to-end revalidation
The request includes "max-age=0" cache-control directive, which forces each cache along the path to the origin server to revalidate its own entry, if any, with the next cache or server. The initial request does not include a cache-validating
conditional; the first cache along the path (if any) that holds a cache entry for this resource includes a cache-validating conditional with its current validator.


#### only-if-cached
**Request**
  Indica que el cliente quiere una repuesta guardada en cache y no desea que se recargue ni se valide nada con el servidor de origen.
  La cache que recibe esta solicitud, o bien devuelve el recurso que tiene almacenado, o devuelve una respuesta 504 (Gateway Timeout) si no tiene el recurso solicitado.

#### must-revalidate
**Response**
Esta directiva permite al servidor de origen requerir que las caches que almacenen la respuesta no la utilicen luego de vencida sin antes revalidarla con el servidor. (Esto existe debido a que una cache puede configurar se para ignorar la fecha de vencimiento establecida por el servidor)
Si por algún motivo la cache no puede contactar al servidor para revalidar, la cache debe devolver una respuesta 504 (Gateway Timeout)

Los servidores deberían utilizar esta directiva sólo si una falla al revalidar genera una operación incorrecta (por ejemeplo, silenciosamente no ejecutar una transacción financiera)

#### proxy-revalidate
**Response**
  Esta directiva es igual a must-revalidate con la diferencia que **NO** aplica a caches privadas de user agent.
  Se podría utilizarse por ejemplo, en la respuesta de un requerimiento autenticado para permitir a la cache del usuario almacene la respuesta, de manera que mas tarde no necesite revalidarla con el servidor, pero requiriendo que las caches compartidas en el camino revaliden cada vez con el servidor (de manera de garantizar que cada request sea autenticado) (Notar que además habría que agregar una directiva "Cache-Control: public" )


#### no-transform
**Request**
  Dado que algunas caches tienen la capacidad de ser configuradas para transformar los datos que almacenan (por ejemplo, comprimir o cambiar de formato una imagen para utilizar menor cantidad de espacio), esta directiva permite al cliente solicitar una repuesta que no haya sido modificada por ninguna cache. 