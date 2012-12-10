---
layout: post
title: New York, New York
excerpt: Una pequeña introducción a Sinatra, un microframework para crear aplicaciones web.
author: Patricio Mac Adden
author_email: patriciomacadden@gmail.com
---

## New York, New York

[Sinatra](http://www.sinatrarb.com/) provee una simple [DSL](http://en.wikipedia.org/wiki/Domain-specific_language)
para desarrollar aplicaciones web.

### Pero... ¿Que es un DSL?

DSL se refiere a Domain-specific language, es decir, que es un lenguaje
específico para el dominio que estamos tratando. En pocas palabras, es una
librería que define cómo tratar un problema (o una serie de problemas
similares).

En nuestro caso puntual, Sinatra provee un DSL para desarrollar aplicaciones
web. Es decir, provee una forma sencilla para atender requerimientos HTTP.

Veamos un ejemplo:

Primero definimos el archivo hello_new_york.rb:

{% highlight ruby %}
require 'rubygems'
require 'sinatra'

get '/' do
  'Hello New York!'
end
{% endhighlight %}

Luego simplemente lo ejecutamos:

{% highlight bash %}
$ ruby hello_new_york.rb
{% endhighlight %}

Luego simplemente visitamos 127.0.0.1:4567 y voilà: **Hello New York!**

Ahora bien, como se puede ver, este ejemplo es autoexplicativo, define que
cuando hagamos 'GET /' en 127.0.0.1:4567, siempre mostrará el mensaje
**Hello New York!**

Básicamente, sinatra traduce las peticiones HTTP a bloques ruby que
atienden a esas peticiones. **¿Bien sencillo, no?**

#### Yo planto la bomba, ahora a aprender por su cuenta!

* [README](http://www.sinatrarb.com/intro)!
* [Documentación](http://www.sinatrarb.com/documentation)
* [Sinatra Contrib](http://www.sinatrarb.com/contrib/) Add-ons para sinatra
* [Sinatra: Up and running](http://shop.oreilly.com/product/0636920019664.do)

