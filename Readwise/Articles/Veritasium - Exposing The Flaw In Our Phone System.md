---
author: "Veritasium"
aliases: "Exposing The Flaw In Our Phone System"
tags: RW_inbox, readwise/articles
url: https://www.youtube.com/watch?v=wVyu7NB7W6Y?__readwiseLocation=
date: 2025-04-24
summary: Linus Tech Tips demonstrated how easy it is to hack phone networks using SS7, allowing them to intercept calls and messages. This vulnerability has been exploited by criminals to steal sensitive information, like two-factor authentication codes. Despite advancements in technology, SS7 remains widely used, making it difficult to fully secure phone communications.
---
# Exposing The Flaw In Our Phone System

![rw-book-cover](https://i.ytimg.com/vi/wVyu7NB7W6Y/maxresdefault.jpg)

## Highlights


First you have to infiltrate SS7, second gain trust and third attack. [](https://read.readwise.io/read/01j8myy5xn6z33xpye1m5jq4k3)



Roaming is one of the main use cases of SS7. [](https://read.readwise.io/read/01j8myymrty2mrwpw53qfv8mqs)



Say Derek, you visit me over here. Your phone would try to connect to a network that's foreign and that network would then have to reach out to your home network in Australia asking, is this a valid customer? Are you willing to pay for the charges that they'll incur on my network? And all of that information is exchanged over SS7. - For this to work, telcos need to communicate with each other.
 So the way they do that is by making sure they're part of the same club. The way they share membership to this club is by using unique addresses to identify where requests are coming from. - SS7 is a global network, just like the internet and like on the internet you need some addressing scheme. So you need some way of saying this is me and this is you. And on the internet we use IP addresses. On SS7 we use what's called Global Titles, GTs. [](https://read.readwise.io/read/01j8mz05n2gecvtdtmxm361b0t)



Telcos generally accept messages only from Global Titles with which they have agreements. And the whole system is designed to be a closed network with few barriers once inside, this is known as the walled garden approach. [](https://read.readwise.io/read/01j8mz0k78rhbcb6z48fk0a28c)



There are so many more players in the garden that not all
 of them are trustworthy. - Those companies, some of them sell services onto third parties, some of them can be bribed, some of them can be hacked. So there's probably thousands of ways into SS7 at reasonable effort or cost. [](https://read.readwise.io/read/01j8mz1k84d4s514s0dar8yxpa)



Now you need something from the SIM card. The real key in a mobile network is a unique 15 digit identifier which belongs exclusively to the SIM card on the phone. It's called an international mobile subscriber identity or IMSI for short. And it is very important. - Basically to be able to collect the IMSI from a subscriber, we would launch some of the messages such as send routing info or send routing info for SM. [](https://read.readwise.io/read/01j8mz2vwe7qmzy0weej11761k)

