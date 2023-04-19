---
Description: Site scraper that find keywords and create targetted wordlists
---

```bash
root@kali:~# cewl [www.megacorpone.com](http://www.megacorpone.com/) -m6 -w megacorpone.txt
CeWL 5.4.8 (Inclusion) Robin Wood (robin@digi.ninja) ([https://digi.ninja/](https://digi.ninja/))
root@kali:~# cat megacorpone.txt |wc -l
318
root@kali:~# head megacorpone.txt
MegaCorp
technology
megacorpone
nanotechnology
Bootstrap
SUPPORT
CAREERS
Contact
company
CONTACT
```