---
title: Pragyan CTF - El33t Articles Hub
layout: post
date: '2018-03-03 9:00:00'
description: Writeup of El33t Articles Hub
image: '/assets/images/posts/writeup/pragyan/0x1.png'
tags:
- ctf
- writeup
author: Guilherme "k33r0k" Assmann
---

### Pragyan CTF - El33t Articles Hub (Web 200pts)

This challenge sent us to the following page:

![alt text]({{ site.url }}/assets/images/posts/writeup/pragyan/0x1.png)

After a few attempts at the `index.php?file`, I opened the source code of the index and I saw a file `favicon.php?id=6`, it seemed strange, so I changed the id by an X and I got something like that back:

`No files named './favicons/x.png', './favicons/x.ico'  or './favicons/x.php' found`

So I decided to try to search for the index and it worked!

![alt text]({{ site.url }}/assets/images/posts/writeup/pragyan/0x2.png)

In the index there were two more files:

![alt text]({{ site.url }}/assets/images/posts/writeup/pragyan/0x3.png)

![alt text]({{ site.url }}/assets/images/posts/writeup/pragyan/helpers.png)

Looking at the helpers.php code, there was a file indicating the location of the flag: `secret/flag_7258689d608c0e2e6a90c33c44409f9d`.

We could not access this file since it is a txt and `favicon.php` does not accept txt files.

Going back to the `index.php?file=`:

![alt text]({{ site.url }}/assets/images/posts/writeup/pragyan/0x5.png)

It's wrong...

Looking at the code more calmly, we can see that it is blocking `php:` and the file of the flag. We need to bypass however, by looking more closely at the `helpers.php` file, it is possible to see that it is being replaced `../` by `` and `./` by ``.

![alt text]({{ site.url }}/assets/images/posts/writeup/pragyan/flag.png)
