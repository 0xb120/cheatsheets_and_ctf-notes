---
Category:
  - GamePwn
Difficulty: Easy
Platform: HackTheBox
Status: 3. Complete
tags: [Unity, memory-editing, memory-inspection, reversing]
---
>[!quote]
> *Gotta collect them all.*


# Set up

- Download the folder
- Download [Cheat Engine](https://www.cheatengine.org/)

# Information Gathering

![Running the game](../../zzz_res/attachments/CubeMadness1%205b6cfd5080cf49a8bdbd63916011018d.png)

Running the game

# Exploitation

Inspect the memory multiple times until you find the possible memory locations for the value you are interested in:

![Untitled](../../zzz_res/attachments/CubeMadness1%205b6cfd5080cf49a8bdbd63916011018d%201.png)

Change the value to the desired one and obtain the flag:

![Untitled](../../zzz_res/attachments/CubeMadness1%205b6cfd5080cf49a8bdbd63916011018d%202.png)

Because the flag was not correct, search in memory the full one:

![Untitled](../../zzz_res/attachments/CubeMadness1%205b6cfd5080cf49a8bdbd63916011018d%203.png)

# Flag

>[!success]
>`HTB{CU83_M4DN355_UNM4DD3N3D}`

