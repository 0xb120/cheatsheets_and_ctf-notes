---
Category:
  - Misc
Difficulty: Medium
Platform: HackTheBox
Status: 3. Complete
tags: [crockford]
---
>[!quote]
> *Some bits are missing*


# Set up

-

# Exploitation

```bash
$ cat crooked_crockford.txt
r,,,,rr,rr,r,rr,r,,,,,rr,rr,r,r,,r,r,rr,,,,rr,,rr,rrr,,,r,,,r,,r,rr,,,r,r,,rrr,r,,,,r,,,,,rr,r,rr,r,,r,rrr,,rrr,r,,,r,,r,,rrr,r,,r,,,,rr,rr,r,,,,,rr,r,rrrr,,r,rrr,,r,rr 

$ cat crooked_crockford.txt | sed 's/r/1/g' | sed 's/,/0/g'
100001101101011010000011011010100101011000011001101110001000100101100010100111010000100000110101101001011100111010001001001110100100001101101000001101011110010111001011
```

```python
$ python3
Python 3.9.2 (default, Feb 28 2021, 17:03:44)
[GCC 10.2.1 20210110] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> bin = '100001101101011010000011011010100101011000011001101110001000100101100010100111010000100000110101101001011100111010001001001110100100001101101000001101011110010111001011'
>>> n = 7
>>> [bin[i:i+n] for i in range(0, len(bin), n)]
['1000011', '0110101', '1010000', '0110110', '1010010', '1011000', '0110011', '0111000', '1000100', '1011000', '1010011', '1010000', '1000001', '1010110', '1001011', '1001110', '1000100', '1001110', '1001000', '0110110', '1000001', '1010111', '1001011', '1001011']
```

In ASCII: `C5P6RX38DXSPAVKNDNH6AWKK`

![Crooked-Crockford_1.png](../../zzz_res/attachments/Crooked-Crockford_1.png)

# Flag

>[!success]
`HTB{allthosenumbers}`

