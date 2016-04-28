---
date: 2015-12-23 14:00:00 -3000
layout: post
title: Flush del DNS en Google Chrome
excerpt: Como limpiar la cache de google chrome cuando modificamos la IP de un sitio
author: Christian Rodriguez
categories: chrome dns
usernames: [ chrodriguez ]
---
Es muy común, sobre todo cuando se trabaja en infraestructura probar una
configuración que ya está funcionando en una IP determinada, pero engañando a
nuestro navegador editando `/etc/hosts` y agregando un nombre determinado con la
IP de nuestra PC o directamente `127.0.0.1`

Sin embargo, Google Chrome cachea las consultas al DNS y sockets que utilizó
previamente, por lo que si tenemos Google Chrome con alguna pestaña que haya
accedido a la URL que intentamos cambiar, es probable que el navegador siga
accediendo a la IP anterior sin tomar los cambios que hicimos en `/etc/hosts` o
incluso en los DNS.

Para forzar al navegador que limpie la cache, es necesario reiniciar la
aplicación o seguir los siguientes pasos. Acceder al las siguientes URLs
limpiando la cache:

* **Flush de sockets:** chrome://net-internals/#sockets
* **Limpiar cache de DNS:** chrome://net-internals/#httpCache

