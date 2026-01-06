---
author: Pierluigi Paganini
aliases:
  - DoubleClickjacking Allows Clickjacking on Major Websites
tags:
  - readwise/articles
url: https://securityaffairs.com/172572/hacking/doubleclickjacking-clickjacking-on-major-websites.html?__readwiseLocation=
created: 2025-04-24
---
# DoubleClickjacking Allows Clickjacking on Major Websites

[DoubleClickjacking](Blog%20-%20DoubleClickjacking%20A%20New%20Era%20of%20UI%20Redressing.md) is a technique that allows attackers to bypass protections on major websites by leveraging a [double-click sequence](https://en.wikipedia.org/wiki/Double-click). [](https://read.readwise.io/read/01jgp3se6hyat1cz5rz3qvp296)

DoubleClickjacking, exploiting double-click sequences, bypasses clickjacking protections like `X-Frame-Options` and `SameSite` cookies, potentially allowing platform account takeovers. [](https://read.readwise.io/read/01jgp3swyrypgq5qbgfvrkgmqq)

> “**DoubleClickjacking** is a new variation on this classic theme: instead of relying on a single click, it takes advantage of a double-click sequence. While it might sound like a small change, it opens the door to new UI manipulation attacks that bypass all known clickjacking protections, including the X-Frame-Options header or a SameSite: Lax/Strict cookie.” [Paulos Yibelo wrote](https://www.paulosyibelo.com/2024/12/doubleclickjacking-what.html). “This technique seemingly affects almost every website, leading to account takeovers on many major platforms.” [](https://read.readwise.io/read/01jgp3tkc8zpr86mfbmqdy6qqm)

DoubleClickjacking exploits timing differences between mousedown and onclick events to hijack user actions. By swiftly swapping windows during a double-click, attackers redirect clicks to sensitive targets, like OAuth prompts, without relying on popunder tricks. [](https://read.readwise.io/read/01jgp3tvr0s6bb89rgj1r4bv5q)

- An attacker starts by opening a new window through a button or automatically on a webpage.
 - Clicking the button opens a new window prompting a double-click, while the parent window is redirected to the target page (e.g., OAuth authorization).
 - The double-click closes the top window and unintentionally triggers authorization on the parent window, granting the attacker access with arbitrary scope.
 [![DoubleClickjacking](https://i0.wp.com/securityaffairs.com/wp-content/uploads/2025/01/image-1.png?resize=1024%2C614&ssl=1)](https://i0.wp.com/securityaffairs.com/wp-content/uploads/2025/01/image-1.png?ssl=1) [](https://read.readwise.io/read/01jgp3vk6y3sc10agmj85b0qgt)

