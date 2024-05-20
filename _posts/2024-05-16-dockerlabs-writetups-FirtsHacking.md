---
layout: single
title: FirstHacking - DockerLabs
excerpt: "Maquina Linux ideal para realizar tu primer CTF"
date: 2024-05-16
classes: wide
header:
  teaser: https://pentest4noob.github.io/Writetups/assets/images/logos/logo_dockerlabs.png
  teaser_home_page: true
categories:
  - DockerLabs
tags:
  - linux
  - Muy facil
---

<p align="center">
  <img width="250" height="250" src="https://pentest4noob.github.io/Writetups/assets/images/logos/logo_dockerlabs.png">
</p>

Esta maquina está clasificada como "Muy Fácil" en la plataforma de [DockerLabs](https://dockerlabs.es/#/), es una maquina que está pensada para todos aquellos que están iniciando en el pentesting y puedan Hackear realizar su primer CTF.

## Herramientas

    nmap
    metasploit

# 1- Enumeración

Comenzamos con la etapa de enumeración para ello vamos a usar la herramienta nmap con los siguientes parámetros

## Nmap

```css
sudo nmap -p- --open -sS -sC -sV -n -Pn 172.17.0.2 -oN nmap.txt
```

Finalizado el escaneo podemos ver que se encuentran un puerto abierto el 21 correspondiente al servicio FTP

```css
tarting Nmap 7.94SVN ( https://nmap.org ) at 2024-05-07 15:52 -03
Nmap scan report for 172.17.0.2
Host is up (0.0000080s latency).
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.3.4
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Unix

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 3.50 seconds
```

analizando mas a profundidad vemos que la versión del servicio FTP es 2.3.4, un poco antigua por lo tanto voy a investigar si existe alguna vulnerabilidad para esta versión

```css
--> 21 FTP vsftpd 2.3.4
```

encontré el CVE-2011-2523 el cual indica que hay un backdoor en el puerto 6200/TCP el cual nos devuelve una shell interactiva

# 3- Exploit

Está vulnerabilidad se puede explotar de varias maneras mediante, scripts en python, con la herramienta metaplsoit y de forma manual, voy a proceder a explicar las 2 ultimas

## Metasploit

Procede a ejecutar metasploit, buscar el servicio vsftpd y seleccionar para ejecuatarlo

```bash
msfconsole
search vsftpd
use 1
```

![000myfirtshacknig.png](https://pentest4noob.github.io/Writetups/assets/images/writetup/dockerlabs/FirstHacking/000firtshacking.png)

ejecutar el comando `show options` para mostrar las opciones que debemos configurar para correr el exploit

```metasploit
show options
```

En esta ocasión solo se deberia configurar el host remoto al cual deseamos atacar el cual ya sabemos es `172.17.0.2`

![001myfirtshacking.png](https://pentest4noob.github.io/Writetups/assets/images/writetup/dockerlabs/FirstHacking/00firtshacking.png)

Configurando opcion rhosts

```metasploit
set rhosts 172.17.0.2
```

Ejecutando el comando run para iniciar el exploit

```metasploit
run
```

Con esto ya tenemos acceso a la maquina y nos devuelve el usuario root por lo tanto no hay que escalar privilegios.

![002myfirtshacking.png](https://pentest4noob.github.io/Writetups/assets/images/writetup/dockerlabs/FirstHacking/002firtshacking.png)

### Explicación del exploit

EL exploit realiza un login con un usuario con caracteres especiales ejemplo user:) esto provoca un desbordamiento de buffer, lo cual aprovecha para crear otra conexion por otro puerto 6200 y permitir la conexion como root

# Explotación manual

## Herramienta

    Netcat

Primero voy hacer la conexión al puerto 21 a través de netcat, la sintaxis es la siguiente `user usuario:)` en usuario puedes ingresar cualquiera lo importante es agregar un carácter especial en este caso dos puntos y paréntesis `:)` luego ingresamos `pass` el cual también vamos a ingresar cualquiera presionar enter y en este momento la consola no va a mostrar nada mas

![003myfirtshacking.png](https://pentest4noob.github.io/Writetups/assets/images/writetup/dockerlabs/FirstHacking/003firtshacking.png)

Por lo tanto procedo abrir otra consola con el comando netcat pero esta vez al puerto 6200 le coloco el parámetro -v para que nos muestre en la consola la conexión, se observa que se estableció una conexión y está abierta, ingreso el comando whoami y soy el usuario root por lo tanto pude ingresar al equipo con el mayor privilegio.

![004myfirtshacking.png](https://pentest4noob.github.io/Writetups/assets/images/writetup/dockerlabs/FirstHacking/004firtshacking.png)
