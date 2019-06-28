--- 
title: Tokyo Westerns CTF - Revolutional Secure Angou Writeup
layout: post 
date: '2018-09-03 22:35:00'
description: Writeup of Revolutional Secure Angou
image: '/assets/images/logo.png'
tags:
- ctf
- writeup
author: Marzano
---

In this challenge we are given an encrypted file, `flag.encrypted`, a public key `publickey.pem`, and a prime generator `generator.rb`.

**flag.encrypted**

```
T+03cnnbLkix6F+jVy2/XS+ianzRCTeihrOjT8ZsE+PQvK8KnTPOMEIC6aTphvsSv4bXYQUGZJpT
bXETFWOH+qylzlYMPNITfegH13yl3DD9b6Pdx5SQi+IwG6m7hgT2wD60hH/15yPYHkJzG7ctjCZG
0qniAyZCqlEoqc1cakvXWw4+IyL7esY93AnJytJXPPKx4a95dOrY1wC4JrvHsxbouiQOu9KmXhP6
po8YzsBdNUY9qbUJw5uOUJDZGxMWP8PZnsM981AegYn807cfwfoasHXLKfZqZNrKtRm3opwBcTBr
7b5YQXYTa1duRyepgFTGjkKF+h8OmAuZV+25pA==
```

**generator.rb**

```ruby
require 'openssl'
e = 65537

while true
    p = OpenSSL::BN.generate_prime(1024, false)
    q = OpenSSL::BN.new(e).mod_inverse(p)
    next unless q.prime?
    key = OpenSSL::PKey::RSA.new
    key.set_key(p.to_i * q.to_i, e, nil)
    File.write('publickey.pem', key.to_pem)
    File.binwrite('flag.encrypted', key.public_encrypt(File.binread('flag')))
    break
end
```

**pubkey.pem**

```
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAhSkGPqCtO0Ypb5L3I1Z3
LqTnA/e3kiDBjeG348oKdyjRnmncSLhoXNYE9Yh6T486lFocoVk88IbTSOxNySFC
CD/J4iA8ZTAxHuUQvlCkKu5KY+f6Zr/ONRL8L7EXQCpVzfCJd3DBu4by2TBtpbiZ
0pTtvLF62H4XWSzMP2KxMFckGBcyrHR0zyO+tyKDM3PvB7apIYjPKLz+8msjaK2j
j39P2JIdvjtkiOS5ICj/vUauJti0PJqG27xj8LUTmLtUCY/3AEtkavtC8kNUq2ot
MO/u6LMzRzq+HMkutopGWBnZ6aD/WP6vLHIq5lt87cnjC+kVAp1pNCUjuYGtg5XN
9wIDAQAB
-----END PUBLIC KEY-----
```

This is a custom RSA implementation. The difference is this line: `q = OpenSSL::BN.new(e).mod_inverse(p)` (generating a prime from another prime is usually a bad idea), so we start from there.

`N` is a 2048-bit integer and `e` is standard (65537).

Since `q := e^-1 mod p`, then `q < p` and `eq = 1 mod p`.
Since `eq = 1 mod p`, then there is an `x` such that `eq = 1 + px` and `0 < x < p`.

Let's approximate this equation by `eq ~= px` for a second. Then `eq^2 ~= nx` and therefore `q^2 ~= Nx/e` and `q ~= sqrt(Nx/e)`.

Let's not forget that `x` is unknown and at first it seems we can't bruteforce it, because it could be as large as `p` (1024 bit). Except that, since `eq ~= Nx`, then `e(q^2) ~= (pq)x`. If `q < p`, then `q^2 < pq`. For this equality to hold, we must have `e > x`. This way we have a small range for `x` (`0 < x < 65537`), allowing us to bruteforce it.

Also note the effect of the approximation is going to be small. If we had kept the `1` we'd have `q = sqrt((nx + p)/e)`. Since `nx/e` is around 2048 bits and `p/e` is around 1024 bits (**a lot** smaller), when we apply the square root the effect of ignoring this extra `p` is going to be very small, although it must be considered.

The idea is that we generate approximations for `q` and then search around these numbers for divisors of `N`:

```python
#!/usr/bin/python
from gmpy2 import isqrt
from sys import exit

e = 0x10001
N = 16809924442712290290403972268146404729136337398387543585587922385691232205208904952456166894756423463681417301476531768597525526095592145907599331332888256802856883222089636138597763209373618772218321592840374842334044137335907260797472710869521753591357268215122104298868917562185292900513866206744431640042086483729385911318269030906569639399362889194207326479627835332258695805485714124959985930862377523511276514446771151440627624648692470758438999548140726103882523526460632932758848850419784646449190855119546581907152400013892131830430363417922752725911748860326944837167427691071306540321213837143845664837111
delta = 50

for x in range(1, e):
    q_approx = isqrt(N*x/e)
    for q in range(q_approx - delta, q_approx + delta):
        if N % q == 0:
            print 'P:', N/q
            print 'Q:', q
            exit(0)
```

With `p` and `q`, we generate a PEM-encoded private key with [rsatool.py](https://github.com/ius/rsatool):

```bash
$ ./rsatool.py -p <P HERE> -q <Q HERE> -o out.pem
```

And now we decrypt the flag with `OpenSSL`:

```bash
$ openssl rsautl -inkey out.pem -in flag.encrypted -decrypt
TWCTF{9c10a83c122a9adfe6586f498655016d3267f195}
```
