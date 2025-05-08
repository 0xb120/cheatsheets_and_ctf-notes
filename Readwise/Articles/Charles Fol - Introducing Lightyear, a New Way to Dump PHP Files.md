---
author: Charles Fol
aliases:
  - lightyear
tags:
  - readwise/articles
url: https://blog.lexfo.fr/lightyear-file-dump.html?__readwiseLocation=
date: 2025-05-08
summary: The article introduces a new technique for efficiently extracting specific digits from base64-encoded PHP files using filters and dechunking. By changing the character set, the method allows for the creation of an alternative base64 digit set, where a newline can replace a digit. This approach enables quick access to digits deep within a file, improving the performance of the extraction process.
---
# Introducing Lightyear, a New Way to Dump PHP Files

![rw-book-cover](https://blog.lexfo.fr/theme/images/favicon.ico)

[PHP wrappers and filters](../../Dev,%20ICT%20&%20Cybersec/Web%20&%20Network%20Hacking/File%20Inclusion%20(LFI%20&%20RFI).md#PHP%20wrappers%20and%20filters) chains are, in my opinion, an amazing research subject, as they seem to offer an infinite, almost ungraspable number of possibilities to an attacker. 

Can we use filters to make [an unknown file look like an image](https://gynvael.coldwind.pl/?lang=en&id=671)? Yes! 
Can we use them to [add a prefix and suffix to a file](https://www.ambionics.io/blog/wrapwrap-php-filters-suffix)? Yes. 
Can we use them to [dump a file, byte per byte, by leveraging memory exhaustion](https://www.synacktiv.com/en/publications/php-filter-chains-file-read-from-error-based-oracle)? Yes as well. [](https://read.readwise.io/read/01jtqjfk1p3hz7rdx2wjf29dtf)

I got new ideas regarding filters, which I was able to use to create [**lightyear**](https://github.com/ambionics/lightyear). This new tool uses a new algorithm **to dump files using an error-based oracle**, making it faster than the already existing implementations. [](https://read.readwise.io/read/01jtqjhq4mbp2547x62x02fhsf)
> #tools 
