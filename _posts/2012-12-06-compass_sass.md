---
layout: post
title: Compass & Sass
author: Magalí Defalco
author_email: mdefalco@gmail.com
---

## {{ page.title }}

### Compass

Es un framework opensource para CSS escrito en Ruby y que utilizando Sass
(extensión de CSS3) hace que el código sea más limpio, y más actualizable.
Compass provee una serie de comandos que pueden utilizarse para configurar
y administrar el proyecto.

Para más detalle visitar el siguiente link que contiene un listado con los
comandos provistos por Compass:

[Comandos Compass ](http://compass-style.org/help/tutorials/command-line/)

#### Instalación

Para comenzar a usar Compass teniendo Ruby ya instalado:

{% highlight bash %}
$ gem install compass [path/to/project]
{% endhighlight %}

Para crear crear el proyecto:

{% highlight bash %}
$ compass create < myproject >
{% endhighlight %}

Este comando genera una carpeta con el nombre especificado en < myproject > y
dentro de la misma dos carpetas más, una llamada "stylesheets" que contiene
los archivos css generados a partir de la compilación de los archivos .sass
ubicados en la carpeta "sass". Estos últimos son los que debemos modificar
para cambiar la vista de nuestra aplicación.

Una vez ejecutado el comando anterior se listan en consola los links que deben
agregarse en la pagina para importar los archivos necesarios:

{% highlight bash %}
	To import your new stylesheets add the following lines of HTML (or equivalent) to your webpage:

	< link href="/stylesheets/screen.css" media="screen, projection" rel="stylesheet" type="text/css" />
	< link href="/stylesheets/print.css" media="print" rel="stylesheet" type="text/css" />
{% endhighlight %}

Para compilar los .sass ejecutar el comando:

{% highlight bash %}
$ compass watch [path/to/project]
{% endhighlight %}

Este comando se queda a la espera de nuevos cambios en los archivos .sass
(cambios guardados), cuando detecta alguno compila automaticamente estos
archivos regenerando los .css correspondientes.

En el directorio raíz de nuestro proyecto Compass también se genera un
archivo "config.rb". La configuración es esencialmente el cerebro de Compass
y define una serie de variables para indicar dónde están los Sass, CSS,
imagenes y los archivos JavaScript, qué extensiones requieren, qué sintaxis
se prefiere, el estilo de salida entre otras.

### Sass

Sass es una extensión de CSS3, que provée anidamiento de reglas, variables,
mixins (son como clases CSS pero se pueden reutilizar y parametrizar) y
herencia.

La implementación oficial de Sass es opensource y está desarrollada en Ruby.

Sass tiene dos sintaxis. La más usada es conocida como SCSS (Sassy CSS, con
extensión .scss), y es un superconjunto de la sintaxis de CSS3, lo que
significa que todas las hojas de estilo CSS3 válidas, son válidas también
en SCSS.

La segunda, es conocida como sintaxis indentada (.sass). En lugar de llaves y
punto y coma, se utilizá la indentación de líneas para especificar bloques.

El ejemplo siguiente muestra las mismas reglas en las diferentes sintaxis.

{% highlight scss %}
$blue: #3bbfce;
$margin: 16px;

.content-navigation {
	border-color: $blue;
	color: darken($blue, 9%);
}
{% endhighlight %}

{% highlight sass %}
$blue: #3bbfce
$margin: 16px

.content-navigation
		border-color: $blue
		color: darken($blue, 9%)
{% endhighlight %}

##### VARIABLES

Sass permite la definición de variables. Comienzan con el signo "$" y se
asignan con ":".

{% highlight scss %}
$blue: #3bbfce;
$margin: 16px;
{% endhighlight %}

Soporta 4 tipos de datos:

- Numbers
- Strings
- Colors 
- Booleans

Las variables pueden ser argumentos o resultado de funciones.

{% highlight sass %}
$blue: #3bbfce
$margin: 16px

.content-navigation
  border-color: $blue
  color: darken($blue, 9%)

.border
  padding: $margin / 2
  margin: $margin / 2
  border-color: $blue
{% endhighlight %}

{% highlight css %}
.content-navigation {
  border-color: #3bbfce;
  color: #2b9eab;
}

.border {
  padding: 8px;
  margin: 8px;
  border-color: #3bbfce;
}
{% endhighlight %}

##### ANIDAMIENTO

{% highlight sass %}
table.hl
  margin: 2em 0
  td.ln
    text-align: right

li
  font:
    family: serif
    weight: bold
    size: 1.2em
{% endhighlight %}

{% highlight css %}
table.hl {
  margin: 2em 0;
}
table.hl td.ln {
  text-align: right;
}

li {
  font-family: serif;
  font-weight: bold;
  font-size: 1.2em;
}
{% endhighlight %}

##### MIXINS

Un Mixin es una sección de código que contiene cualquier código válido de
Sass. Siempre que un mixin es llamado, el resultado de la traducción del mismo
se inserta en la posición de la llamada.

{% highlight sass %}
@mixin table-base
  th
    text-align: center
    font-weight: bold
  td, th
    padding: 2px

@mixin left($dist)
  float: left
  margin-left: $dist

#data
  @include left(10px)
  @include table-base
{% endhighlight %}

{% highlight css %}
#data {
  float: left;
  margin-left: 10px;
}

#data th {
  text-align: center;
  font-weight: bold;
}

#data td, #data th {
  padding: 2px;
}
{% endhighlight %}

##### HERENCIA

La herencia se realiza insertando una línea dentro de un bloque de código
utilizando el @extender como palabra clave, que hace referencia a otro
selector. El selector de atributos extendido se aplica al selector que llama.

{% highlight sass %}
.error
  border: 1px #f00
  background: #fdd

.error.intrusion
  font-size: 1.3em
  font-weight: bold

.badError
  @extend .error
  border-width: 3px
{% endhighlight %}

{% highlight css %}
.error, .badError {
  border: 1px #f00;
  background: #fdd;
}

.error.intrusion,
.badError.intrusion {
  font-size: 1.3em;
  font-weight: bold;
}

.badError {
  border-width: 3px;
}
{% endhighlight %}

Referencias:

* http://compass-style.org/
* http://sass-lang.com/
* http://en.wikipedia.org/wiki/Sass_%28stylesheet_language%29
