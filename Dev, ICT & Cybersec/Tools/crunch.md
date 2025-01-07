---
Description: Crunch is a wordlist generator where you can specify a standard character set or any set of characters to be used in generating the wordlists. The wordlists are created through combination and permutation of a set of characters. You can determine the amount of characters and list size.
---

```bash
crunch [min] [max] [charset] (-t [pattern]) -o [file]
crunch 10 10 -f /usr/share/rainbowcrack/charset.txt mixalpa-numeric -t @@@@010180 -o /url/share/wordlist/personalizzata
crunch 4 4 -f /usr/share/crunch/charset.lst mixalpha -o mixalpha.txt

# @ Lower case alpha char
# , Upper case alpha char
# % Numeric char
# ^ Special char (space included)

# generate wordlists with exactly 850 entires per file
crunch 16 16 FLTUSEN,11,%0410 -c 850 -o START
```