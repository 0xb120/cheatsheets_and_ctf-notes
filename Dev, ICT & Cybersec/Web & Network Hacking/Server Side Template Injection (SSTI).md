>[!question] What is server-side template injection?
>Server-side template injection is when an attacker is able to **use native template syntax to inject a malicious payload into a template**, which is then executed server-side. Server-side template injection attacks can **occur when user input is concatenated directly into a template**, rather than passed in as data.

# Attacking server-side templates

As **templates are simply strings**, web developers sometimes directly concatenate user input into templates prior to rendering, causing the issue:

```php
// Vulnerable code
$output = $twig->render("Dear " . $_GET['name']);

// Safe code
$output = $twig->render("Dear {first_name},", array("first_name" => $user.first_name) );
```

## Detect

As with any vulnerability, the first step towards exploitation is being able to find it. Perhaps the simplest initial approach is to try fuzzing the template by injecting a sequence of special characters commonly used in template expressions, such as `${{<%[%'"}}%\`. If an exception is raised, this indicates that the injected template syntax is potentially being interpreted by the server in some way. This is one sign that a vulnerability to server-side template injection may exist.

SSTI exists in two contexts:
- **Plaintext context**:
```python
render('Hello ' + username)

http://vulnerable-website.com/?username=${7*7}
```

- **Code context**:
```python
greeting = getQueryParameter('greeting')
engine.render("Hello {{"+greeting+"}}", data)

http://vulnerable-website.com/?greeting=data.username}}<tag>
```

## Identify

Once you have detected the template injection potential, the next step is to identify the template engine. Although there are a huge number of templating languages, many of them use very similar syntax.

![SSTI](../../zzz_res/attachments/SSTI.png)

## Exploit

Once you discover a server-side template injection vulnerability, and identify the template engine being used, successful exploitation typically involves:

- Reading the template engine documentation and existing PoC
	- Basic template syntax
	- Security implication
	- Known exploits
- Explore
	- Explore the environment and try to discover all the objects to which you have access ( `self`, `debug`, `env`, `environment`, etc.)
	- Discover custom, site-specific non-standard objects that have been supplied by the web developer
- Create a custom attack
	- Object chain (eg. `$class.inspect("java.lang.Runtime").type.getRuntime().exec("bad-stuff-here")`)
	- Chain exploiting developer-supplied objects

# External Resources

- [Server-Side Template Injection | James Kettle](https://portswigger.net/research/server-side-template-injection)
- [SSTI - HackTricks](https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection)
- [SSTI - PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/README.md)