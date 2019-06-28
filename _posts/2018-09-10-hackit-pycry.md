--- 
title: HackIT CTF 2018 - PyCry Writeup
layout: post 
date: '2018-09-10 16:55:00'
description: Writeup of PyCry
image: '/assets/images/logo.png'
tags:
- ctf
- writeup
author: Marzano
---
This challenge was a teamwork challenge.

First, @diofeher discovered that we could run python on the server by sending code to the second input. He found this:

```python
<< print locals()
<< {'funcwtf': <function funcwtf at 0x7f22e83de758>, 'string': <module 'string' from '/usr/lib/python2.7/string.pyc'>, 'name': 'test', 'hint': <function hint at 0x7f22e83de848>, '__builtins__': <module '__builtin__' (built-in)>, 'func4e': <function func4e at 0x7f22e83de7d0>, '__file__': '/home/pychal/chal.py', 'random': <module 'random' from '/usr/lib/python2.7/random.pyc'>, 'func3e': <function func3e at 0x7f22e83de6e0>, '__package__': None, 'sys': <module 'sys' (built-in)>, 'func1e': <function func1e at 0x7f22e83cd230>, '__name__': '__main__', 'func2e': <function func2e at 0x7f22e83de668>, 'os': <module 'os' from '/usr/lib/python2.7/os.pyc'>, '__doc__': None, 'drunkenc': <function drunkenc at 0x7f22e83de8c0>, 'dis': <module 'dis' from '/usr/lib/python2.7/dis.pyc'>}
```

Inside the `hint` function he found some guidance:

```python
<< print locals().__getitem__('hint')().func_code
<< The flag was encrypted using func2e(func4e(func3e(func1e('flag{redacted}','a6105c0a611b41b08f1209506350279e'),'looooool')))
<< I think the output was : 0A3BFDA6EBFD5CEBFD1ADBFDBEDBFD1DFBFDF10CFD51FBFD51FBFDBEEBFDB3ABFD3DABFD2DABFD589BFDD79BFD9E9BFD10ABFDDFBBFDAFBBFDD4CBFD77CBFD42BBFD91BBFDC2BBFDB4BBFDE7BBFDB6BBFDF0BBFD2ABBFD22BBFD39BBFDE2BBFDB1CE
<< You guessed right, your goal is to retrieve the flag using your python, reversing and crypto skills
<< Good luck!
```

Then, he disassembled `func1e`, `func2e`, `func3e`, `func4e` and `funcwtf` for us:

