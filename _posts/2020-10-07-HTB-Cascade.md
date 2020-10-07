---
title: "Hack The Box - Cascade"
tags: [windows,medium,rpc,ldap,vnc,smb,dnspy,re,aes,AD Recyle Bin Group]
categories: HackTheBox-Active-Directory
---

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled.png)

⇒ Cascade is an medium difficulty windows machine made by VBScrub that teaches us about :

- Using rpc to enumerate users and ldapsearch
- SMB Enumeration and VNC Registry password decryption
- Disassembling .exe and .dll files using dnSpy
- Decrypting AES encrypted hash
- Recovering Deleted AD Objects [ AD Recycle Bin group ]

---

## NMAP

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%201.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%201.png)

---

## Enumerating and Password Spraying

- So we are able to authenticate to rpc with null authentication and enumerate domusers :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%202.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%202.png)

- We used ldapsearch to perform ldapqueries and we found an base64 encoded string which we decoded and got : **rY4n5eva**

```bash
ldapsearch -h cascade.htb -x -b "dc=cascade,dc=local" > ldapsearch.txt
strings ldapsearch.txt | grep Pwd
```

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%203.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%203.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%204.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%204.png)

- Next we ran crackmapexec and discovered that the password works for user r.thompson

```bash
crackmapexec smb 10.10.10.182 -u user.txt -p rY4n5eva
```

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%205.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%205.png)

**r.thompson:rY4n5eva** [ User only has smb access no winrm ]

---

### Enumerating SMB Shares and decrypting vnc_registry pass

- So user r.thompson can read the following shares. The Data share looks interesting so lets download all the files in the share and look around for something useful :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%206.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%206.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%207.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%207.png)

```bash
recurse on
prompt off
mget *
```

- So we were able to download the following files from the Data share

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%208.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%208.png)

⇒ Analyzing the files :

- The meeting notes june 2018 talks about an TempAdmin user which will be Deleted and has same password as Administrator account.

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%209.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%209.png)

- ArkADRecyclebin.log has the logs of the TempAdmin user account being moved to AD Recycle Bin :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%2010.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%2010.png)

- User s.smith VNC_Install.reg has an hex password string :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%2011.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%2011.png)

⇒ This password hex can be decoded using the tool : [http://aluigi.org/pwdrec/vncpwd.zip](http://aluigi.org/pwdrec/vncpwd.zip)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%2012.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%2012.png)

**s.smith : sT333ve2**

→ We were able to WinRM using user s.smith and read the user flag :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%2013.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%2013.png)

---

### Privilege Escalation to ArkSvc [ RE and decrypting AES cipher ]

⇒ User s.smith can read Audit$ share. We downloaded all the files and started enumerating :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%2014.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%2014.png)

We downloaded and got the following files :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%2015.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%2015.png)

- DB\Audit.db has ArkSvc password which is AES encrypted :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%2016.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%2016.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%2017.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%2017.png)

Pwd = BQO5l5Kj9MdErXx6Q6AGOw==

- We disassembled the exe and dll files and found the key and IV key

We used dnspy : [https://github.com/0xd4d/dnSpy/releases](https://github.com/0xd4d/dnSpy/releases)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%2018.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%2018.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%2019.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%2019.png)

key : **c4scadek3y654321**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%2020.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%2020.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%2021.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%2021.png)

IV Key : **1tdyjCbY1Ix49842**

- Finally we decrypted the cipher and got the password for ArkSvc user

**ArkSvc : w3lc0meFr31nd**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%2022.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%2022.png)

---

### Privilege Escalation to Root [ Getting Deleted AD Objects ]

⇒ So after getting arksvc we see that this user is in AD Recycle Bin groups. So looking back we saw that there was user TempAdmin which has same password as Administrator and was moved to Recycle Bin.

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%2023.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%2023.png)

The following post explains how to recover deleted ad ojbects : [https://blog.stealthbits.com/active-directory-object-recovery-recycle-bin/](https://blog.stealthbits.com/active-directory-object-recovery-recycle-bin/)

- Lets get all deleted objects on the Domain

```powershell
Get-ADObject -filter 'isDeleted -eq $true -and name -ne "Deleted Objects"' -includeDeletedObjects
```

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%2024.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%2024.png)

- We see that TempAdmin is in deleted objects. Lets read the objects property

```powershell
Get-ADObject -filter 'isdeleted -eq $true -and name -ne "Deleted Objects"' -includeDeletedObjects -property *
```

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%2025.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%2025.png)

After decoding it we get the password for TempAdmin whose password is same as Administrator : 

**baCT3r1aN00dles**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%2026.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Cascade%20Box%20e9898716717a4280b147a952c0b0a0a5/Untitled%2026.png)
