---
title: The Ultimate Double-Clickjacking PoC | Jorian Woltjer
source: https://jorianwoltjer.com/blog/p/research/ultimate-doubleclickjacking-poc
author: jorianwoltjer.com
published: 
created: 2025-07-29
description: "Combing a lot of browser tricks to create a realistic Proof of Concept for the Double-Clickjacking attack. Moving a real popunder with your mouse cursor and triggering it right as you're trying to beat your Flappy Bird high score."
tags: [clippings/articles, _inbox]
---
# The Ultimate Double-Clickjacking PoC | Jorian Woltjer

![](https://jorianwoltjer.com/img/blog/ultimatepoc.png)

> [!summary]
> This webpage is a blog post by Jorian Woltjer detailing a sophisticated "Double-Clickjacking" proof-of-concept (PoC) exploit. The attack involves tricking a user into clicking on a target button within a popup window that's moved dynamically under the user's cursor while they are engaged in another activity (like playing a game). The PoC combines techniques like moving popups, using a fake Cloudflare captcha, exploiting window references across navigations, and leveraging a popunder technique. The author explains how to create a convincing attack scenario and discusses potential mitigations.

[Paulos Yibelo](https://www.paulosyibelo.com/2024/12/doubleclickjacking-what.html) came with a fun showcase of a technique dubbed "Double-Clickjacking". I later [expanded on that](https://jorianwoltjer.com/blog/p/hacking/pressing-buttons-with-popups) with some more vulnerable targets, and this is the 2nd and likely last post I'll write about it.

Working on my 1st blog post and the people talking about it later gave me an idea: make the ultimate double-clickjacking proof of concept.

Below is how I could take over someone's Gitlab account by them playing Flappy Bird:

<video controls="" src="/img/blog/ultimatepoc.mp4" style="height: 360px; width: auto;"></video>

Below is a link to the source code if you want to jump ahead:

[https://github.com/JorianWoltjer/popup-research/tree/main/ultimate-poc/gitlab](https://github.com/JorianWoltjer/popup-research/tree/main/ultimate-poc/gitlab)

[Idea: Moving popup in background](https://jorianwoltjer.com/blog/p/research/#idea-moving-popup-in-background)

As you can see in the video, it doesn't rely on holding space anymore like my last writeup, not even double-clicking specifically. Instead, it relies on a user repeatedly clicking on any position of your site, while still not requiring any iframes.  
The TLDR of how this works is: **The target page is opened in a popup under the attacker's page, which moves with the user's cursor until the time is right, and it comes into focus right under the user who's about to click**.