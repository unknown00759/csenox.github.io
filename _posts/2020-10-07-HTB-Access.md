---
title: "Hack The Box - Access"
tags: [windows,easy,mdb ,zip,lnk, runas.exe , runas]
categories: HackTheBox-Windows
---

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Access%20Box%20a1736dce02b24e089481f9e686a31c68/Untitled.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Access%20Box%20a1736dce02b24e089481f9e686a31c68/Untitled.png)

## NMAP :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Access%20Box%20a1736dce02b24e089481f9e686a31c68/Untitled%201.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Access%20Box%20a1736dce02b24e089481f9e686a31c68/Untitled%201.png)

## Enumerating

⇒ There was nothing on the webpage.....

⇒ From ftp we found 2 files :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Access%20Box%20a1736dce02b24e089481f9e686a31c68/Untitled%202.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Access%20Box%20a1736dce02b24e089481f9e686a31c68/Untitled%202.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Access%20Box%20a1736dce02b24e089481f9e686a31c68/Untitled%203.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Access%20Box%20a1736dce02b24e089481f9e686a31c68/Untitled%203.png)

→ The zip file has a password.

→ We ran strings on backup.mdb and found password :

**`access4u@security`**

→ Unzipped the file and got a **.pst** file.

⇒ Opened the file in an **pst** viewer :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Access%20Box%20a1736dce02b24e089481f9e686a31c68/Untitled%204.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Access%20Box%20a1736dce02b24e089481f9e686a31c68/Untitled%204.png)

Password for security user : **`4Cc3ssC0ntr0ller`**

**⇒ We have a login on port 23**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Access%20Box%20a1736dce02b24e089481f9e686a31c68/Untitled%205.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Access%20Box%20a1736dce02b24e089481f9e686a31c68/Untitled%205.png)

→ Logged in using the credentials we found :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Access%20Box%20a1736dce02b24e089481f9e686a31c68/Untitled%206.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Access%20Box%20a1736dce02b24e089481f9e686a31c68/Untitled%206.png)

## Privilege Escalation

⇒ On public user desktop we find an lnk file :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Access%20Box%20a1736dce02b24e089481f9e686a31c68/Untitled%207.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Access%20Box%20a1736dce02b24e089481f9e686a31c68/Untitled%207.png)

`**runas.exe#..\..\..\Windows\System32\runas.exeC:\ZKTeco\ZKAccess3.5G/user:ACCESS\Administrator /savecred**`

→ Its runs runas.exe on a file as administrator using /savecred

→ We can run our msfvenom shell and get administrator shell :

**`runas.exe /user:ACCESS\Administrator /savecred "C:\temp\jake.exe"`**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Access%20Box%20a1736dce02b24e089481f9e686a31c68/Untitled%208.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Access%20Box%20a1736dce02b24e089481f9e686a31c68/Untitled%208.png)

