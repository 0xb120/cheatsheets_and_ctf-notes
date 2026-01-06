---
author: News ≈ Packet Storm
aliases:
  - Using YouTube to Steal Your Files
tags:
  - readwise/articles
url: https://packetstormsecurity.com/news/view/36373/Using-YouTube-To-Steal-Your-Files.html
created: 2024-09-25
---
# Using YouTube to Steal Your Files

![rw-book-cover](https://ssl.google-analytics.com/__utm.gif?utmwv=1.3&utmn=1377320288&utmcs=ISO-8859-1&utmsr=31337x31337&utmsc=32-bit&utmul=en-us&utmje=0&utmfl=-&utmcn=1&utmdt=News%u2248%20Packet%20Storm&utmhn=packetstormsecurity.com&utmr=-&utmp=%2Fnews%2F&utmac=UA-18885198-1&utmcc=__utma%3D32867617.1377320288.1657598775.1657598775.1657598775.1%3B%2B__utmz%3D32867617.1657598775.1.1.utmccn%3D(direct)%7Cutmcsr%3D(direct)%7Cutmcmd%3D(none))

## Highlights


Google Slides has this neat feature that lets you add YouTube videos to your presentations.
> [View Highlight](https://read.readwise.io/read/01j8mz4aaxv3m0keqe399k740m)



What appears is an iframe that links to [www.youtube.com/embed/{VIDEOID}](https://www.youtube.com/embed/%7bVIDEOID%7d)
> [View Highlight](https://read.readwise.io/read/01j8mz4jgr9dbp02qybwv1tmyw)



Looking at the network traffic, it seems like adding a video onto a slide will send Slides the videoid, which it then uses to construct the embed URL for the iframe. We can’t control the full URL, just the videoid part. Can we still do something?
 The obvious thing to try here is path traversal - if we change the videoid to **../**, the full url will be [www.youtube.com/embed/../](https://www.youtube.com/embed/../), which should turn into just [www.youtube.com/](https://www.youtube.com/), leading us straight to the YouTube home page.
> [View Highlight](https://read.readwise.io/read/01j8mz5hz21csxqjkkjmfrqgpy)



as we’re stuck in an iframe on **youtube.com**, an open redirect would be pretty lovely.
> [View Highlight](https://read.readwise.io/read/01j8mz6bmdtw0hxr22zcec35xe)



obvious place to look for open redirects is usually the authentication flow of a website - generally sites want to return you to the same page you were on before logging in. It’s no different for YouTube, logging into a Google account takes you back to the page you were originally on. This is achieved through the **/signin** endpoint
> [View Highlight](https://read.readwise.io/read/01j8mz7ftdqee5396bpaqampkf)



That **/signin** redirect wasn’t the only one I found though - there was another one present on a different YouTube subdomain:
 This one seems to be for Google account logins. For example, if you log in on **google.ee**, you’d get redirected through **accounts.google.com** and **accounts.youtube.com** to update the cookies on both of those domains. I played around with it a little and found that while it once again wasn’t a full open redirect, it did allow a variety of Google’s own domains
> [View Highlight](https://read.readwise.io/read/01j8mz8xgfbc192y7633t2rr26)



Let’s try chaining our previous path-traversed **/signin** redirect to the new **accounts.youtube.com** one and see if we can make it embed Docs pages within itself.
 And meow - Docs inside Docs!! So epic!
> [View Highlight](https://read.readwise.io/read/01j8mz9ch4fy0fgs1byjgy3e96)



So we have Docs inside of Docs, which is incredibly fun for a few minutes, but can we actually do anything useful with this?
> [View Highlight](https://read.readwise.io/read/01j8mz9pk6g9err132qrw1wk80)



Going through link after link, I eventually stumbled upon this url: docs.google.com/file/d/{ID}/edit. This page lets us preview and perform actions (such as sharing) on Google Drive files, and unlike the other links I found earlier, it stays on the **docs.google.com** domain instead of redirecting to Drive. And not only does it work with Drive files, it also works with folders and other such entities
> [View Highlight](https://read.readwise.io/read/01j8mzapt6ntyjt3yva7536vz4)



The page has a share button that stays enabled even within an iframe. If we can trick someone into clicking the Share button, typing in our e-mail, and changing the permissions on some important folder, we’ll gain access to it.
> [View Highlight](https://read.readwise.io/read/01j8mzbb4tw9qyqx2ggkq8pwmr)



Thinking of ways to improve the attack, I remembered the feature in Drive that lets you request access to other people’s documents.
> [View Highlight](https://read.readwise.io/read/01j8mzbj23m9n7fws3139d1kap)



The button in that e-mail links to [https://drive.google.com/drive/folders/{ID}?usp=sharing_esp&userstoinvite=lyra.horse@gmail.com&sharingaction=manageaccess&role=writer&ts=66e724ba](https://drive.google.com/drive/folders/%7bID%7d?usp=sharing_esp&userstoinvite=lyra.horse@gmail.com&sharingaction=manageaccess&role=writer&ts=66e724ba) , which when opened, pops up the Share dialog with a notification of the request.
> [View Highlight](https://read.readwise.io/read/01j8mzbzmexkhah1j5tnax4tjz)



I pulled out my DevTools and began digging through the JavaScript of the page to see how the query parameters are handled. As a simple test, I started off with just the *userstoinvite* query parameter.
 And wow!? I had accidentally stumbled upon the perfect share dialog URL. For some reason, leaving out all the other query parameters makes the share dialog just auto-fill the e-mail field from the query parameter
> [View Highlight](https://read.readwise.io/read/01j8mzd6c0xvgmndwj42w7d25k)



I began putting the attack together, combining all the cool tricks we’ve come up with so far.
 1. We first take cool little docs invite url. 
 https://docs.google.com/file/d/1sHy3aQXsIlnOCj-mBFxQ0ZXm4TzjjfFL/edit?userstoinvite=lyra.horse@gmail.com
 2. Then we put it inside the **accounts.youtube.com** redirect. 
 https://accounts.youtube.com/accounts/SetSID?continue=https%3A%2F%2Fdocs.google.com%2Ffile%2Fd%2F1sHy3aQXsIlnOCj-mBFxQ0ZXm4TzjjfFL%2Fedit%3Fuserstoinvite%3Dlyra.horse%40gmail.com
 3. Then we put *that* into the **youtube.com/signin** redirect. 
 https://www.youtube.com/signin?next=https%3A%2F%2Faccounts.youtube.com%2Faccounts%2FSetSID%3Fcontinue%3Dhttps%3A%2F%2Fdocs.google.com%252Ffile%252Fd%252F1sHy3aQXsIlnOCj-mBFxQ0ZXm4TzjjfFL%252Fedit%253Fuserstoinvite%253Dlyra.horse%2540gmail.com
 4. And finally, we turn it into a path traversed "videoid" we can embed in our slides. 
 ../signin?next=https%3A%2F%2Faccounts.youtube.com%2Faccounts%2FSetSID%3Fcontinue%3Dhttps%3A%2F%2Fdocs.google.com%252Fa%252Fa%252Ffile%252Fd%252F1sHy3aQXsIlnOCj-mBFxQ0ZXm4TzjjfFL%252Fedit%253Fuserstoinvite%253Dlyra.horse%2540gmail.com
> [View Highlight](https://read.readwise.io/read/01j8mzdhe2e2k5bdzg52sbymr5)



It seems like Docs has some sort of a mitigation in place that prevents me from using a cross-site redirect for the file page within an iframe. More precisely, it checks for the *Sec-Fetch-Dest* and and *Sec-Fetch-Site* headers, and if they’re both set to *iframe* and *cross-site* respectively, we get a 403 back.
> [View Highlight](https://read.readwise.io/read/01j8mzf3f1egbewfeky44cf56g)



To bypass *that*, we need to perform a same-origin redirect inside of the iframe.
> [View Highlight](https://read.readwise.io/read/01j8mzfbp0e5xyzqs0v5ah36vq)



To put it simply, we’re currently doing:
 accounts.youtube.com (cross-site) → docs.google.com/file/d/…/edit (403)
 so to bypass that, we want to chain a redirect like this:
 accounts.youtube.com (cross-site) → docs.google.com/??? (same-origin) → docs.google.com/file/d/…/edit (200)
 and it should work! But we have to find something that’d work for that part in the middle.
> [View Highlight](https://read.readwise.io/read/01j8mzfr4pg2tpvqkedhcxfe35)



It seems like there’s an old legacy GSuite URL format of **docs.google.com/a/<domain>/…**, which probably did something useful years ago (edit: and still does[6](https://lyra.horse/blog/2024/09/using-youtube-to-steal-your-files/#fn:6)), but these days just disappears when you open an URL. If you’re logged out, you must find some working donor URL to use, such as **/a/wyo.gov/**[7](https://lyra.horse/blog/2024/09/using-youtube-to-steal-your-files/#fn:7), but logged in you can even do **/a/a/** and it’ll just work.
> [View Highlight](https://read.readwise.io/read/01j8mzgs4pr7vz8kzdknar3hgn)



With that figured out, let’s throw the **/a/a/** thing into our “videoid” from earlier: ../signin?next=https%3A%2F%2Faccounts.youtube.com%2Faccounts%2FSetSID%3Fcontinue%3Dhttps%3A%2F%2Fdocs.google.com%252Ffile%252Fd%252F1sHy3aQXsIlnOCj-mBFxQ0ZXm4TzjjfFL%252Fedit%253Fuserstoinvite%253Dlyra.horse%2540gmail.com
> [View Highlight](https://read.readwise.io/read/01j8mzgmhpx6bpad7de7wtnzwe)

