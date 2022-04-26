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
img 

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