```python
<< print __builtins__.__import__('dis').dis(func1e)
<<  10           0 LOAD_CONST               1 ('')
<<               3 STORE_FAST               2 (tmp)
<<
<<  11           6 SETUP_LOOP             133 (to 142)
<<               9 LOAD_GLOBAL              0 (range)
<<              12 LOAD_GLOBAL              1 (len)
<<              15 LOAD_FAST                0 (str1)
<<              18 CALL_FUNCTION            1
<<              21 CALL_FUNCTION            1
<<              24 GET_ITER
<<         >>   25 FOR_ITER               113 (to 141)
<<              28 STORE_FAST               3 (i)
<<
<<  12          31 LOAD_FAST                3 (i)
<<              34 LOAD_CONST               2 (2)
<<              37 BINARY_MODULO
<<              38 LOAD_CONST               3 (0)
<<              41 COMPARE_OP               2 (==)
<<              44 POP_JUMP_IF_FALSE       94
<<
<<  13          47 LOAD_FAST                2 (tmp)
<<              50 LOAD_GLOBAL              2 (chr)
<<              53 LOAD_GLOBAL              3 (ord)
<<              56 LOAD_FAST                0 (str1)
<<              59 LOAD_FAST                3 (i)
<<              62 BINARY_SUBSCR
<<              63 CALL_FUNCTION            1
<<              66 LOAD_GLOBAL              3 (ord)
<<              69 LOAD_FAST                1 (str2)
<<              72 LOAD_FAST                3 (i)
<<              75 BINARY_SUBSCR
<<              76 CALL_FUNCTION            1
<<              79 BINARY_XOR
<<              80
<<  LOAD_CONST               4 (4)
<<              83 BINARY_ADD
<<              84 CALL_FUNCTION            1
<<              87 INPLACE_ADD
<<              88 STORE_FAST               2 (tmp)
<<              91 JUMP_ABSOLUTE           25
<<
<<  15     >>   94 LOAD_FAST                2 (tmp)
<<              97 LOAD_GLOBAL              2 (chr)
<<             100 LOAD_GLOBAL              3 (ord)
<<             103 LOAD_FAST                0 (str1)
<<             106 LOAD_FAST                3 (i)
<<             109 BINARY_SUBSCR
<<             110 CALL_FUNCTION            1
<<             113 LOAD_GLOBAL              3 (ord)
<<             116 LOAD_FAST                1 (str2)
<<             119 LOAD_FAST                3 (i)
<<             122 BINARY_SUBSCR
<<             123 CALL_FUNCTION            1
<<             126 BINARY_XOR
<<             127 LOAD_CONST               2 (2)
<<             130 BINARY_SUBTRACT
<<             131 CALL_FUNCTION            1
<<             134 INPLACE_ADD
<<             135 STORE_FAST               2 (tmp)
<<             138 JUMP_ABSOLUTE           25
<<         >>  141 POP_BLOCK
<<
<<  16     >>  142 LOAD_FAST                2 (tmp)
<<             145 LOAD_CONST               0 (None)
<<             148 LOAD_CONST               0 (None)
<<             151 LOAD_CONST               5 (-1)
<<             154 BUILD_SLICE              3
<<             157 BINARY_SUBSCR
<<
<<             158 RETURN_VALUE
<< None


<< print __builtins__.__import__('dis').dis(func3e)
<<
<<  37           0 BUILD_LIST               0
<<               3 STORE_FAST               2 (encryped)
<<
<<  38           6 SETUP_LOOP              91 (to 100)
<<               9 LOAD_GLOBAL              0 (enumerate)
<<              12 LOAD_FAST                0 (msg)
<<              15 CALL_FUNCTION            1
<<              18 GET_ITER
<<         >>   19 FOR_ITER                77 (to 99)
<<              22 UNPACK_SEQUENCE          2
<<              25 STORE_FAST               3 (i)
<<              28 STORE_FAST               4 (c)
<<
<<  39          31 LOAD_GLOBAL              1 (ord)
<<              34 LOAD_FAST                1 (key)
<<              37 LOAD_FAST                3 (i)
<<              40 LOAD_GLOBAL              2 (len)
<<              43 LOAD_FAST                1 (key)
<<              46 CALL_FUNCTION            1
<<              49 BINARY_MODULO
<<              50 BINARY_SUBSCR
<<              51 CALL_FUNCTION            1
<<              54 STORE_FAST               5 (key_c)
<<
<<  40          57 LOAD_GLOBAL              1 (ord)
<<              60 LOAD_FAST                4 (c)
<<              63 CALL_FUNCTION            1
<<              66 STORE_FAST               6 (msg_c)
<<
<<  41          69 LOAD_FAST                2 (encryped)
<<              72 LOAD_ATTR                3 (append)
<<              75 LOAD_GLOBAL              4 (chr)
<<              78 LOAD_FAST                6 (msg_c)
<<              81 LOAD_FAST                5 (key_c)
<<              84 BINARY_ADD
<<              85 LOAD_CONST               1 (127)
<<              88 BINARY_MODULO
<<              89 CALL_FUNCTION            1
<<              92 CALL_FUNCTION            1
<<              95 POP_TOP
<<              96 JUMP_ABSOLUTE           19
<<         >>   99 POP_BLOCK
<<
<<  42     >>  100 LOAD_CONST               2 ('')
<<             103 LOAD_ATTR                5 (join)
<<             106 LOAD_FAST                2 (encryped)
<<             109 CALL_FUNCTION            1
<<             112 RETURN_VALUE
<< None
<<

<< print __builtins__.__import__('dis').dis(funcwtf)
<<
<<  47           0 LOAD_FAST                0 (n)
<<               3 LOAD_CONST               1 (0)
<<               6 COMPARE_OP               4 (>)
<<               9 POP_JUMP_IF_FALSE       25
<<
<<  48          12 LOAD_FAST                0 (n)
<<              15 LOAD_CONST               2 (20)
<<              18 BINARY_MODULO
<<              19 STORE_FAST               0 (n)
<<              22 JUMP_FORWARD            12 (to 37)
<<
<<  50     >>   25 LOAD_FAST                0 (n)
<<              28 UNARY_NEGATIVE
<<              29 LOAD_CONST               2 (20)
<<              32 BINARY_MODULO
<<              33 UNARY_NEGATIVE
<<              34 STORE_FAST               0 (n)
<<
<<  51     >>   37 LOAD_GLOBAL              0 (string)
<<              40 LOAD_ATTR                1 (ascii_lowercase)
<<              43 STORE_FAST               1 (lc)
<<
<<  52          46 LOAD_GLOBAL              0 (string)
<<              49 LOAD_ATTR                2 (ascii_uppercase)
<<              52 STORE_FAST               2 (uc)
<<
<<  53          55 LOAD_GLOBAL              0 (string)
<<              58 LOAD_ATTR                3 (maketrans)
<<              61 LOAD_FAST                1 (lc)
<<              64 LOAD_FAST                2 (uc)
<<              67 BINARY_ADD
<<
<<  54          68 LOAD_FAST                1 (lc)
<<              71 LOAD_FAST                0 (n)
<<              74 SLICE+1
<<              75 LOAD_FAST                1 (lc)
<<              78 LOAD_FAST                0 (n)
<<              81 SLICE+2
<<              82 BINARY_ADD
<<              83 LOAD_FAST                2 (uc)
<<              86 LOAD_FAST                0 (n)
<<              89 SLICE+1
<<              90 BINARY_ADD
<<              91 LOAD_FAST                2 (uc)
<<              94 LOAD_FAST                0 (n)
<<              97 SLICE+2
<<              98 BINARY_ADD
<<              99 CALL_FUNCTION            2
<<             102 STORE_DEREF              0 (trans)
<<
<<  55         105 LOAD_CLOSURE             0 (trans)
<<             108 BUILD_TUPLE              1
<<             111 LOAD_CONST               3 (<code object <lambda> at 0x7f78b15b80b0, file "/home/pychal/chal.py", line 55>)
<<             114 MAKE_CLOSURE             0
<<             117 RETURN_VALUE
<< None
<<

<< print __builtins__.__import__('dis').dis(func2e)
<<
<<  21           0 SETUP_EXCEPT           212 (to 215)
<<
<<  22           3 LOAD_GLOBAL              0 (random)
<<               6 LOAD_ATTR                1 (randint)
<<               9 LOAD_CONST               1 (1)
<<              12 LOAD_CONST               2 (1024)
<<              15 CALL_FUNCTION            2
<<              18 STORE_FAST               1 (k)
<<
<<  23          21 LOAD_CONST               3 (0)
<<              24 STORE_FAST               2 (n)
<<
<<  24          27 LOAD_CONST               4 ('')
<<              30 STORE_FAST               3 (f)
<<
<<  25          33 SETUP_LOOP             116 (to 152)
<<              36 LOAD_GLOBAL              2 (range)
<<              39 LOAD_GLOBAL              3 (len)
<<              42 LOAD_FAST                0 (d)
<<              45 CALL_FUNCTION            1
<<              48 CALL_FUNCTION            1
<<              51 GET_ITER
<<         >>   52 FOR_ITER                96 (to 151)
<<              55 STORE_FAST               4 (i)
<<
<<  26          58 LOAD_FAST                2 (n)
<<              61 LOAD_CONST               1 (1)
<<              64 INPLACE_ADD
<<              65 STORE_FAST               2 (n)
<<
<<  27          68 LOAD_FAST                2 (n)
<<              71 LOAD_FAST                2 (n)
<<              74 BINARY_MULTIPLY
<<              75 LOAD_CONST               5 (62)
<<              78 BINARY_XOR
<<              79 STORE_FAST
<<   5 (c)
<<
<<  28          82 LOAD_FAST                3 (f)
<<              85 LOAD_CONST               6 ('00000')
<<              88 LOAD_GLOBAL              4 (hex)
<<              91 LOAD_GLOBAL              5 (int)
<<              94 LOAD_GLOBAL              6 (oct)
<<              97 LOAD_GLOBAL              7 (ord)
<<             100 LOAD_FAST                0 (d)
<<             103 LOAD_FAST                4 (i)
<<             106 BINARY_SUBSCR
<<             107 CALL_FUNCTION            1
<<             110 LOAD_FAST                1 (k)
<<             113 LOAD_CONST               7 (720451)
<<             116 BINARY_XOR
<<             117 LOAD_CONST               8 (3775139)
<<             120 BINARY_XOR
<<             121 LOAD_FAST                5 (c)
<<             124 BINARY_XOR
<<             125 BINARY_XOR
<<             126 CALL_FUNCTION            1
<<             129 CALL_FUNCTION            1
<<             132 CALL_FUNCTION            1
<<             135 LOAD_CONST               9 (2)
<<             138 SLICE+1
<<             139 BINARY_ADD
<<             140 LOAD_CONST              10 (-6)
<<             143 SLICE+1
<<             144 INPLACE_ADD
<<             145 STORE_FAST               3 (f)
<<             148 JUMP_ABSOLUTE           52
<<         >>  151 POP_BLOCK
<<
<<  29     >>  152 LOAD_CONST              11 ('000')
<<             155 L
<< OAD_GLOBAL              4 (hex)
<<             158 LOAD_FAST                1 (k)
<<             161 LOAD_CONST              12 (2719)
<<             164 BINARY_XOR
<<             165 LOAD_CONST              13 (59262)
<<             168 BINARY_XOR
<<             169 CALL_FUNCTION            1
<<             172 LOAD_CONST               9 (2)
<<             175 SLICE+1
<<             176 BINARY_ADD
<<             177 LOAD_CONST              14 (-4)
<<             180 SLICE+1
<<             181 LOAD_FAST                3 (f)
<<             184 BINARY_ADD
<<             185 LOAD_CONST               0 (None)
<<             188 LOAD_CONST               0 (None)
<<             191 LOAD_CONST              15 (-1)
<<             194 BUILD_SLICE              3
<<             197 BINARY_SUBSCR
<<             198 LOAD_ATTR                8 (upper)
<<             201 CALL_FUNCTION            0
<<             204 STORE_FAST               3 (f)
<<
<<  30         207 LOAD_FAST                3 (f)
<<             210 RETURN_VALUE
<<             211 POP_BLOCK
<<             212 JUMP_FORWARD             8 (to 223)
<<
<<  31     >>  215 POP_TOP
<<             216 POP_TOP
<<             217 POP_TOP
<<
<<  32         218 LOAD_CONST              15 (-1)
<<             221 RETURN_VALUE
<<             222 END_FINALLY
<<         >>  223 LOAD_CONST               0 (None)
<<             226 RETURN_VALUE
<< None
<<

<< print __builtins__.__import__('dis').dis(func4e)
<<
<<  59           0 LOAD_GLOBAL              0 (funcwtf)
<<               3 LOAD_CONST               1 (1337)
<<               6 CALL_FUNCTION            1
<<               9 STORE_FAST               1 (enc)
<<
<<  60          12 LOAD_FAST                1 (enc)
<<              15 LOAD_FAST                0 (str)
<<              18 CALL_FUNCTION            1
<<              21 RETURN_VALUE
<< None
<<
```

