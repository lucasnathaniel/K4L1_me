---
title: SunshineCTF - Home Sweet Home Writeup
layout: post
date: '2018-04-10 08:00:00'
description: Writeup of Home Sweet Home
image: https://i.imgur.com/0IGcZ3V.png
tags:
- ctf
- writeup
author: Guilherme "k33r0k" Assmann
---
### SunshineCtf - Home Sweet Home Writeup

`"Home Sweet Home
Looks like this site is doing some IP filtering. That's very FORWARD thinking of them.
Have fun!
http://web1.sunshinectf.org:50005"`

The following page was given by the task:

![](https://i.imgur.com/0IGcZ3V.png)

Given the task's title and the IP address being displayed I quickly figured out it was referring to the `127.0.0.1` address. I tried passing it through the `X-Forwarded-For` header...

![](https://i.imgur.com/az4Tp04.png)

And it worked :). 
