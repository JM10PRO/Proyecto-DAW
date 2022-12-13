# Práctica de Servidores Web 1º trimestre 2ºDAW
#### Proyecto del módulo Despliegue de Aplicaciones Web

## Apache
###  · Instalación de Apache:
```bash
sudo apt update
sudo apt install apache2
```
### · Activación de los módulos necesarios para ejecutar PHP y acceder a MySql
### Instalación de MySql:
```bash
sudo apt install mysql-server
```
### Instalación de PHP:
```bash
sudo apt install php libapache2-mod-php php-mysql
```
### Comprobamos la instalación:
![apachestatus](img/apachestatus.png)
### · Instalando y cofigurando WordPress:
```bash
sudo mkdir -p /srv/www
sudo chown www-data: /srv/www
sudo apt install curl
curl https://wordpress.org/latest.tar.gz | sudo -u www-data tar zx -C /srv/www
```
### Creamos un archivo WordPress.conf
```bash
sudo nano /etc/apache2/sites-available/wordPress.conf
```
![wordpress3](img/wordpress3.png)

### Añadimos estas líneas al fichero:
![wordpress4](img/wordpress4.png)

### Habilitamos el sitio con:
```bash
sudo a2ensite wordpress
```
### Habilitamos la reescritura de URL con:
```bash
sudo a2enmod rewrite
```
### Desactivamos el sitio predeterminado _"It Works"_ para agregar un nombre de host para que WordPress responda a las solicitudes:
```bash
sudo a2dissite 000-default
```
### En el fichero _wordpress.conf_ añadimos el host donde servirá WordPress (centro.intranet):
```bash
<VirtualHost *:80>
    ServerName centro.intranet
    ServerAlias www.centro.intranet
    ServerAdmin webmaster@localhost
    ... # el resto de la configuración de VHost
</VirtualHost>
```
### Recargamos apache2:
```bash
sudo service apache2 reload
```
## Configurando la base de datos
### Para configurar WordPress, necesitamos crear una base de datos MySQL.
```bash
sudo mysql -u root
```
![mysql](img/mysql.png)
![mysql2](img/mysql2.png)
![mysql3](img/mysql3.png)

### Habilitamos MySQL:
```bash
sudo service mysql start
```
### Configuramos la conexión a la base de datos:
```bash
sudo -u www-data cp /srv/www/wordpress/wp-config-sample.php /srv/www/wordpress/wp-config.php
sudo -u www-data sed -i 's/database_name_here/wordpress/' /srv/www/wordpress/wp-config.php
sudo -u www-data sed -i 's/username_here/wordpress/' /srv/www/wordpress/wp-config.php
sudo -u www-data sed -i 's/password_here/<your-password>/' /srv/www/wordpress/wp-config.php
```
### Abrimos en la terminal el siguente fichero y terminamos de configurarlo:
```bash
sudo -u www-data nano /srv/www/wordpress/wp-config.php
```
```bash
define( 'AUTH_KEY',         'put your unique phrase here' );
define( 'SECURE_AUTH_KEY',  'put your unique phrase here' );
define( 'LOGGED_IN_KEY',    'put your unique phrase here' );
define( 'NONCE_KEY',        'put your unique phrase here' );
define( 'AUTH_SALT',        'put your unique phrase here' );
define( 'SECURE_AUTH_SALT', 'put your unique phrase here' );
define( 'LOGGED_IN_SALT',   'put your unique phrase here' );
define( 'NONCE_SALT',       'put your unique phrase here' );
```
#### Abrimos http://localhost/ en el navegador. Nos pedirá el título del nuevo sitio, nombre de usuario, contraseña y dirección de correo electrónico. Esta información es solo para WordPress y no proporcionan acceso a ninguna otra parte del servidor. Elije un nombre de usuario y una contraseña que sean diferentes a las credenciales de MySQL. 
#### Ahora podemos iniciar sesión en http://localhost/wp-login.php. En el panel de WordPress, aparecerán un montón de iconos y opciones. En la opción de agregar post podemos escribir nuestro primer post, seguimos las opciones y ya estaría listo nuestro.

# Ahora vamos a configurar un servidor con Python y el módulo WSGI de Apache.

## · Activamos el módulo "wsgi" para permitir la ejecución de aplicaciones Python.
```bash
apt install libapache2-mod-wsgi-py3 -y
```
![pythonwsgi](img/pythonwsgi.png)
### Comprobamos que está activado
![wsgipython](img/wsgipython.png)

## · Creamos y desplegamos una pequeña aplicación python para comprobar que funciona correctamente.
### Escribimos en un script de Python lo siguiente:
```bash
sudo nano /var/www/html/app.py
```
```bash
def application(environ, start_response):
    status = '200 OK'
    output = b'APP CREADA CON PYTHON QUE SE EJECUTA DESDE WSGI EN APACHE\n'
    response_headers = [('Content-type', 'text/plain'),
                        ('Content-Length', str(len(output)))]
    start_response(status, response_headers)
    return [output] 
```
### Cambiamos los permisos con el siguiente comando:sudo
```bash
sudo chown www-data:www-data /var/www/html/app.py
sudo chmod 775 /var/www/html/app.py
```
### Luego añadimos estas líneas de código al fichero 000-default.conf:
![wsgi-conf](img/wsgi-conf.png)

### Ahora en el navegador, si escribimos http://departamentos.centro.intranet/app nos aparecerá el mensaje de nuestro script de Python desde el servidor.
![pythonapp3](img/pythonapp3.png)

## · Adicionalmente protegeremos el acceso a la aplicación python mediante autenticación.
### Creamos el archivo necesario con un usuario y en la siguiente línea indicamos la contraseña.
```bash
sudo htpasswd -c /etc/apache2/.htpasswd userjm
```
![htpasswd](img/htpasswd.png)

