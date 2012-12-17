---
layout:   post
title:    Rapid prototyping con Serve
excerpt:  Un breve paseo por Serve, una herramienta para realizar prototipos HTML muy versátil.
author:   Nahuel Cuesta Luengo
author_email: nahuelcuestaluengo@gmail.com
---

## {{ page.title }}

[Serve](http://get-serve.com/) es una herramienta para realizar sitios web tanto estáticos como dinámicos, que - entre otras - tiene las siguientes bondades:

* Permite usar partials y layouts (y por ende, respetar el principio [DRY](http://en.wikipedia.org/wiki/Don't_repeat_yourself)),
* Soporta distintos markups además de HTML:
    * [ERB](http://ruby-doc.org/stdlib-1.9.3/libdoc/erb/rdoc/ERB.html),
    * [Haml](http://haml.info/),
    * [Markdown](http://daringfireball.net/projects/markdown/),
    * [Textile](http://textile.thresholdstate.com/),
    * [Slim](http://slim-lang.com/),
* Soporta a su vez diferentes supersets de CSS:
    * [SCSS o SASS](http://sass-lang.com/) + [Compass](http://desarrollo-cespi.github.com/2012/12/06/compass_sass.html),
    * [LESS](http://lesscss.org/),
* No tiene grandes dependencias,
* Puede generar sitios con HTML que se pueden servir estáticamente,
* Provee una estructura básica de proyecto práctica para organizar los archivos lógicamente,

### Instalación

Serve es una [gema](https://rubygems.org/gems/serve), por lo que instalarla es sencillo:

{% highlight bash %}
$ gem install serve
{% endhighlight %}

Una vez instalada, tendremos disponible el comando `serve` para interactuar con la herramienta desde la CLI.

## Creando un prototipo con Serve

Una vez instalado Serve, podemos comenzar a utilizarlo para crear un prototipo de una aplicación. A modo de ejemplo, vamos a realizar el prototipo de una pequeña aplicación: **Gabbo**.

### Creación del proyecto con Serve

Situados en el directorio padre de donde querramos crear el proyecto del prototipo, debemos indicar a Serve que queremos crear un nuevo proyecto. Para esto, ejecutamos el siguiente comando:

{% highlight bash %}
$ serve create gabbo-prototype
{% endhighlight %}

Si todo salió bien, deberíamos ver algo similar a esto:

{% highlight bash %}
To start serving your project, run:

    cd "gabbo-prototype"
    serve

Then go to http://localhost:4000 in your web browser.

Have fun!
{% endhighlight %}

Si hacemos lo que indica, vamos a ver [en nuestro browser](http://localhost:4000/) una página de bienvenida como esta:

![Bienvenida de Serve](/static/serve-welcome-page.png)

#### Pero yo veo algo feo con un error!

Si en lugar de la página linda de bienvenida ven una página sin estilos con un error en la cabecera como este:


    Syntax error: Invalid US-ASCII character "\xC2"
            on line 9 of ./stylesheets/partials/_content.scss
            from line 6 of ./stylesheets/screen.scss


Es porque tienen instalado Ruby >1.9 y necesitan especificar el charset que las hojas de estilos SCSS usan.

Para solucionar esto, podemos configurar el encoding global del sistema, mediante los siguientes `export` en nuestro `.bashrc` o `.zshrc`:

{% highlight bash %}
export LC_ALL=en_US.UTF-8
export LANG=en_US.UTF-8
{% endhighlight %}

O agregar una directiva a los archivos SCSS que hay en el proyecto:

{% highlight scss %}
@charset "UTF-8";

// ... resto del archivo ...
{% endhighlight %}

O ejecutar el siguiente comando:

{% highlight bash %}
for scss in `find stylesheets -name '*.scss'`; do echo '@charset "UTF-8";' | cat - $scss > /tmp/tmp.scss && mv /tmp/tmp.scss $scss; done
{% endhighlight %}

Luego de esto, refrescando en el browser la página, el error debería desaparecer.

### Estructura del proyecto

A esta altura, deberíamos tener en nuestro proyecto una estructura similar a la siguiente:

    gabbo-prototype
    |-- Gemfile
    |-- Gemfile.lock
    |-- README.md
    |-- compass.config
    |-- config.ru
    |-- public
    |   |-- images
    |   |   `-- serve-logo.png
    |   |-- javascripts
    |   `-- stylesheets
    |       `-- screen.css
    |-- stylesheets
    |   |-- modules
    |   |   |-- _all.scss
    |   |   |-- _links.scss
    |   |   |-- _typography.scss
    |   |   `-- _utility.scss
    |   |-- partials
    |   |   |-- _base.scss
    |   |   |-- _content.scss
    |   |   `-- _layout.scss
    |   `-- screen.scss
    |-- tmp
    |   `-- restart.txt
    `-- views
        |-- _layout.html.erb
        |-- index.redirect
        |-- layouts
        |   `-- default.html.erb
        |-- view_helpers.rb
        `-- welcome.html.erb


## Posibilidades

Si bien este post se centró en utilizar Serve para generar prototipos web, siendo esta una herramienta que utiliza contenidos semi-dinámicos para generar sitios estáticos, se la podría utilizar para servir contenidos y no únicamente realizar prototipos. Por ejemplo, podríamos usarlo como un CMS, blog o como generador de documentación de algún proyecto.

## Recursos

* [Serve](http://get-serve.com/)
* [Serve en GitHub](https://github.com/jlong/serve)
* [gabbo-prototype en GitHub](https://github.com/ncuesta/gabbo-prototype)