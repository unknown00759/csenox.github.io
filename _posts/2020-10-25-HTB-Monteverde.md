---
title: "Hack The Box - Monteverde"
tags: [windows,medium,active directory,rpc,password,usernames,azure admins,azure]
categories: HackTheBox-Active-Directory
---

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Monteverde%20Box%20b0260a1c1ca7425da49daee88b891ea7/Untitled.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Monteverde%20Box%20b0260a1c1ca7425da49daee88b891ea7/Untitled.png)

⇒ Monteverde is an easy active directory machine that teaches us :

- Enumerating users through RPC
- Password spraying using usernames
- Abusing Azure Admins group for privesc to Domain Admin

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Monteverde%20Box%20b0260a1c1ca7425da49daee88b891ea7/Untitled%201.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Monteverde%20Box%20b0260a1c1ca7425da49daee88b891ea7/Untitled%201.png)

---

## Getting User through mostly Enumeration

⇒ So we are able to login in rpc through null authentication and enumere domain users using the command **`enumdomusers`** 

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Monteverde%20Box%20b0260a1c1ca7425da49daee88b891ea7/Untitled%202.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Monteverde%20Box%20b0260a1c1ca7425da49daee88b891ea7/Untitled%202.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Monteverde%20Box%20b0260a1c1ca7425da49daee88b891ea7/Untitled%203.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Monteverde%20Box%20b0260a1c1ca7425da49daee88b891ea7/Untitled%203.png)

- Next we created an list of users and performed password spraying using crackmapexec

```php
crackmapexec smb 10.10.10.172 -u users -p users
```

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Monteverde%20Box%20b0260a1c1ca7425da49daee88b891ea7/Untitled%204.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Monteverde%20Box%20b0260a1c1ca7425da49daee88b891ea7/Untitled%204.png)

**SABatchJobs : SABatchJobs**

---

### Enumerating SMB

⇒ So we are able to access users$ smb share as SABatchJobs user :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Monteverde%20Box%20b0260a1c1ca7425da49daee88b891ea7/Untitled%205.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Monteverde%20Box%20b0260a1c1ca7425da49daee88b891ea7/Untitled%205.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Monteverde%20Box%20b0260a1c1ca7425da49daee88b891ea7/Untitled%206.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Monteverde%20Box%20b0260a1c1ca7425da49daee88b891ea7/Untitled%206.png)

- We find an xml file named axure.xml in mhope directory which contains a password :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Monteverde%20Box%20b0260a1c1ca7425da49daee88b891ea7/Untitled%207.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Monteverde%20Box%20b0260a1c1ca7425da49daee88b891ea7/Untitled%207.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Monteverde%20Box%20b0260a1c1ca7425da49daee88b891ea7/Untitled%208.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Monteverde%20Box%20b0260a1c1ca7425da49daee88b891ea7/Untitled%208.png)

**mhope : 4n0therD4y@n0th3r$**

⇒ Next we just evil-winrm as mhope using the credentials we found :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Monteverde%20Box%20b0260a1c1ca7425da49daee88b891ea7/Untitled%209.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Monteverde%20Box%20b0260a1c1ca7425da49daee88b891ea7/Untitled%209.png)

---

## Privilege Escalation to Domain Admin [ AzureAdmins Group ]

⇒ So after owning mhope user we see that we are in AzureAdmins group which we can abuse to get Domain Admin on megabank domain.

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Monteverde%20Box%20b0260a1c1ca7425da49daee88b891ea7/Untitled%2010.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Monteverde%20Box%20b0260a1c1ca7425da49daee88b891ea7/Untitled%2010.png)

### Abusing Azure Admins

Reference : [https://blog.xpnsec.com/azuread-connect-for-redteam/](https://blog.xpnsec.com/azuread-connect-for-redteam/)

- So the most interesting is Password Hash Synchronisation (**PHS**), which uploads user accounts and password hashes from Active Directory into Azure.

- A new user is created during the installation of Azure AD Connect with the username **MSOL_**[HEX]

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Monteverde%20Box%20b0260a1c1ca7425da49daee88b891ea7/Untitled%2011.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Monteverde%20Box%20b0260a1c1ca7425da49daee88b891ea7/Untitled%2011.png)

- Now by default when deploying the connector a new database is created on the host using SQL Server's **LOCALDB**. The database supports the Azure AD Sync service by storing **metadata** and **configuration** data for the **service**.

- There's a table named **mms_management_agent** which contains several fields including **private_configuration_xml** which holds detail regarding **MSOL** user.

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Monteverde%20Box%20b0260a1c1ca7425da49daee88b891ea7/Untitled%2012.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Monteverde%20Box%20b0260a1c1ca7425da49daee88b891ea7/Untitled%2012.png)

- However the password is omitted from the XML returned . The encrypted password is actually stored within another field, **encrypted_configuration**

- So C:\Program Files\Microsoft Azure AD Sync\Binn\mcrypt.dll is responsible for key management and the decryption of the data in **encrypted_configuration**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Monteverde%20Box%20b0260a1c1ca7425da49daee88b891ea7/Untitled%2013.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Monteverde%20Box%20b0260a1c1ca7425da49daee88b891ea7/Untitled%2013.png)

⇒ So to decrypt the password from **encrypted_configuration** we will have to retrieve the keying material from the **LocalDB** instance before passing it to the **mcrypt.dll** assembly to decrypt

> So what are the requirements to complete this exfiltration of credentials? Well we will need to have access to the **LocalDB** (if configured to use this DB), which by default holds the following **security configuration**

> This means that if you are able to compromise a server containing the Azure AD Connect service, and gain access to either the **ADSyncAdmins** or local Administrators groups, what you have is the ability to retrieve the credentials for an account capable of performing a **DCSync**

⇒ **In short :**

- MSOL account is capable of peforming dcsync attack
- Retrieve the keyring from LocalDB
- Retrieve the encrypted password of MSOL user from **encrypted_configuration**
- Pass it to mcrypt.dll to decrypt and get the password for MSOL user.

Exploit Script : [https://github.com/Hackplayers/PsCabesha-tools/blob/master/Privesc/Azure-ADConnect.ps1](https://github.com/Hackplayers/PsCabesha-tools/blob/master/Privesc/Azure-ADConnect.ps1)

```php
upload Azure-ADConnect.ps1
import-module .\Azure-ADConnect.ps1
Azure-ADConnect -server 127.0.0.1 -db ADSync
```

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Monteverde%20Box%20b0260a1c1ca7425da49daee88b891ea7/Untitled%2014.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Monteverde%20Box%20b0260a1c1ca7425da49daee88b891ea7/Untitled%2014.png)

**administrator : d0m@in4dminyeah!**

⇒ Next we just evil-winrm using the administrator credentials :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Monteverde%20Box%20b0260a1c1ca7425da49daee88b891ea7/Untitled%2015.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Monteverde%20Box%20b0260a1c1ca7425da49daee88b891ea7/Untitled%2015.png)

---
