---
Category:
  - Misc
Difficulty: Easy
Platform: HackTheBox
Status: 3. Complete
tags:
  - cracking-pdf-files
---
>[!quote]
> Old algorithms are not a waste, but are really precious...

# Set up

Download the provided file

## Information Gathering

PDF file is password protected

## The bug

Easy brute-forceble password

## Exploitation

```bash
$ pdfcrack -w /usr/share/wordlists/rockyou.txt -f "0ld is g0ld.pdf"
...
found user-password: 'jumanji69'
```

Open the file and extract the morse code from the bottom of the page:

![old-is-gold_1.png](../../zzz_res/attachments/old-is-gold_1.png)

`.-. .---- .--. ... .- -- ..- ...-- .-.. -- ----- .-. ... ...--`

Translate the morse code:

![old-is-gold_2.png](../../zzz_res/attachments/old-is-gold_2.png)

**r1psamu3lm0rs3** (*convertire in maiuscolo*)

## Flag

`HTB{r1psamu3lm0rs3}`