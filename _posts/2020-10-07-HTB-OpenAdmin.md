---
title: "Hack The Box - OpenAdmin"
tags: [linux,easy,opennetadmin,ona,open net admin,apache config,sha512,ssh local port forwarding,nano gtfobins]
categories: HackTheBox-Linux
---

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/OpenAdmin%20Box%20c241e91d326e4ebaa21079be9e4fa7a7/Untitled.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/OpenAdmin%20Box%20c241e91d326e4ebaa21079be9e4fa7a7/Untitled.png)

## NMAP :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/OpenAdmin%20Box%20c241e91d326e4ebaa21079be9e4fa7a7/Untitled%201.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/OpenAdmin%20Box%20c241e91d326e4ebaa21079be9e4fa7a7/Untitled%201.png)

## Enumerating Web

→ After running gobuster we find /ona directory which is running OpenNetAdmin

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/OpenAdmin%20Box%20c241e91d326e4ebaa21079be9e4fa7a7/Untitled%202.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/OpenAdmin%20Box%20c241e91d326e4ebaa21079be9e4fa7a7/Untitled%202.png)

→ OpenNetAdmin v18.1.1

⇒ Exploit : [https://github.com/amriunix/ona-rce](https://github.com/amriunix/ona-rce)

→ Checking if it is vulnerable.

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/OpenAdmin%20Box%20c241e91d326e4ebaa21079be9e4fa7a7/Untitled%203.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/OpenAdmin%20Box%20c241e91d326e4ebaa21079be9e4fa7a7/Untitled%203.png)

→ Exploiting it .

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/OpenAdmin%20Box%20c241e91d326e4ebaa21079be9e4fa7a7/Untitled%204.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/OpenAdmin%20Box%20c241e91d326e4ebaa21079be9e4fa7a7/Untitled%204.png)

## Privilege Escalation to User

→ In ona config files we find an password :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/OpenAdmin%20Box%20c241e91d326e4ebaa21079be9e4fa7a7/Untitled%205.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/OpenAdmin%20Box%20c241e91d326e4ebaa21079be9e4fa7a7/Untitled%205.png)

**n1nj4W4rri0R!**

→ Used it to switch to user jimmy.

⇒ There's an internal directory in /var/www/ which only jimmy has read to . Checking apache configs we see that its an website running locally :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/OpenAdmin%20Box%20c241e91d326e4ebaa21079be9e4fa7a7/Untitled%206.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/OpenAdmin%20Box%20c241e91d326e4ebaa21079be9e4fa7a7/Untitled%206.png)

- Its running locally on port 52846

→ From the folder index.php we find an sha512 password hash which we cracked using [crackstation.net](http://crackstation.net) :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/OpenAdmin%20Box%20c241e91d326e4ebaa21079be9e4fa7a7/Untitled%207.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/OpenAdmin%20Box%20c241e91d326e4ebaa21079be9e4fa7a7/Untitled%207.png)

→ There's a php page which reads user joanna ssh key :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/OpenAdmin%20Box%20c241e91d326e4ebaa21079be9e4fa7a7/Untitled%208.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/OpenAdmin%20Box%20c241e91d326e4ebaa21079be9e4fa7a7/Untitled%208.png)

⇒ SSH Local Port Forwarding to access the page :

**`ssh -L 52846:127.0.0.1:52846 [jimmy@10.10.10.171](mailto:jimmy@10.10.10.171)`**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/OpenAdmin%20Box%20c241e91d326e4ebaa21079be9e4fa7a7/Untitled%209.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/OpenAdmin%20Box%20c241e91d326e4ebaa21079be9e4fa7a7/Untitled%209.png)

→ Logged in with **jimmy : Revealed**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/OpenAdmin%20Box%20c241e91d326e4ebaa21079be9e4fa7a7/Untitled%2010.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/OpenAdmin%20Box%20c241e91d326e4ebaa21079be9e4fa7a7/Untitled%2010.png)

→ Cracked key password using john : **bloodninjas**

→ SSH'ed in as user joanna using the ssh key :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/OpenAdmin%20Box%20c241e91d326e4ebaa21079be9e4fa7a7/Untitled%2011.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/OpenAdmin%20Box%20c241e91d326e4ebaa21079be9e4fa7a7/Untitled%2011.png)

## Privilege Escalation to root

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/OpenAdmin%20Box%20c241e91d326e4ebaa21079be9e4fa7a7/Untitled%2012.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/OpenAdmin%20Box%20c241e91d326e4ebaa21079be9e4fa7a7/Untitled%2012.png)

→ We can run nano on /opt/priv file as sudo. Used gtfobins to get root :

```
sudo /bin/nano /opt/priv
^R^X
reset; sh 1>&0 2>&0
```

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/OpenAdmin%20Box%20c241e91d326e4ebaa21079be9e4fa7a7/Untitled%2013.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/OpenAdmin%20Box%20c241e91d326e4ebaa21079be9e4fa7a7/Untitled%2013.png)

