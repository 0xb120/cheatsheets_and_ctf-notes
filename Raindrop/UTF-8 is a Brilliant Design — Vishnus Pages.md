---
raindrop_id: 1340419984
raindrop_highlights:
  68d2a72f6ce16004f42999f7: 423a6f3dea59a4d7bf512ac1d572ff85
  68d2a84823c4900726812027: 03dfb0387a6edd0988b567dc4fef67a9
  68d2a86ea2be5671f2804a6e: d00d78ceed683f15949569303995c959
  68d2a8c615d4063d77b54778: 4525e780a9813c061f3a1602dc69d9d9
  68d2a8de91c768ef7ec2e3ed: 4f5a42671d762ff25ea6eec0576297fe
  68d2a8ed91c768ef7ec2e699: 127d9cf6a445f428e5ed815c4fce5f2c
  68d2a8ff3640ed9ab6fc5336: 1e76a89b94b3be4d709943c338969e51
title: "UTF-8 is a Brilliant Design — Vishnu's Pages"

description: |-
  Exploring the brilliant design of UTF-8 encoding system that represents millions of characters while being backward compatible with ASCII

source: https://iamvishnu.com/posts/utf8-is-brilliant-design

created: 1757868270349
type: link
tags:
  - "_index"

 
  - "tech-blog" 
  - "Tools"

---
# UTF-8 is a Brilliant Design — Vishnu's Pages

![](https://iamvishnu.com/images/utf8/thumbnail.png)

> [!summary]
> Exploring the brilliant design of UTF-8 encoding system that represents millions of characters while being backward compatible with ASCII





UTF-8 Playground

When I was exploring the UTF-8 encoding, I couldn't find any good tool to interactively visualize how UTF-8 encoding works. So I built UTF-8 Playground to visualize and play around with UTF-8 encoding. Give it a try!.
UTF-8 is a Brilliant Design
Basically UTF-8 uses 32 bits and the old ASCII uses 7 bits, but UTF-8 is designed in such a way that:

Every ASCII encoded file is a valid UTF-8 file.
Every UTF-8 encoded file that has only ASCII characters is a valid ASCII file.
How Does UTF-8 Do It?
UTF-8 is a variable-width character encoding designed to represent every character in the Unicode character set, encompassing characters from most of the world's writing systems.

It encodes characters using one to four bytes.
The first 128 characters (U+0000 to U+007F) are encoded with a single byte, ensuring backward compatibility with ASCII
Other characters require two, three, or four bytes. The leading bits of the first byte determine the total number of bytes that represents the current character. These bits follow one of four specific patterns, which indicate how many continuation bytes follow.