---
title: TJCTF - Request Me Writeup
layout: post
date: '2018-08-09 16:53:00'
description: Writeup of Request Me
image: https://i.imgur.com/KjTElM6.png
tags:
- ctf
- writeup
author: Guilherme "k33r0k" Assmann
---
This challenge is really silly, but what is annoying, is that he was blocking the use of the `Burp Suite` and `mitmproxy`, but using `postman` it is possible to make the requests!

Initially, I tried to do everything straight through the curl, but it did not work, I'm not sure exactly why, but I performed exactly the same `curl` things on `postman` and it worked:

![](https://i.imgur.com/5NYBuc0.png)

In the first requests the challenge induced us to send the `OPTIONS` to see the accepted methods:

![](https://i.imgur.com/KjTElM6.png)

Within `OPTIONS` it would tell us that there were 2 parameters to be sent, however, we did not know where, so I tried to send the parameters in both `GET` and `POST` as well.

I noticed that none worked so I tested the two together in other methods until I got through the `DELETE` method:

![](https://i.imgur.com/QmixNfh.png)