After about 3 hours of hard, manual work, @diofeher and I managed to decompile this ugly thing:

```python
def func1e(str1, str2):
    tmp = ''
    for i in range(len(str1)):
        if i % 2 == 0:
            tmp += chr((ord(str1[i]) ^ ord(str2[i])) + 4)
        else:
            tmp += chr((ord(str1[i]) ^ ord(str2[i])) - 2)

    return tmp[::-1]

def func2e(d):
    try:
        k = random.randint(1, 1024)
        n = 0
        f = ''

        for i in range(len(d)):
            n += 1
            c = (n*n) ^ 62
            f += ('00000' + hex(int(oct(ord(d[i]) ^ (k ^ 720451 ^ 3775139 ^ c))))[2:])[-6:]

        f = (('000' + hex((k ^ 2719) ^ 59262)[2:])[-4:] + f)[::-1].upper()
        return f
    except:
        return -1

def func3e(msg, key):
    encryped = []
    for i, c in enumerate(msg):
        key_c = ord(key[i % len(key)])
        msg_c = ord(c)

        encryped.append(chr((msg_c + key_c) % 127))

    return ''.join(encryped)

def func4e(str_):
    enc = funcwtf(1337)
    return enc(str_)

def funcwtf(n):
    if n > 0:
        n = n % 20
    else:
        n = -(-n % 20)

    lc = string.ascii_lowercase
    uc = string.ascii_uppercase

    trans = string.maketrans(lc + uc, lc[n:] + lc[:n] + uc[n:] + uc[:n])
    return lambda x: x.translate(trans)
```

