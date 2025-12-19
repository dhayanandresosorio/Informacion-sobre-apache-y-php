# Apache 2 – Instalación y Configuración en Ubuntu

---

## ¿Qué es Apache?

Apache es un software de servidor web de código abierto que permite alojar sitios web y aplicaciones en Internet. Es gratuito, estable y ampliamente utilizado. Actualmente sigue siendo uno de los servidores web más populares, aunque alternativas como **Nginx** están ganando terreno.

Como servidor web, Apache gestiona solicitudes HTTP. Su función principal es responder a las peticiones de los usuarios que acceden a una URL, enviándoles el contenido solicitado.

Fue desarrollado inicialmente por una comunidad de voluntarios y actualmente está mantenido por la **Apache Software Foundation (ASF)**, una organización sin ánimo de lucro que gestiona múltiples proyectos de software libre.

---

## Instalación y comprobación de funcionamiento

Para instalar Apache utilizamos el siguiente comando:

```bash
sudo apt install apache2
```

Una vez finalizada la instalación, comprobamos que el servicio se ha iniciado correctamente:

```bash
sudo systemctl status apache2
```

Si el servicio aparece como **active (running)**, Apache está funcionando correctamente.

---

## Directorios y archivos de configuración

### Directorio principal de configuración

```text
/etc/apache2/
```

Este directorio contiene todos los archivos de configuración de Apache.

### Archivo de configuración principal

```text
/etc/apache2/apache2.conf
```

Aquí se define la configuración global del servidor: puertos, módulos cargados y directivas generales.

### Directorios de módulos

```text
/etc/apache2/mods-available/
/etc/apache2/mods-enabled/
```

* **mods-available**: módulos disponibles.
* **mods-enabled**: módulos activos (enlaces simbólicos).

Activar un módulo:

```bash
sudo a2enmod ssl
```

Desactivar un módulo:

```bash
sudo a2dismod ssl
```

### Directorios de sitios web

```text
/etc/apache2/sites-available/
/etc/apache2/sites-enabled/
```

* `sites-available`: configuraciones de VirtualHost.
* `sites-enabled`: enlaces a los sitios activos.

### Directorio de contenido web

```text
/var/www/html/
```

Aquí se almacenan los archivos del sitio web, por ejemplo `index.html`.

---

## Gestión del servicio

Comandos más habituales para gestionar Apache:

```bash
sudo systemctl start apache2
sudo systemctl stop apache2
sudo systemctl restart apache2
sudo systemctl reload apache2
sudo systemctl status apache2
sudo systemctl enable apache2
sudo systemctl disable apache2
```

---

## Creación y configuración de un VirtualHost

Un **VirtualHost** permite alojar múltiples sitios web en un mismo servidor.

Primero visualizamos los sitios disponibles:

```bash
ls /etc/apache2/sites-available
```

Visualizamos el sitio por defecto:

```bash
cat /etc/apache2/sites-available/000-default.conf
```

Copiamos el VirtualHost por defecto y lo renombramos:

```bash
sudo cp 000-default.conf dhayan.conf
```

Editamos el archivo:

```bash
sudo nano dhayan.conf
```

Configuramos el dominio y la ruta del sitio web.

Creamos el directorio del sitio y el archivo `index.html`:

```bash
sudo mkdir -p /var/www/dhayan
sudo nano /var/www/dhayan/index.html
```

Ejemplo de HTML utilizado:

```html
<h1>Bienvenido a mi página Dhayan</h1>
```

Activamos el sitio:

```bash
sudo a2ensite dhayan.conf
sudo systemctl reload apache2
```

Añadimos el dominio al archivo `/etc/hosts`:

```text
127.0.0.1   www.dhayan.com
```

y si ahora buscamos en el navegador nuestra página web (virtualhost), se habrá hecho correctamente. 


---

## Módulos

### mod_alias

Su principal función es cambiar o redirigir direcciones web.
Por ejemplo, hace que "dhayan.com/webantigua" lleve a "dhayan.com/wennueva" o conecta una carpeta a una URL corta.


### mod_rewrite

