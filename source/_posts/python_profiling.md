---
title: python profiling
date: 2022-08-15
categories:
- Programming_Language
tags:
- random_notes
---

This blog show simple steps that can profile a python code and then visualize the profile info. 

1. Run the `python` code with `cProfile`

```bash
python3 -m cProfile -o test.profile __main__.py
```

2. Install `snakeviz` profile visualization tool

```bash
pip3 install snakeviz
```

3. Run the following command. The tool will host a local webpage that we can access. 

```bash
snakeviz test.profile
```

That's it. 
