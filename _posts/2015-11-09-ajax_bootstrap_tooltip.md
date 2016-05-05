---
date: 2015-11-09 12:50:00 -3000
layout: post
title: Bootstrap tooltip ajax
excerpt: Carga de tooltip bootstrap por medio de ajax
author: Federico Otaran
categories: ajax bootstrap
usernames: [ fedeotaran ] 
---
Surgió la necesidad de mostrar mayor cantidad información de un registro dentro de una tabla. Me pareció buena idea que esa información podía visualizarse dentro un tooltip cuando posiciono el cursor sobre el registro.
<!-- more -->
Dado que en el sistema que quiero realizar la funcionalidad utiliza Bootstrap, lo lógico es que use los tooltips que la librería me provee.
Como no tengo esta información disponible en la vista, debo realizar consultas ajax para obtener la información dentro cada tooltips. El problema con esto, es que cuando inicializamos los tooltips de bootstrap debemos ya tener disponible la información que queremos mostrar dentro del atributo `title` del tag `a` o `button`. Por esta razón por mas que realicé la consulta y obtuve la información correctamente, no la visualizaba dentro del tooltip.
![Tooltip sin datos](/assets/images/tooltip_without_data.png){: .center-image }

## Solución
Una solución simple para esto sería colgarme del evento onmouseover del registro donde quiero que se muestre el tooltip.
{% highlight javascript %}
  $(document).on('mouseover', '.target', load_tooltip);
{% endhighlight %}
Éste es el código html del registro donde quiero mostrar el tooltip:
{% highlight erb %}
<td>
  <div class="target" data-id="<%= data.id %>" data-url="<%= ajax_path %>">
    <%= data %>
  </div>
</td>
{% endhighlight %}
En la función `load_tooltip()` lanzar una petición ajax que traiga los datos que quiero mostrar.
{% highlight javascript %}
function load_tooltip() {
    var $element = $(this),
    url = $element.data('url');

    $.ajax(url, { beforeSend: function() { return !(cache.hasOwnProperty(url)); } })
      .done(function(data) { cache[url] = data; })
      .always(function() { render_html_tooltip($element, cache[url]["html"]); });
}
{% endhighlight %}
Para **no realizar tantas consultas ajax al servidor** se guarda el resultado de las peticiones dentro de un arreglo llamado `cache` utilizando como clave la url del de la petición que se terminó de realizar.
Una vez que ya tengo la información y la petición finalizó, inicializamos los tooltips.
{% highlight javascript %}
function render_html_tooltip(elem, html) {
    elem.tooltip({
      title: html,
      html: true,
      container: 'body'
    }).tooltip('show');
  }
{% endhighlight %}
## Resultado:
![Tooltip sin datos](/assets/images/tooltip_with_data.png){: .center-image }

Ésto mismo puede aplicarse al componente **popover** de Bootstrap ya que tienen el mismo comportamiento que los **tooltips**.

