---
title: EvlzCTF 2019 - Don't Cry
layout: post
date: '2019-02-03 17:00:00'
description: Writeup of Don't Cry
image: https://i.imgur.com/C1NweL5.png
tags:
- ctf
- writeup
author: shrimpgo
---

### "Don't Cry" (400)

Description:
How deep can you go?(0>0)
![Image](https://storage.cloud.google.com/evlzctf2019/crying.jpg)
While submitting put the flag in evlz{}

Author: Achilles
---
It's just a JPEG file called crying.jpg. I tried common tools like `stegsolve.jar`, `exiftool` and `binwalk` but they didn't find anything relevant. So I decided to look deeper with `hexdump`, trying to find something beyond JPEG headers. Looking for first JPEG bytes (FF D8) and last JPEG bytes (FF D9), I've found an extra bytes after last JPEG bytes (ff d9 __42 5a 68 ...__):

```bash
000535a0  51 4d 5c ff d9 42 5a 68  30 31 41 59 26 53 59 74  |QM\..BZh01AY&SYt|
000535b0  99 24 00 00 19 34 7f ff  ff ef ff ff ff ff ff ff  |.$...4..........|
000535c0  ff ff ff ff ff ff ff ff  ff ff ff ff ff ff ff ff  |................|
000535d0  ff ff ff ff ff ff ff ff  e0 ee fa dd ee fa ef 65  |...............e|
000535e0  ab 6f 77 bc 7b ef ae ee  dd ee de bd 3b 7b b7 dd  |.ow.{.......;{..|
000535f0  f6 9f 6c fb e6 3a d5 b7  d7 b7 8f bb 57 1d 6f a5  |..l..:......W.o.|
00053600  6f 76 fb ce bd 2a f9 ee  d6 ed bd bb de e8 cb 57  |ov...*.........W|
00053610  dd 97 db ed 65 69 f6 f7  1b ef 75 bb 8f 7b e5 e9  |....ei....u..{..|
00053620  db d7 de 5b d6 b7 77 a7  ae fb ba de 25 a7 de 7b  |...[..w.....%..{|
00053630  de ef 88 2f 75 ee ea f7  ac 35 f7 9b 91 f7 2e f7  |.../u....5......|
00053640  79 aa db be f2 ef 76 fb  74 d5 73 6d f7 9d af 5e  |y.....v.t.sm...^|
00053650  e6 fb b0 1d bd b6 f6 be  1e f7 0d cc 3e 6e f7 9e  |............>n..|
00053660  7b ea e6 ee de fb 1e e8  ee db 7d 1d d7 be df 4f  |{.........}....O|
00053670  a4 db 6b 7a b5 f6 f3 ef  9b ef 7a b9 62 f6 6b ee  |..kz......z.b.k.|
00053680  f5 b9 ee fa ab 77 bd f7  bb bc be b5 ce 6e 22 ea  |.....w.......n".|
00053690  ef bb 7b bd f7 57 7d 7d  dd 3e fb 6f 7b ef 39 f7  |..{..W}}.>.o{.9.|
000536a0  77 de fa f7 5d f7 bd 76  2f 9e ec ec 2d bb 9a f7  |w...]..v/...-...|
000536b0  df 37 7b db cb 5b 7b dd  eb b7 71 3b be f2 d7 7b  |.7{..[{...q;...{|
000536c0  63 dd 6f 7d db b7 bb e9  b7 79 67 de 7b df 4c 54  |c.o}.....yg.{.LT|
...
```

I grabbed this extra bytes with `hexedit` (or any hex editor), saved it in another file (let's call it foo for a while) and tried to figure out what type this file is with `file` tools:

```bash
$ file foo
foo: bzip2 compressed data, block size = 000k
```

Nice! I renamed to foo.bz2 and decompressed it, but it threw me an error:

```bash
$ bunzip2 foo.bz2 
bunzip2: foo.bz2 is not a bzip2 file.
```

What is wrong with this file? Checking bz2 structure in [Wikipedia](https://en.wikipedia.org/wiki/Bzip2), I've found this:

![link](https://i.imgur.com/C1NweL5.png)

foo file has following first bytes: 42 5a 68 30 31 41 59 26 53 59 74. Comparing it to bzip2 structure, I noted that `hundred_k_blocksize` was filled with 0 (0x30), so I tried replacing it with 1 (0x31) (100k block size):

```bash
$ bunzip2 foo.bz2
$ ls
crying.jpg  foo
```

Alright! What kind foo file is now?

```bash
$ file foo
foo: PDF document, version 1.7
```

Before I open this file, I checked it's "sanity":

```bash
$ pdfinfo foo.pdf 
Creator:        Nitro Pro 12
ModDate:        Thu Jan  3 13:15:06 2019 -02
Tagged:         no
UserProperties: no
Suspects:       no
Form:           none
Syntax Error: Couldn't find trailer dictionary
Syntax Error: Invalid XRef entry
Internal Error: xref num 21 not found but needed, try to reconstruct<0a>
Syntax Error: Invalid XRef entry
JavaScript:     no
Pages:          1
Encrypted:      no
Page size:      612 x 792 pts (letter)
Page rot:       0
File size:      96378 bytes
Optimized:      no
PDF version:    1.7
```

And when I tried to open it with a PDF reader, there was a blank page with errors in my terminal, with the same error messages when I ran pdfinfo. Now I had 3 options:

* Learn PDF version 1.7 structure to recriate trailer dictionary;
* Compare foo.pdf structre with another PDF version 1.7 file and fill it with lack trailer dictionary, accordingly with foo.pdf values;
* Search for some PDF Repair online;

I chose the third one, because those tools are easy and quick to use. I've found this [link](https://www.pdf-online.com/osa/repair.aspx) and it fixed foo.pdf! OK, it puts watermark on my PDF file but I don't care! I only want it's contents! I opened foo.pdf and saw this text:

```
Wake
From your sleep
The drying of
Your tears
T4oday
We escape
We escape
Pack
And get dressed
Beyfore your father hears Jus
Before
All he2ll
Breaks loose
Breathe
Keep breathing
Don't lvoose
Your nerve
Breathe
Keep bmreathing
I can't do this
A_lone
Sing
qUs a song
A song to keep
UVs warm
Th3ere's
Such a chill
Such _a chill
You can laugh
A spineless laugh
We hopse your
Rules and wisdom choke you
No1w
We are one
In eeverlasting peace
We hope that you choke
That you choke
We hfope that you choke
That you choke
We ho1pe
That you choke
That you choke
```

When I read this, I noticed there were misplaced characters in the middle of some words, so I collected all of them and formed this: `4yJ2vm_V3_s1ef1`. It's a kind of cipher, I don't know. My guess is it's Caesar Cipher and I've made a bruteforce attack using every possible rotations. I used this [site](https://www.dcode.fr/caesar-cipher) to do the job for me and I've found, in rot13, this: `4lW2iz_dI3_f1rs1`. So strange for a flag, right? At my first try I was correct, the flag was `evlz{4lW2iz_dI3_f1rs1}ctf`.

Nice chall!
