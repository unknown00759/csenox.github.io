---
title: "CyberSecLabs - Casino"
tags: [Linux,medium,ssti,setenv,python,ssrf]
categories: CyberSecLabs-Linux
---

⇒ Casino is an medium difficulty linux machine that teaches us about :

- Flask SSTI [ Server Side Template Injection ]
- SSRF [ Server Side Request Forgery ]
- Python SETENV abuse to hijack library being imported.

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Casino%20Machine%208fe92e2ee5994748856aed9a8b960335/Untitled.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Casino%20Machine%208fe92e2ee5994748856aed9a8b960335/Untitled.png)

---

## NMAP

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Casino%20Machine%208fe92e2ee5994748856aed9a8b960335/Untitled%201.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Casino%20Machine%208fe92e2ee5994748856aed9a8b960335/Untitled%201.png)

---

### Enumerating and performing SSTI

⇒ So when we visit the website we find the employees working and a few more names in the banned list

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Casino%20Machine%208fe92e2ee5994748856aed9a8b960335/asa.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Casino%20Machine%208fe92e2ee5994748856aed9a8b960335/asa.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Casino%20Machine%208fe92e2ee5994748856aed9a8b960335/Untitled%202.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Casino%20Machine%208fe92e2ee5994748856aed9a8b960335/Untitled%202.png)

- So we have an user list now :

```powershell
grey
Carla
Erlich
Blockman
Phil
Alan
Stu
Mr.Chow
Doug
```

⇒ After looking around for a file we see that search is vulnerable to SSTI as it executes `${{7*7}}` and returns **49** . We then read the items in the config using **{{config.items()}}**

Reference : [https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server Side Template Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Casino%20Machine%208fe92e2ee5994748856aed9a8b960335/ree.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Casino%20Machine%208fe92e2ee5994748856aed9a8b960335/ree.png)

- So we got the secret key : **i_L0v3$$$**

---

### SSRF

⇒ We are able to login as Erlich user using the secret key we found from the config. When we try to play the game we get the following page :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Casino%20Machine%208fe92e2ee5994748856aed9a8b960335/Untitled%203.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Casino%20Machine%208fe92e2ee5994748856aed9a8b960335/Untitled%203.png)

⇒ Intercepting the request we notice something interesting , an BTC parameter that has an url. We previously saw that there was port 9000/tcp which is filtered, it could be only accessible locally. So we tried and visit it through this request and it worked :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Casino%20Machine%208fe92e2ee5994748856aed9a8b960335/Untitled%204.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Casino%20Machine%208fe92e2ee5994748856aed9a8b960335/Untitled%204.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Casino%20Machine%208fe92e2ee5994748856aed9a8b960335/Untitled%205.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Casino%20Machine%208fe92e2ee5994748856aed9a8b960335/Untitled%205.png)

- On the /admin page there's an option that allows us to execute commands on the system :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Casino%20Machine%208fe92e2ee5994748856aed9a8b960335/Untitled%206.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Casino%20Machine%208fe92e2ee5994748856aed9a8b960335/Untitled%206.png)

- We ran id command and saw that it is running commands as user **grey.**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Casino%20Machine%208fe92e2ee5994748856aed9a8b960335/Untitled%207.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Casino%20Machine%208fe92e2ee5994748856aed9a8b960335/Untitled%207.png)

- We popped an shell as user grey :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Casino%20Machine%208fe92e2ee5994748856aed9a8b960335/Untitled%208.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Casino%20Machine%208fe92e2ee5994748856aed9a8b960335/Untitled%208.png)

---

### Privilege Escalation ( grey to carla )

⇒ There's an github repository in **adminPanel** folder

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Casino%20Machine%208fe92e2ee5994748856aed9a8b960335/Untitled%209.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Casino%20Machine%208fe92e2ee5994748856aed9a8b960335/Untitled%209.png)

- Running **git show .** We find an commit that has carla user password :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Casino%20Machine%208fe92e2ee5994748856aed9a8b960335/Untitled%2010.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Casino%20Machine%208fe92e2ee5994748856aed9a8b960335/Untitled%2010.png)

**carla : >F73SzS36>V$tJmc**

- We switched to carla :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Casino%20Machine%208fe92e2ee5994748856aed9a8b960335/Untitled%2011.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Casino%20Machine%208fe92e2ee5994748856aed9a8b960335/Untitled%2011.png)

---

### Privilege Escalation ( carla to root )

⇒ We can run an python script as root user and we also have SETENV permissions : 

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Casino%20Machine%208fe92e2ee5994748856aed9a8b960335/Untitled%2012.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Casino%20Machine%208fe92e2ee5994748856aed9a8b960335/Untitled%2012.png)

- Reading the python script we see that it performs simple things nothing vulnerable here.

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Casino%20Machine%208fe92e2ee5994748856aed9a8b960335/Untitled%2013.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Casino%20Machine%208fe92e2ee5994748856aed9a8b960335/Untitled%2013.png)

⇒ But we saw that we have SETENV permissions which would allow us to specify PYTHONPATH to import an module. We can leverage this to hijack the library **datetime** being imported .

- Creating [datetime.py](http://datetime.py) in /dev/shm that has an python reverse shell

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Casino%20Machine%208fe92e2ee5994748856aed9a8b960335/Untitled%2014.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Casino%20Machine%208fe92e2ee5994748856aed9a8b960335/Untitled%2014.png)

- Running the script as root with PYTHONPATH to /dev/shm 

**sudo PYTHONPATH=/dev/shm /opt/updateBTCPrice.py**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Casino%20Machine%208fe92e2ee5994748856aed9a8b960335/Untitled%2015.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Casino%20Machine%208fe92e2ee5994748856aed9a8b960335/Untitled%2015.png)

---
