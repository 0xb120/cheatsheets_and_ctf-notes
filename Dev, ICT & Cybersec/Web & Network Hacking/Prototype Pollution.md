# Prototype Pollution 101

> [!question] What is Prototype Pollution?
>Prototype pollution is a [JavaScript & NodeJS](../Dev,%20scripting%20&%20OS/JavaScript%20&%20NodeJS.md) (but not only!) vulnerability that enables an attacker to add arbitrary properties to global object prototypes, which may then be inherited by user-defined objects.

It lets an attacker control properties of objects that would otherwise be inaccessible. If the application subsequently handles an attacker-controlled property in an unsafe way, this can potentially be chained with other vulnerabilities, like [DOM-based vulnerabilities](DOM-based%20vulnerabilities.md) for client-side prototype pollution, or even [Remote Code Execution (RCE)](Remote%20Code%20Execution%20(RCE).md) for server-side ones.

# High level overview

Prototype pollution vulnerabilities typically arise when a function recursively merges an object containing user-controllable properties into an existing object (see also [Mass Assignment](Mass%20Assignment.md)), without first sanitizing the keys. This can allow an attacker to inject a property with a key like `__proto__`, along with arbitrary nested properties.

It's possible to pollute any prototype object, but this most commonly occurs with the built-in global `Object.prototype.

Successful exploitation of prototype pollution requires the following components:

1. **A prototype pollution source**: input that enables you to poison prototype objects with arbitrary properties
2. **A sink**: a JavaScript function or DOM element that enables arbitrary code execution.
3. **An exploitable gadget**: any property that is passed into a sink without proper filtering or sanitization.

## Prototype pollution sources

A prototype pollution source is any user-controllable input that enables you to add arbitrary properties to prototype objects.

The most common are:
- URL (either the query or fragment string)
  >[!example] 
  >`https://vulnerable-website.com/?__proto__[evilProperty]=payload`

- JSON-based input (eg. `JSON.parse()`)
  >[!example]
  >```json
  >{
    >"__proto__": {
>        "evilProperty": "payload"
>    }
>}
>```
- Web messages

## Prototype pollution sinks

A prototype pollution sink is essentially just a JavaScript function or DOM element that you're able to access via prototype pollution, which enables you to execute arbitrary JavaScript or system commands. You can find some examples in [DOM-based vulnerabilities](DOM-based%20vulnerabilities.md).

## Prototype pollution gadgets

A gadget provides a means of turning the prototype pollution vulnerability into an actual exploit.

This is any property that is:

- Used by the application in an unsafe way, such as passing it to a sink without proper filtering or sanitization.
- Attacker-controllable via prototype pollution. In other words, the object must be able to inherit a malicious version of the property added to the prototype by an attacker.

>[!example]
>
>Many JavaScript libraries accept an object that developers can use to set different configuration options. The library code checks whether the developer has explicitly added certain properties to this object and, if so, adjusts the configuration accordingly. If a property that represents a particular option is not present, a predefined default option is often used instead.
>
>```js
let transport_url = config.transport_url || defaults.transport_url;
>```
>
>Now imagine the library code uses this `transport_url` to add a script reference to the page:
>
>```js
let script = document.createElement('script');
script.src = `${transport_url}/example.js`;
document.body.appendChild(script);
>```
>
>If the website's developers haven't set a `transport_url` property on their `config` object, this is a potential gadget. In cases where an attacker is able to pollute the global `Object.prototype` with their own `transport_url` property, this will be inherited by the `config` object and, therefore, set as the `src` for this script to a domain of the attacker's choosing.

---

# Prototype pollution vulnerabilities

- [Prototype Pollution client-side](Prototype%20Pollution%20client-side.md)
	- [JavaScript prototype pollution examples](Prototype%20Pollution%20client-side.md#JavaScript%20prototype%20pollution%20examples)
- [Prototype Pollution server-side](Prototype%20Pollution%20server-side.md)
	- [NodeJS prototype pollution examples](Prototype%20Pollution%20server-side.md#NodeJS%20prototype%20pollution%20examples)
	- [AST Prototype Pollution](Prototype%20Pollution%20server-side.md#AST%20Prototype%20Pollution)
- [Class Pollution](Class%20Pollution.md)