```bash
sudo nano /etc/apache2/sites-available/000-default.conf
```
Escribimos en el fichero lo siguiente:
```bash
<Directory /var/www/html/app.py>
        AuthType Basic
        AuthName "Autenticacion Obligatorio"
        AuthUserFile /etc/apache2/.htpasswd
        Require valid-user
</Directory>
```
### Ahora nuestra aplicación estará restringida para los usuarios que hemos incluido en el archivo _htpasswd_
![htpasswdexample](img/htpasswdexample.png)

## · Instala y configura awstat.

Instalamos awstat con el siguiente comando:

```bash
  sudo apt install awstats
```

![awstats1](img/awstats1.png)

Después habilitamos el módulo CGI y reiniciamos apache2:

```bash
sudo a2enmod cgi
systemctl restart apache2
```

![awstats2](img/awstats2.png)

Creamos un archivo de configuracion para el dominio "centro.intranet" en nuestro caso con el siguiente comando:

```bash
  sudo cp /etc/awstats/awstats.conf /etc/awstats/awstats.centro.intranet.conf
```

Entramos al archivo de configuración y escribimos lo siguiente:

```bash
  sudo nano /etc/awstats/awstats.centro.intranet.conf
``` 

```bash
  SiteDomain="centro.intranet"
  HostAliases="centro.intranet localhost 127.0.0.1"
```

![awstats3](img/awstats3.png)

Por último, accedemos desde el siguiente link en un navegador "http://centro.intranet/cgi-bin/awstats.pl?config=centro.intranet" y nos mostrará esta web donde se estará ejecutando el AWSTATS.

![awstats4](img/awstats4.png)

## · Instala un segundo servidor de tu elección (nginx, lighttpd) bajo el dominio “servidor2.centro.intranet”. Debes configurarlo para que sirva en el puerto 8080 y haz los cambios necesarios para ejecutar php. Instala phpmyadmin.

### Instalamos NGINX con el siguiente comando:
```bash
  sudo apt install nginx
```

![nginx1](img/nginx1.png)

### Aplicamos ajustes al firewall, enumeramos las configuraciones con: 
```bash
    sudo ufw app list
```
![nginx1-1](img/nginx1-1.png)

### Habilitamos escribiendo lo siguiente:
```bash
    sudo ufw allow 'Nginx HTTP'
```

### Instalamos el paquete PHP:

```bash
  apt install php-fpm
```

![nginx2](img/nginx2.png)

### Añadimos el nuevo dominio en /etc/hosts:

![nginx3](img/nginx3.png)

### Instalamos y descomprimimos el paquete _phpmyadmin_:

```bash
  sudo wget https://files.phpmyadmin.net/phpMyAdmin/5.2.0/phpMyAdmin-5.2.0-all-languages.tar.gz
  tar -xf phpMyAdmin-5.2.0-all-languages.tar.gz
```
  
### Movemos el directorio de phpMyAdmin a la carpeta donde tendremos phpmyadmin **/var/www/phpmyadmin**

```bash
  sudo mv phpMyAdmin-5.2.0-all-languages /var/www/phpmyadmin
```

### A continuación, movemos la plantilla del archivo de configuración de PHPMYADMIN y le cambiamos el nombre.

```bash
  sudo cp /var/www/phpmyadmin/config.sample.inc.php /var/www/phpmyadmin/config.inc.php
```

### Generamos el "blowfish_secret" con este comando

```bash
  sudo apt install pwgen
  pwgen -s 32 1
```

### Entramos al archivo de configuración con el siguiente comando:

```bash
  sudo nano /var/www/phpmyadmin/config.inc.php
```
### Realizamos los siguientes cambios:

```bash
  $cfg['blowfish_secret'] = 'codigo generado automáticamente anteriormente';
  $cfg['Servers'][$i]['controlhost'] = 'localhost';
  $cfg['Servers'][$i]['controlport'] = '';
  $cfg['Servers'][$i]['controluser'] = 'josemaria';
  $cfg['Servers'][$i]['controlpass'] = 'password1234';
```

![bdphpmyadmin](img/bdphpmyadmin.png)

### Importamos la base de datos que ya viene incluida en phpMyAdmin por defecto:

```bash
  sudo mysql < /var/www/phpmyadmin/sql/create_tables.sql -u root -p
```

### Nos damos permisos a nuestro usuario a la base de datos de phpmyadmin:

![phpmyadmin6](img/phpmyadmin6.png)

### Configuramos el servidor para poder acceder a phpmyadmin con el siguiente comando:

```bash
  sudo nano /etc/nginx/sites-available/phpmyadmin.conf
```

### Y añadimos lo siguiente:

![image](https://user-images.githubusercontent.com/91668406/204386019-f02a2eb6-a4e3-48d0-9ff4-ca59c702528b.png)

### Para finalizar cambiamos los permisos de phpmyadmin con el siguiente comando y reiniciamos los servicios.

```bash
  sudo chown -R www-data:www-data /var/www/phpmyadmin
  sudo systemctl restart nginx php8.1-fpm  
```

### Si accedemos con "servidor2.centro.intranet:8080 nos aparecerá la siguiente ventana:

![image](https://user-images.githubusercontent.com/91668406/204386376-5e1a3f4e-1ca8-4a5d-b7c9-21516b843f7c.png)

### Y si ponemos las credenciales entraremos dentro del phpmyadmin:

![image](https://user-images.githubusercontent.com/91668406/204379493-76d4ee13-37a7-409a-b635-23417832d49d.png)
