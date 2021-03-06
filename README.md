# NGINX
## Índice

1. [Introducción](#int)
2. [Comparativa con Apache](#com)
3. [Esquema de red](#esq)
4. [Instalación](#ins)
5. [Casos prácticos](#cas)
6. [Referencias](#ref)

<a name="int"></a>
## 1.- Introducción

En este proyecto vamos a crear un servidor web usando NGINX, estudiando el proceso de creación instalación y realizaremos una comparativa con Apache.

<a name="com"></a>
## 2.- Comparativa con Apache

NGINX y Apache son servidor web muy populares usados para ofrecer páginas web a nuestros clientes. Ambos son usados en mas de 500 grandes empresas de todo el mundo.

### Apache

  - Apache fue lanzado al mercado en 1995
  - Apache esta integrado en muchas de las distribuciones mas populares de Linux, como Red Hat o CentOS.
  - Apache usa .htaccess para su configuración. Hay muchos tutoriales de como trabajar con dicho fichero. Permite configurar cosas como redireccionamiento, máxima capacidad de subida de ficheros, limites de memoria, cookies, etc.
  - Cada directorio de Apache tiene su propio archivo de configuración .htaccess.
  - Una de las grandes características de Apache es la facilidad para trabajar con modulos. Con un par de comandos podemos activar y desactivar modulos sin tener que acceder a ningun archivo de configuración.

### Nginx

  - NGINX fue lanzado al mercado en 2004.
  - El mercado de NGINX ha crecido de forma estable durante años.
  - NGINX no tiene configuraciones a nivel de directorio, a diferencia de Apache, lo cual ayuda enormemente en el rendimiento.

<a name="esq"></a>
## 3.- Esquema de red

Para esta tarea, tendremos un servidor virtual con dos tarjeta de red. Una en modo adaptador puente y otra en red interna. Acto seguido, iniciamos la máquina y seguimos el siguiente procedimiento:

1.- Detenemos el servicio de NetworkManager
```
$ systemctl stop NetworkManager
```
2.- Deshabilidad el servicio de NetworkManager
```
$ systemctl disable NetworkManager
```
3.- Accedemos a /etc/network/interfaces y editamos el fichero para configurar nuestros dos adaptadores de forma estática, debería de quedar de una forma similar:

![/img/4.png](/img/4.png)

4.- Reiniciamos el servicio networking:
```
$ systemctl restart networking
```

5.- Si nos hemos tenido ningun problema, con el comando "ip a" veremos el resultado de la configuración:

![/img/5.png](/img/5.png)

<a name="ins"></a>
## 4.- Instalación

El proceso para instalar Nginx es muy sencillo, sigamos los siguientes comandos para ello:

1.- Actualizamos los repositorios:
```
$ apt update
```
2.- Instalamos Nginx
```
$ apt install nginx
```
3.- Comprobamos el estado del servicio Nginx:
```
$ systemctl status nginx
```

<a name="cas"></a>
## 5.- Casos prácticos

### Versión usada de Nginx

Con este comando podemos saber la versión que estamos usado de Nginx:
```
nginx -v
```
Como podemos ver, la versión de Nginx actual es la 1.14.2.
### Servicio asociado

En este apartado vamos a ver como trabajar con el servicio de Nginx:

- Reiniciar Nginx
```
$ systemctl restart nginx
```
- Habilitar Nginx
```
$ systemctl enable nginx
```
- Deshabilitar Nginx
```
$ systemctl disable nginx
```
- Iniciar Nginx
```
$ systemctl start nginx
```
- Parar Nginx
```
$ systemctl stop nginx
```
### Ficheros de configuración
Aquí vamos a ver los ficheros y rutas mas importantes a la hora de configurar Nginx:

- La ruta principal de Nginx es /etc/nginx/
- La ruta de los sitios web esta en /var/www/html/
- El fichero de configuración principal de Nginx se encuentra en /etc/nginx/nginx.conf. En este fichero podremos ajustar cosas como directivas del trafico de red, puertos de escucha o rutas de ficheros.
- En la carpeta sites-available tenemos los sitios web disponibles.
- En la carpeta sites-enabled tenemos los sitios web activados.
- En la carpeta modules-available tenemos todos los modulos disponibles.
- En la carpeta modules-enabled tenemos todos los modulos acivados.

### Editar la página web por defecto
Vamos a modificar la página web que se nos crea por defecto al instalar nginx:

1.- Accedemos al directorio de lo sitios web, veremos que tenemos un documento .html.
```
cd /var/www/html
```
2.- Lo editamos
```
nano index.nginx-debian.html
```
3.- Veremos algo similar: 

![/img/1.png](/img/1.png)

4.- Procedemos a editar el fichero a nuestro gusto, en mi caso solo lo modificaré ligeramente:

![/img/2.png](/img/2.png)

5.- Recargamos la página de Nginx y veremos que se han aplicado los cambios:

![/img/3.png](/img/3.png)

### Virtual Hosting

Vamos a proceder a crear dos sitios web y a acceder a ellos usando nombres. Sigamos los siguientes pasos para ello:

1.- Creamos los directorio dentro de /var/www/html
```
$ sudo mkdir /var/www/html/web1
$ sudo mkdir /var/www/html/web2
```
2.- Le damos los siguientes permisos:
```
$ chown -R www-data:www-data web1
$ chown -R www-data:www-data web2
```
3.- Creamos un archivo index.html en ambas carpetas y escribimos un código similar a este:
```
<!DOCTYPE html>
<html>
<head>
<title>Bienvenidos a Nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Bienvenidos a Nginx!</h1>
<p>Antonio J. Holguin.</p>
<p><em>Esta es la web 1</em></p>
</body>
</html>
```
4.- Vamos a crear nuestros nuevos servidores virtuales, para ello accedemos a /etc/nginx/sites-available y copiamos el fichero "default" con el nombre "web1.conf" y "web2.conf". Acto seguido abrimos el nuevo fichero y lo editamos de la siguiente forma:
```
server {
        listen 80;
        listen [::]:80;

        server_name www.web1.org;

        root /var/www/web1;
        index index.html;

        location / {
                try_files $uri $uri/ =404;
        }

        access_log /var/log/nginx/web1.org-access.log;
        error_log /var/log/nginx/web1.org-error.log;
}
```
5.- Acto seguido, vamos a limitar que rango de IPs pueden acceder a ambos sitios web. Para www.web1.org permitiremos únicamente las redes 192.168.2.0 y 192.168.3.0:
```
server {
        listen 80;
        listen [::]:80;

        server_name www.web1.org;

        root /var/www/web1;
        index index.html;

        location / {
                try_files $uri $uri/ =404;
                allow 192.168.3.0/24;
                allow 192.168.2.0/24;
                deny all;
        }

        access_log /var/log/nginx/web1.org-access.log;
        error_log /var/log/nginx/web1.org-error.log;
}
```
En el caso de www.web2.org, solo le permitiremos que accedan equipos de 192.168.3.0:
```
server {
        listen 80;
        listen [::]:80;

        server_name www.web2;

        root /var/www/web2;
        index index.html;

        location / {
                try_files $uri $uri/ =404;
                allow 192.168.3.0/24;
                deny all;
        }

        access_log /var/log/nginx/web2org-access.log;
        error_log /var/log/nginx/web2org-error.log;
}
```
6.- Copiamos los archivos a sites-enabled:
```
$ sudo ln -s /etc/nginx/sites-available/web1.conf /etc/nginx/sites-enabled/
$ sudo ln -s /etc/nginx/sites-available/web2.conf /etc/nginx/sites-enabled/
```
7.- Reiniciamos el servicio Nginx:
```
$ sudo systemctl restart nginx
```
8.- Si todo ha ido bien, si escribimos www.web1.org o www.web2.org en nuestro buscador nos apareceran nuestras páginas web:

![/img/7.png](/img/7.png)

### Autorización

Una parte de vital importancia de Nginx es la autorización, en esta práctica vamos a crear una carpeta dentro de nuestro sitio web que requerirá a los usuarios loguearse:

1.- Vamos a crear una carpeta llamada "privado" en /var/www/web1/:
```
$ cd /var/www/web1
$ mkdir privado
```
2.- Vamos a la carpeta principal de Nginx para crear el documento que contendrá las credenciales, es decir, los usuarios y sus contraseñas:
```
$ cd /etc/nginx/
```
3.- Para crear el documento debemos de ejecutar el siguiente comando:
```
$ htpasswd -c -m htpasswd user1
```
4.- Si echamos un vistazo al documento, veremos el nombre de usuario acompañado de la contraseña la cual esta encriptada:

![/img/11.png](/img/11.png)

5.- Tras crear el documento, vamos a la carpeta sites-available y editamos el documento web1.conf, de forma que tenga lo siguiente:
```
server {
	listen 80;
	listen [::]:80;
	
	server_name www.web1.org;
	
	location / {
		root /var/www/web1/;
		index index.html;
		try_files $uri $uri/ =404;
	
		access_log /var/log/nginx/web1.org-access.log;
		error_log /var/log/nginx/web1.org-error.log;
	}
	location /privado {
		auth_basic		"Acceso restingido, requiere autorización";
		auth_basic_user_file	/etc/nginx/htpasswd;
	}
}
```
6.- Reiniciamos Nginx:
```
$ systemctl restart nginx
```
7.- Vamos a nuestro navegador e intentamos acceder a la carpeta privado en web1:

![/img/10.png](/img/10.png)

8.- Es posible permitir este metodo de autorización para una red y negar cualquier intento de conexión de otra, para ello lo único que debemos de añadir es lo siguiente en el documento anterior:
```
location /privado {
  satisfy any;
  deny all;
  allow 192.168.3.0/24;
}
```
9.- Reiniciamos Nginx:
```
$ systemctl restart nginx
```
10.- Comprobemos los resultados si entramos por la red 192.168.2.0 y la red 192.168.3.0:

![/img/12.png](/img/12.png)

![/img/12.png](/img/12.png)

### SSL/TLS

Vamos a crear un certificado SSL para poder acceder a nuestro sitio web de forma segura. Tendremos que seguir los siguientes pasos para ello:

1.- Generamos una clave usando la herramienta openssl, la instalamos con apt:
```
$ apt install openssl
```
2.- Creamos una clave con el siguiente comando:
```
$ openssl genrsa -out web1.key 2048
```
3.- A continuación generamos un certificado:
```
$ openssl req -new -key web1.key web1.csr
```
4.- Vamos a firmar el certificado siguiendo los pasos que el comando nos irá indicando:
```
$ openssl x509 -days 365 -in web1.csr -signkey web1.key -out web1.crt
```
5.- Una vez hecho eso, entramos en /etc/nginx/sites-enabled y copiamos el fichero de configuración de nuestro sitio web con un nuevo nombre (web1.conf -> web1-ssl.conf). Acto seguido editamos el nuevo fichero con la siguiente configuración:
```
server {
        listen  443 ssl;

        server_name www.web1.org;
        ssl_certificate         /etc/nginx/claves/web1.crt;
        ssl_certificate_key     /etc/nginx/claves/web1.key;
        ssl_protocols           TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers             HIGH:!aNULL:!MD5;
        ...
}
```
6.- Guardamos la configuración y reniciamos nginx:
```
$ systemctl restart nginx
```
7.- Ahora podremos acceder a nuestro sitio web con https, nuestro navegador nos indicará que el sitio web no es seguro:

![/img/8.png](/img/8.png)

8.- Si continuamos y accedemos al sitio web veremos los siguiente:

![/img/9.png](/img/9.png)

<a name="ref"></a>
## 6.- Referencias
- [www.digitalocean.com](https://www.digitalocean.com/)
- [www.linode.com](https://www.linode.com/docs/guides/how-to-configure-nginx/)
- [www.chachocool.com](https://chachocool.com/como-instalar-nginx-en-debian-10-buster/#Como_configurar_Nginx_en_Debian_10)
