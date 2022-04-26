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

website wouldnt be  not accessible untill you have added the victim hostname to your attack host name 
you need to add this url  on your host  to access them 

![https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/THM_WREATH/gedit.png](https://raw.githubusercontent.com/unknown00759/unknown00759.github.io/master/img/THM_WREATH/gedit.png)


As Now you can see the Website running 

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
## *Task-6-Exploitation*

Now we have to run that exploit which we have founded earlier in the task 5 
the thm instruction will guide you how to run that exploit step by step 


```
Exploit Explaination :- "Basically what this script does is there one configuration    in webmin manager " user password change " 
in which we will exploit our vulnerability  and this is the only condiition . the Webmin 1.890 is vulnerable in the default configuration
This 0day has been published at DEFCON-AppSec Village.
While researching on the Webmin application, I noticed some interesting ".cgi" files. One of them is "password_change.cgi"

in the "Webmin> Webmin Configuration> Authentication" section, 
the " password_change.cgi" sends the old password to "encrypt_password" function in "acl/acl-lib.pl 
( An ACL is a list of user permissions for a file, folder, or other object. It defines what users and groups can access
the object and what operations they can perform

then this function call the other function "unix_crypt"
in other section , the same function "unix_crypt" is called again for validate the password 
At this point, we will use "vertical bar (|)" by reading the shadow file during validating the old password.
And then we can Try To run the different comman on the server 
```
Exploit Explaination 
