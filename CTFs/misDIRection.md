---
Category:
  - Misc
Difficulty: Easy
Platform: HackTheBox
Status: 3. Complete
---
>[!quote]
>*During an assessment of a unix system the HTB team found a suspicious directory. They looked at everything within but couldn't find any files with malicious intent.*


# Set up

-

# Information Gathering

```bash
┌──(kali㉿kali)-[~/…/challenge/misc/misDIRection/.secret]
└─$ find . -type f -exec ls -al {} \;
-rw-r--r-- 1 kali kali 0 May  2  2018 ./J/8
-rw-r--r-- 1 kali kali 0 May  2  2018 ./9/36
-rw-r--r-- 1 kali kali 0 May  2  2018 ./e/5
-rw-r--r-- 1 kali kali 0 May  2  2018 ./5/16
-rw-r--r-- 1 kali kali 0 May  2  2018 ./V/35
-rw-r--r-- 1 kali kali 0 May  2  2018 ./E/14
-rw-r--r-- 1 kali kali 0 May  2  2018 ./X/29
-rw-r--r-- 1 kali kali 0 May  2  2018 ./X/21
-rw-r--r-- 1 kali kali 0 May  2  2018 ./X/17
-rw-r--r-- 1 kali kali 0 May  2  2018 ./C/4
-rw-r--r-- 1 kali kali 0 May  2  2018 ./p/32
-rw-r--r-- 1 kali kali 0 May  2  2018 ./N/33
-rw-r--r-- 1 kali kali 0 May  2  2018 ./N/25
-rw-r--r-- 1 kali kali 0 May  2  2018 ./N/11
-rw-r--r-- 1 kali kali 0 May  2  2018 ./N/31
-rw-r--r-- 1 kali kali 0 May  2  2018 ./s/24
-rw-r--r-- 1 kali kali 0 May  2  2018 ./j/10
-rw-r--r-- 1 kali kali 0 May  2  2018 ./j/12
-rw-r--r-- 1 kali kali 0 May  2  2018 ./D/26
-rw-r--r-- 1 kali kali 0 May  2  2018 ./x/15
-rw-r--r-- 1 kali kali 0 May  2  2018 ./U/9
-rw-r--r-- 1 kali kali 0 May  2  2018 ./0/6
-rw-r--r-- 1 kali kali 0 May  2  2018 ./2/34
-rw-r--r-- 1 kali kali 0 May  2  2018 ./d/13
-rw-r--r-- 1 kali kali 0 May  2  2018 ./u/28
-rw-r--r-- 1 kali kali 0 May  2  2018 ./u/20
-rw-r--r-- 1 kali kali 0 May  2  2018 ./F/27
-rw-r--r-- 1 kali kali 0 May  2  2018 ./F/2
-rw-r--r-- 1 kali kali 0 May  2  2018 ./F/19
-rw-r--r-- 1 kali kali 0 May  2  2018 ./R/3
-rw-r--r-- 1 kali kali 0 May  2  2018 ./R/7
-rw-r--r-- 1 kali kali 0 May  2  2018 ./1/30
-rw-r--r-- 1 kali kali 0 May  2  2018 ./1/22
-rw-r--r-- 1 kali kali 0 May  2  2018 ./z/18
-rw-r--r-- 1 kali kali 0 May  2  2018 ./S/1
-rw-r--r-- 1 kali kali 0 May  2  2018 ./B/23

┌──(kali㉿kali)-[~/…/challenge/misc/misDIRection/.secret]
└─$ find . -type f -exec ls {} \; | sort | awk -F'/' '{print $3 "\t" $2}'       
6       0
22      1
30      1
34      2
16      5
36      9
23      B
4       C
13      d
26      D
14      E
5       e
19      F
2       F
27      F
10      j
12      j
8       J
11      N
25      N
31      N
33      N
32      p
3       R
7       R
1       S
24      s
20      u
28      u
9       U
35      V
15      x
17      X
21      X
29      X
18      z
```

Folder sequence ordered by file name:

```bash
┌──(kali㉿kali)-[~/…/challenge/misc/misDIRection/.secret]
└─$ find . -type f -exec ls {} \; | sort | awk -F'/' '{print $3 ":" $2}' | sort -n | cut -d':' -f2 | tr -d '\n'
SFRCe0RJUjNjdEx5XzFuX1BsNDFuX1NpN2V9 

┌──(kali㉿kali)-[~/…/challenge/misc/misDIRection/.secret]
└─$ find . -type f -exec ls {} \; | sort | awk -F'/' '{print $3 ":" $2}' | sort -n | cut -d':' -f2 | tr -d '\n' | base64 -d
HTB{DIR3ctLy_1n_Pl41n_Si7e}
```

# Flag

>[!success]
>`HTB{DIR3ctLy_1n_Pl41n_Si7e}`

