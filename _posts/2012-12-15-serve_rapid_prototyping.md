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

Una vez instalado Serve, podemos comenzar a utilizarlo para crear un prototipo de una aplicación. A modo de ejemplo, vamos a realizar el prototipo de una pequeña aplicación imaginaria: **Gabbo**.

### Descripción de la aplicación a prototipar

**Gabbo** es una pequeña aplicación que mostrará frases como si fueran dichas por [Gabbo](http://simpsons.wikia.com/wiki/Gabbo), con una pequeña página de inicio (`/index` - o simplemente `/`), la página principal donde se muestra una frase al azar (`/gabbo`) y una interfaz de administración que permita realizar el CRUD de las frases.

Para no excedernos en el desarrollo de este post, en nuestro prototipo armaremos la introducción y la página donde se muestran las frases.

El resultado final se puede ver en [la página del proyecto en GitHub](http://ncuesta.github.com/gabbo-prototype/).

### Creación del proyecto con Serve

Situados en el directorio padre de donde querramos crear el proyecto del prototipo, debemos indicar a Serve que queremos crear un nuevo proyecto. Para esto, ejecutamos el siguiente comando:

{% highlight bash %}
$ serve create gabbo-prototype
{% endhighlight %}

Si todo salió bien, deberíamos ver algo similar a esto:

    To start serving your project, run:

        cd "gabbo-prototype"
        serve

    Then go to http://localhost:4000 in your web browser.

    Have fun!

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

En la estructura se distinguen principalmente los siguientes elementos:

* `compass.config` - Archivo de configuración utilizado por Compass al momento de convertir los archivos SASS en CSS.
* `config.ru` - Configuración e inicialización de la aplicación [Rack](http://rack.github.com/) con los Middlewares necesarios (por defecto SASS).
* `public/` - Directorio para los assets estáticos (esto incluye las hojas de estilos en formato CSS). Los assets ubicados en este directorio se referencian a partir del `/` de la aplicación. Es decir, la imagen `public/images/serve-logo.png` se referencia desde la página como `/images/serve-logo.png`.
* `stylesheets/` - Directorio para los fuentes SASS (o LESS).
* `tmp/` - Directorio necesario para [Phusion Passenger](https://www.phusionpassenger.com/).
* `views/` - Directorio contenedor de las vistas y los Redirects:
    * `layouts/` - Directorio de layouts. Por defecto contiene el layout `default.html.erb`, que deberemos modificar para quitar los elementos propios de Serve.
    * `*.redirect` - Archivos de redirección. Cuando un request matchea el nombre del archivo sin la extensión (para `index.redirect`, la URL `/index` por ejemplo) se tomará **la última linea de este archivo** y se usará para responder con un HTTP Status [`302 Found`](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.3.3) que redirija a la URL especificada en esa última linea. Cualquier linea anterior en el archivo es considerada comentario.
    * `*.html.erb` - Vistas en formato Embedded RuBy.
    * `view_helpers.rb` - Módulo Ruby con helpers de vista: métodos que podemos usar en nuestras views para reutilizar código o simplificar lógica del lado de las vistas.
    
### Personalización inicial del proyecto

Para empezar a armar el prototipo vamos a necesitar borrar los ejemplos que Serve pone en el proyecto. Para esto, tenemos que:

* Eliminar el Redirect para `index`: `views/index.redirect`.
* Eliminar la vista `views/welcome.html.erb` ya que no la vamos a necesitar más.
* Eliminar la imagen `public/images/serve-logo.png`.
* Limpiar el layout `views/layouts/default.html.erb` para que únicamente contenga la línea {% highlight erb %}<%= yield %>{% endhighlight %} en el tag `<body>`.

Una vez hecho esto, podemos extender un poco la lógica del layout por defecto para permitirnos agregar una hoja de estilo particular para cada acción, si así lo deseáramos - en el esquema actual se carga siempre la misma hoja de estilos global.

Para agregar esa lógica, simplemente tenemos que modificar el layout `views/layouts/default.html.erb` y agregar las siguientes líneas en el tag `<head>`:

{% highlight erb %}
<% if @stylesheet %>
  <link rel="stylesheet" href="./stylesheets/<%= @stylesheet %>">
<% end %>
{% endhighlight %}

De esta forma, si está definida la variable `@stylesheet`, se incluirá una hoja de estilos de nombre igual al valor de esa variable.

### Primera página - Introducción

Ahora que tenemos el proyecto personalizado, pasemos a crear nuestra primera vista. Comenzando por la página de intro, vamos a crear un archivo de vista que responda al request `/index.html` - `views/index.html.erb`. Ponemos un HTML sencillo y adicionalmente indicamos valores para las dos variables del layout:

* `@title` - Con el título de la página, y
* `@stylesheet` - Con el nombre de la hoja de estilos específica para esta página.

{% highlight erb %}
<!-- views/index.html.erb -->
<% @title = "Gabbo!" %>
<% @stylesheet = "index.css" %>

<div id="tv-container">
  <span class="screen name">GABBO</span>
  <a href="./gabbo.html" class="power-button">&rarr;</a>
</div>
{% endhighlight %}

Lo que restaría antes de ir a ver la página es agregar los estilos específicos para esta página, ya que los indicamos con la variable `@stylesheet`. Para esto, creamos el archivo `stylesheets/index.scss` y colocamos en él los estilos para la página de introducción. Al hacer esto, con cada Request se compilará automáticamente una hoja de estilos CSS en `public/stylesheets/index.css` con los estilos especificados en nuestra hoja SASS.

**Nota:** Al no ser el objetivo de este post centrarse en el desarrollo del prototipo en sí, no incluyo los estilos incorporados, pero se los puede ver en el [repo del prototipo terminado en GitHub](https://github.com/ncuesta/gabbo-prototype/blob/master/stylesheets/index.scss).

Ahora, [cargando en el browser la página](http://localhost:4000) veremos la introducción.

### Refreshing

![Refreshing](/static/serve-refreshing.jpg)

Uno de los puntos más cómodos de desarrollar los prototipos en Serve es que una vez que tengamos el server corriendo, con solo refrescar la página veremos los cambios que hagamos, sean de HTML, SASS o assets.

### Segunda página - Gabbo

En este punto, si hacemos clic en el botón de acceso a la página de frases, Serve nos indicará que el recurso no existe:

    Entity not found: /gabbo.html

Para evitar esto, creamos la vista de la segunda página - `views/gabbo.html.erb` - y le agregamos el HTML necesario, junto con las variables para el layout:

{% highlight erb %}
<!-- views/gabbo.html.erb -->
<% @title = "Gabbo!" %>
<% @stylesheet = "gabbo.css" %>
<% @quote = random_quote %>

<div id="tv-container">
    <span class="screen">
        <span id="quote" class="balloon"><%= @quote %></span>
        <img id="gabbo-character" src="./images/gabbo.png" />
    </span>
    <a href="https://twitter.com/share?text=<%= encode @quote %>"
       class="power-button tweet">
        <img src="./images/tweet.png" alt="Tweet" />
    </a>
</div>
{% endhighlight %}

Al igual que cuando creamos la página anterior, tendremos que crear el archivo de estilos para esta página: `stylesheets/gabbo.scss` y agregarle las reglas de estilos propias de esta página.

Ahora, si miramos con un poco de detalle el código HTML de esta vista, vamos a encontrar 2 métodos que utilizamos pero que no están definidos aún: `random_quote` y `encode`. Estos métodos son **helpers de vista**, y como tales deberemos incluirlos en el módulo `views/view_helpers.rb`:

{% highlight ruby %}
module ViewHelpers
  # Returns a random quote
  def random_quote()
    require "net/http"
    require "uri"

    url = "http://iheartquotes.com/api/v1/random?format=text&max_lines=4&max_characters=140&show_permalink=false&show_source=false&source=simpsons_ralph+simpsons_homer+simpsons_chalkboard"
    uri = URI.parse url

    response = Net::HTTP.get_response(uri)
    response.body.chomp!
  end

  # Encodes a string in a URL-safe way
  def encode(string)
    require "uri"

    URI::encode string
  end
end
{% endhighlight %}

Con esto tendremos estas funciones disponibles en cualquier vista.

Además de los helpers, en el HTML se hace referencia a 2 imágenes: `images/gabbo.png` y `images/tweet.png`. Cualquier imagen, por ser asset estático, deberá ser incluida en el directorio `public/` del proyecto y dentro de este en el subdirectorio `images/`.

**Nota:** Todas las referencias a assets o páginas dentro del proyecto las estamos manejando de manera relativa (prefijadas por `./`) porque dependiendo de la configuración del servidor - como es el caso de nuestro entorno de Testing - puede llegar a necesitarlo para poder referenciar correctamente los contenidos.

## Exportando el prototipo

Una vez terminado el desarrollo del prototipo, el siguiente paso va a ser hacerlo accesible. Para esto, podemos montarlo como una aplicación Rack pública o generar el HTML estático del sitio, lo cual además de ahorrar recursos en el servidor, nos permite entregar copias a las partes interesadas directamente, sin dependencias externas.

### Exportando en HTML estático

La forma de exportar el proyecto a HTML estático es muy simple. Asumiendo que se tiene un directorio `build` donde deseamos exportar el prototipo, debemos ejecutar:

{% highlight bash %}
$ serve export . build
{% endhighlight %}

Una vez exportado el prototipo, podremos redistribuir el directorio `build/` y/o publicarlo en algún servidor para que quede accesible. Después de todo, es solo HTML.

**Nota:** Tener en cuenta que si bien nuestro prototipo en la aplicación Rack muestra frases de manera aleatoria, una vez exportado mostrará siempre la misma frase. *Agregando un poco de JS se podría hacer aleatorio también el prototipo exportado*, pero eso excede el alcance del presente post.

## Posibilidades

Si bien este post se centró en utilizar Serve para generar prototipos web, siendo esta una herramienta que utiliza contenidos semi-dinámicos para generar sitios estáticos, se la podría utilizar para servir contenidos y no únicamente realizar prototipos. Por ejemplo, podríamos usarlo como un CMS, blog o como generador de documentación de algún proyecto.

## Recursos

* [Serve](http://get-serve.com/)
* [Serve en GitHub](https://github.com/jlong/serve)
* [gabbo-prototype](http://ncuesta.github.com/gabbo-prototype/)
* [gabbo-prototype en GitHub](https://github.com/ncuesta/gabbo-prototype)