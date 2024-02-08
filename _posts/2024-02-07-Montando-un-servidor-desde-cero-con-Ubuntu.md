
Que tal Haxxors, hace algún tiempo quise montar un servidor ONION casero y mantenerlo en funcionamiento durante algunos meses, pero por falta de tiempo y ganas nunca pude configurarlo y correrlo.

Lo quise montar todo desde cero y configurarlo lo mas seguro posible aplicando Hardening sobre mi Server, después de todo seria muy peligroso exponer mi red en una Darknet muy hostil como lo es la red Tor.

Antes de instalar Tor y configurar nuestro Hidden Service, vamos instalar y configurar ciertas aplicaciones las cuales nos serviran para montar nuestra pagina Web totalmente funcional.

### Instalación de herramientas básicas para el servidor

**Para usar el comando ifconfig y otros comando para administración de red vamos a instalar net-tools:

```bash
sudo apt install net-tools
```

### Instalación de Nginx

Vamos hacer uso Nginx como servidor Web ya que es el mas poderoso actualmente, a diferencia de Apache, Nginx no consume muchos recursos y tampoco posee vulnerabilidades  de consideracion o criticas.

```bash
sudo apt update && sudo apt install nginx
```

### Usando ufw como Firewall:

Configurando el firewall para que acepte conexiones hacia SSH (22) y Nginx (80), todo esto usando ufw, el cual es un Firewall para personas las cuales no le gusat complicarse la vida con Iptable, facil de usar y directo al grano.

```bash
sudo ufw enable 
sudo ufw allow ssh 
sudo ufw allow 'Nginx HTTP' 
sudo ufw status
```

Verificamos que nginx esta corriendo en nuestro sistema

```bash
sudo systemctl status nginx
```

Si todo va bien comenzamos con la creación del directorio de nuestro sitio, sin permisos de *root* para prevenir que un atacante tenga todos los permisos al piratear nuestro Server.

**Creación de directorio para nuestra Web

```bash 
sudo mkdir -p /var/www/sitio/html
```

*La opción -p crea directorios y subdirectorios de forma automática*

**Asignando permisos al directorio que hemos creado

```bash
sudo chmod -R 755 /www/sitio/html
```

**Creando nuestro fichero index.html

```bash
vi /var/www/sitio/html/index.html
```

### Configurando del bloque del servidor

Vamos a indicarle a Nginx una nueva ruta para nuestra Web.

```bash
sudo vi /etc/nginx/sites-available/sitio
```

Y añadimos la siguiente configuracion...

```bash
server { 
	listen 80; 
	listen [::]:80; 
	root /var/www/hektorprofe.tk/html; 
	index index.html index.htm; server_name hektorprofe.tk www.hektorprofe.tk; 
	
	location / { 
		try_files $uri $uri/ =404; 
		} 
	}
```


Es hora de crear un enlace de nuestra configuración en la ruta */etc/nginx/sites-enabled

```bash
sudo ln -s /etc/nginx/sites-available/sitio /etc/nginx/sites-enabled/
```


**Verificando si no hay errores en nuestra configuración

```bash
sudo nginx -t
```

Reiniciamos el servidor

**Opción #1

```bash
sudo systemctl restart nginx
```

**Opción #2

```bash
sudo service nginx restart
```


### Ficheros de configuración y rutas para configurar:

- **/var/www/html**: Ruta donde se almacena el contenido por defecto que muestra Nginx (su default), puede ser cambiado en el fichero de configuración principal.

- **/etc/nginx/nginx.conf**: Ruta del fichero de configuración principal.

- **/etc/nginx/sites-available/**: Directorio que contiene las configuraciones de los bloques de servidor disponibles pero no activados

- **/etc/nginx/sites-enabled/**: Directorio que contiene las configuraciones de los bloques de servidor activados, normalmente enlazado desde el directorios de bloques de servidor disponibles

- **/etc/nginx/snippets**: Directorio que contiene fragmentos configurables incluidos globalmente en la configuración de Nginx.

- **/var/log/nginx/access.log**: Fichero que almacena un registro de las peticiones recibidas.

- **/var/log/nginx/error.log**: Fichero que almacena un registro de los posibles errores.


### Instalación de Servidor FTP

Ya tenemos nuestro servidor Web corriendo en nuestro sistema pro nos falta una manera de subir los ficheros de nuestro sitio Web hacia el directorio /var/www/html/.

**Instalación de vsftpd

```bash
sudo apt install -y vsftpd
```

**Verificamos si vsftpd esta corriendo en nuestro sistema

```bash
sudo systemctl status vsftpd
```


**Configurando el fichero vsftpd.conf

Deshabilitando el usuario anonymous

```bash
anonymous_enable=No
```

Restringir que el usuario pueda ver datos de otros usuarios del sistema

```bash
chroot_local_user=YES
```


Indicarle el usuario y la ruta en donde el usuario podrá subir sus ficheros

```bash
user_sub_token=$USER
local_root=/var/www/html
```

**Darle acceso a los usuarios del sistema:

```bash
userlist_enable=YES
userlist_fle=/etc/vsftpd.userlist
userlist_deny=NO
```

**Configurando el Firewall

```bash
sudo ufw allow 21
```

**Añadiendo usuarios a la lista de usuarios 

```bash
echo $USER ! sudo tee -a /etc/vsftpd.userlist
```


Recuerda mantener el servidor actualizado

```bash
sudo apt update && sudo apt upgrade
```

Por si deseas desinstalar el servidor FTP

```bash
sudo apt autoremove --purge vsftpd*
```

### Instalación de PHP

Hasta ahora nuestro server por Default solo interpreta ficheros HTML, para realizar pagina dinámicas vamos a instalar PHP y poder programar del lado del Servidor.

**Instalando PHP**

```bash
sudo apt install -y php-fpm php-mysql
```

**Configurando Nginx para la ejecución de nuestros Script PHP nos dirigimos hacia /etc/nginx/sites-available/prueba

Añadimos estas líneas...

```php
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
     }

    location ~ /\.ht {
        deny all;
    }
```


### Instalación de Mysql

Ahora que tenemos PHP instalado vamos a necesitar una base de datos donde guardar datos de nuestros usuarios.

```bash
sudo apt install mysql-server
```

**Creación de usuario para mysql:

Creando usuario y contraseña

```mysql
CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'password';
```

Añadiendo permisos

```mysql
GRANT ALL ON example_database.* TO 'example_user'@'%';
```


#### Actualizando contraseña de usuario

**Verificando la contraseña si es necesario

```mysql
SELECT User, Host, authentication_string FROM mysql.user WHERE User='root';
```

**Cambiando la contraseña**

```mysql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'nueva_contraseña';
```

Recuerda que la contraseña debe ser una contraseña alphanumerica (letras y numeros) de 10 o mas digitos. Por ejemplo: ```haxxor_2024_admin_server```

**Actualizar y salir**

```mysql
FLUSH PRIVILEGES;
EXIT;
```

Y Listo ya puedes crear tus bases de datos

```mysql
CREATE DATABASE wordpress;
```

Listo!!, ya tenemos nuestro servidor LEMP (Linux, Nginx, Mysql y PHP) (se pronuncia “Engine-X” de hay sale la "E" :) ), Pero hace falta algo de seguridad, seguridad dura y agresiva al puro estilo Haxxor, en el siguiente Topic se tocara algo de Hardening para cada servicio de nuestro servidor.

Happy Hacking Haxxors!!!
