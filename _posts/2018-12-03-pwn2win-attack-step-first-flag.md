--- 
title: Pwn2Win 2018 - Attack Step [First Flag]
layout: post 
date: '2018-12-03 17:15:00'
description: Writeup of Attack Step [First Flag]
image: https://i.imgur.com/fnQFZ4f.jpg
tags:
- ctf
- writeup
author: Rafael "rasknikov" Correia
---

These were the steps to solve the First Flag of Attack Step. We solved this with little time left so we couldn't try to solve the Second Part.

## Description

>We have gotten information that The Bavarian have their own authentication system, which allows through their firewall only those who manage to pass their sophisticate handshake, leading to the group's private News System. There are also rumors that they manage their botnet through a control panel located in their internal network, and that they have control of a very important Russian computer. We must try to access this machine/victim, directly in the currently logged user's screen. Help us in this long and complex mission!
>
>We figured out there is a Control Packet that is sent to a random port. It's fundamental to understand that handshake.
>
>This challenge's steps include:
>
>Networking -> Networking[1st flag here] -> Networking -> Web -> Pwnage[2nd flag here]
>
>nc 200.136.252.32 9449

## Write-up

The first step was to connect to a remote port. 

```
nc 200.136.252.32 9449
```

When connecting to this port one message was returned saying that my IP was not inside the servers.*.bavarian.world domain.

```
Your connection is not coming from servers.*.bavarian.world, handshake denied!
```

I asked the guys from my team and they already had cracked this.

Marzano and ShrimpGo found out the bavarian.world domain was free and it was possible to configure an A DNS record to our IP in afraid.org.

```
bavarian.world IN NS ns1.afraid.org 3600s (01:00:00)
bavarian.world IN NS ns4.afraid.org 3600s (01:00:00)
bavarian.world IN NS ns3.afraid.org 3600s (01:00:00)
bavarian.world IN NS ns2.afraid.org 3600s (01:00:00)
```

