# Proyecto DAW
#### Proyecto del módulo Despliegue de Aplicaciones Web

## Instalación de Apache:
```bash
sudo apt update
sudo apt install apache2
```
## Activación de los módulos necesarios para ejecutar PHP y acceder a MySql
### Instalación de MySql:
```bash
sudo apt install mysql-server
```
### Instalación de PHP:
```bash
sudo apt install php libapache2-mod-php php-mysql
```
## Comprobamos la instalación:
![apachestatus](img/apachestatus.png)
## Instalando y cofigurando WordPress:
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

## Ahora vamos a configurar un servidor con Python y Django.

### Activamos el módulo "wsgi" para permitir la ejecución de aplicaciones Python.
```bash
apt install libapache2-mod-wsgi-py3 -y
```
![pythonwsgi](img/pythonwsgi.png)

### Creamos y desplegamos una pequeña aplicación python para comprobar que funciona correctamente.

