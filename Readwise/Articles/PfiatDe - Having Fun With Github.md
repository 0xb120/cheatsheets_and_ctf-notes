---
author: "PfiatDe"
aliases: "Having Fun With Github"
tags: RW_inbox, readwise/articles
url: https://badoption.eu/blog/2025/04/25/github.html?__readwiseLocation=
date: 2025-04-30
summary: Github has security weaknesses that allow users to spoof commit authors and contributors. This can be used for malicious activities like phishing and tampering. Despite improvements, some vulnerabilities still exist, making it easy to create misleading content on the platform.
---
# Having Fun With Github

![rw-book-cover](https://readwise-assets.s3.amazonaws.com/static/images/article2.74d541386bbf.png)

## Highlights


Github has some holes in their basic security which allow some tampering and spoofing. [](https://read.readwise.io/read/01jt3a71k0bbqa516qjctc0fk4)



TL;DR: On Github it is possible to:
 • Spoof Commit authors
 • Spoof Contributors
 • Host hidden payloads under github.com (mostly solved)
 • Use Issues for phishing [](https://read.readwise.io/read/01jt3a7tyh1hp472tphxd2jm7r)



Spoof Commit Authors
 We can spoof users on Github as long as we know the username and the email. [](https://read.readwise.io/read/01jt3a97k8gq7m7y659k9cwa2z)



Open the commit details for a real commit, not a **merge pull request** like [https://github.com/ashtom/hkimport/commit/e2a386c0f2fe4b8ec939eb132b5532c0396c9133](https://github.com/ashtom/hkimport/commit/e2a386c0f2fe4b8ec939eb132b5532c0396c9133).
 Add a `.patch` to the URL to see the real git patch message: 
 [https://github.com/ashtom/hkimport/commit/e2a386c0f2fe4b8ec939eb132b5532c0396c9133.patch](https://github.com/ashtom/hkimport/commit/e2a386c0f2fe4b8ec939eb132b5532c0396c9133.patch) [](https://read.readwise.io/read/01jt3aa9r63wrwfeppabsfpey6)



Here we have our needed **user.name** `Thomas Dohmke` and the **user.email** `thomas@dohmke.de`. [](https://read.readwise.io/read/01jt3aayaf62xfg7rb3rhrjzxd)



Spoof some commits
 Create a new repository and check it out via the git CLI.
 > You will need a Github Token for that, which you can generate under Developer Settings.
 Now we make some nice commits and push them with another user.name.
 user@localhost ~/must-be-legit (main)> git config user.name Thomas Dohmke
 user@localhost ~/must-be-legit (main)> git config user.email thomas@dohmke.de
 user@localhost ~/must-be-legit (main)> code -n .
 user@localhost ~/must-be-legit (main)> git add *
 user@localhost ~/must-be-legit (main)> git commit -m "Hello from Thomas"
 [main (root-commit) 9d6c889] Hello from Thomas
 2 files changed, 1 insertion(+)
 create mode 100644 README.MD
 create mode 100644 image.png
 user@localhost ~/must-be-legit (main)> git push
 Username for 'https://github.com': <<myemail>>
 Password for 'https://<<myemail>>@github.com': 
 Enumerating objects: 4, done.
 Counting objects: 100% (4/4), done.
 Delta compression using up to 4 threads
 Compressing objects: 100% (3/3), done.
 Writing objects: 100% (4/4), 308.67 KiB | 17.15 MiB/s, done.
 Total 4 (delta 0), reused 0 (delta 0), pack-reused 0
 To https://github.com/PfiatDe/must-be-legit.git
 * [new branch] main -> main [](https://read.readwise.io/read/01jt3ac88tz1n8efz2qbja9nhz)



However, there is still some discrepancy, which you might have noticed, the contributor did not show up on the repository main page. [](https://read.readwise.io/read/01jt3aeafk09tp0h6xxenwy7zz)



Actually, that’s quite easy. Looking at: [Why are my contributions not showing up on my profile?](https://docs.github.com/en/account-and-profile/setting-up-and-managing-your-github-profile/managing-contribution-settings-on-your-profile/why-are-my-contributions-not-showing-up-on-my-profile) we find several actions which might add the contributors.
 We are going for a Pull Request, as we cannot create issues directly in the git CLI. [](https://read.readwise.io/read/01jt3afmp22jds8tv1j24b95be)



I was playing around with this when suddenly I realized it is way simpler. Just set the repository to `public` before you commit & push and you get your contributors badge. [](https://read.readwise.io/read/01jt3afss76dtsn3k3w35km2w4)



Unverified commit status
 If we look at the commit history, there is a status showing if the commit could be verified and bound to the user. [](https://read.readwise.io/read/01jt3ah9n6hfn8zfa1qt6nt8vv)



For some spoofed users, we see an unpleasant `Unverified` status. 
 This happens when the user has the vigilant mode enabled ([Learn about vigilant mode](https://docs.github.com/github/authenticating-to-github/displaying-verification-statuses-for-all-of-your-commits)) and commits must be signed. [](https://read.readwise.io/read/01jt3ahp9evhdqq0b6p29hmf4h)



Luckily we can use another feature of Github, [`Co-authored-by`](https://docs.github.com/en/pull-requests/committing-changes-to-your-project/creating-and-editing-commits/creating-a-commit-with-multiple-authors) in that case. [](https://read.readwise.io/read/01jt3ajd2ctr82ag7k09xqmd9n)



where one could be verified and therefore it is `partially verified`. [](https://read.readwise.io/read/01jt3akbyyd04975at36dsg2p6)



Host payloads under other repos [](https://read.readwise.io/read/01jt3amtv4az2mdafb8wdwgnbs)



it was possible to host payloads in issues for other repositories. You can check e.g. the following report for details:
 [https://research.openanalysis.net/github/lua/2024/03/03/lua-malware.html](https://research.openanalysis.net/github/lua/2024/03/03/lua-malware.html) [](https://read.readwise.io/read/01jt3an8bemsm1nhwv6wxj4kaf)



After some malicious campaigns using this, Github finally fixed it after several months and now temporary uploads are hosted under some user folder now: 
 `[copilot-language-server.zip](https://github.com/user-attachments/files/19891659/copilot-language-server-darwin-arm64-1.306.0.zip)`. 
 So, this still works, but isn’t that great anymore. [](https://read.readwise.io/read/01jt3anzc2v0k2dk632av0fq7x)



Host under Tickets
 Another approach is to host the file at a ticket.
 Go to: `Account` –> `Github Support` –> `My Tickets` –> `New Ticket` –> `open a support ticket` to create a new service ticket.
 Upload your file and it will be available under a Zendesk URL `https://github.zendesk.com/attachments`.
 For example:
 [https://github.zendesk.com/attachments/token/jwxXlXEbwo0IJFzrhy1OVPkEZ/?name=copilot-language-server-darwin-arm64-1.306.0.zip](https://github.zendesk.com/attachments/token/jwxXlXEbwo0IJFzrhy1OVPkEZ/?name=copilot-language-server-darwin-arm64-1.306.0.zip) [](https://read.readwise.io/read/01jt3ar22gdva00kfwrmmfjhcj)



Use issues for phishing
 If we drop somebody an issue in their repository, typically an e-mail will be sent to the owner. [](https://read.readwise.io/read/01jt3arhsjgw4fhb5ca980ssvz)



Somebody might choose a nice Github name and set the display name and a fitting profile picture.
 The display name of the profile will be the from field, which is … *interesting*. [](https://read.readwise.io/read/01jt3aseeqj13ddcmgana49ny2)



The HTML which will be sent via e-mail notification is quite limited, however, it might be possible to craft something. [](https://read.readwise.io/read/01jt3av4emgpnz1fyqmhmzj190)



e.g. here: [https://www.bleepingcomputer.com/news/security/fake-security-alert-issues-on-github-use-oauth-app-to-hijack-accounts/](https://www.bleepingcomputer.com/news/security/fake-security-alert-issues-on-github-use-oauth-app-to-hijack-accounts/) [](https://read.readwise.io/read/01jt3av9ha3vr4xaxbyr28wvzh)



This is how some example might look like: [](https://read.readwise.io/read/01jt3avsbs57aknrb2f3e0za9c)



We can edit the issue after a few seconds. [](https://read.readwise.io/read/01jt3awcvytpyz8cy5xxd1appy)



And then delete the history: [](https://read.readwise.io/read/01jt3awfb49pdk2qge9gcng4t8)

