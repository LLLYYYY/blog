---
title: Ubuntu CUDA installation
date: 2019-10-19
categories:
- Linux
tags:
- blogs
---

**To install CUDA in Ubuntu.**
1. Update the system first. Enable updates from Ubuntu system settings. And then run ` sudo apt upgrade `, and then ` sudo apt update `.
2. Install required libraries. `sudo apt install gcc g++ make dkms`.
3. Download official drivers from Nvidia. Install CUDA.

**Possible Problems**
- When updating the kernel, drivers fail.
- The CUDA included drivers might be too old for the system. But the gaming version is not. If the CUDA package drivers doesn't work. Try the gaming version.