<img title="Tor for the freedom" src="tor-browser.jpg">

## Instalación de Tor 

```bash
sudo apt update 
sudo apt install -y tor
```

## Editando el fichero de configuración para levantar el Hidden Service

Vamos a editar el fichero ```/etc/tor/torrc``` 

y vamos a añadir o a descomentar las lineas...


```bash
HiddenServiceDir /var/lib/tor/hidden_service
HiddenServicePort 80 127.0.0.1:80 <----- Nuestro servidor local
				  /\
Puerto a la escucha en el sitio ONION 	
```

Para poder usar SSH desde Tor podemos también descomentar

```bash
HiddenServicePort 22 127.0.0.1:22
```

Lo mas recomendable seria cambiar el numero de puerto expuesto en la red Tor 
Kali seria de esta forma...

```bash
HiddenServicePort 2222 127.0.0.1:22
```

## Creación de nuestra URL personalizada

### Usando oniongen-go

Vamos a generar nuestra URL personalizada usando para ello un pequeño proyecto escrito en Go 
en su uso e instalación no hay mucha ciencia, por lo tanto al grano...

##### Clonación de repositorio

```bash 
git clone https://github.com/rdkr/oniongen-go
``` 

Nos dirigimos hacia nuestra carpeta oniongen-go y tenemos 2 opciones ejecutarlo rápidamente usando el siguiente comando...

```bash
go run main.go "^test" 1
```

o compilar y correr

```bash
go build
oniongen-go "prefijo" <numeros de urls generadas>
```

En mi caso oniongen-go me dio algunos problemas, pero tenemos otra herramienta llamada mkp224o, la cual esta escrita en C, lo que significa que debemos compilarlo para usarlo.

### Usando mkp224o

Con esta herramienta vamos a poder generar nuestra url ONION personalizable, esta herramienta esta programada en C y Assembler lo que lo hace una herramienta veloz a la hora de generar nuestra URL.

#### Instalación de las librerías necesarias

Antes de realizar la compilación vamos a instalar las librerías necesarias para poder compilar con éxito la herramienta.

```bash
apt install gcc libc6-dev libsodium-dev make autoconf
```

#### Clonación del repositorio 

```bash
git clone https://github.com/cathugger/mkp224o
```

#### Compilación

Para generar nuestro Script de configuración

```bash
./autogen.sh
```

Creación de nuestro makefile útil para la compilación de nuestra aplicación 

```bash
./configure
```

y procedemos a compilar...

```bash
make
```


Y listo tenemos nuestro programa generado **mkp224o** 

#### Uso: 

```bash
mkp224o -n <numeros de urls generadas> <prefijo a usar>
```

**Ejemplo:**

```bash
mkp224o -n 1 test 

test67pqshefizpib47jwnfrh57esinxhvq74ghv3v2luno2sjg4jxid.onion
```

Se nos generar un directorio con las claves privadas de la url lo exportas hacia la carpeta
 **/var/lib/tor/hidden_service** reseteamos tor y listo!!.

## Notas finales

 - **Si deseas añadir mas seguridad a tu servidor puedes seguir nuestra guía de Hardening.**
 - **Ten en cuenta que no solo puedes montar un servidor Web, también puedes montar un servidor SSH o FTP.**
 - **Los mayores problemas para un servidor Web ONION son los constantes ataques DDoS, por lo tanto establece algunas medidas de seguridad en tu servidor.**