![Insertion of subdomains in Afraid.org](https://i.imgur.com/95HCK5a.png)

Shrimp used a server on Scaleway and it was easy to configure a reverse lookup, as shown here: `https://www.scaleway.com/docs/how-to-configure-reverse-dns/`

(I tried the same with Google and Amazon but it failed. Both need you to own the domain)

With the server configured, the nc worked and this was shown:

```
user@server1:~$ nc 200.136.252.32 9449

Handshake started, disable your firewall.
Step 1 done.
Sending control packet to port 1943.
Step 2 done.
Code? blah
Wrong Key, authentication failure!
```

Code typed, message of authentication failure received. So, we needed to understand how the handshake works.

The control packet was something like that:

`xor 194`

The operator changed randomly from `xor` to `mod` and the number was apparently random too.

These two guys saw that a lot of `SYN` packets were being received by the server after the control packet. All the communication can be see below:

![Packets between our server and the challenge server](https://i.imgur.com/doV3NAp.png)

Here I started to try to decipher the handshake together with Marzano and ShrimpGo and we failed for several hours. They already had a script with all implemented, including the sniffer for the `SYN` packets. I tried to concat the ports, sum all, xor/mod after sum, xor/mod after concat... Nothing.

Early in the sunday I woke up and I saw a hint:

```
admin: [Attack Step] To complete the handshake, the code expected is in ASCII (of course) and starts with Code (you will know when you find it) :)
```

All made sense to me after this message. I saw 9 SYN packets that were received and sometimes (when the control packet was mod) it was easy to distinct the two groups of 4 and 5 characters. I tried to `xor` the first 4 ports and the `Code` word showed up. And then the only guessing part of the challenge showed up.

xor'ring'/mod'ding' the last 5 bytes and converting them to ASCII didn't work either because some bytes got more than decimal 255 and could not be converted to ASCII. I asked the admin and he told me to not convert the last parts. The first 4 we converted from byte to text, the last 5 converted from the byte to its decimal format. Different conversions for the two parts sounds guessing to me.

Authentication completed, the firewall opened for us.

(below is an idea of the script's output after running it, as the service is unavailable at the time of the write-up)

```bash
user@server1:~$ sudo python solve.py

Handshake started, disable your firewall.
Step 1 done.
Sending control packet to port 1112.
Step 2 done.
Code? Code64699234114
Authentication completed.
Firewall has been opened for you.
```

The finished script for the handshake:

```python
#!/usr/bin/env python

from pwn import *
from scapy.all import *
from threading import Thread
import os
import sys

dports = []
pkts = []

class MySniffer(Thread):
    def  __init__(self):
        super(MySniffer, self).__init__()

    def run(self):
        sniff(filter="tcp and src host 200.136.252.32 and tcp[tcpflags] & (tcp-syn) != 0", prn=self.print_packet)

    def print_packet(self, packet):
        tcp = packet.getlayer(TCP)
        pkts.append(packet)
        if tcp:
            dports.append(tcp.dport)

sniffer = MySniffer()
sniffer.start()
r = remote('200.136.252.32', 9449)
if 'Handshake' in r.recv(1024):
    r.recvuntil('port')

    port = int(r.recvuntil('.')[:-1])

    print 'Port', port

    l = listen(port)
    conn = l.wait_for_connection()

    control = l.recv(1024).rstrip()
    print control

    op = control.split(' ')

    r.recvuntil('Code? ')
    dports = dports[-5:]
    print 'Dports', dports
    code = ""
    if op[0] == 'xor':
        for port in dports:
            code += str(port ^ int(op[1]))
            print code
    if op[0] == 'mod':
        for port in dports:
            code += str(port % int(op[1]))
            print code
    r.sendline('Code'+code)
    r.interactive()
    os._exit(0)
```

I ran nmap on the IP and the port 22 showed up.

(The first time I ran nmap only the port 65222 showed up and I spent some time on this. It was the management port of the server :P)

I connected to port 22 and the system asked the input of an IP address to send the news:

```
user@server1:~$ nc 200.136.252.32 22
Bavarian News System v1.0 - To receive the document containing the news, enter your IP address:
```

After inputting the IP address, the connection was kept stablished for sometime but closed with no further info. I ran tcpdump to see if they were connecting on us:

![Connection to the 6201 port](https://i.imgur.com/iXn4ZWP.png)

As I thought, they are connecting to another port. In this case, 6201. Let's listen to this port to check what data will be received:

```
user@server1:~$ sudo nc -nlvp 6201
listening on [any] 6201 ...
connect to [10.7.10.171] from (200.136.252.32) [200.136.252.32] 44727
l
```

What is this l? Maybe it is expecting another text? But what? Looking into the packets saved with tcpdump I found these bytes:

![Data to the 6201](https://i.imgur.com/QfeVd9I.png)

Just the `6c` was printable, so that's why I saw a l in netcat. 6c000b000000000000000000 looks like the start of some protocol, but which one? I threw this inside our team's chat and Adriano, our captain, found this on Google:

```html
<ServiceTrigger timeout="6000" name="X11" data="6c000b000000000000000000">
<Endpoint protocol="tcp">6000-6009</Endpoint>

Source: https://github.com/brl/netifera/blob/master/platform/com.netifera.platform.net/com.netifera.platform.net.services.detection/Triggers.xml
```

Nice! But... How to publish X11 to the world? While I was trying to install Xorg on the cloud Adriano shone again.

```
Xvfb :0 -screen - 1024x768x24 -ac &

Source: https://github.com/FFCrewCTF/ctf-writeups/blob/master/2016-10-26-ekoparty/misc_150/README.md
```

Great! I didn't even know about Xvfb. Now I know Xvfb is like a normal X server, but using a virtual memory instead a screen output. In the middle of the CTF I was calling it X simulator. :D

I needed to ask Xvfb to listen to tcp, as it is disable by default now. Here I started to use another server to segregate duties and to not be confused in the middle of the CTF:

```
user@server2:~$ sudo Xvfb :0 -screen - 1024x768x24 -listen tcp -ac &
```

Xvfb listens on port 6000 by default, and the challenge server was connecting to 6201. Without enough time to study how to change the port, I created a NAT rule on iptables to make it easier for me:

```
user@server2:~$ sudo iptables -t nat -I PREROUTING --src 200.136.252.32 -p tcp --dport 1:65535 -j REDIRECT --to-ports 6000
```

Let's capture the communication.

```
user@server2:~$ sudo tcpdump -vvvv host 200.136.252.32 -w capture.cap
```

Lots of packets were captured - quite different of the single one packet captured when netcat was listening - so this worked, I think. Now, how to extract images from the capture file?

As I did not find on Wireshark nothing ready to extract X11 images I analyzed the full pcap. I found nothing relevant besides a command that the challenge server wrote (xview file.jpg, I think). I searched on Google and found a tool named Chaosreader that could help me. However, it didn't work for me.

And then I thought: maybe I can take screenshots of a X server. I read some articles and tested some commands until I found one using ImageMagick:

```
import -window root screen.jpeg
```

I wrote a bash script to run it every second, for 100 seconds:

```bash
#!/bin/bash

for i in `seq 1 100`
do
    import -window root screen$i.jpg
    sleep 1
done
```

A lot of screenshots were taken (most of them black screens). And then this image is shown:

![News document after server X screenshot](https://i.imgur.com/fnQFZ4f.jpg)

Ok, but **WHERE IS THE GOD DAMN FLAG???**

Oh, maybe the screen is to small for the image...

I changed my Xvfb command and captured the screenshots again.

```
user@server2:~$ sudo Xvfb :0 -screen - 1920x1080x24 -listen tcp -ac &
```

![Full document after enlarge the screen](https://i.imgur.com/ZCLMUgs.jpg)

Here is it... finally! The flag was at the bottom of the image, so a small screen would not show it.

Flag: `CTF-BR{n3tw0rk1ng_1s_alw4ys_v3ry_funNNNNNNNNN}`

## Thanks

Thank you, Pwn2Win! Challenging challenge. :)

And thank you, FireShell. Our team work is awesome!
