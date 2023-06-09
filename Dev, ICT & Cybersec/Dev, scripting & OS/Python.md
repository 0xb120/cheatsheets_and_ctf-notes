
## Versions and environment management

>[!note]
>See also how to use **multiple isolated versions of Python** using [pyenv](../Tools/pyenv.md).

```bash
# create a virtual environemnt named .venv containing the python standalone environment
$ python3 -m venv .venv

# active the virtual environment
$ source .venv/bin/activate

# disable the environment
$ deactivate
```

# Python3 snippets

>[!tip] AWAE snippets
>Pre-built examples and trick available here: https://mlcsec.com/posts/python-awae-prep/

## Basic I/O

```python
input = ""

print("""
Print on
multiple lines!
""")

input = input("Please Subscribe!")
print(f"You replied with {input}")

print(f"You replied with {input}", flush=True) # Debug for web server

print("This is a {} using {}".format('formatter','format'))
```

## argv

```python
from sys import argv
script, param1, param2 = argv
arg2 = argv[2]
```

## argparse

>[!summary]
>- Parser for command-line options, arguments and sub-commands - Python 3.10.5 documentation [^1]
>- Argparse Tutorial - Python 3.10.5 documentation [^2]

[^1]: https://docs.python.org/3/library/argparse.html
[^2]: https://docs.python.org/3/howto/argparse.html

```python
#!/usr/bin/python3
import urllib.parse, argparse

parser = argparse.ArgumentParser(description="Exploit Spiky Tamagotchi")
parser.add_argument("-i","--ip", help="target IP")
parser.add_argument("-p","--port", help="target port")
parser.add_argument("-c","--cmd", help="command to execute")
parser.add_argument("--username", help="login username (deafult: admin)", default="admin")

args = parser.parse_args()

if not(args.ip and args.port and args.cmd):
    print("Usage: exploit.py -i <ip> -p <port> -c <cmd>")
    exit 0

...
```

## Basic statements

```python
# If, else
if x > y:
	...
elif x = y:
	...
else:
	...

# Fuctions
def function(arg1, arg2):
	...
	return val
	
# While
while i < 10:
	...
	break		# Exit from the loop
	...
	continue	# Skip the current iteration
	...
	pass		# Skip the current condition (if)
	
# For
for i in range(1,10):
	...
	break		# Exit from the loop
	...
	continue	# Skip the current iteration
	...
	pass	
	
# Try wxcept
try:
	...
except NullPointer:
	...
```

## Lists, dictionaries and tuples

```python
list = [1,'str1',2,'str2',3,'str3']
element_2 = list[1]

dict = {
	'name' : 'maoutis', 
	'age' : 23, 
	'other' : 'hacking lover'
}
age = dict['age']
dict['isNew'] = 1 # Add new element to the dict
dict = {
	'name' : 'maoutis', 
	'age' : 23, 
	'other' : 'hacking lover',
	'isNew' : 1
}

tuple = (1,'str1',2,'str2',3,'str3')
```

## R/W files

```python
file = open("filename","w") # w or r for string; wb or rb for bytes
# "x" - Create - will create a file, returns an error if the file exist
# "a" - Append - will create a file if the specified file does not exist
# "w" - Write - will create a file if the specified file does not exist
data = file.read()
file.truncate()
file.write("Writing into the file!")
file.seek(0)		# Rewind to the first char
file.write("Writing into the file again from the start!")
file.close()
```

## Class and Objects

```python
class MyClass(object):
	def __init__(self):
		self.name = "MaOutis"
		
	def printName(self):
		print(self.name)

	def add(a,b):
		return a+b
		
person = MyClass()
person.printName()
print(person.name)
```

### `__class__`

The attribute points to the class that the object is an instance of:
>[!tip]
>You can think about `<instance>.__class__` in Python as `<instance>.constructor` in JavaScript.

```python
class Employee: pass # Creating an empty class

emp = Employee()
print(emp.__class__)
#> <class '__main__.Employee'>
```

#### `__class__` attributes

- `__qualname__` is an attribute that contains the class name:

```python
class Employee: pass # Creating an empty class
emp = Employee()

print(emp.__class__.__qualname__)
#> Employee
```

- `__getattribute__` and `__setattr__` are *getter* and *setter* for a class attribute and returns/overwrites the attribute value:

```python
class Employee: pass # Creating an empty class

emp = Employee()
emp.name = "test"
emp.__getattribute__('name')
#> 'test'
emp.name
#> 'test'
emp.__setattr__("name","new val")
emp.name
#> 'new val'
```

-  `__base__` points to the nearest parent class that it’s inheriting from, so if there is an inheritance chain, it will point to the last class that we inherit:

```python
class Employee: pass # Creating an empty class

emp = Employee()
emp
#> <__main__.Employee object at 0x7f9ff0e4f3d0>
emp.__class__
#> <class '__main__.Employee'>
emp.__class__.__base__
#> <class 'object'>
```



---

## Print raw bytes

Python2:

