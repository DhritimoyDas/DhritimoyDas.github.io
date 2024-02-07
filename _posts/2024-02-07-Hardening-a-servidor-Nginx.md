
En esta sección vamos a dedicarnos a securizar nuestro servidor para que algún Haxxor no logre piratearnos, vamos a comenzar tocando el fichero de configuración de Nginx 
```/etc/nginx/nginx.conf``` y vamos a cambiar el nombre de nuestro servidor en la cabecera 

Vamos a empezar con los 3 principios importantes dentro del Hardening de sistemas:

- **Mínimo punto de Exposición:** Mante instalado solo el Software necesario y trata de no brindar mucha información de tu servidor.

- **Mínimo privilegio posible:** Nuestro servidor no debe nunca correr como Root, esto hará que ante un compromiso de nuestro sistema el atacante podrá fácilmente pivotear o instalar software para moverse sobre nuestra infraestructura.

- **Defensa en profundidad:** Se debe implementar todas la medidas de seguridad necesarias, teniendo en cuanto 2 factores:

	1) Una medida de seguridad no debe anular a otra
	2) Las medida de protección no deben dañar la disponibilidad del sistema o causar problemas en ciertos servicios.


```
HTTP/1.1 200 OK
Date: Tue, 05 Dec 2023 19:21:13 GMT
Content-Type: text/html; charset=UTF-8
Connection: keep-alive
Server: Nginx
```

Vamos realizar la instalación de *nginx-extras*

```bash
sudo apt install -y nginx-extras
```

Una vez instalado proseguimos a editar el fichero de configuración nginx.conf he ingresamos las siguientes líneas


```bash
# /etc/nginx/nginx.conf 
http { 
	# Basic Settings 
	more_set_headers 'Server: Server Onion';
...
```


## Mitigar ataques de DDoS / DoS 

Uno de los ataques mas frecuentes dentro de la red Tor son los ataques DDos y por lo tanto tenemos que estar preparados para que nuestro servidor reciba una buena cantidad de consultas.

Vamos a añadir las siguientes configuraciones...

### Limitar la tasa de solicitudes

Utiliza la directiva `limit_req` para limitar el número de solicitudes por segundo desde una IP específica. Esto ayuda a mitigar ataques DDoS al limitar la cantidad de solicitudes que un atacante puede realizar desde una sola dirección IP.

Ejemplo de configuración en el bloque `http` de Nginx:

```php
limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;

server {
    # Otras configuraciones
    location / {
        limit_req zone=one burst=5;
        # Resto de configuración
    }
}
```

Esto limitará las solicitudes a 1 por segundo por dirección IP, con una capacidad de ráfaga de 5 solicitudes.
### Limitar conexiones simultáneas

Utiliza la directiva `limit_conn` para limitar el número de conexiones simultáneas desde una dirección IP específica.

```php
limit_conn_zone $binary_remote_addr zone=addr:10m;

server {
    # Otras configuraciones
    location / {
        limit_conn addr 10;
        # Resto de configuración
    }
}
```

Esto limitará a 10 conexiones simultáneas desde una dirección IP.

### Contrarrestando ataques de DoS 

```bash
##buffer policy 
client_body_buffer_size 1K;
client_header_buffer_size 1k;
client_max_body_size 1k;
large_client_header_buffers 2 1k; 
##end buffer policy
```

## Deshabilitando métodos inseguros

Los métodos habituales en un servidor son: GET, POST, HEAD si tu Server tiene métodos como TRACE, PUT, DELETE entre otros, tu servidor puede ser vulnerable a ciertos ataques como ser cross-site tracking.

Vamos a añadir la siguiente linea en nuestro fichero **nginx.conf** en la cual le estamos diciendo a Nginx que devuelva un error 405 a ante la consulta a esos métodos. 

```bash
	if ($request_method !~ ^(GET|HEAD|POST)$ ) {
		return 405;
	}
```

## Restringir acceso por contraseña

Si deseamos que usuarios ingresen a un sector de nuestra pagina usando contraseña, podemos hacerlo de la siguiente manera...

```bash
location /wp-admin {
    auth_basic "Admin Area";
    auth_basic_user_file /path/to/.htpasswd;
}
```

Las contraseñas se almacenaran en **/



## Consejos finales

- Mantener monitoreado los logs: **/etc/logs/errors.log** para ver errores del servidor, **/etc/log/access.log**  para ver datos de los usuarios.

- Mantener actualizado Nginx y tu sistema en general

**Recursos:**
https://medium.com/guayoyo/hardening-asegurando-nginx-e295f4850562


