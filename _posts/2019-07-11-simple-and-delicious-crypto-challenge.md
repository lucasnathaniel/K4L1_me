---
title: A delicious soup Crypto Challenge
layout: post 
date: '2019-07-11 15:00:00'
description: Writeup for crypto of Asis Quals CTF 2019
image: 'https://i.imgur.com/LnGay3i.jpg'
tags:
- ctf
- writeup
- crypto
- infosec
---

## A delicious soup(crypto) - Asis Quals 2019

>Hi everyone! C:
>This is a crypto challenge that I liked a lot and I was wanted to do my first post on my blog with this writeup, so finally now I have that :D

## So basically we have...

This flag encrypted:

```
11d.3ilVk_d3CpIO_4nlS.ncnz3e_0S}M_kn5scpm345n3nSe_u_S{iy__4EYLP_aAAall
```

and this is the code that encrypted that:

```python
import random
from flag import flag

def encrypt(msg, perm):
	W = len(perm)
	while len(msg) % (2*W):
		msg += "."
	msg = msg[1:] + msg[:1]
	msg = msg[0::2] + msg[1::2]
	msg = msg[1:] + msg[:1]
	res = ""
	for j in xrange(0, len(msg), W):
		for k in xrange(W):
			res += msg[j:j+W][perm[k]]
	msg = res
	return msg

def encord(msg, perm, l):
	for _ in xrange(l):
		msg = encrypt(msg, perm)
	return msg

W, l = 7, random.randint(0, 1337)
perm = range(W)
random.shuffle(perm)

enc = encord(flag, perm, l)
f = open('flag.enc', 'w')
f.write(enc)
f.close()
```

We need to reverse this code and create a decrypt function.

## Reversing...

* So, first lets check the initials lines: we have a 0 to 6 random list and a random number between 0 and 1337;
* The encord function call encrypt function L(random number(0,1337)) times;
* On encrypt function, the first while will just add 2 points on encrypted flag;
* The msg[1:] + msg[:1] will put the first char in the end;
* The msg[0::2] + msg[1::2] will split the msg and zip;
* The msg[1:] + msg[:1] will put the first char in the end again;
* The last double for will shuffle the msg with the random list logic.

## Solution...

The weakness of this crypto is that the key(The random list) has 5040 possibilities, so we just need to brute that C:

>We can do it with itertools.permutations


## The ~~*Jurandir*~~ Solver

```python 
import itertools

def str_perm(flag, p):
	res = ""
	
	for j in xrange(0, 70, 7):
		for k in xrange(7):
			res += flag[j:j+7][p.index(k)]
	return res

def shuf(flag):
	flag = flag[-1:] + flag[:-1]
	new = ""
	
	for i, j in zip(flag[:len(flag)//2], flag[len(flag)//2:]):
		new += i+j
	flag = new
	flag = flag[-1:] + flag[:-1]
	return flag

def main():
	perm = list(itertools.permutations([0,1,2,3,4,5,6]))
	first_flag = open("flag.enc", "r").readlines()[0]
	
	for idx, p in enumerate(perm):
		flag = first_flag
		print str(idx)+"/5040"
		
		for _ in xrange(1337):
			res = str_perm(flag, p)
			res = shuf(res)

			if res.startswith("ASIS{"):
				print res
				exit(1)
			flag = res

if __name__ == "__main__":
	main()
```

>Running the solver, after 805 iterations, we have the flag: ASIS{1n54n3ly_Simpl3_And_d3lic1Ous_5n4ckS_eVEn_l4zY_Pe0pL3_Can_Mak3}

By: Lucas ~K4L1~ Nathaniel | FireShell
