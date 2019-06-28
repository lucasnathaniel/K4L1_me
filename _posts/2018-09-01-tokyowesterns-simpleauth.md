--- 
title: Tokyo Westerns CTF - SimpleAuth Writeup 
layout: post 
date: '2018-09-01 22:55:00'
description: Writeup of SimpleAuth
image: https://i.imgur.com/zvsZqHA.png
tags:
- ctf
- writeup
author: Guilherme "k33r0k" Assmann
---

This challenge when I got it, I did not even know its description, but I really like that kind of challenge that shows us the source code and it was not so time consuming to kill this challenge.

![](https://i.imgur.com/zvsZqHA.png)

Analyzing the code above, we can see that it receives the parameters via `$_SERVER['QUERY_STRING']` and creates a variable `action` to separate what comes from `parse_str()`

A little further down, we have some checks being made, sending a get with the value of "auth", we entered into the condition we need to solve the challenge.

From this moment, we have 3 _if's_, of which in fact they are of no use, however, we have the variable `$hashed_password` which is used in the last if to check its value and give us the flag!

The real intent of the challenge was to explore the `parse_str()`.

By doing a small test, we can see that using the `parse_str()` to get a value already set, will cause this value to be overwritten.

```php
$var = 'xxxxx'; 
parse_str($_SERVER['QUERY_STRING']); 
echo $var;
``` 

![](https://i.imgur.com/GTrRI2W.png) 

![](https://i.imgur.com/OlVQRFH.png)

Sending the value via GET, it is possible to notice that the overwriting of the values happens, so we just need to do this in the challenge.

![](https://i.imgur.com/YQjGbdb.png)
