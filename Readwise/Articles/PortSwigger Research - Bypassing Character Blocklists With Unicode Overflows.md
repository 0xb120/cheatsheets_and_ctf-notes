---
author: PortSwigger Research
aliases: [Bypassing Character Blocklists With Unicode Overflows]
tags: [readwise/articles]
url: https://portswigger.net/research/bypassing-character-blocklists-with-unicode-overflows?__readwiseLocation
date: 2025-02-03
---
# Bypassing Character Blocklists With Unicode Overflows

[Unicode codepoint truncation](https://docs.google.com/presentation/d/1jW0o1YO3FNXlXVkAziM_wSGQqRdLP2kmfoBb6mF1bGY/edit#slide=id.g2f056d28156_1_250) - also called a [Unicode overflow](https://portswigger.net/research/splitting-the-email-atom#unicode-overflows) attack - happens when a server tries to store a #Unicode character in a single byte. Because the maximum value of a byte is 255, **an overflow can be crafted to produce a specific ASCII character**. [](https://read.readwise.io/read/01jk64myk47rb4hpe6y32kdct7)

Here are a couple of examples that end with 0x41 which represents A: `0x4e41 0x4f41 0x5041 0x5141` [](https://read.readwise.io/read/01jk64nfbwm3p17eqyfx9sh9nm)

It's not only bytes that have this problem, JavaScript itself has a codepoint overflow in the `fromCharCode()` method. This method allows you to generate a character between `0`-`0xffff` but if you go above this range it will be overflowed and produce a character by the overflow amount. [](https://read.readwise.io/read/01jk64qb0bempbqp6n4f7fh45m)

```js
String.fromCharCode(0x10000 + 0x31, 0x10000 + 0x33, 0x10000 + 0x33, 0x10000 + 0x37) //1337`
```

This is being actively used by [bug bounty hunters](https://infosecwriteups.com/6000-with-microsoft-hall-of-fame-microsoft-firewall-bypass-crlf-to-xss-microsoft-bug-bounty-8f6615c47922) and was brought to our attention by [Ryan Barnett](https://x.com/ryancbarnett). [](https://read.readwise.io/read/01jk64rkrr64jbkcq2msp2rena)

## Shazzer Unicode Table

We've also updated the [Shazzer unicode table](https://shazzer.co.uk/unicode-table?fromTo=0x41&highlightsFromTo=) to display potential unicode truncation characters. [](https://read.readwise.io/read/01jk64sm547c6n1fwhmr1pfz4s)
> #tools