Modifica URLs para que sean más claras.
Se usa para cambiar algo como "pagina.php?id=123" a "pagina/123", haciéndola  más fácil de leer.


### mod_dir

Decide qué archivo se muestra si no se indica uno.
Sirve para que si entras a "dhayan.com/directoriorandom/" sin indicar un archivo, te muestra automáticamente "index.html".


### mod_auth_basic

Esto como hemos visto en clase, es un mod para pedir un usuario y contraseña para entrar a una página.
Lo que hace, es proteger secciones privadas, como un área de administración.

### mod_status

Como su nombre indica, muestra cómo está funcionando el servidor.
Por lo general, nos dice cuántos usuarios están conectados o si hay problemas.


### mod_expires

Este controla cuánto tiempo guarda el navegador archivos como imágenes.
Activar esto, evita que los usuarios descarguen lo mismo repetidamente, haciendo la web más rápida.


---

## Activación del protocolo HTTPS

Comprobamos que OpenSSL está instalado:

```bash
openssl version
```

Creamos un directorio para certificados:

```bash
sudo mkdir /etc/ssl/dhayansitio
```

Ahora, dentro de ese directorio, generamos un certificado autofirmado que servirá para activar HTTPS en Apache.
Usaremos los siguientes comandos y rellenaremos la información que nos pide de manera personalizada, en mi caso llamaré a todo dhayansitio:

```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
-keyout /etc/ssl/dhayansitio/dhayansitio.key \
-out /etc/ssl/dhayansitio/dhayansitio.crt
```

Activamos SSL en Apache:

```bash
sudo a2enmod ssl
sudo systemctl restart apache2
```

El siguiente paso en nuestra práctica, es crear un nuevo virtualhost que escuche por el puerto 443, el puerto que usa HTTPS.
Para ello, copiamos el archivo del sitio por defecto y lo adaptamos con las siguientes ordenes:

```bash
sudo cp /etc/apache2/sites-available/default-ssl.conf /etc/apache2/sites-available/dhayansitio-ssl.conf
sudo nano /etc/apache2/sites-available/dhayansitio-ssl.conf
```

Configuramos certificados y rutas.

Activamos el sitio HTTPS:

```bash
sudo a2ensite dhayansitio-ssl.conf
sudo systemctl reload apache2
```

Añadimos el dominio dhayansitio.com a `/etc/hosts`.

```bash
sudo nano /etc/hosts
```

---

## UFW

Comprobamos estado:

```bash
sudo ufw status
```

Activamos firewall:

```bash
sudo ufw enable
```

Permitimos Apache:

```bash
sudo ufw allow 'Apache'
sudo ufw allow 'Apache Secure'
```

---

## Autenticación

### Autenticación básica

Crearemos un usuario, que en mi caso se llamará dhayan y le asignaremos una contraseña para que pueda acceder a la página web:

```bash
sudo htpasswd -c /etc/apache2/.htpasswd dhayan
```

Creamos directorio protegido:

```bash
sudo mkdir /var/www/dhayan/privado
sudo nano /var/www/dhayan/privado/index.html
```

Configuramos el VirtualHost:

```apache
<Directory /var/www/dhayan/privado>
AuthType Basic
AuthName "Zona Protegida"
AuthUserFile /etc/apache2/.htpasswd
Require valid-user
</Directory>
```

Reiniciamos Apache y podemos proceder a intentar entrar en la página web secreta y efectivamente, nos pide el usuario que hemos creado con su contraseña.


---

### Autenticación Digest

Creamos directorio:

```bash
mkdir /var/www/dhayan/html/nuevodirectorio
```

Creamos archivo HTML.

Creamos archivo Digest:

```bash
htdigest -c /etc/apache2/.htdigest "Espacio Protegido Por Digest" dhayan
```

Configuramos el VirtualHost:

```apache
<Directory /var/www/dhayan/html/nuevodirectorio>
AuthType Digest
AuthName "Espacio Protegido Por Digest"
AuthUserFile /etc/apache2/.htdigest
Require valid-user
</Directory>
```

Activamos módulo:

