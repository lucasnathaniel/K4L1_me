---
title: ISITDTU CTF 2018 - Drill Writeup
layout: post
date: '2018-07-29 10:45:00'
description: Writeup of Drill
image: '/assets/images/logo.png'
tags:
- ctf
- writeup
author: shrimpgo
---
## "Drill" (1000)

Description:
There is no description, only this [link](https://mega.nz/#!u3QCSYxB!ftCpT9C-jTm09U2tLwdnAYWtLw06wS2i15BbC4aZ5xk).

---

Downloaded the file named `drill` and used the `cat` command. Inside, there was a line with a hex sequence. Converted it into bytes with `xxd` and tried to discover what kind of file it is:

```bash
$ xxd -r -p drill > 500
$ file 500
500: Zip archive data
```

Decompressed it using `unzip` but got an error:

```bash
$ unzip 500
Archive:  500
error [500]:  missing 4 bytes in zipfile
  (attempting to process anyway)
error [500]:  attempt to seek before beginning of zipfile
  (please check that you have transferred or created the zipfile in the
  appropriate BINARY mode and that you have compiled UnZip properly)
  (attempting to re-compensate)
file #1:  bad zipfile offset (local header sig):  0
  (attempting to re-compensate)
error [500]:  attempt to seek before beginning of zipfile
  (please check that you have transferred or created the zipfile in the
  appropriate BINARY mode and that you have compiled UnZip properly)
```

The error message said there was 4 bytes missing in the zipfile. I took a look inside it with `hd` to find out which bytes were missing and noticed that were the first four (50 4B 03 04), at this [link](https://en.wikipedia.org/wiki/List_of_file_signatures). Inserted these bytes and created a new one:

```bash
$ printf "\x50\x4B\x03\x04" | cat - 500 > 500.zip
$ unzip 500.zip
Archive:  500.zip
[500.zip] 499.zip password: 
```

It's asking for a password. Let's crack this with John The Ripper + rockyou list and decompress it:

```bash
$ zip2john 500.zip > 500hash
john --wordlist=rockyou.txt 500hash
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
brandon1         (500.zip)
1g 0:00:00:00 DONE (2018-07-29 17:10) 3.571g/s 58514p/s 58514c/s 58514C/s 123456..christal
Use the "--show" option to display all of the cracked passwords reliably
Session completed

$ unzip -P brandon1 500.zip
Archive:  500.zip
  inflating: 499.zip
```

Cool! Now trying to decompress 499.zip file, it is also encrypted! I had to make all of the above steps to decompress 498.zip file and so on. I made a little script to decompress all of them and save time:

```bash
$ for i in $(seq 500 -1 1); do zip2john "$i".zip > "$i"hash; key=$(john --wordlist=rockyou.txt "$i"hash | grep "$i".zip | awk '{print $1}'); unzip -P $key $i.zip; done; unzip 0.zip
```

P.S.: 0.zip file is not encrypted.

Now I received two files when I decompressed 0.zip: box.zip and key.png. box.zip was encrypted and I didn't waste my time trying to crack it. Instead, I focused on key.png and on my first try I found something:

```bash
$ zsteg -a key.png 
b3,r,msb,xy         .. text: "]1t-6lC[tX7t-"
b3,rgb,msb,xy       .. file: IRIS Showcase file - version 0
b5,bgr,lsb,xy       .. file: BS image, Version 0, Quantization 26624, (Decompresses to 6656 words)
b8,r,lsb,xy         .. text: "-.- . -.-- .. ... --- -. .-.. -.-- ..-. --- .-. .-.. ..- -.-. -.- -.-- .... ..- -. - . .-. ... "
b3,bgr,msb,xy,prime .. file: MIPSEB-LE MIPS-III ECOFF executable - version 12.6
b8,r,lsb,xy,prime   .. text: "-  ..  -. .--  .- ... -."
b3,r,msb,yx         .. file: Hitachi SH big-endian COFF object file, not stripped, 0 section
b3,rgb,msb,yx       .. file: Hitachi SH big-endian COFF object file, not stripped, 0 section
b3,bgr,msb,yx       .. file: MIPSEB-LE MIPS-III ECOFF executable stripped - version 0.0
b6,bgr,lsb,yx       .. file: MacBinary, Mon Feb  6 04:28:16 2040 INVALID date, modified Mon Feb  6 04:28:16 2040 "@"
b8,r,lsb,Xy         .. text: " ... .-. . - .- -.. .... --.- -.- .-.- -.. ..-. .-. --- .-.. --.- ..-. .- --- ... .. --.- . -.-"
b8,r,lsb,Xy,prime   .. text: "... .. -- ... .  "
```

There are four lines with dots and dashes so it's a morse code! Translating these codes with any online tool, we have:

```bash
"-.- . -.-- .. ... --- -. .-.. -.-- ..-. --- .-. .-.. ..- -.-. -.- -.-- .... ..- -. - . .-. ... " = KEYISONLYFORLUCKYHUNTERS
"-  ..  -. .--  .- ... -." = T<CNF>I<CNF>NW<CNF>AS<CNF>
" ... .-. . - .- -.. .... --.- -.- .-.- -.. ..-. .-. --- .-.. --.- ..-. .- --- ... .. --.- . -.-" = <CNF>SRETADHQK<CNF>DFROLQFAOSIQE<CNF>
"... .. -- ... .  " = SIMSE<CNF><CNF>
```

P.S.: `<CNF>` means _Code Not Found_.

I grabbed the first result (KEYISONLYFORLUCKYHUNTERS) and tried to decompress it: didn't work. My next step was to convert it to lower case and decompressing box.zip again:

```bash
$ unzip -P keyisonlyforluckyhunters box.zip
Archive:  box.zip
  inflating: flag.txt
```

Nice! Let's get our flag:

```bash
$ cat flag.txt
ISITDTU{4_g00d_hunt3r_0n_th3_c0mput3r!}
```

End of story!
