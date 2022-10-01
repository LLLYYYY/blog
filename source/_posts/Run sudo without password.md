---
title: Run `SUDO` without password
date: 2022-10-01
categories:
- Linux
tags:
- blogs
---

In the Linux terminal , type the following command:
```bash
sudo visudo
```

A text editor would popup. Change the following line to: 
```diff
- <your-machine-name> ALL=(ALL) ALL
+ <your-machine-name> ALL=(ALL: ALL) NOPASSWD: ALL
```

Save and exit. Next time you won't be necessary to enter password when using `sudo`. Warning: could cause security concerns. 