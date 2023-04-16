---
title: Nmap para Ninja Hackers
published: true
---

<center><img src="https://i.ibb.co/DCnfyLP/b43a4374d2bca6f6f57a5.png"></center>

Nmap es una de las herramientas indispensable para cualquier experto en ciberseguidad y administrador de red es una de las herramientas que no debe faltar dentro de tu arsenal de herramientas, acontinuacion aprenderemos como usar algunas opciones de Nmap para escanear puertos de manera sigilosa volando por debajo el radar. 

### Escaneando puertos de manera sigilosa con TCP SYN stealth

Escaneo sigiloso usando el metodo SYN para el escaneo de puertos, La ventaja de un escaneo de tipo SYN es que no establece una conexión completa con el servidor, lo que significa que el escaneo es menos intrusivo y más rápido.

```nmap -Pn -n -sSCV -T4 192.168.0.1```

Con la opcion **-Pn**  le decimos a Nmap que no realize un Ping al Target, con la opcion **-n**
le decimos que no realize un resolucion DNS esto para agilizar el escaneo y ahorrarnos tiempo, luego esta la opcion **-sSCV** la cual es una combinacion de varias opciones:

**-sS** :  Con esta opcion le estamos diciendo a Nmap que realize un escaneo usando el metodo SYN.

**-sV** : Para obtener version de los puertos abiertos.

**-sC** : Para correr Scripts NSE en los puertos abiertos y asi obtener la mayor informacion posible del servicio abierto.

Y como ultima opcion tenemos la opcion **-T4**  que es un temporizador agresivo para acelerar el escaneo.

### Escaneo de puertos TCP utilizando un puerto aleatorio de origen

```nmap -sS -T4 -A -v -p 1-65535 -g 53 <IP del objetivo>```

Este comando utiliza un puerto aleatorio de origen para cada paquete enviado, lo que dificulta que el objetivo lo detecte. También se utiliza el puerto 53 (DNS) como puerto de salida para engañar a algunos firewalls.

### Escaneo de puertos TCP y UDP en modo sigiloso con detección de sistemas operativos

```nmap -sS -sU -T4 -A -v <IP del objetivo>```

Este comando utiliza tanto el escaneo TCP SYN stealth como el escaneo UDP para explorar los puertos abiertos. Además, muestra información detallada sobre el sistema operativo del objetivo.

### EVASIÓN DE CORTAFUEGOS/IDS Y FALSIFICACIÓN

Estas opciones solo son aplicables dentro de una red local.

**Usando señuelos para nuestros escaneos**

```nmap -D <IP señuelo 1>, <IP señuelo 2>, <IP señuelo 3> -Pn -n -sCV -T4 <TARGET>```

**Ejemplo de uso:**

```sudo nmap -Pn -n -sCV -T4 -D 192.168.0.9, 192.168.0.1, 192.168.0.2 192.168.0.12```

Realiza un sondeo con señuelos. Esto hace creer que el/los equipo/s que utilice como señuelos están también haciendo un sondeo de la red. De esta manera sus IDS pueden
llegar a informar de que se están realizando de 5 a 10 sondeos de puertos desde distintas direcciones IP, pero no sabrán qué dirección IP está realizando el análisis y cuáles son señuelos inocentes. 

Tenga en cuenta que los equipos que utilice como distracción deberían estar conectados o puede que accidentalmente causes un ataque de inundación SYN a sus objetivos. Además, sería bastante sencillo determinar qué equipo está realmente haciendo el sondeo si sólo uno está disponible en la red.

**Falsificacion de  la dirección MAC**

Esta opcion nos permite falsificar nuestra direccion MAC y podemos usar esta opcion de diferentes formas por ejemplo...

```nmap --spoof-mac -Pn -n -sCV -T4 <TARGET>```

En el comando escrito solo hemos puesto la opcion **--spoof-mac** con esta opcion Nmap nos dara una direccion MAC de forma aleatoria.

Pero si lo queremos es sustituir nuestra direccion MAC por la de un dispositivo en especial podemos hacerlo dandole los 3 primeros bytes de la direccion

```nmap -Pn -n -sCV -T4 --spoof-mac 00:05:c9 192.168.0.1```

Puedes conseguir los prefijos de 3 bytes usando la herramienta **Macchanger** con la opcion **-l** y con **grep** puedes escribir el dispositivo que deseas, por ejemplo...

```macchanger -l | grep "Sansumg" ```

**Salida:**

```
0633 - 00:02:78 - Samsung Electro-Mechanics Co., Ltd.
1964 - 00:07:ab - Samsung Electronics Co.,Ltd
3530 - 00:0d:e5 - Samsung Thales
...
```

**Un Oneliner para conseguir solamente una sola direccion...**

```macchanger -l | grep "Samsung" | head -n 1 | cut -d " " -f 3```



## Algunas otras opciones para volar bajo el radar

**Evasión de Firewall mediante fragmentación de paquetes:**

```nmap -f <objetivo>```

Este comando fragmenta los paquetes de Nmap para evitar ser detectado por algunos firewalls.

Cuando un escaneo de Nmap se fragmenta, el paquete de escaneo se divide en varios fragmentos más pequeños y se envían de manera individual a través de la red. El firewall receptor del tráfico no puede detectar la intención del escaneo, ya que sólo ve fragmentos de paquetes aparentemente no relacionados.

**Falsificación de origen IP:**

```nmap -S <IP Falsa> <objetivo>```

Este comando permite falsificar la dirección IP de origen de los paquetes de Nmap.

**Falsificación de origen de puerto:**

```nmap -g <Puerto Falso> <objetivo>```

Este comando permite falsificar el número de puerto de origen de los paquetes de Nmap.

**Escaneo de puertos TCP mediante proxy:**

```nmap -sT <objetivo> --proxies <servidor proxy>:<puerto>```

Este comando permite utilizar un servidor proxy para escanear los puertos TCP del objetivo

**Escaneo de puertos UDP mediante falsificación de paquetes ICMP:**

```nmap -sU <objetivo> -P0 --data-length 0 --fuzzy```

Este comando utiliza la falsificación de paquetes ICMP para escanear los puertos UDP del objetivo.





