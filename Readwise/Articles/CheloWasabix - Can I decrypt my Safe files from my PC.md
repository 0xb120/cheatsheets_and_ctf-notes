---
author: CheloWasabix
aliases:
  - Can I decrypt my Safe files from my PC
tags:
  - readwise/articles
  - vulnerability-research/todo
url: https://www.reddit.com/r/Huawei/comments/1awnosg/can_i_decrypt_my_safe_files_from_my_pc/?rdt=55695?__readwiseLocation=
date: 2025-04-28
---
# Can I decrypt my Safe files from my PC

![rw-book-cover](https://readwise-assets.s3.amazonaws.com/static/images/article4.6bc1851654a0.png)

## Highlights


Backed up the .File_SafeBox folder to my PC, factory reset my phone, created an empty safe on the newly wiped phone, and just copied and overwrote the current safe with the one I had backed up. Whit that procedure I recovered everything.
 Huawei tech support had literally told me it could not be recovered and that "they could not do anything" what a load of bs. [](https://read.readwise.io/read/01jsxwghs624r038877c1exx3j)



Huawei encrypts the files with AES. The decryption key is held by the msb.db inside your .File_SafeBox and is also encrypted with your password and a salt. The salt is also encrypted. So as long as you have your password and the msb.db a decryption is possible. I‘ve written a tool for it in the past du decrypt the files without need of a smartphone. Another solution is to create an empty safe and replace the created directory. So if you need help send me a DM [](https://read.readwise.io/read/01jsxwhcqdnpqz6vjwpgtj2rnw)