I quickly wrote functions to reverse `func1e`, `func3e` and `func4e`, but we were left with `func2e`, which had a random component. Luckily, @Alisson googled some pieces of it and found out it was `stypr` and that the random component doesn't matter at all. We grabbed the `decrypt` function from [here](https://github.com/stypr/stypr_crypt/blob/master/crypto.py).

Final exploit:

```python
#! /usr/bin/env python
import string

def func1e(str1, str2):
    tmp = ''
    for i in range(len(str1)):
        if i % 2 == 0:
            tmp += chr((ord(str1[i]) ^ ord(str2[i])) + 4)
        else:
            tmp += chr((ord(str1[i]) ^ ord(str2[i])) - 2)

    return tmp[::-1]

def func1erev(str1, str2):
    tmp = ''
    str1 = str1[::-1]
    for i in range(len(str1)):
        if i % 2 == 0:
            tmp += chr((ord(str1[i]) - 4) ^ ord(str2[i]))
        else:
            tmp += chr((ord(str1[i]) + 2) ^ ord(str2[i]))

    return tmp

def func2ek(d, k):
    try:
        n = 0
        f = ''

        for i in range(len(d)):
            n += 1
            c = (n*n) ^ 62
            f += ('00000' + hex(int(oct(ord(d[i]) ^ (k ^ 720451 ^ 3775139 ^ c))))[2:])[-6:]

        f = (('000' + hex((k ^ 2719) ^ 59262)[2:])[-4:] + f)[::-1].upper()
        return f
    except:
        return -1

def func2erev(d):
    try:
        e = d[::-1]
        k = int(e[:4], 16) ^ 0xA9F ^ 0xE77E
        t = e[4:]
        f = ""
        n = 0
        for i in range(0, len(t), 6):
            n += 1
            c = (n * n) ^ 0x3E
            # $tmp = $tmp . chr(octdec(hexdec(substr($Temps, $i, 6))) ^ ($MyKey ^ hexdec("AFE43") ^ hexdec("399AA3") ^ $cal));
            f += chr(int(str(int(t[i:i+6], 16)), 8) ^ (k ^ 0xAFE43 ^ 0x399AA3 ^ c))
        return f
    except:
        return -1

def func3e(msg, key):
    encryped = []
    for i, c in enumerate(msg):
        key_c = ord(key[i % len(key)])
        msg_c = ord(c)

        encryped.append(chr((msg_c + key_c) % 127))

    return ''.join(encryped)

def func3erev(msg, key):
    encryped = []
    for i, c in enumerate(msg):
        key_c = ord(key[i % len(key)])
        msg_c = ord(c)

        encryped.append(chr((msg_c - key_c) % 127))

    return ''.join(encryped)

def func4e(str_):
    enc = funcwtf(1337)
    return enc(str_)

def func4erev(str_):
    dec = funcwtfrev(1337)
    return dec(str_)

def funcwtf(n):
    if n > 0:
        n = n % 20
    else:
        n = -(-n % 20)

    lc = string.ascii_lowercase
    uc = string.ascii_uppercase

    trans = string.maketrans(lc + uc, lc[n:] + lc[:n] + uc[n:] + uc[:n])
    return lambda x: x.translate(trans)

def funcwtfrev(n):
    if n > 0:
        n = n % 20
    else:
        n = -(-n % 20)

    lc = string.ascii_lowercase
    uc = string.ascii_uppercase

    trans = string.maketrans(lc[n:] + lc[:n] + uc[n:] + uc[:n], lc + uc)
    return lambda x: x.translate(trans)

if __name__ == '__main__':
    out = '0A3BFDA6EBFD5CEBFD1ADBFDBEDBFD1DFBFDF10CFD51FBFD51FBFDBEEBFDB3ABFD3DABFD2DABFD589BFDD79BFD9E9BFD10ABFDDFBBFDAFBBFDD4CBFD77CBFD42BBFD91BBFDC2BBFDB4BBFDE7BBFDB6BBFDF0BBFD2ABBFD22BBFD39BBFDE2BBFDB1CE'

    print func1erev(func3erev(func4erev(func2erev(out)), 'looooool'), 'a6105c0a611b41b08f1209506350279e')
```
