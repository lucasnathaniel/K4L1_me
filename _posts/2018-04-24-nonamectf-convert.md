---
title: NoNameConCTF - Convert Writeup
layout: post
date: '2018-04-24 08:17:00'
description: Writeup of Convert
image: https://i.imgur.com/839eg3Y.png
tags:
- ctf
- writeup
author: Guilherme "k33r0k" Assmann
---
### NoNameConCTF - Convert Writeup
>Convert
>100
>
>If you need to convert something to Markdown, you can try our service: http://convert.nonameconctf2018.xyz


This challenge quickly illustrates a ssrf.

One of the first things I tried was to access `localhost / 127.0.0.1`, but without success:

![](https://i.imgur.com/839eg3Y.png)

So I decided to try to do with `0.0.0.0` and it worked!

![](https://i.imgur.com/kc81Z6l.png)

But I did not know what to look for, so I did the simple, a brute of directories:

![](https://i.imgur.com/uCThGlR.jpg)

So finally, I just did the ssrf asking for the file:

![](https://i.imgur.com/CtqseLe.png)

