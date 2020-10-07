---
title: "Hack The Box - Kotarak"
tags: [linux,hard,ssrf,tomcat,ntds.dit,ntlm,secretsdump,wget,cronjob]
categories: HackTheBox
---


![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled.png)

## *NMAP*

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%201.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%201.png)

Full Port :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%202.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%202.png)

## *Enumerating [ SSRF ]*

⇒ So on port 8080 we have tomcat running. On port 60000 we have an webpage that sends an GET request on the url provided :  

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%203.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%203.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%204.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%204.png)

→ The word **`file`** is filtered which stops us from reading local files. Although we are able to use [localhost](http://localhost) and access this and the tomcat page :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%205.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%205.png)

⇒ Scanning for ports on [localhost](http://localhost) using wfuzz 

**`wfuzz -w wordlist.txt -u [http://10.10.10.55:60000/url.php?path=localhost:FUZZ/](http://10.10.10.55:60000/url.php?path=localhost:FUZZ/) --hl=2`**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%206.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%206.png)

→ [localhost:8888](http://localhost:8888) has interesting things :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%207.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%207.png)

→ Taking a look at **backup** option.

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%208.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%208.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%209.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%209.png)

We got credentials for tomcat

```
admin : 3@g01PdhB!
```

⇒ Logged in on tomcat manager page. Uploaded a WAR msfvenom shell and popped a shell :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%2010.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%2010.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%2011.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%2011.png)

---

### *Privilege Escalation [ tomcat to atanas ]*

⇒ In tomcat home folder we find 2 interesting files which looks like one of them is ntds.dit and the other is system registry file :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%2012.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%2012.png)

Lets run [secretsdump.py](http://secretsdump.py) [ a tool from impacket ] to retrieve ntlm hashes.

```bash
python3 /usr/share/doc/python3-impacket/examples/secretsdump.py -ntds 20170721114636_default_192.168.110.133_psexec.ntdsgrab._333512.dit -system 20170721114637_default_192.168.110.133_psexec.ntdsgrab._089134.bin LOCAL
```

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%2013.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%2013.png)

→ Cracking the NTLM hash

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%2014.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%2014.png)

→ The credentials work and we are able to switch to user atanas : 

**`atanas : f16tomcat!`**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%2015.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%2015.png)

---

### *Privilege Escalation to Root*

⇒ We are able to read and write files in root directory. We find the following :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%2016.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%2016.png)

→ Looks like there's another machine on the network with the ip [ 10.0.3.133 ] which is running an wget cron to our http server. We can also see the version of Wget being used  ( Wget/1.16 ). There's an exploit for this version of wget being used

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

⇒ So in this exploit when an user downloads an file using wget an attacker who controls the server can make wget create an arbitary file with arbitary contents by issuing a HTTP 30X Redirect containing ftp server reference in response to the victim's wget request. So now lets start exploiting it 🙂

→ Creating malicious .wgetrc that we will be serving :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%2017.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%2017.png)

→ When we normally try to host the ftp server it doesn't allow us to do so. There is authbind on the system which allows a program which does not or should not run as root to bind to low-numbered ports in a controlled way. 

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%2018.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%2018.png)

→ Editing our exploit and then running it on the system :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%2019.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%2019.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%2020.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%2020.png)

→ Waiting :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%2021.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%2021.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%2022.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Kotarak%20Box%2024e466eadc03400ea6b6c13ae24bb20f/Untitled%2022.png)

Now we have the root flag.

---

⇒ I tried getting a root shell on the 2nd machine, but i failed horribly . After talking with my friend about it he said that you can achieve an root shell by editing the cron file contents in the exploit script so that it runs an one liner reverse shell. 


