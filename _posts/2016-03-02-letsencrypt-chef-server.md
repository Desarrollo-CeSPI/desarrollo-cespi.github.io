---
layout: post
title: Letsencrypt y Chef 12 server
author: Christian Rodriguez
categories: chef ssl
---

La idea es que el server de chef utilice certificados válidos para evitar
problemas con el uso de knife, así como la integración con el supermarket de una
intranet.

## Instalando letsencrypt

En el servidor donde se encuentra instalado el server de chef, instalar:

{% highlight bash %}
sudo apt-get install bc git
{% endhighlight %}

Luego descargar letsencrypt:

{% highlight bash %}
sudo git clone https://github.com/letsencrypt/letsencrypt /opt/letsencrypt
{% endhighlight %}

## Obtenemos los certificados

Asumiendo que deseamos un certificado para el dominio
chef.miorganizacion.com.ar, entonces procedemos con los siguientes pasos:

### Liberamos provisoriamente el puerto 80

Para poder obtener el primer certificado, debemos bajar el actual servicio de
nginx:

{% highlight bash %}
sudo chef-server-ctl stop nginx
{% endhighlight %}

### Corremos letsencrypt

{% highlight bash %}
cd /opt/letsencrypt
sudo ./letsencrypt-auto certonly --standalone
{% endhighlight %}

Seguimos la ayuda en pantalla y se completa la instalación dejando los
certificados en el directorio `/etc/letsencrypt/live/chef.miorganizacion.com.ar`

## Configuramos chef

Editamos `/etc/opscode/chef-server.rb` que probablemente no exista, con el
siguiente contenido:

{% highlight ruby linenos %}
nginx['ssl_certificate'] = '/etc/letsencrypt/live/chef.miorganizacion.com.ar/fullchain.pem'
nginx['ssl_certificate_key'] = '/etc/letsencrypt/live/chef.miorganizacion.com.ar/privkey.pem'
nginx['ssl_ciphers'] = 'EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH'
nginx['ssl_protocols'] = 'TLSv1 TLSv1.1 TLSv1.2'
{% endhighlight %}

Una vez creado este archivo, se procede a reconfigurar chef:

{% highlight bash %}
sudo chef-server-ctl reconfigure
{% endhighlight %}

## Configurando la auto renovación de los certificados

Para simplificar este paso, utilizaremos un plugin de `letsencrypt` llamado
`webroot` que permite *actualizar los certificados en caliente*, esto es sin reiniciar el
servidor. La idea detrás de este script es:

1. Al correr el script de `letsencrypt` se crea un archivo en un directorio bajo
nuestro document root
2. El servicio de letsencrypt consultará nuestro server en busca de este archivo,
por lo que el mismo debe estar disponible desde la WEB
  1. Como el script se crea con un nombre aleatorio, el servicio, y sólo el
servicio de letsencrypt conocerá la ubicación real del mismo, y la
contravalidación se hará con nuestro server

Para poder habilitar este plugin, debemos modificar la configuración del nginx
embebido que provee `chef-server`. 

### Habilitando el directorio `.well-known/` en nuestro web server

Para habilitar este `location` en nginx, debemos agregar una configuración en el
archivo `/var/opt/opscode/nginx/etc/addon.d/10-letsencrypt_external.conf` con el
contenido:

{% highlight bash %}

location ~ /.well-known {
  allow all;
}

{% endhighlight %}

Y luego reiniciamos nginx:

{% highlight bash %}
sudo chef-server-ctl restart nginx
{% endhighlight %}

### Probamos la actualización del certificado

Ejecutamos el siguiente comando, modificando con los datos del dominio
correspondiente para el caso de su chef server

{% highlight bash %}
cd /opt/letsencrypt && \
  sudo ./letsencrypt-auto certonly \
      -a webroot --agree-tos \
      --renew-by-default \
      --webroot-path=/var/opt/opscode/nginx/html \
      -d chef.miorganizacion.com.ar
{% endhighlight %}

### Simplificamos el script anterior

Creando un archivo de configuración con los dato que enviamos como parámetro,
podemos simplificar la ejecución del comando anterior. Creamos el archivo
`/etc/letsencrypt/miorganizacion-webroot.ini` con el siguiente contenido:

{% highlight bash  %}
rsa-key-size = 4096
email = user@miorganizacion.com.ar
domains = miorganizacion.com.ar
webroot-path = /var/opt/opscode/nginx/html
{% endhighlight %}

y probamos el sctipt:

{% highlight bash %}
cd /opt/letsencrypt && \
  sudo ./letsencrypt-auto certonly \
    -a webroot \
    --renew-by-default \
    --config /etc/letsencrypt/miorganizacion-webroot.ini
{% endhighlight %}

### Script de renovacion por demanda

El siguiente script verificará la validez del certificado y generará un nuevo
certificado, sólo si es necesario:

{% highlight bash linenos %}
#!/bin/bash

web_service='nginx'
config_file='/etc/letsencrypt/miorganizacion-webroot.ini'

le_path='/opt/letsencrypt'
exp_limit=30;

if [ ! -f $config_file ]; then
        echo "[ERROR] config file does not exist: $config_file"
        exit 1;
fi

domain=`grep "^\s*domains" $config_file | sed "s/^\s*domains\s*=\s*//" | sed 's/(\s*)\|,.*$//'`
cert_file="/etc/letsencrypt/live/$domain/fullchain.pem"

if [ ! -f $cert_file ]; then
  echo "[ERROR] certificate file not found for domain $domain."
fi

exp=$(date -d "`openssl x509 -in $cert_file -text -noout|grep "Not After"|cut -c 25-`" +%s)
datenow=$(date -d "now" +%s)
days_exp=$(echo \( $exp - $datenow \) / 86400 |bc)

echo "Checking expiration date for $domain..."

if [ "$days_exp" -gt "$exp_limit" ] ; then
  echo "The certificate is up to date, no need for renewal ($days_exp days left)."
  exit 0;
else
  echo "The certificate for $domain is about to expire soon. Starting webroot renewal script..."
        $le_path/letsencrypt-auto certonly -a webroot --agree-tos --renew-by-default --config $config_file
  echo "Reloading chef $web_service"
  chef-server-ctl restart $web_service
  echo "Renewal process finished for domain $domain"
  exit 0;
fi
{% endhighlight %}

Creamos un archivo con el contenido de arriba llamado `/usr/local/sbin/le-renew`
y modificamoss la segunda línea donde hace mención al archivo de configuración.
Damos permiso de ejecución:

{% highlight bash %}
sudo chmod +x /usr/local/sbin/le-renew 
{% endhighlight %}

Finalmente probamos el scipt:

{% highlight bash %}
sudo le-renew 
{% endhighlight %}

Verificando que todo funciona bien

### Agregamos el cron

Corremos

{% highlight bash %}
crontab -e
{% endhighlight %}

y agregamos:

{% highlight bash %}
30 2 * * 1 /usr/local/sbin/le-renew >> /var/log/le-renewal.log
{% endhighlight %}

Esto significa que correrá todos los lunes a las 2:30 am
