---
title: Bandit Level 1 → Level 2
layout: page
permalink: /writeups/bandit/level1/
---
**Goal:** Find the password for the next level.

### 🔍 Objective:
> *"The password for the next level is stored in a file called `-`. It is in the home directory."*


### After Googling how to read files starting with -
![2025-05-21 12_43_24-ubuntu - How can I open a file whose name starts with _-__ - Server Fault](https://github.com/user-attachments/assets/cf6837f2-0377-4f83-9bd3-9d238eab7d86)
[Serverfault](https://serverfault.com/questions/124659/how-can-i-open-a-file-whose-name-starts-with)

I found that ./ should work so lets try it.

```bash
bandit1@bandit:~$ ls
-
bandit1@bandit:~$ cat ./-
263JGJPfgU6LtdEvgfWU1XP5yac29mFx
```
TaaaaDaaaa! 
### Why this works?:
- Simply doing ``` cat - ``` tells cat to read from standard input (stdin), not read a file named -. 
- using ```./``` explicitly refers to the file named - in the current working directory.
- another method that worked for me was ``` cat -- - ```
  

