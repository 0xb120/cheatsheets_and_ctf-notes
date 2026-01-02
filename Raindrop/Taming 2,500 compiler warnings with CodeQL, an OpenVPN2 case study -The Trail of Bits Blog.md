---
raindrop_id: 1358480146
raindrop_highlights:
  68d69212f9a75cbba1147a82: f8efe984e0f7ddd3181d53dfee60ef52
  68d69219f9a75cbba1147b6c: 8a7339024f3350ec55e1963a898f2964
  68d6922cb3703b68c1452e89: 697195f8698fc6104fe662951affc70d
  68d69252a2be5671f237d1e2: d15b6418d0eb280e898926dc527f4c91
  68d692665430135b7f9a4667: fc87db1ff5f90364e5e0e9be7cb7b30c
  68d69317c09d890017841348: 157dcf75e45e6c375c220f9e76b017db
  68d694673b739d334336f227: bd30a6e532d8ebe63517f394ee91bbea
title: "Taming 2,500 compiler warnings with CodeQL, an OpenVPN2 case study -The Trail of Bits Blog"

description: |-
  We created a CodeQL query that reduced 2,500+ compiler warnings about implicit conversions in OpenVPN2 to just 20 high-priority cases, demonstrating how to effectively identify potentially dangerous type conversions in C code.

source: https://blog.trailofbits.com/2025/09/25/taming-2500-compiler-warnings-with-codeql-an-openvpn2-case-study/

created: 1758835729021
type: article
tags: ["_index"]

 
  - "tech-blog" 
  - "vuln-research-blog" 
  - "SAST"

---
# Taming 2,500 compiler warnings with CodeQL, an OpenVPN2 case study -The Trail of Bits Blog

![](https://blog.trailofbits.com/img/tob.png)

> [!summary]
> We created a CodeQL query that reduced 2,500+ compiler warnings about implicit conversions in OpenVPN2 to just 20 high-priority cases, demonstrating how to effectively identify potentially dangerous type conversions in C code.





When conversions matter for security
C’s relaxed type system allows for implicit conversions, which is when the compiler automatically changes the type of a variable to make code compile. Not all conversions are problematic, but this behavior creates space for vulnerabilities. One problematic case is when the result of the conversion is used to alter data.
we have broken it down into three categories: truncation, reinterpretation, and widening.
unsigned int x = 0x80000000; unsigned char a = x; // truncation int b = x; // reinterpretation uint64_t c = b; // widening
The examples above were all altered via the same type of conversion: conversion as if by assignment. There are two other types of conversions that C programmers often encounter.
Usual arithmetic conversion occurs when variables of different types are operated on and reconciled:

unsigned short header_size = 0x13;

int offset = 0x37;

return header_size + offset;  // usual arithmetic conversion


Integer promotions happen when unary bitwise, arithmetic, or shift operations happen on a single variable:

uint8_t val = 0x13;

int val2 = (~val) >> 3;  // integer promotion


By combining the conversion types with the data alteration types mentioned above, we can create a table to clarify which implicit conversions we should further analyze for possible security issues.

	Truncation	Reinterpretation	Widening
As if by assignment	Possible	Possible	Possible
Integer promotions	Not possible	Not possible	Possible
Usual arithmetic conversions	Not possible	Possible	Possible
During our security review of OpenVPN2, we faced a daunting challenge: which of the about 2,500 implicit conversions compiler warnings could actually lead to a vulnerability? To answer this, we created a new CodeQL query that reduced the number of flagged implicit conversions to just 20. Here is how we built the query, what we learned, and how you can run the queries on your code. Our query is available on GitHub, and you can dig deeper into the details in our full case study paper.