---
author: Crisdeo Nuel Siahaan
aliases:
  - Bounty of an Insecure WebView (Part 1)
tags:
  - readwise/articles
  - Android
url: https://infosecwriteups.com/bounty-of-an-insecure-webview-part-1-xss-but-with-steroids-1a41cf654048
date: 2024-08-20
---
# Bounty of an Insecure WebView (Part 1): XSS, but With Steroids

![rw-book-cover](https://miro.medium.com/v2/resize:fit:1200/1*DrXexgcRloxJCFMPqe-C1g.jpeg)

## Highlights


> WebViewActivity (or any activity that passes a URI to WebViewActivity) **could** be dangerous if it’s being exported, especially if the URL is not being validated.
> [View Highlight](https://read.readwise.io/read/01hc8h7x63xq7z6a7358yha6vf)



> However, we can’t use deeplink method to launch the webview like before due to the intent filter is not being defined in the AndroidManifest.
> [View Highlight](https://read.readwise.io/read/01hc8hd5rg2rzeb14w8gdrks32)



> Quoted from [oversecured.com](https://blog.oversecured.com/Android-security-checklist-webview/#universal-xss): `onNewIntent` is a method that is called each time the activity receives a new Intent. Since `onNewIntent` also call “**parseIntent()**” and load a new URL, we could smuggle JavaScript code to be executed on the WebView.
> [View Highlight](https://read.readwise.io/read/01hc8hkyz5c46jjbtddazdtq8z)