```python
$ python2 -c 'print("\xff")' | hexdump -C
00000000  ff 0a                                             |..|
00000002

$ python2 -c 'import sys;sys.stdout.write("\xff")' | hexdump -C
00000000  ff                                                |.|
00000001
```

Python3:

```python
$ python3 -c 'import sys;sys.stdout.buffer.write(b"\xff")' | hexdump -C
00000000  ff                                                |.|
00000001
```

## Mixing strings and byte-strings

```python
>>> "ABCD".encode("utf-8")
b'ABCD'

>>> b"\x41\x42\x43\x44".decode("ascii")
'ABCD'

# From python 3.5

>>> b'\xde\xad\xbe\xef'.hex()
'deadbeef'

>>> bytes.fromhex('deadbeef')
b'\xde\xad\xbe\xef'
```

## Number conversion

```python
# Define format statically
>>> binary = 0b1010
>>> hexadecimal = "0xa"

# String --> Dec
>>> int("10")
10

# String --> Hex
>>> int("0x12F", base=16)
303

# Int (dec/hex/bin) --> String
>>> str(10)
'10'
>>> str(0b11010010)
'210'

# int (dec) --> String (Hex)
>>> hex(240)
'0xf0'
```

## struct

This module performs conversions between Python values and C structs represented as Python `bytes` objects. This can be used in handling binary data stored in files or from network connections, among other sources. It uses [Format Strings](https://docs.python.org/3/library/struct.html#struct-format-strings) as compact descriptions of the layout of the C structs and the intended conversion to/from Python values [^3].

[^3]: https://docs.python.org/3/library/struct.html

```python
import struct

# pack into a x64 little endian string
struct.pack("<Q",0x1234)
b'4\x12\x00\x00\x00\x00\x00\x00'

# pack into a x64 big endian string
struct.pack(">Q",1234)
b'\x00\x00\x00\x00\x00\x00\x04\xd2'

# pack into a x86 little endian string
struct.pack("<I",0x1234)
b'4\x12\x00\x00'

# pack into a x86 big endian string
struct.pack(">I",0x1234)
b'\x00\x00\x124'

# unpack from a x86 big endian string
struct.unpack(">I",b'\x00\x00\x124')[0]
4660
hex(struct.unpack(">I",b'\x00\x00\x12\x34')[0])
'0x1234'
```

## socket

```python
import socket

host = 127.0.0.1
port = 10099

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
connect = s.connect((host, port))
s.recv(1024)
s.send('Data')
s.recv(1024)
s.close()
```

## requests

```python
import requests

cookies_dict = {"php-console-server":"5", "php-console-client":b64_php_console_client.decode()}
resp = requests.get(url, cookies=cookies_dict)

print(resp.headers["PHP-Console"])
print(resp.content)
print(resp.text)
print(resp.status_code)
print(resp.cookies)

# Multipart / File upload
files = {'file': ("image.jpg", exploit, 'image/jpg')}

url = f'http://{target}:{port}/api/upload'
print(f"Uploading file to {url}")

resp = requests.post(url, files=files, proxies={"http":"http://127.0.0.1:8080"})

# Send JSON data
url = 'http://' + args.ip + ':' + args.port + '/api/login'
payload = {
    "username":f"{args.username}",
    "password":{
        "password":1
    }
}
jsonData = json.dumps(payload)
print(jsonData)
headers = {'Content-type': 'application/json'}
resp = requests.post(url, data = jsonData, headers=headers)

# Manage cookies
s = requests.Session()
resp = s.post(url, data = jsonData, headers=headers)
session = s.cookies.get_dict()
print(session)
resp = s.post(url, data = jsonData, headers=headers, cookies=session)
```

## pwntools

>[!summary]
>pwntools - pwntools 4.8.0 documentation [^4]

[^4]: https://docs.pwntools.com/en/stable/index.html

`pwntools` is a CTF framework and exploit development library. Written in Python, it is designed for rapid prototyping and development, and intended to make exploit writing as simple as possible [^5].

[^5]: https://docs.pwntools.com/en/stable/#

```python
from pwn import *

# Interact with local process
p = process('sp_going_deeper')
# or
elf = pwn.ELF('<nome>')
p = elf.process()

# Interact with remote process
p = remote("46.101.27.51",30934)

# Basic I/O
print(p.recvuntil(b">> ", timeout=1))
p.sendline(p40)
p.recvline()

# Keep session interactive
p.interactive()
```

## builtins
<iframe width="1060" height="415" src="https://sadh.life/post/builtins/" title="Python buildins" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

>[!tldr]- 
>Dunder methods (also known as magic methods) are special methods that are implicitly invoked by all objects in Python during various operations, such as 
`__str__()`, `__eq__()`, and `__call__()`. They are used to specify what objects of a class should do when used in various statements and with various operators.
>
>Dunder methods have their own default implementation for built-in classes, which we will be implicitly inheriting from when creating a new class, however, developers can override these methods and provide their own implementation when defining new classes.
>
>There are also other special attributes in every object in Python, such as `__class__`, `__doc__`, etc.

# Python 2 vs 3 for binary exploitation

<iframe width="660" height="415" src="https://www.youtube.com/embed/FxNS-zSS7MQ" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
