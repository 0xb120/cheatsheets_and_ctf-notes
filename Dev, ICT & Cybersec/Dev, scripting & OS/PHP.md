
# PHP 101
## Print info inside logs

```php
error_log("message", 0);
```

## PHP wrappers and filters

![PHP wrappers and filters](../Web%20&%20Network%20Hacking/File%20Inclusion%20(LFI%20&%20RFI).md#PHP%20wrappers%20and%20filters)

## Enable remote debug in PHP with Xdebug

Guide from *How to setup PHP remote debugging* [^remote-debug-php] and Debugging PHP on Docker with VS Code [^remote-debug-php-docker].

```sh
sudo apt-get install php-xdebug
sudo pecl install xdebug
```

[^remote-debug-php]: [How to setup PHP remote debugging](https://blog.devsense.com/2020/remote-debugging), devsense.com
[^remote-debug-php-docker]: [Debugging PHP on Docker with VS Code](https://blog.devsense.com/2019/debugging-php-on-docker-with-visual-studio-code), devsense.com

If you are not using docker, you can simply enable remote debugging by editing `php.ini`

```ini title:"php.ini"
[Xdebug]
zend_extension=xdebug.so
xdebug.remote_enable=1
xdebug.remote_host=<debugger ip>
xdebug.remote_port=<debugger port>
```

If you are using docker, you can specify some docker environment variables and install xdebug from the Dockerfile:

```Dockerfile title:Dockerfile
FROM php:8.2-apache

RUN apt-get update \
  && apt-get install -y \
      wait-for-it \
      unzip \
      libfreetype6-dev \
      libjpeg62-turbo-dev \
      libpng-dev \
      libzip-dev \
      libcurl3-dev \
	  libwebp-dev

RUN pecl install xdebug

RUN docker-php-ext-configure gd --with-freetype --with-jpeg --with-webp \
  && docker-php-ext-install -j$(nproc) gd zip mysqli curl \
  && docker-php-ext-enable gd zip mysqli curl \
  && docker-php-ext-enable xdebug

RUN chmod -R 777 /var/www/

RUN a2enmod rewrite
```

```yml title:docker-compose.yml
services:
  opencart:
    build: 
      dockerfile: Dockerfile
    user: root
    ports:
      - "80:80"
      - "9000:9000"
    volumes:
      - ./upload:/var/www/html:rw
    environment:
      XDEBUG_MODE: debug
      XDEBUG_CONFIG: client_host=172.17.0.1 client_port=9003
      ...
```

- **XDEBUG_MODE: debug** configures which Xdebug features should be enabled. For our case we will just set `debug`. For more possible values, click [here](https://xdebug.org/docs/step_debug#mode)

You can verify if XDEBUG is enabled searching the xdebug info inside the `phpinfo()` output. Then you can debug the application using the vscode debugger extension "PHP Debug" and the following launcher:
```json title:"launch-xdebug.json"
{
"name": "Listen for Xdebug",
"type": "php",
"request": "launch",
"port": 9003,
"pathMappings": {
"/var/www/html": "${workspaceFolder}\\data"
}
```

When you make a request to [http://localhost:8080?XDEBUG_SESSION_START=1](http://localhost:8080/?XDEBUG_SESSION_START=1) the execution should be halted. `XDEBUG_SESSION_START` argument instructs Xdebug to start debugging the request. [^xdebug-step-debug]
You can also set a cookie names `xDEBUG_SESSION` with any value to automatically debug EVERY request.

[^xdebug-step-debug]: [Step debugging](https://xdebug.org/docs/step_debug), xdebug.org

### Inspect available in-memory classes and objects

While your code is paused at a breakpoint, open the **"Evaluate Expression"** or **"Debug Console"** and use:
```php
// every class
get_declared_classes();

// filter for namespace
array_filter(get_declared_classes(), fn($className) => str_starts_with($className, 'App\\'));

// unique root namespaces currently loaded
array_unique(array_map(fn($c) => explode('\\', $c)[0], get_declared_classes()));

// If you find a class in that list and want to see its methods, properties, or even the **file path** it was loaded from, use the **Reflection** API in your debug evaluator
$reflector = new ReflectionClass('App\Services\MyService');
print_r([
    'file' => $reflector->getFileName(),
    'methods' => $reflector->getMethods(),
    'interfaces' => $reflector->getInterfaceNames()
]);

// inspect what the autoload.php "knows". Place a breakpoint on 
// require 'vendor/autoload.php0 and then
$loader = require 'vendor/autoload.php';
print_r($loader->getPrefixesPsr4());
```

See also [Exploiting `new $a($b)` via Built-In Classes](../../Readwise/Articles/Arseniy%20Sharoglazov%20-%20Exploiting%20Arbitrary%20Object%20Instantiations%20in%20PHP%20Without%20Custom%20Classes.md#Exploiting%20`new%20$a($b)`%20via%20Built-In%20Classes)

If you need to find if a specific class has already instantiated some objects, you can use this code:
```php
call_user_func(function() {
    $targetClass = 'WPvividGuzzleHttp\Cookie\FileCookieJar'; // <-- SET YOUR TARGET CLASS HERE
    $results = [];
    $visitedObjects = [];
    
    // Recursive search function
    $search = function($path, $value, $depth = 0) use (&$search, &$results, &$visitedObjects, $targetClass) {
        // Hard limit to prevent infinite recursion on massive arrays
        if ($depth > 10) return; 

        if (is_object($value)) {
            // Track object hashes to prevent infinite loops from circular references
            $hash = spl_object_hash($value);
            if (isset($visitedObjects[$hash])) return;
            $visitedObjects[$hash] = true;
            
            // Check if it's our target
            if ($value instanceof $targetClass || get_class($value) === $targetClass) {
                $results[] = $path;
            }
            
            // Reflect into the object to search its private/protected properties
            $ref = new ReflectionObject($value);
            foreach ($ref->getProperties() as $prop) {
                $prop->setAccessible(true);
                if ($prop->isInitialized($value)) {
                    $search($path . '->' . $prop->getName(), $prop->getValue($value), $depth + 1);
                }
            }
        } elseif (is_array($value)) {
            foreach ($value as $k => $v) {
                if (is_object($v) || is_array($v)) {
                    $search($path . "['" . $k . "']", $v, $depth + 1);
                }
            }
        }
    };

    // 1. Search all global variables
    foreach ($GLOBALS as $k => $v) {
        if ($k === 'GLOBALS') continue; // Skip recursive GLOBALS reference
        $search("\$GLOBALS['$k']", $v);
    }

    // 2. Search all static properties of loaded classes
    foreach (get_declared_classes() as $class) {
        $ref = new ReflectionClass($class);
        foreach ($ref->getProperties(ReflectionProperty::IS_STATIC) as $prop) {
            $prop->setAccessible(true);
            if ($prop->isInitialized()) {
                $search("$class::$" . $prop->getName(), $prop->getValue());
            }
        }
    }
    
    return $results ?: "No instances of {$targetClass} found in current scope.";
});
```
## PHP libraries and components

- [PHP PDO](PHP%20PDO.md): PDO is one of the most commonly used libraries for connecting PHP services to databases like [MySQL](../Services/MySQL.md) and [PostgreSQL](../Services/PostgreSQL.md) and use prepared statements for query them.

## PHP quirks and tricks

- `register_argc_argv` + `$_SERVER['argv']` [^1] 
- FrankenPHP and Unicode Cade-folding [^2]
- [PDO Preparaed Statements](PHP%20PDO.md#PDO%20Preparaed%20Statements) with query emulation = [Novel SQL Injection](../../Clippings/Adam%20Kues%20-%20Novel%20SQL%20Injection%20Technique%20in%20PDO%20Prepared%20Statements.md#Novel%20SQL%20Injection%20Technique%20in%20PDO%20Prepared%20Statements)
- `spl_autoload_register` [^5] and PHP `autoload` [^6] exploitation [^3] [^4] [^7]
- `mt_rand` [^8]


[^1]: [Assetnote Research - How an Obscure PHP Footgun Led to RCE in Craft CMS](../../Readwise/Articles/Assetnote%20Research%20-%20How%20an%20Obscure%20PHP%20Footgun%20Led%20to%20RCE%20in%20Craft%20CMS.md)

[^2]: [FrankenPHP - Unicode Case‑Folding Bug and PHP Sandbox Escape](../../Clippings/FrankenPHP%20-%20Unicode%20Case‑Folding%20Bug%20and%20PHP%20Sandbox%20Escape.md)

[^3]: [You Already Have Our Personal Data, Take Our Phone Calls Too (FreePBX CVE-2025-57819)](../../Raindrop/You%20Already%20Have%20Our%20Personal%20Data,%20Take%20Our%20Phone%20Calls%20Too%20(FreePBX%20CVE-2025-57819).md)

[^4]: [Arseniy Sharoglazov - Exploiting Arbitrary Object Instantiations in PHP Without Custom Classes](../../Readwise/Articles/Arseniy%20Sharoglazov%20-%20Exploiting%20Arbitrary%20Object%20Instantiations%20in%20PHP%20Without%20Custom%20Classes.md)

[^5]: https://www.php.net/manual/en/function.spl-autoload-register.php

[^6]: https://www.php.net/manual/en/language.oop5.autoload.php

[^7]: [PHP Object Injection exploiting `spl_autoload_register`](../Web%20&%20Network%20Hacking/Insecure%20Deserialization%20&%20Object%20Injection.md#PHP%20Object%20Injection%20exploiting%20`spl_autoload_register`)

[^8]: [Hassan Khafaji - Exploiting (GH-13690) mt_rand in php in 2024](../../Clippings/Hassan%20Khafaji%20-%20Exploiting%20(GH-13690)%20mt_rand%20in%20php%20in%202024.md)
