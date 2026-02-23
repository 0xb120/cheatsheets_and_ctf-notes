Arbitrary Object Instantiation vulnerabilities occur when an application instantiates objects using user-controlled data without proper validation, allowing attackers to **instantiate arbitrary classes** and potentially execute malicious code.

```php
$attacker_controlled_variable = $_GET['module'];
new $attacker_controlled_variable();
```

This allows an attacker to trigger `__construct()` methods of any class loaded in the system, which can be dangerous if those constructors perform sensitive actions.

> Because of that, Arbitrary Object Instantiation vulnerabilities are very close to [Insecure Deserialization & Object Injection](Insecure%20Deserialization%20&%20Object%20Injection.md) vulnerabilities. They exploit the same internal feature and mechanism.

## Examples

- [Arseniy Sharoglazov - Exploiting Arbitrary Object Instantiations in PHP Without Custom Classes](../../Readwise/Articles/Arseniy%20Sharoglazov%20-%20Exploiting%20Arbitrary%20Object%20Instantiations%20in%20PHP%20Without%20Custom%20Classes.md)
- [LFI via Autoloading](File%20Inclusion%20(LFI%20&%20RFI).md#LFI%20via%20Autoloading)