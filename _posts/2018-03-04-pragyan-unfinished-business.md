---
title: Pragyan CTF - Unfinished Business
layout: post
date: '2018-03-04 15:35:00'
description: Writeup of Unfinished Business
image: '/assets/images/posts/writeup/pragyan/0x1-2.png'
tags:
- ctf
- writeup
author: Guilherme "k33r0k" Assmann
---
### Pragyan CTF - Unfinished business (Web 100pts)

This challenge sent to the following page:

![alt text]({{ site.url }}/assets/images/posts/writeup/pragyan/0x1-2.png)

When I tried to login, the page returned a message that the dashboard was not complete, but making a small brute of directories, I found a file `admin.php`, with the burp, I logged in and redirect the `POST` to the `admin.php` file.

![alt text]({{ site.url }}/assets/images/posts/writeup/pragyan/flag-2.png)
