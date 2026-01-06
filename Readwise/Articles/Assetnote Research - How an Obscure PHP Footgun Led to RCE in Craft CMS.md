---
author: Assetnote Research
aliases:
  - How an Obscure PHP Footgun Led to RCE in Craft CMS
tags:
  - readwise/articles
url: https://www.assetnote.io/resources/research/how-an-obscure-php-footgun-led-to-rce-in-craft-cms?__readwiseLocation
created: 2025-02-26
---
# How an Obscure PHP Footgun Led to RCE in Craft CMS

![rw-book-cover](https://cdn.prod.website-files.com/6422e507d5004f85d107063a/649c2686dd142039d6d5ea8e_Frame%201428.png)


## register_argc_argv 101

Any developer familiar with developing PHP to be used on the command line will be familiar with `$_SERVER['argc']` and `$_SERVER['argv']`. [](https://read.readwise.io/read/01jn1f8gtj7ap4jpzbycpnnw7w)

```php
<?php var_dump($_SERVER['argv']);
```


Run `php test.php foo bar baz`, you will get: [](https://read.readwise.io/read/01jn1f8w2272tm9vfnjf9fe40a)
```json
array(4) {
[0]=>
string(7) "test.php"
[1]=>
string(3) "foo"
[2]=>
string(3) "bar"
[3]=>
string(3) "baz"
}
```

What happens if you host this file on a web server? This is controlled by the `register_argc_argv` configuration variable in the `php.ini`. In PHP's default configuration, `register_argc_argv` is on, and PHP will actually take `argv` from the query string, separated by spaces: [](https://read.readwise.io/read/01jn1f9j4g5fb642ntnktahcaq)
```http
GET /test.php?foo+bar+baz
```

```json
array(3) {
[1]=>
string(3) "foo"
[2]=>
string(3) "bar"
[3]=>
string(3) "baz"
}
```

If you are a developer wanting to test if a file is being executed via the command line or via the web, you may be tempted to test with something like: [](https://read.readwise.io/read/01jn1fa3mpkzdfgh3hpx3bk198)
```php
if (isset($_SERVER['argv'])) {
// cli ...
}
else {
// web ...
}
```

This will work, some of the time! But it will only work if `register_argc_argv` *is set to off*. If you run this code on a webserver in a default installation of PHP and pass a query string, this code will think it's being run via the CLI. [](https://read.readwise.io/read/01jn1fb8gec0wnbb25c6dbsm6d)

