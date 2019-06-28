---
title: N1CTF 2018 - Funning eating cms
layout: post
date: '2018-03-12 00:10:00'
description: Writeup of Funning eating cms
image: '/assets/images/posts/writeup/n1ctf/0x1.png'
tags:
- ctf
- writeup
author: Guilherme "k33r0k" Assmann
---
### N1CTF - Funning eating cms

“a strange online reservation system for restaurants, please hacking it”

This challenge told us little, just said it was to hack a restaurant website.

Upon accessing the link, he showed us the following page:

![alt text]({{ site.url }}/assets/images/posts/writeup/n1ctf/0x1.png)

By logging in we are redirected to another page:

![alt text]({{ site.url }}/assets/images/posts/writeup/n1ctf/0x2.png)

![alt text]({{ site.url }}/assets/images/posts/writeup/n1ctf/0x3.png)

Here we can see what would be the first vulnerability found, an LFI in `user.php?page=`.

![alt text]({{ site.url }}/assets/images/posts/writeup/n1ctf/0x4.png)

![alt text]({{ site.url }}/assets/images/posts/writeup/n1ctf/0x5.png)

Ok, the `guest` file did not give us much information, so I went after other files:

![alt text]({{ site.url }}/assets/images/posts/writeup/n1ctf/0x6.png)

![alt text]({{ site.url }}/assets/images/posts/writeup/n1ctf/userPHP.png)

Now, from this code, we have already been able to go to other files that were not in our view.

![alt text]({{ site.url }}/assets/images/posts/writeup/n1ctf/0x7.png)

![alt text]({{ site.url }}/assets/images/posts/writeup/n1ctf/directoryFunctions.png)

In this `function.php` file, it is good to point out some things that are being done: we have two types of filters blocking some words, like `manage`, `flag` and `ffffllllaaaaggg`, so we can not access these files by lfi casual.

Doing small tests to see if we get anything, we had a good answer in the `info` file:

![alt text]({{ site.url }}/assets/images/posts/writeup/n1ctf/ffflag.png)

We find a hint that we already know exists.

Well, we've got everything we wanted right now, we need to move up the search and find a bypass to access files that were blocked at first.

The filter of the url is being done with the help of the `parse_url` and `parse_str` function, but this function has a weakness, when we add more (`/`) bars in the url, we are able to make the function parsing correct and therefore does not read the entire url:

![alt text]({{ site.url }}/assets/images/posts/writeup/n1ctf/firstBypass.png)

![alt text]({{ site.url }}/assets/images/posts/writeup/n1ctf/decode.png)

We found another file that was not in our view, looking for it in lfi and seeing its contents, we see that it is doing the `include` of a template. So this means that the file is something that can be manipulated by the user.

![alt text]({{ site.url }}/assets/images/posts/writeup/n1ctf/m4nnge.png)

![alt text]({{ site.url }}/assets/images/posts/writeup/n1ctf/decode2.png)

Entering the page, we have a form to send files.

![alt text]({{ site.url }}/assets/images/posts/writeup/n1ctf/upload.png)

Checking the code that we had already obtained, I did not see any entries for files, so I opened the source and found the `upllloadddd.php` file.

![alt text]({{ site.url }}/assets/images/posts/writeup/n1ctf/uploadddd.png)

![alt text]({{ site.url }}/assets/images/posts/writeup/n1ctf/uploaddddddd.png)

By doing a little analysis in the code, we were able to highlight two things for the bypass:

1.It is using the `system` function with a concatenation without filters, this gives us the possibility of a RCE.

![alt text]({{ site.url }}/assets/images/posts/writeup/n1ctf/AnalystCode1.png)

2.It is checking the extension of our file, so we need a way to send an extension.

![alt text]({{ site.url }}/assets/images/posts/writeup/n1ctf/AnalystCode2.png)

The easiest way I thought of bypassing was by using a `#` comment and passing the extension right after:

![alt text]({{ site.url }}/assets/images/posts/writeup/n1ctf/shell01.png)

After a few attempts looking at all these files listed, I thought maybe the flag was in the database, so I went behind the contents of the `config.php` file.

![alt text]({{ site.url }}/assets/images/posts/writeup/n1ctf/config.png)

I used the `mysql` credentials of the `config.php` file to log into `mysql` and dumped the contents of the database.

![alt text]({{ site.url }}/assets/images/posts/writeup/n1ctf/shell02.png)

![alt text]({{ site.url }}/assets/images/posts/writeup/n1ctf/shell03.png)

![alt text]({{ site.url }}/assets/images/posts/writeup/n1ctf/shell04.png)

I found many users and nothing flag, so I thought the obvious, look for the flag in the system.

![alt text]({{ site.url }}/assets/images/posts/writeup/n1ctf/findFlag.png)

We find the flag `:)`.

![alt text]({{ site.url }}/assets/images/posts/writeup/n1ctf/shit.png)

So I came across that we could not use `/` :P

But it does not matter, we can use `cd`.

![alt text]({{ site.url }}/assets/images/posts/writeup/n1ctf/flag.png)
