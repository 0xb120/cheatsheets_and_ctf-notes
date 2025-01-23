---
author: Ram Gall
aliases:
  - "PSA: Critical POP Chain Allowing Remote Code Execution Patched in WordPress 6.4.2"
  - POP Chain in WordPress 6.4.2
tags:
  - readwise/articles
  - wordpress
url: https://www.wordfence.com/blog/2023/12/psa-critical-pop-chain-allowing-remote-code-execution-patched-in-wordpress-6-4-2/
date: 2024-08-20
---
# PSA: Critical POP Chain Allowing Remote Code Execution Patched in WordPress 6.4.2

![rw-book-cover](https://www.wordfence.com/wp-content/uploads/2023/12/PSA-Critical-POP-Chain-Allowing-Remote-Code-Execution-Patched-in-WordPress-6.4.2.png)

## Highlights


> the primary reason most Object Injection vulnerabilities are difficult to exploit is the **lack of useful POP chains**.
> [View Highlight](https://read.readwise.io/read/01hhg1ntqj7m4wtnt4h4zfy1hk)



> The problem here resides in the `WP_HTML_Token` class, which was introduced in WordPress 6.4 and is used to improve HTML parsing in the block editor. It includes a `__destruct` magic method that is automatically executed after PHP has processed the request. This `__destruct` method uses `call_user_func` to execute the function passed in through the `on_destroy` property, accepting the `bookmark_name` property as an argument:

```php
public function __destruct() {
    if ( is_callable( $this->on_destroy ) ) {
        call_user_func( $this->on_destroy, $this->bookmark_name );
        }
}
```

> [View Highlight](https://read.readwise.io/read/01hhg1qxxy18xnz6z9dkfa7nc7)