```bash
sudo a2enmod auth_digest
sudo systemctl restart apache2
```

---

## PHP

### Instalación y comprobación con Apache

Instalamos PHP y módulos:

```bash
sudo apt install php libapache2-mod-php
```

Comprobamos versión:

```bash
php -v
```

Creamos archivo PHP:

```php
<?php
phpinfo();
?>
```

Lo copiamos al directorio web:

```bash
sudo cp phpinfo.php /var/www/html/
```

Accedemos desde el navegador.


---

## Archivo de configuración principal (php.ini)

Ruta:

```text
/etc/php/8.3/apache2/php.ini
```

Directivas configuradas:

```ini
display_errors = Off
expose_php = Off
allow_url_fopen = On
allow_url_include = Off
session.cookie_httponly = On
session.cookie_secure = On
disable_functions = exec
open_basedir = /var/www:/tmp
max_execution_time = 30
memory_limit = 128M
file_uploads = On
upload_max_filesize = 2M
```

Lo que hace cada una de las directivas será lo siguiente: 

display_errors = Off: Lo que indicamos aquí es que no muestre errores en la pantalla, es decir, si algo falla, no aparece el error en el navegador y lo guarda dentro de un log para que solo yo lo pueda ver. 

expose_php = Off: Con esta configuración, hacemos que nuestra estructura de php sea más segura ya que esconde su versión en las respuestas del servidor.

allow_url_fopen On y allow_url_include Off: Aquí lo que indicamos es que podamos descargar datos con el fopen en On pero que no se pueda meter código malicioso desde fuera con el Include Off, es decir, deja la puerta abierta para leer cosas de internet, pero no deja ejecutarlas.

session.cookie_httponly = On y session.cookie_secure = On: En este caso, hacemos que las cookies sean invisibles con el secure en On, pero además, le indicamos que solo viaje por vía HTTPS o vía segura como su nombre indica con el secure On.

disable_functions: Como el propio nombre indica, esta directiva lo que hace es desactivar aquellas herramientas que podrían ser peligrosas si un atacante lograra entrar a nuestro servicio. Alguna muy básica que he añadido es por ejemplo la de exec, que permite ejecutar comando del sistema, añadiendola, he prohibido esta opción.

open_basedir:  Con esta configuración y con la ruta que he puesto, le indico a PHP que solo puede leer y escribir en mi carpeta de mi web, y en /tmp. limitando su movimiento a rutas alternativas, haciéndolo más seguro.

max_execution_time y memory_limit: Con estas directivas lo que se consigue es evitar que un script o programa se quede colgado para siempre o que se coma toda la ram, entonces con le indico que el máximo tiempo de ejecución sea de 30 segundos y el límite de memoria sea de 128 MB. Es una forma muy sencilla de proteger al servidor de ataques DoS por ejemplo.

file_uploads y upload_max_filesize: Aquí permitimos que se puedan subir archivos pero a la vez, indicamos el límite de cuánto ha de pesar dicho archivo, es una herramienta muy útil para formularios por ejemplo, y al limitarlo, evitamos que se colapse el servidor.

Comprobamos configuración:

```bash
apachectl configtest
```

Solucionamos warning de ServerName:

```bash
sudo nano /etc/apache2/apache2.conf
ServerName localhost
```

Reiniciamos Apache.

---

## Webs de consulta

* [https://www.arsys.es/blog/que-es-apache-y-para-que-sirve](https://www.arsys.es/blog/que-es-apache-y-para-que-sirve)
  
* [https://es.wikipedia.org/wiki/Apache_Software_Foundation](https://es.wikipedia.org/wiki/Apache_Software_Foundation)
  
* [https://httpd.apache.org/docs/current/es/mod/](https://httpd.apache.org/docs/current/es/mod/)
  
* [https://www.php.net/manual/es/ini.list.php](https://www.php.net/manual/es/ini.list.php)
  
* [https://www.php.net/manual/es/errorfunc.configuration.php#ini.display-errors](https://www.php.net/manual/es/errorfunc.configuration.php#ini.display-errors)

---
