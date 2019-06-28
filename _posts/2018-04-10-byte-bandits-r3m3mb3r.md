---
title: Byte Bandits CTF - R3M3MB3R Writeup
layout: post
date: '2018-04-10 08:00:00'
description: Writeup of R3M3MB3R
image: https://i.imgur.com/qgWzjcj.png
tags:
- ctf
- writeup
author: Guilherme "k33r0k" Assmann
---
### Byte Bandits CTF - R3M3MB3R Writeup

The following page was given by the task:

![](https://i.imgur.com/qgWzjcj.png)

Without second thoughts, it's clear this is about an LFI so I took the straightforward approach:

![](https://i.imgur.com/CPW5SnH.png)

![](https://i.imgur.com/FcA2IOS.png)

The source code of `eg.php` had nothing useful, so I tried `index.php` instead and found a filter.
After some attempts on getting the `index.php` source code without success, I decided to try other ways, like Apache's logs.

![](https://i.imgur.com/y9rKFVc.png)

Some hours went by and the log infecction attempts were unsuccessful, because the URLs in the log files where URL encoded. Then an idea on trying to infect the log files through the `User-agent` came up, via "Alisson Bezerra".
And that worked...

![](https://i.imgur.com/MI86zkV.png)

![](https://i.imgur.com/2MzSXxk.png)
