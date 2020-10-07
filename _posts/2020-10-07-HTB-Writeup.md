---
title: "Hack The Box - Writeup"
tags: [linux,easy,cms made simple, path hijack]
categories: HackTheBox-Linux
---

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Writeup%20Box%20c8abaf66b7694b32be2810ee9ed1af0b/Untitled.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Writeup%20Box%20c8abaf66b7694b32be2810ee9ed1af0b/Untitled.png)

## NMAP

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Writeup%20Box%20c8abaf66b7694b32be2810ee9ed1af0b/Untitled%201.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Writeup%20Box%20c8abaf66b7694b32be2810ee9ed1af0b/Untitled%201.png)

### Enumerating Web

⇒ On homepage we have the following. There is Eeyore Dos protection which restricts us from dirbusting as we will get ip banned :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Writeup%20Box%20c8abaf66b7694b32be2810ee9ed1af0b/Untitled%202.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Writeup%20Box%20c8abaf66b7694b32be2810ee9ed1af0b/Untitled%202.png)

⇒ We found an directory name in robots.txt . Which is running CMS Made Simple :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Writeup%20Box%20c8abaf66b7694b32be2810ee9ed1af0b/Untitled%203.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Writeup%20Box%20c8abaf66b7694b32be2810ee9ed1af0b/Untitled%203.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Writeup%20Box%20c8abaf66b7694b32be2810ee9ed1af0b/Untitled%204.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Writeup%20Box%20c8abaf66b7694b32be2810ee9ed1af0b/Untitled%204.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Writeup%20Box%20c8abaf66b7694b32be2810ee9ed1af0b/Untitled%205.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Writeup%20Box%20c8abaf66b7694b32be2810ee9ed1af0b/Untitled%205.png)

→ We enumerated the pages and couldnt find anything useful. But CMS Made Simple has an Blind SQLi Exploit 

⇒ Running Exploit

[https://www.exploit-db.com/exploits/46635](https://www.exploit-db.com/exploits/46635) [Unauthenticated Blind SQLi ]

→ We increased the TIME to 3 seconds in the exploit as we kept getting IP Banned

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Writeup%20Box%20c8abaf66b7694b32be2810ee9ed1af0b/Untitled%206.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Writeup%20Box%20c8abaf66b7694b32be2810ee9ed1af0b/Untitled%206.png)

Password : **raykayjay9**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Writeup%20Box%20c8abaf66b7694b32be2810ee9ed1af0b/Untitled%207.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Writeup%20Box%20c8abaf66b7694b32be2810ee9ed1af0b/Untitled%207.png)

### Privilege Escalation to Root

⇒ After running pspy64 we notice that a cron is triggered when a user ssh in :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Writeup%20Box%20c8abaf66b7694b32be2810ee9ed1af0b/Untitled%208.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Writeup%20Box%20c8abaf66b7694b32be2810ee9ed1af0b/Untitled%208.png)

→ We see that its running **`run-parts`** and full path to it isnt specified. We can perform path hijacking.

⇒ Performing path hijacking 

→ We create an bash reverse shell named run-parts in **`/usr/local/bin`** :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Writeup%20Box%20c8abaf66b7694b32be2810ee9ed1af0b/Untitled%209.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Writeup%20Box%20c8abaf66b7694b32be2810ee9ed1af0b/Untitled%209.png)

→ We ssh to trigger the cron :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Writeup%20Box%20c8abaf66b7694b32be2810ee9ed1af0b/Untitled%2010.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Writeup%20Box%20c8abaf66b7694b32be2810ee9ed1af0b/Untitled%2010.png)

