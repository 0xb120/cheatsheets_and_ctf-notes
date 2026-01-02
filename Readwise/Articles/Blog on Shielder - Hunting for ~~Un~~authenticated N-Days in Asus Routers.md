---
author: Blog on Shielder
aliases: [Hunting for ~~Un~~authenticated N-Days in Asus Routers]
tags: [readwise/articles]
url: https://www.shielder.com/blog/2024/01/hunting-for-~~un~~authenticated-n-days-in-asus-routers/
date: 2024-08-20
---
# Hunting for ~~Un~~authenticated N-Days in Asus Routers

![rw-book-cover](https://www.shielder.com/favicon/favicon-192x192.png)

## Highlights


> research by reversing IoT stuff, and most of those CVEs didn’t have much public details or Proof-of-Concepts yet, we got the chance to apply the [CVE North Stars](https://cve-north-stars.github.io/) approach
> [View Highlight](https://read.readwise.io/read/01hsjxxrpa70p57nv87ajynxqn)



> “An unauthenticated remote attacker can exploit this vulnerability without privilege to perform remote arbitrary code execution”
> [View Highlight](https://read.readwise.io/read/01hsjxy5r9gxfr418hb6y70r0d)



> From the details of the CVEs we can already infer some interesting information, such as the affected devices and versions. The following firmware versions contain patches for each device:
>  • Asus RT-AX55: 3.0.0.4.386_51948 or later
>  • Asus RT-AX56U_V2: 3.0.0.4.386_51948 or later
>  • Asus RT-AC86U: 3.0.0.4.386_51915 or later
>  Also, we can learn that the vulnerability is supposedly a **format string**, and that the affected modules are `set_iperf3_cli.cgi`, `set_iperf3_srv.cgi`, and `apply.cgi`.
> [View Highlight](https://read.readwise.io/read/01hsjxya245b3h91f9eg2zhfn6)



> #patch-diffing with BinDiff
>  Once we got hold of the firmware, we proceeded by extracting them using [Unblob](https://github.com/onekey-sec/unblob).
>  By doing a quick `find`/`ripgrep` search we figured out that the affected modules are not CGI files as one would expect, but they are compiled functions handled inside the `/usr/sbin/httpd` binary.
>  #tools 
> [View Highlight](https://read.readwise.io/read/01hsjxyhtp2kyra2pzcsgqe6d6)



> We then loaded the new and the old httpd binary inside of Ghidra, analyzed them and exported the relevant information with BinDiff’s [BinExport](https://github.com/google/binexport/) to perform a patch diff.
> [View Highlight](https://read.readwise.io/read/01hsjxzexrcr5zsmd44vz65aj6)
> #tools 



> if we take a look at the handlers of the vulnerable CGI modules, we can see that they were not changed at all.
> [View Highlight](https://read.readwise.io/read/01hsjy5pq4hmjyct8hkvjtvfw2)



> Interestingly, all of them shared a common pattern. The input of the `notify_rc` function was not fixed and was instead coming from the user-controlled JSON request. :money_with_wings:
>  The `notify_rc` function is defined in `/usr/lib/libshared.so`: this explains why diffing the `httpd` binary was ineffective.
> [View Highlight](https://read.readwise.io/read/01hsjy60bjbnfevt0rxp2d92tn)



> Diffing `libshared.so` resulted in a nice discovery: in the first few lines of the `notify_rc` function, a call to a new function named `validate_rc_service` was added
> [View Highlight](https://read.readwise.io/read/01hsjy6vc5nc1qwcr48jan5t5s)



> ![](https://www.shielder.com/img/blog/asus-bindiff2.png)
>  ![](https://www.shielder.com/img/blog/asus-validaterc1.png)
>  ![](https://www.shielder.com/img/blog/asus-validaterc2.png)
> [View Highlight](https://read.readwise.io/read/01hsjy7yynstfzva9kvyyk2q21)



> Let’s emulate!
>  Enter the Dragon, Emulating with Qiling
>  If you know us, we bet you know that we love [Qiling](https://github.com/qilingframework/qiling/), so our first thought was “What if we try to emulate the firmware with Qiling and reproduce the vulnerability there?”.
> [View Highlight](https://read.readwise.io/read/01hsjyazxyt7n9x0c0bdqdphd2)

