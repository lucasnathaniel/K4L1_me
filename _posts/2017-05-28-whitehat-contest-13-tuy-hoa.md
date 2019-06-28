---
title: WhiteHat Contest 13 - Tuy Hoa
layout: post
date: '2017-05-28 13:46:55'
description: Writeup using angr
image: '/assets/images/posts/writeup/main.png'
tags:
- ctf
- writeup
author: Alisson Bezerra
---

There's a binary file that checks for a password. 

```sh
$ file re100
re100: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.24, BuildID[sha1]=d06aaba2cfbcbc4f43313fa30f2b42f079472a11, not stripped
```

Here's a screenshot of pseudocode of main function:

![Pseudocode]({{ site.url }}/assets/images/posts/writeup/main.png)


As I'm lazy as hell, I thought I could solve it with [angr](https://github.com/angr){:target="_blank" rel="noopenner noreferrer"}. Here's my script:

```python
import angr

p = angr.Project('./re100', load_options={'auto_load_libs':False})
st = p.factory.blank_state()
pg = p.factory.path_group(st)
pg.explore(find=0x400eb2, avoid=0x400ebf)

if len(pg.found) > 0:
    print "Flag: %s" % pg.found[0].state.posix.dumps(0)
```

After I ran the script, some seconds later I've got my flag.

```sh
$ python solver.py
Flag: 5a62af9a23b56ee49370808a0cf1e8096757257

$ ./re100 
input password: 
5a62af9a23b56ee49370808a0cf1e8096757257
Good password!!!
```

When I've tried to submit, I've received an error message telling me that the flag was wrong! So, I contacted the admin and they gave me the correct flag. There was an error on the binary file.
