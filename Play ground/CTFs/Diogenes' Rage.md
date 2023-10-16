---
Category:
  - Web
Difficulty: Easy
Platform: HackTheBox
Status: 3. Complete
tags:
  - nodejs
  - race-condition
---
>[!quote]
> *Having missed the flight as you walk down the street, a wild vending machine appears in your way. You check your pocket and there it is, yet another half torn voucher coupon to feed to the consumerism. You start wondering why should you buy things that you don't like with the money you don't have for the people you don't like. You're Jack's raging bile duct.*


# Set up

- **build-docker.sh**
    
    ```bash
    #!/bin/bash
    docker rm -f web_diogenes_rage
    docker build -t web_diogenes_rage . 
    docker run --name=web_diogenes_rage --rm -p1337:1337 -it web_diogenes_rage
    ```
    
- **Dockerfile**
    
    ```bash
    FROM node:current-buster-slim
    
    # Install packages
    RUN apt-get update \
        && apt-get install -y supervisor \
        && rm -rf /var/lib/apt/lists/*
        
    # Setup app
    RUN mkdir -p /app
    
    # Add application
    WORKDIR /app
    COPY challenge .
    
    # Install dependencies
    RUN yarn
    
    # Setup superivsord
    COPY config/supervisord.conf /etc/supervisord.conf
    
    # Expose the port node-js is reachable on
    EXPOSE 1337
    
    # Start the node-js application
    CMD ["/usr/bin/supervisord", "-c", "/etc/supervisord.conf"]
    ```
    

# Information Gathering

## The application at-a-glance 🔍

The application allows to order some items (3) from the dispenser using virtual money (1).

1. Insert the ticket inside the dispenser
2. Digit on the keyboard (2) the product’s code (3) you wanna buy
3. Take your item

