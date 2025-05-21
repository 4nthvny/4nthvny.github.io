---
title: Bandit Level 2 → Level 3
layout: page
permalink: /writeups/bandit/level2/
---
**Goal:** Find the password for the next level.

### 🔍 Objective:
> *"The Password for the next level is stored in a file called "spaces in this filename" located in the home directory."*

### Once again utilizing google, I searched "spaces in filename" and this is what I found: 
![2025-05-21 13_09_07-How can I go to a directory whose file name has spaces between them in the Linux](https://github.com/user-attachments/assets/f11a154c-0a8e-4ca7-b2d3-8068f3626dbf)

With this information, I tried what google said. Shoutout google and StackOverflow. 
```bash
bandit2@bandit:~$ ls
spaces in this filename

bandit2@bandit:~$ cat 'spaces in this filename'
MNK8KNH3Usiio4lPRUEoDFPqfxLPLSmx
```
### Why this works? : 
- using '' tells cat to use the entire string as a single argument
- also using \ as spaces also works as a replacement for spaces in a filename 





