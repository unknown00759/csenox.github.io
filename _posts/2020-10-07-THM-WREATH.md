---
title: "Task 1-"
tags: [windows,medium]
categories: THM-Wreath
---
![https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/THM_WREATH/wreath_banner-e1647485665429.png](https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/THM_WREATH/wreath_banner-e1647485665429.png)

From Task 1-4 All Those stuff are about setting up the lab which you can find out easily 
Without wasting both our time will move to the next step 

## *Task-5-Enumeration*
So we will begin with wreath by Task 5 - Enumeration 
i have performed the full port scan 
```
nmap -sC -sV -p-  --min-rate=500 -oN wreath.txt 10.200.105.200
```
![https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/THM_WREATH/nmap.png](https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/THM_WREATH/nmap.png)

Port 22 - ssh ( not exploitable by looking at that version 
Port 80 / 443 - will redirect you to https:.[]//thomaswreath.thm/

you need to add this url  on your host  to access them 

![https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/THM_WREATH/gedit.png](https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/THM_WREATH/gedit.png)


Now you can see the Website running 

![https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/THM_WREATH/web.png](https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/THM_WREATH/web.png)

```
Answer 1 :- 4
Note :- only open ports 

Answer 2 - centos

Answer 3 :- https://thomaswreath.thm/

Answer 4 :- +447821548812

Answer 5 - MiniServ 1.890 (Webmin httpd)

Answer 6 - CVE-2019-15107 
Note :- by searching miniserv 1.890 (webmin httpd) exploit on  google 
we get this link https://www.rapid7.com/db/vulnerabilities/http-webmin-cve-2019-15107/

```