![Untitled](../../zzz_res/attachments/Diogenes'%20Rage%2027860513f86a42b2aec2640a41096060.png)

![Untitled](../../zzz_res/attachments/Diogenes'%20Rage%2027860513f86a42b2aec2640a41096060%201.png)

### HTTP Requests

The requests performed by the app are the following:

```
POST /api/coupons/apply HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9,it-IT;q=0.8,it;q=0.7
Cache-Control: no-cache
Connection: keep-alive
Content-Length: 25
Content-Type: application/json
Host: 206.189.26.97:31176
Origin: http://206.189.26.97:31176
Pragma: no-cache
Referer: http://206.189.26.97:31176/
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36

{"coupon_code":"HTB_100"}
```

```
POST /api/purchase HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9,it-IT;q=0.8,it;q=0.7
Cache-Control: no-cache
Connection: keep-alive
Content-Length: 13
Content-Type: application/json
Cookie: session=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6InR5bGVyXzg0NTIyNjVhY2MiLCJpYXQiOjE2NTY3NjU2OTJ9.an7jXxXq0eAPMfXlbSmTbBQqeasXfJXtaAln6pGy3bs
Host: 206.189.26.97:31176
Origin: http://206.189.26.97:31176
Pragma: no-cache
Referer: http://206.189.26.97:31176/
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36

{"item":"A1"}
```

```
GET /api/reset HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9,it-IT;q=0.8,it;q=0.7
Cache-Control: no-cache
Connection: keep-alive
Cookie: session=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6InR5bGVyXzg0NTIyNjVhY2MiLCJpYXQiOjE2NTY3NjU2OTJ9.an7jXxXq0eAPMfXlbSmTbBQqeasXfJXtaAln6pGy3bs
Host: 206.189.26.97:31176
Pragma: no-cache
Referer: http://206.189.26.97:31176/
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36
```

All the responses are reflected within the dispenser’s monitor

### JWT content

![Untitled](../../zzz_res/attachments/Diogenes'%20Rage%2027860513f86a42b2aec2640a41096060%202.png)

## Source code review

- **package.json**
    
    ```json
    {
    	"name": "diogenes_rage",
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
    		"body-parser": "^1.19.0",
    		"cookie-parser": "^1.4.4",
    		"express": "^4.17.1",
    		"jsonwebtoken": "^8.5.1",
    		"nunjucks": "^3.2.0",
    		"sqlite-async": "^1.1.2"
    	},
    	"devDependencies": {
    		"nodemon": "^1.19.1"
    	}
    }
    ```
    
- **index.js**
    
    ```jsx
    const express      = require('express');
    const app          = express();
    const path         = require('path');
    const bodyParser   = require('body-parser');
    const cookieParser = require('cookie-parser');
    const nunjucks     = require('nunjucks');
    const routes       = require('./routes');
    const Database     = require('./database');
    
    const db = new Database('diogenes-rage.db');
    
    app.use(bodyParser.json());
    app.use(bodyParser.urlencoded({ extended: true }));
    app.use(cookieParser());
    
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
    
- **routes/index.js**
    
    ```jsx
    const fs             = require('fs');
    const express        = require('express');
    const router         = express.Router();
    const JWTHelper      = require('../helpers/JWTHelper');
    const AuthMiddleware = require('../middleware/AuthMiddleware');
    
    let db;
    
    const response = data => ({ message: data });
    
    router.get('/', (req, res) => {
    	return res.render('index.html');
    });
    
    router.post('/api/purchase', AuthMiddleware, async (req, res) => {
    	return db.getUser(req.data.username)
    		.then(async user => {
    			if (user === undefined) {
    				await db.registerUser(req.data.username);
    				user = { username: req.data.username, balance: 0.00, coupons: '' };
    			}
    			const { item } = req.body;
    			if (item) {
    				return db.getProduct(item)
    					.then(product => {
    						if (product == undefined) return res.send(response("Invalid item code supplied!"));
    						if (product.price <= user.balance) {
    							newBalance = parseFloat(user.balance - product.price).toFixed(2);
    							return db.setBalance(req.data.username, newBalance)
    								.then(() => {
    									if (product.item_name == 'C8') return res.json({
    										flag: fs.readFileSync('/app/flag').toString(),
    										message: `Thank you for your order! $${newBalance} coupon credits left!`
    									})
    									res.send(response(`Thank you for your order! $${newBalance} coupon credits left!`))
    								});
    						}
    						return res.status(403).send(response("Insufficient balance!"));
    
    					})
    			}
    			return res.status(401).send(response('Missing required parameters!'));
    		});
    });
    
    router.post('/api/coupons/apply', AuthMiddleware, async (req, res) => {
    	return db.getUser(req.data.username)
    		.then(async user => {
    			if (user === undefined) {
    				await db.registerUser(req.data.username);
    				user = { username: req.data.username, balance: 0.00, coupons: '' };
    			}
    			const { coupon_code } = req.body;
    			if (coupon_code) {
    				if (user.coupons.includes(coupon_code)) {
    					return res.status(401).send(response("This coupon is already redeemed!"));
    				}
    				return db.getCouponValue(coupon_code)
    					.then(coupon => {
    						if (coupon) {
    							return db.addBalance(user.username, coupon.value)
    								.then(() => {
    									db.setCoupon(user.username, coupon_code)
    										.then(() => res.send(response(`$${coupon.value} coupon redeemed successfully! Please select an item for order.`)))
    								})
    								.catch(() => res.send(response("Failed to redeem the coupon!")));
    						}
    						res.send(response("No such coupon exists!"));
    					})
    			}
    			return res.status(401).send(response("Missing required parameters!"));
    		});
    });
    
    router.get('/api/reset', async (req, res) => {
    	res.clearCookie('session');
    	res.send(response("Insert coins below!"));
    });
    
    module.exports = database => {
    	db = database;
    	return router;
    };
    ```
    
- **middleware/AuthMiddleware.js**
    
    ```jsx
    const JWTHelper = require('../helpers/JWTHelper');
    const crypto    = require('crypto');
    
    module.exports = async (req, res, next) => {
    	try{
    		if (req.cookies.session === undefined) {
    			let username = `tyler_${crypto.randomBytes(5).toString('hex')}`;
    			let token = await JWTHelper.sign({
    				username
    			});
    			res.cookie('session', token, { maxAge: 48132000 });
    			req.data = {
    				username: username
    			};
    			return next();
    		}
    		let { username } = await JWTHelper.verify(req.cookies.session);
    		req.data = {
    			username: username
    		};
    		next();
    	} catch(e) {
    		console.log(e);
    		return res.status(500).send('Internal server error');
    	}
    }
    ```
    
- **helpers/JWTHelper.js**
    
    ```jsx
    const crypto = require('crypto');
    const jwt    = require('jsonwebtoken');
    const SECRET = crypto.randomBytes(69).toString('hex');
    
    module.exports = {
    	async sign(data) {
    		return jwt.sign(data, SECRET, { algorithm: 'HS256' });
    	},
    	verify(token) {
    		return jwt.verify(token, SECRET, { algorithm: 'HS256' });
    	}
    };
    ```
    

# The Bug - Race condition allows to reuse the same coupon

>[!info]
>Race conditions are vulnerabilities that appear in webs that limit the number of times you can perform an action.

Every time a coupon is used, the server assigns a different JWT creating a new user

![Untitled](../../zzz_res/attachments/Diogenes'%20Rage%2027860513f86a42b2aec2640a41096060%203.png)

![Untitled](../../zzz_res/attachments/Diogenes'%20Rage%2027860513f86a42b2aec2640a41096060%204.png)

Trying to reuse the same coupon with valid session also does not work, because the server recognizes the coupon has already been used:

![Untitled](../../zzz_res/attachments/Diogenes'%20Rage%2027860513f86a42b2aec2640a41096060%205.png)

However, if we manage to find a way to generate a valid user without having to use the coupon, then we can try to send multiple concurrent requests redeeming the coupon and maybe re-using it multiple times.

# Exploitation

Generate a valid session cookie without redeeming the coupon:

![Untitled](../../zzz_res/attachments/Diogenes'%20Rage%2027860513f86a42b2aec2640a41096060%206.png)

Sent the apply request to the turbointruder extension in order to use multiple concurrent requests and so using multiple time the same coupon:

![Untitled](../../zzz_res/attachments/Diogenes'%20Rage%2027860513f86a42b2aec2640a41096060%207.png)

![Untitled](../../zzz_res/attachments/Diogenes'%20Rage%2027860513f86a42b2aec2640a41096060%208.png)

Purchase the flag:

![Untitled](../../zzz_res/attachments/Diogenes'%20Rage%2027860513f86a42b2aec2640a41096060%209.png)

# Flag

>[!success]
>`HTB{r4c3_w3b_d3f34t_c0nsum3r1sm}`
