---
title: "Hack The Box - Player"
tags: [linux,hard,subdomain,ssrf,bfac,ffmpeg,xauth,codiad,cronjob]
categories: HackTheBox-Linux
---

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled.png)

## *NMAP*

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%201.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%201.png)

**Full Port :**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%202.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%202.png)

## *Enumerating Web*

⇒ Running gobuster on the website we discover directory /launcher which has an request access form.

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%203.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%203.png)

→ When we intercepted the request we notice an json web token ( access ) :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%204.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%204.png)

Decoded the jwt :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%205.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%205.png)

Enumerated a lot on the website but couldn't find anything.....

⇒ Running gobuster to hunt for subdomains :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%206.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%206.png)

### *dev.player.htb*

→ Its running codiad and has an autenticated rce exploit which will get us an shell on the system.

EXPLOIT : [https://github.com/WangYihang/Codiad-Remote-Code-Execute-Exploit](https://github.com/WangYihang/Codiad-Remote-Code-Execute-Exploit)

### *chat.player.htb*

→ Its an chat with the developer. We see an message talking about exposing sensitive files on staging site and main domain exposing source code which allows access to product before release :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%207.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%207.png)

### *staging.player.htb*

⇒ There's an contact us page , we see that in the response it reveals path to service_config and fix.php :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%208.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%208.png)

---

### *Hunting for files that reveal source code*

⇒ We tried looking for the file on main domain ( player.htb ) that is exposing source code which would allow us to get access to their product. 

→ We come upon an tool named Backup File Artifcats Checker which checks for backup artifacts that may disclose the web-application's source code.

[https://github.com/mazen160/bfac](https://github.com/mazen160/bfac)

**bfac --url [http://player.htb/launcher/dee8dc8a47256c64630d803a4c40786c.php](http://player.htb/launcher/dee8dc8a47256c64630d803a4c40786c.php)**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%209.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%209.png)

This page reveals the **access_code** and the **signature key** being used to create the jwt ( json web token / **access** cookie ).

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%2010.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%2010.png)

⇒ Now we can forge ourselves an jwt using the key and with the access code that allows us access to their product :

[https://jwt.io/](https://jwt.io/)

→ We get redirected to : [http://player.htb/launcher/7F2dcsSdZo6nj3SNMTQ1/](http://player.htb/launcher/7F2dcsSdZo6nj3SNMTQ1/)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%2011.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%2011.png)

---

### *Exploiting PlayBuff - Compact [ SSRF ]*

⇒ So we are able to upload video files . It compresses and converts mp4 file to .avi and lets us download it :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%2012.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%2012.png)

⇒ Performing SSRF in Video Converters : [https://hydrasky.com/network-security/exploiting-ssrf-in-video-converters/](https://www.youtube.com/watch?v=tZil9j7TTps&ab_channel=BlackHat)

→ Here **ffmpeg** is being used to convert the video file. **HTTP Live Streaming** is live and on-demand streaming which was developed by apple and supports ffmpeg. An file with **.m3u8** extension is a UTF-8 Encoded Audio Playlist file. These are plain text files which can be used by both audio and video players to describe where media files are located.

⇒ Making it send an http request to us :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%2013.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%2013.png)

Uploaded it to the site and then we recieve an request :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%2014.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%2014.png)

**So if an website / application utilizes ffmpeg for video conversion it is vulnerable to SSRF**

⇒ We will read local files on the system. We found this neat script which we can use to create the avi file : [https://github.com/cujanovic/SSRF-Testing/blob/master/ffmpeg/gen_avi.py](https://github.com/cujanovic/SSRF-Testing/blob/master/ffmpeg/gen_avi.py)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%2015.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%2015.png)

We are able to read /etc/passwd as when its being converted it sends request to file:///etc/passwd and stores the output to the converted avi file.

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%2016.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%2016.png)

→ Lets read the server_config that we previously found on the staging subdomain :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%2017.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%2017.png)

username = **telegen** 

password = **d-bC|jC!2uepS/w**

→ We are able to ssh in on port 6686 as user telegen. But we are in lshell ( restricted shell ) :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%2018.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%2018.png)

We tried escaping it but failed ☹️

### *Exploiting OpenSSH 7.2 [ xauth ]*

[https://www.exploit-db.com/exploits/39569](https://www.exploit-db.com/exploits/39569)

An authenticated user may inject arbitrary **xauth** commands by sending an **x11 channel** request that includes a **newline character** in the **x11 cookie**. The newline acts as a **command separator** to the **xauth binary**. This attack **requires** the server to have **X11Forwarding yes enabled**. Disabling it, mitigates this vector.

By injecting **xauth** commands one gains **limited* read/write arbitrary files**, information leakage or xauth-connect capabilities. These capabilities can be leveraged by an **authenticated restricted use**r ( i.e us ) .

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%2019.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%2019.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%2020.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%2020.png)

→ Lets read fix.php that we found on staging subdomain :

**.readfile /var/www/staging/fix.php**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%2021.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%2021.png)

⇒ Used the credentials and logged in on dev.player.htb ( codiad ).

**peter : CQXpm\z)G5D#%S$y=**

### *Exploiting Codiad [ Authenticated RCE ]*

The file **components/filemanager/class.filemanager.php** in Codiad prior to 2.8.4 is vulnerable to remote command execution because shell commands can be embedded in parameter values, as demonstrated by **search_file_type**.

EXPLOIT : [https://github.com/WangYihang/Codiad-Remote-Code-Execute-Exploit](https://github.com/WangYihang/Codiad-Remote-Code-Execute-Exploit)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%2022.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%2022.png)

Now if we try switching to user telegen we still get an restricted shell. So lets just try and privesc from www-data to root.

### *Privilege Escalation to Root [ Cronjob ]*

⇒ Running pspy we discover an cronjob running /var/lib/playbuff/buff.php :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%2023.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%2023.png)

⇒ Viewing the script :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%2024.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%2024.png)

- Includes /var/www/html/launcher/dee8dc8a47256c64630d803a4c40786g.php
- serializes buff variable
- unserializes /var/lib/playbuff/merge.log

⇒ We see that we have write access to the php file being included. We can simply replace its content with an php reverse shell from pentest monkey and wait for the cron to be triggered :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%2025.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%2025.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%2026.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Player%20Box%20762f0a919a0948b2bfe663bcf20aef16/Untitled%2026.png)
