---
title: CSAW 2017 Quals - Serial - [PT-BR]
date: '2017-10-12 16:28:18'
image: '/assets/images/posts/writeup/byte.png'
tags:
- writeup
- ctf
layout: post
description: Solução da chall Serial do CTF CSAW 2017 Quals
author: Gilmar
---

### Challenge:

Temos um servidor dando para a porta 4239, conectando via nc o servidor retornou o seguinte:

``` ruby
nc misc.chal.csaw.io 4239
```

### Análise:

Quando enviamos dados, podemos ter algumas vezes um erro na transmissão, então cada vez que enviamos um byte, contamos o número de "1" nesse byte e, se o número é mesmo, adicionamos "0" no nosso byte, se o número de "1" é estranho, adicionamos "1" ao nosso byte.
Exemplo:

> 0110 1100 0: aqui o número de "1" é 4, 4% 2 == 0, então 4 é mesmo assim, o bit de paridade deve ser 0, sem erro na transmissão

> 0001 0110 0: aqui o número de "1" é 3, 3% 2 == 1, então 3 é estranho, o bit de paridade deve ser 1, eles têm um erro na transmissão

> 0101 1110 1: aqui o número de "1" é 5, 5% 2 == 1, então 5 é estranho, o número de paridade deve ser 1, sem erro na transmissão

O N em 8-n-1 não é teste de paridade, em nosso desafio temos 8-1-1, então nós temos um teste de paridade em nosso byte.

![byte]({{ site.url }}/assets/images/posts/writeup/byte.png)

Cada vez que temos 11 bits, o primeiro bit é sempre "0", ele é o byte inicial, então nós temos o nosso byte (8bits), então um bit para paridade, um bit que é sempre "1", isso significa que ele irá parar!

### Solução:

Em cada etapa verificamos se eles não têm erro de transmissão contando o número de "1" em nosso byte, então se for igual ao nosso bit de paridade enviamos "1" para obter o próximo byte, caso contrário enviamos 0 para obter o byte correto e convertemos cada byte no caracter ascci para obter a flag!.

```
solve.py
```
``` python
from pwn import *
from re import compile

r = remote("misc.chal.csaw.io", 4239)

response = compile(r'[01]{11}')
query = r.recvline(512).rstrip()

flag = ''
solve = response.search(query)
while True:
    string = solve.group()
    serial = string[1:9]
    parity = int(string[-2])
    if (serial.count('1') % 2) == parity:
        flag += chr(int(serial,2))
        print flag
        r.sendline('1\n')
    else :
        r.sendline('0\n')
    query = r.recvline(512).strip()
    solve = response.search(query)
    
print  flag
```

![solution]({{ site.url }}/assets/images/posts/writeup/flag.png)

```
flag{@n_int3rface_betw33n_data_term1nal_3quipment_and_d@t@_circuit-term1nating_3quipment}
```

### Referência

[Misc 50 Serial - By Nazime](https://noobsinthehood.gitbooks.io/nith/content/misc-50-serial.html){:target="_blank" rel="noopenner noreferrer"}
