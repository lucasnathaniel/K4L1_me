---
title: SunshineCTF - Marceau Writeup
layout: post
date: '2018-04-10 08:00:00'
description: Writeup of Marceau
image: '/assets/images/logo.png'
tags:
- ctf
- writeup
author: Guilherme "k33r0k" Assmann
---
### SunshineCtf - Marceau Writeup

>"Marceau
>
>Hey my friend tells me that the flag is in this site's source code. Idk how to read that though, lol (🅱️retty lame tbh 😂)
>http://marceau.web1.sunshinectf.org
>Author: charlton
>Hint 2018-04-06 00:20 UTC: There are many different types of MIMEs, but only a handful were truly legendary..."

The given web page only tells us that we are supposedly not accepting something...
By association it's refering to the `Accept` field in the `HTTP`'s header.

I listed some `PHP`-relative mimetypes and tried passing them in the `Accept` field until it finally worked.
The resulting `Accept` is:

`Accept: text/html,application/xhtml+xml,application/xml,application/x-php,text/php;q=0.9,*/*;q=0.8`

`flag: sun{45k_4nd_y3_5h411_r3c31v3}`
