---
date: 2015-11-09 12:50:00 -3000
layout: post
title: Bootstrap tooltip ajax
excerpt: Carga de tooltip bootstrap por medio de ajax
author: Federico Otaran
categories: ajax bootstrap
---

# Problema
Surgió la necesidad de mostrar más información de un registro dentro de una tabla. Para no recurrir a una redirección me pareció buena idea que esa información se puede visualizar dentro un tooltip cuando posiciono el cursor sobre dicho registro.
Dado que en el sistema que quiero realizar la funcionalidad utiliza Bootstrap lo adecuado es que utilice los tooltips que la librería nos provee. El problema con esto es que cuando inicializamos los tooltips de bootstrap debemos tener ya disponible la información que queremos mostrar dentro del él. Esta información debe estar en el atributo "title" del tag "a" o "button".

# Solución
Una solución simple para esto sería colgarme del evento onmouseover de la porción de html donde quiero que se muestre el tooptip, lanzar una petición ajax en ese momento que traiga los datos que quiero mostrar y los pegue en el atributo title del tag en cuestion. Luego le decimos al tooltips que se muestre. ¡Así de simple!
Ahora el código:

