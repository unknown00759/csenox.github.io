---
title: "Hack The Box - Bastion"
tags: [windows,easy,vhd,sam,system,samdump2,mRemoteng]
categories: HackTheBox-Windows
---

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Bastion%20Box%204e317b6dd5cc42b6995d05eba0c48fb4/Untitled.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Bastion%20Box%204e317b6dd5cc42b6995d05eba0c48fb4/Untitled.png)

## NMAP

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Bastion%20Box%204e317b6dd5cc42b6995d05eba0c48fb4/Untitled%201.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Bastion%20Box%204e317b6dd5cc42b6995d05eba0c48fb4/Untitled%201.png)

### Enumerating SMB

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Bastion%20Box%204e317b6dd5cc42b6995d05eba0c48fb4/Untitled%202.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Bastion%20Box%204e317b6dd5cc42b6995d05eba0c48fb4/Untitled%202.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Bastion%20Box%204e317b6dd5cc42b6995d05eba0c48fb4/Untitled%203.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Bastion%20Box%204e317b6dd5cc42b6995d05eba0c48fb4/Untitled%203.png)

⇒ We mounted the smb share and found vhd ( Virtual Hard Disk ) files :

**mkdir /mnt/L4mpje-PC/**

**mount -t cifs //10.10.10.134/Backups/WindowsImageBackup/L4mpje-PC /mnt/L4mpje-PC/ -o user=anonymous**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Bastion%20Box%204e317b6dd5cc42b6995d05eba0c48fb4/Untitled%204.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Bastion%20Box%204e317b6dd5cc42b6995d05eba0c48fb4/Untitled%204.png)

⇒ Mounting the vhd file :

**mkdir /mnt/vhd**

**sudo modprobe nbd max_part=16**

**qemu-nbd -r -c /dev/nbd0 "/mnt/L4mpje-PC/Backup 2019-02-22 124351/9b9cfbc4-369e-11e9-a17c-806e6f6e6963.vhd"**

**mount -r /dev/nbd0p1 /mnt/vhd**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Bastion%20Box%204e317b6dd5cc42b6995d05eba0c48fb4/Untitled%205.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Bastion%20Box%204e317b6dd5cc42b6995d05eba0c48fb4/Untitled%205.png)

⇒ We grabbed the SAM and SYSTEM config files and dumped the hashes using **samdump2**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Bastion%20Box%204e317b6dd5cc42b6995d05eba0c48fb4/Untitled%206.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Bastion%20Box%204e317b6dd5cc42b6995d05eba0c48fb4/Untitled%206.png)

→ Cracked the hash :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Bastion%20Box%204e317b6dd5cc42b6995d05eba0c48fb4/Untitled%207.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Bastion%20Box%204e317b6dd5cc42b6995d05eba0c48fb4/Untitled%207.png)

**L4mpje** : **bureaulampje**

→ We ssh'ed in as user L4mpje :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Bastion%20Box%204e317b6dd5cc42b6995d05eba0c48fb4/Untitled%208.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Bastion%20Box%204e317b6dd5cc42b6995d05eba0c48fb4/Untitled%208.png)

### Privilege Escalation :

⇒ In program files folder we see mRemoteNG. Which has a vulnerability of storing passwords in config files which can be decrypted using mrRemoteNG-decrypt tool on github :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Bastion%20Box%204e317b6dd5cc42b6995d05eba0c48fb4/Untitled%209.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Bastion%20Box%204e317b6dd5cc42b6995d05eba0c48fb4/Untitled%209.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Bastion%20Box%204e317b6dd5cc42b6995d05eba0c48fb4/Untitled%2010.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Bastion%20Box%204e317b6dd5cc42b6995d05eba0c48fb4/Untitled%2010.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Bastion%20Box%204e317b6dd5cc42b6995d05eba0c48fb4/Untitled%2011.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Bastion%20Box%204e317b6dd5cc42b6995d05eba0c48fb4/Untitled%2011.png)

→ The encrypted password is in **`confCons.xml`**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Bastion%20Box%204e317b6dd5cc42b6995d05eba0c48fb4/Untitled%2012.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Bastion%20Box%204e317b6dd5cc42b6995d05eba0c48fb4/Untitled%2012.png)

Hash :

```
aEWNFV5uGcjUHF0uS17QTdT9kVqtKCPeoC0Nw5dmaPFjNQ2kt/zO5xDqE4HdVmHAowVRdC7emf7lWWA10dQKiw==
```

→ Decrypting it using the tool : [https://github.com/haseebT/mRemoteNG-Decrypt](https://github.com/haseebT/mRemoteNG-Decrypt)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Bastion%20Box%204e317b6dd5cc42b6995d05eba0c48fb4/Untitled%2013.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Bastion%20Box%204e317b6dd5cc42b6995d05eba0c48fb4/Untitled%2013.png)

Password : **thXLHM96BeKL0ER2**

⇒ SSH'ed in as administrator :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Bastion%20Box%204e317b6dd5cc42b6995d05eba0c48fb4/Untitled%2014.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Bastion%20Box%204e317b6dd5cc42b6995d05eba0c48fb4/Untitled%2014.png)

