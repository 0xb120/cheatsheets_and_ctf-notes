---
author: Paul Gerste
aliases:
  - Remote Code Execution in Tutanota Desktop Due to Code Flaw
tags:
  - readwise/articles
url: https://www.sonarsource.com/blog/remote-code-execution-in-tutanota-desktop-due-to-code-flaw
date: 2024-08-20
---
# Remote Code Execution in Tutanota Desktop Due to Code Flaw

![rw-book-cover](https://www.sonarsource.com/app-icon.png)

## Highlights


> In June 2022, the Sonar Research team discovered critical code vulnerabilities in multiple encrypted email solutions, including Proton Mail, Skiff, and Tutanota.
> [View Highlight](https://read.readwise.io/read/01hbtymw14awtvfmesc219q70s)



> what does the `'self'` CSP value mean for `file://` URLs? Turns out it allows *any* file from the file system to be loaded!
> [View Highlight](https://read.readwise.io/read/01hcvs4cnkwm0ej3wvr3wv36zn)



> For Tutanota, there was no isolation between the application itself and the email body, so any CSS styles included in the email may also apply to other elements of the UI.
>  This can be abused by an attacker to make the *Save Attachment* button transparent and also stretch it over the whole application's UI. This form of UI redressing leaves the victim no choice but to unknowingly click the invisible button
> [View Highlight](https://read.readwise.io/read/01hcvs9jreb7eyayfx0atze0da)



> Once the attachment is downloaded, the attacker knows its file path because Tutanota saves files to a known location that includes the file's name. This allows the attacker to include the saved attachment as a script, bypassing the CSP.
> [View Highlight](https://read.readwise.io/read/01hcvsavwfbb3sn7r0b5255gdg)



> Context isolation was enabled, node integration was disabled, and so on.
>  The remaining attack surfaces are the inter-process communication (IPC) calls that can be sent between the UI and the main world.
> [View Highlight](https://read.readwise.io/read/01hcvsdmcfavgd7fhwfffmstne)



> We mapped all available IPC calls and found two interesting ones: `download` and `open`.
> [View Highlight](https://read.readwise.io/read/01hcvsdzpx9hb1jjxfkx3m98z7)



> On Windows, attackers can easily use the combination of the two calls to download and run a malicious executable. However, there is a final security mechanism in place that prevents this. The `open` IPC call implements a blocklist that tries to prevent any executable file format from being opened.
> [View Highlight](https://read.readwise.io/read/01hcvsek99w5yzwy7he4z0cpm5)



> To get the extension, the application uses the `path.extname()` function from Node.js
> [View Highlight](https://read.readwise.io/read/01hcvsgtr565mx679vxxyr1dtw)



> If there is a file called `C:\Temp\.exe`, then `path.extname()` will return an empty string. Checking the file extension blocklist of Tutanota, we can observe that the empty string is not blocked.
> [View Highlight](https://read.readwise.io/read/01hcvsh35xpazndvppbste4ze6)

