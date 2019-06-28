---
title: TJCTF - Ess Kyoo Ell Writeup
layout: post
date: '2018-08-09 16:55:00'
description: Writeup of Ess Kyoo Ell
image: https://i.imgur.com/RtVTVug.png
tags:
- ctf
- writeup
author: Guilherme "k33r0k" Assmann
---
This challenge is interesting, sometimes it does not seem very realistic, but if you take the side of that every time the developers try to make everything more automatic, it fits perfectly in this question!

![](https://i.imgur.com/ByOv3s8.png)

At first it seemed like a lot of sql injection for auth bypass, I tried some simple injections like `'or 1 = 1 #` and derivatives

![](https://i.imgur.com/RtVTVug.png)

But looking more closely, it is trying to get something out of a `password` column that does not even seem to exist, if this was bugged, they probably would have corrected it, but in fact, what happens is that the parameters of the post are the columns!

![](https://i.imgur.com/zztxytX.png)

By doing a simple test, we can confirm this:

![](https://i.imgur.com/vdXCOZV.png)

In order not to confuse the parameter with its value, and leave the injection in the parameter key, I used the `=` encode, not to occur is confusion internally:

![](https://i.imgur.com/K4tif3T.png)

Well, it was clear that we were able to dump a user, but, the description tells us that we need the ip of the admin, so what I did was try to look for the user admin through the injection...

![](https://i.imgur.com/vAqpvFW.png)

Then the flag is `tjctf{145.3.1.213}`
