---
title: "Hack The Box - Kotarak"
tags: [linux,hard,ssrf,tomcat,ntds.dit,ntlm,secretsdump,wget,cronjob]
categories: HackTheBox-Linux
---


![https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/title.png](https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/title.png)

## *NMAP*
This Machine is the Hard level machine so i will prefer the Full Port Scan :- 

![https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/Untitled%203.png](https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/Untitled%203.png)


## *Enumerating [ SSRF ]*

-> So on port 8080 we have our Tomcat running . On port 60000 we have an webpage that sends an GET request on the url provided :

![https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/Untitled%203.png](https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/Untitled%203.png)

-> If we try to read /etc/passwd the The word **`file`** is filtered which stops us from reading local files. Although we are able to use [localhost](http://localhost) and access this and the tomcat page :

![https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/burp.png](https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/burp.png)




