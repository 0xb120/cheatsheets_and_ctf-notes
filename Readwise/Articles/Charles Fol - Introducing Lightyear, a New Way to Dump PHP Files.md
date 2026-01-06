---
author: Charles Fol
aliases:
  - lightyear
tags:
  - readwise/articles
url: https://blog.lexfo.fr/lightyear-file-dump.html?__readwiseLocation=
created: 2025-05-08
---
# Introducing Lightyear, a New Way to Dump PHP Files

![rw-book-cover](https://blog.lexfo.fr/theme/images/favicon.ico)

[PHP wrappers and filters](../../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/File%20Inclusion%20(LFI%20&%20RFI).md#PHP%20wrappers%20and%20filters) chains are, in my opinion, an amazing research subject, as they seem to offer an infinite, almost ungraspable number of possibilities to an attacker. 

Can we use filters to make [an unknown file look like an image](https://gynvael.coldwind.pl/?lang=en&id=671)? Yes! 
Can we use them to [add a prefix and suffix to a file](Charles%20Fol%20-%20Introducing%20Wrapwrap%20Using%20PHP%20Filters%20to%20Wrap%20a%20File%20With%20a%20Prefix%20and%20Suffix.md)? Yes. 
Can we use them to [dump a file, byte per byte, by leveraging memory exhaustion](https://www.synacktiv.com/en/publications/php-filter-chains-file-read-from-error-based-oracle)? Yes as well. [](https://read.readwise.io/read/01jtqjfk1p3hz7rdx2wjf29dtf)

I got new ideas regarding filters, which I was able to use to create [**lightyear**](https://github.com/ambionics/lightyear). This new tool uses a new algorithm **to dump files using an error-based oracle**, making it faster than the already existing implementations. [](https://read.readwise.io/read/01jtqjhq4mbp2547x62x02fhsf)
> #tools 

The tool is the current state-of-the-art to dump the contents of a PHP file using a **blind file read primitive**!
- can dump files of **tens of thousands** of bytes using **small payloads** (a few thousand characters);
- determines the value of each byte **using dichotomy** (6 requests), which is the most efficient;
- can greatly **speed up request time** by refraining from making PHP run out of memory;
- does not produce any unwanted PHP warnings or errors.