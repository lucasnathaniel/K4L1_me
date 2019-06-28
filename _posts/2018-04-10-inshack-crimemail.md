---
title: INS'hAck - Crimemail Writeup
layout: post
date: '2018-04-10 08:00:00'
description: Writeup of Crimemail
image: https://i.imgur.com/aAxVWWy.png
tags:
- ctf
- writeup
author: Guilherme "k33r0k" Assmann
---
### INS'hAck - Crimemail Writeup

>"service, to communicate with his associates.
>Let's see if you can hack your way in his account...
>Hint: his password's md5 is computed as followed: `md5 = md5($password + $salt)` and Collins Hackle has a password which can be found in an english dictionary"

There was a simple login form in the web page so, obviously, the usual login form routine tests were run against it.

![](https://i.imgur.com/aAxVWWy.png) 

After a couple of tries I followed the "Lost password?" link and there was another form that seemed a lot like the regular one we get in these kind of links.
Running some tests on it I got an error message printed on the screen:

![](https://i.imgur.com/SdD6gCm.png)

The most logical thing to do in a SQLi attack is to try dumping the number of columns using the `union` operator.

![](https://i.imgur.com/jjeqTSE.png)

![](https://i.imgur.com/9jIQY9Z.png)

![](https://i.imgur.com/nMRG3A9.png)

![](https://i.imgur.com/eWKJJw1.png)

![](https://i.imgur.com/RZywjQ2.png)


Knowing that there was only one column, I tried to dump the table's data: `' union select concat(table_name,":",column_name) from information_schema.columns#`

![](https://i.imgur.com/Vd0mdl1.png)

After taking a look at the end of the dumped info I found what I needed.

![](https://i.imgur.com/Qw9aAQE.png)

Having the table's columns full names it was easy to dump the content.
Using the following payload I could dump all the `users` table columns' contents: `' union select concat(userID,":",username,":",pass_salt,":",pass_md5) from users#`

![](https://i.imgur.com/IzfwEk4.png)

Ok we have the dump but the passwords were hashed and salted

![](https://i.imgur.com/lFVbuRF.png)

At this point I wrote my own brute force script:

![](https://i.imgur.com/O8d6QZy.png)

Some seconds later the password was found:

`pizza` is the user password we needed :).

![](https://i.imgur.com/INAkOZS.png)

Then I logged in with the `c.hackle:pizza` credentials:

![](https://i.imgur.com/rEQsmfH.png)
