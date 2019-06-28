--- 
title: RITSEC CTF 2018 - PCAP Me If You Can 
layout: post 
date: '2018-11-19 08:30:00'
description: Writeup of PCAP Me If You Can
image: '/assets/images/logo.png'
tags:
- ctf
- writeup
author: Kauê Doretto
---

>PCAP Me If You Can (forensics 300)
>
>The hackers have written their own protocol for their MALL-ware. Can you figure out what they're saying?
>Hint 1: custom protocols are hard
>Hint 2: Don't worry, I won't make you **decrypt** anything in this challenge.
>
>_Author: hulto_
[pcapmeifyoucan.pcapng](https://ctf.ritsec.club/files/0c7298192e137cfdb71a1102f6495d75/pcapmeifyoucan.pcapng)

The challenge was about locating a custom protocol among other usual traffic, so the first task was to look for abnormal packets in regular protocols or an unrecognized protocol itself. The first approach was to list all unique protocols captured:

```bash
$ tshark -r pcapmeifyoucan.pcapng -T fields -e frame.protocols | sort | uniq

eth:ethertype:arp
eth:ethertype:ip:icmp:ip:udp:data
eth:ethertype:ip:icmp:ip:udp:dns
eth:ethertype:ip:tcp
eth:ethertype:ip:tcp:data
eth:ethertype:ip:tcp:http
eth:ethertype:ip:tcp:http:data-text-lines
eth:ethertype:ip:tcp:ssh
eth:ethertype:ip:tcp:ssl
eth:ethertype:ip:tcp:ssl:ocsp:ssl
eth:ethertype:ip:tcp:ssl:ssl
eth:ethertype:ip:tcp:ssl:x509sat:x509sat:x509sat:x509sat:x509ce:x509ce:x509ce:x509ce:x509ce:pkix1implicit:x509ce:x509ce:pkix1explicit:pkix1implicit:ssl:x509sat:x509sat:x509sat:x509sat:x509sat:x509ce:x509ce:pkix1implicit:x509ce:x509ce:pkix1explicit:x509ce:x509ce:ssl
eth:ethertype:ip:tcp:ssl:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:pkcs-1:x509ce:x509ce:x509ce:pkix1implicit:x509ce:x509ce:x509ce:x509ce:x509ce:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509ce:x509ce:x509ce:x509ce:x509ce:pkix1implicit:x509ce:x509ce:pkix1explicit:ssl
eth:ethertype:ip:tcp:ssl:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509ce:x509ce:pkix1implicit:x509ce:x509ce:x509ce:x509ce:x509ce:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509ce:x509ce:x509ce:x509ce:x509ce:pkix1implicit:x509ce:x509ce:pkix1explicit:ssl
eth:ethertype:ip:tcp:ssl:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509ce:x509ce:x509ce:x509ce:x509ce:x509ce:pkix1explicit:x509ce:pkix1implicit:x509ce:ssl:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509ce:x509ce:x509ce:x509ce:x509ce:x509ce:x509ce:pkix1implicit:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509ce:x509ce:x509ce:x509ce:x509ce:x509ce:pkix1implicit:ssl:ocsp
eth:ethertype:ip:tcp:ssl:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509ce:x509ce:x509ce:x509ce:x509ce:x509ce:x509ce:pkix1explicit:pkix1implicit:x509ce:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509ce:x509ce:pkix1implicit:x509ce:x509ce:pkix1explicit:x509ce:x509ce:ssl
eth:ethertype:ip:tcp:ssl:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509ce:x509ce:x509ce:x509ce:x509ce:x509ce:x509ce:pkix1explicit:pkix1implicit:x509ce:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509ce:x509ce:pkix1implicit:x509ce:x509ce:pkix1explicit:x509ce:x509ce:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509ce:x509ce:x509ce:x509ce:ssl
eth:ethertype:ip:tcp:ssl:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509ce:x509ce:x509ce:x509ce:x509ce:x509ce:x509ce:pkix1explicit:pkix1implicit:x509ce:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509ce:x509ce:pkix1implicit:x509ce:x509ce:pkix1explicit:x509ce:x509ce:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509ce:x509ce:x509ce:x509ce:ssl:ocsp
eth:ethertype:ip:tcp:ssl:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509ce:x509ce:x509ce:x509ce:x509ce:x509ce:x509ce:pkix1explicit:pkix1implicit:x509ce:ssl:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509ce:x509ce:pkix1implicit:x509ce:x509ce:pkix1explicit:x509ce:x509ce:ssl
eth:ethertype:ip:tcp:ssl:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509ce:x509ce:x509ce:x509ce:x509ce:x509ce:x509ce:pkix1explicit:pkix1implicit:x509ce:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509ce:x509ce:pkix1implicit:x509ce:x509ce:pkix1explicit:x509ce:x509ce:ssl
eth:ethertype:ip:tcp:ssl:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509ce:x509ce:x509ce:x509ce:x509ce:x509ce:x509ce:pkix1explicit:pkix1implicit:x509ce:ssl:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509ce:x509ce:x509ce:pkix1implicit:x509ce:x509ce:pkix1explicit:x509ce:x509ce:ssl
eth:ethertype:ip:tcp:ssl:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509ce:x509ce:x509ce:x509ce:x509ce:x509ce:x509ce:pkix1explicit:pkix1implicit:x509ce:ssl:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509sat:x509ce:x509ce:x509ce:pkix1implicit:x509ce:x509ce:pkix1explicit:x509ce:x509ce:ssl
eth:ethertype:ip:udp:data
eth:ethertype:ip:udp:dns
eth:ethertype:ip:udp:mdns
eth:ethertype:ip:udp:nbns
eth:ethertype:ipv6:udp:mdns
```

Discarding the ssl protocols a few other regular ones came up. The first choice was the DNS protocol, used for data exfiltration, but nothing out of the ordinary was found. Next the ARP and HTTP protocols were examined. Nothing, except this: 

```
GET /watch?v=dQw4w9WgXcQ HTTP/1.1
Host: www.youtube.com
User-Agent: curl/7.61.0
Accept: */*
```

Let's try to get something from the used ports. By building a filter to list the unique destination ports (therefore, identifying servers) and ignoring the ones in the ephemeral range (32768+)...:


```bash
$ tshark -r pcapmeifyoucan.pcapng -T fields -e tcp.dstport tcp.dstport lt 32768 | sort | uniq
22
443
80
8888
```

What is that 8888 port doing there? Looks promising, let's check these packets. 


```bash
$ tshark -r pcapmeifyoucan.pcapng tcp.port eq 8888
13990  68.058515 172.16.140.131 45826 172.16.140.1 8888 TCP 74  45826 → 8888 [SYN] Seq=0 Win=29200 Len=0 MSS=1460 SACK_PERM=1 TSval=2111755176 TSecr=0 WS=128
13991  68.058661 172.16.140.1 8888 172.16.140.131 45826 TCP 78  8888 → 45826 [SYN, ACK] Seq=0 Ack=1 Win=65535 Len=0 MSS=1460 WS=32 TSval=676410590 TSecr=2111755176 SACK_PERM=1
13992  68.058814 172.16.140.131 45826 172.16.140.1 8888 TCP 66  45826 → 8888 [ACK] Seq=1 Ack=1 Win=29312 Len=0 TSval=2111755177 TSecr=676410590
13993  68.058874 172.16.140.1 8888 172.16.140.131 45826 TCP 66  [TCP Window Update] 8888 → 45826 [ACK] Seq=1 Ack=1 Win=131744 Len=0 TSval=676410591 TSecr=2111755177
13994  68.058952 172.16.140.131 45826 172.16.140.1 8888 TCP 112  45826 → 8888 [PSH, ACK] Seq=1 Ack=1 Win=29312 Len=46 TSval=2111755177 TSecr=676410590
13995  68.059025 172.16.140.1 8888 172.16.140.131 45826 TCP 66  8888 → 45826 [ACK] Seq=1 Ack=47 Win=131712 Len=0 TSval=676410591 TSecr=2111755177
13998  68.071416 172.16.140.1 8888 172.16.140.131 45826 TCP 74  8888 → 45826 [PSH, ACK] Seq=1 Ack=47 Win=131712 Len=8 TSval=676410603 TSecr=2111755177
13999  68.071475 172.16.140.1 8888 172.16.140.131 45826 TCP 66  8888 → 45826 [FIN, ACK] Seq=9 Ack=47 Win=131712 Len=0 TSval=676410603 TSecr=2111755177
14000  68.071634 172.16.140.131 45826 172.16.140.1 8888 TCP 66  45826 → 8888 [ACK] Seq=47 Ack=9 Win=29312 Len=0 TSval=2111755190 TSecr=676410603
14001  68.071832 172.16.140.131 45826 172.16.140.1 8888 TCP 66  45826 → 8888 [FIN, ACK] Seq=47 Ack=10 Win=29312 Len=0 TSval=2111755190 TSecr=676410603
14002  68.071904 172.16.140.1 8888 172.16.140.131 45826 TCP 66  8888 → 45826 [ACK] Seq=10 Ack=48 Win=131712 Len=0 TSval=676410603 TSecr=2111755190
[...]
```

It's a brief but regular TCP communication: a handshake, some data goes, some data comes and finish. Data, huh? The data section was present in the PSH,ACK packets, so these were filtered:

```bash
$ tshark -r pcapmeifyoucan.pcapng -T fields -e data tcp.flags eq 0x018 and tcp.port eq 8888
566572622e5245414400eee3ecf2dface5edece300afb1b1b5e6dff6edf0f1d0e1aeaeea00adf2ebeeadedf3f200
c3d0d0cdd09e1337
566572622e575249544500eee3ecf2dface5edece300afb1b1b5e6dff6edf0f1d0e1aeaeea00a0f5e3eaea9ee6edf5e2f7a000
f5e3eaea9ee6edf5e2f79eadf2ebeeadedf3f2881337
566572622e5245414400eee3ecf2dface5edece300afb1b1b5e6dff6edf0f1d0e1aeaeea00adf2ebeeadedf3f200
c3d0d0cdd09e1337
566572622e575249544500eee3ecf2dface5edece300afb1b1b5e6dff6edf0f1d0e1aeaeea00cdcbc59ee6edf59edff0e39ef7edf3bd00
cdcbc59ee6edf59edff0e39ef7edf3bd9eadf2ebeeadedf3f2881337
566572622e5245414400eee3ecf2dface5edece300afb1b1b5e6dff6edf0f1d0e1aeaeea00adf2ebeeadedf3f200
cdcbc59ee6edf59edff0e39ef7edf3bd9e881337
566572622e5245414400eee3ecf2dface5edece300afb1b1b5e6dff6edf0f1d0e1aeaeea00f5e6eddfebe700
c3d0d0cdd09e1337
566572622e5245414400eee3ecf2dface5edece300afb1b1b5e6dff6edf0f1d0e1aeaeea00adf2ebeeadedf3f2b99ef5e6eddfebe700
cdcbc59ee6edf59edff0e39ef7edf3bd9e88e6f3eaf2ed881337
566572622e5245414400eee3ecf2dface5edece300afb1b1b5e6dff6edf0f1d0e1aeaeea00adf2ebeeade2dff2df00
d0c7d2d1c3c1f9d2e6aff1dde7f1ddcbf7ddcedfb3f1f5aef0e2ddd2e6b1f0e3dddff0e3ddcbdfecf7ddeae7e9e3dde7f2dde0f3b5ddf2e6e7b3ddafdde7f1ddebe7ece3fb881337
566572622e575249544500eee3ecf2dface5edece300afb1b1b5e6dff6edf0f1d0e1aeaeea00f7aeeaae9eebdfec00
f7aeeaae9eebdfec9eadf2ebeeadedf3f2881337
566572622e5245414400eee3ecf2dface5edece300afb1b1b5e6dff6edf0f1d0e1aeaeea00adf2ebeeadedf3f200
f7aeeaae9eebdfec881337
```

This output was saved in a file named `comm.txt` and analyzed. First things noted were:
1) it looks like a request and a response communication style; 
2) the `00` value looks like as a separator;
3) there's a "1337" (leet) in all the reponses, maybe a breadcrumb hint;

The longer string (the request) was converted to ASCII, using every pair of chars as representation of a byte:

```
566572622e52454144  eee3ecf2dface5edece3  afb1b1b5e6dff6edf0f1d0e1aeaeea  adf2ebeeadedf3f200
Verb.READ           îãìòß¬åíìã            ¯±±µæßöíðñÐá®®ê ­                òëî­íóò
```

"Verb.READ" sounds good. There are three other similar strings with different value for the first part:

```
566572622e5752495445  eee3ecf2dface5edece3  afb1b1b5e6dff6edf0f1d0e1aeaeea  a0f5e3eaea9ee6edf5e2f7a0
Verb.WRITE            îãìòß¬åíìã            ¯±±µæßöíðñÐá®®ê                  õãêêæíõâ÷
```

"Verb.WRITE". Sweet. Now let's remove everything that's redundant and work only with the differences:

```
V E R B . R E A D |
adf2ebeeadedf3f2                    00
c3d0d0cdd09e1337

V E R B . W R I T E |
a0  f5e3eaea9ee6edf5e2f7    a0      00
    f5e3eaea9ee6edf5e2f7            9eadf2ebeeadedf3f2881337
                                                                                                                                
V E R B . R E A D |
adf2ebeeadedf3f2                    00
c3d0d0cdd09e1337

V E R B . W R I T E |
cdcbc59ee6edf59edff0e39ef7edf3bd    00
cdcbc59ee6edf59edff0e39ef7edf3bd    9eadf2ebeeadedf3f2881337
                                                                                                                                
V E R B . R E A D |
adf2ebeeadedf3f2                    00
cdcbc59ee6edf59edff0e39ef7edf3bd9e881337

V E R B . R E A D |
f5e6eddfebe7                        00
c3d0d0cdd09e1337

V E R B . R E A D |
adf2ebeeadedf3f2b99ef5e6eddfebe7    00
cdcbc59ee6edf59edff0e39ef7edf3bd9e88e6f3eaf2ed881337

V E R B . R E A D |
adf2ebeeade2dff2df                  00
d0c7d2d1c3c1f9d2e6aff1dde7f1ddcbf7ddcedfb3f1f5aef0e2ddd2e6b1f0e3dddff0e3ddcbdfecf7ddeae7e9e3dde7f2dde0f3b5ddf2e6e7b3ddafdde7f1ddebe7ece3fb881337

V E R B . W R I T E |
f7aeeaae9eebdfec                    00
f7aeeaae9eebdfec                    9eadf2ebeeadedf3f2881337
                                                                                                                                
V E R B . R E A D |
adf2ebeeadedf3f2                    00
f7aeeaae9eebdfec881337
```

The first visible pattern is that every "WRITE" receives as response the request itself and some other fixed-value string. Probably a success or error message. Let's ignore this for now. The response to the sixth "READ" seems off from the others, let's dig deeper there.

```
    d0  c7  d2  d1 [...]
dec 208 199 210 209
```

We know that the flag begins with "RITSEC", right? The decimal distance from the first to the fourth value is 1, from the fourth to the third is also 1 and from the first to the third is 2. You get the picture:

```
    d0  c7  d2  d1
dec 208 199 210 209
    R   I   T   S 
dec 82  73  84  83

208 - 82 = 199 - 73 = 210 - 84 = 209 - 83 = 126
```

Seems that we found the secret protocol. The python program `decode.py` below decodes all the communication.

```python
# Open and read the file containing all the packets' data field
with open('comm.txt') as f:
    data = f.read().strip().split("\n")
    f.close()

# Read every two lines as one "communication" (request + response)
for communication in range(0, len(data), 2):
    print("Request: ")

    # Group the data on a 2-char basis and split it when equal to "00"
    request = []
    str = ""
    for i in range(0, len(data[communication]), 2):
        if(data[communication][i:i+2] == "00"):
            request.append(str)
            str = ""
        else:
            str += data[communication][i:i+2]

    # Now that the request was split into separated parts, we'll convert
    # each of these parts to a string, grouping it into pairs of chars todo
    # make one byte
    for part in range(0, len(request)):

        # "Bytefy" the part being parsed
        for i in range(0, len(request[part]), 2):

            byte = int(request[part][i:i+2], 16)

            if byte >= 126:
                byte -= 126

            # Print the byte's equivalent ASCII char
            print(chr(byte), end="")

        print(" | ", end="")

    print("\n")
    print("Response: ")

    # The response is less complex. Every strings ends with the "1337" string
    # and has no internal separators
    response = data[communication + 1].strip("1337")
    # "Bytefy" the string
    response = [(response[i:i+2]) for i in range(0, len(response), 2)]

    for byte in range(0, len(response)):
        print(chr(int(response[byte], 16) - 126), end="")

    print("")

    print("=================================================================")
```

Result:

```bash
$ python3 decode.py 
Request: 
Verb.READ | penta.gone | 1337haxorsRc00l | /tmp/out | 

Response: 
ERROR 
=================================================================
Request: 
Verb.WRITE | penta.gone | 1337haxorsRc00l | "well howdy" | 

Response: 
well howdy /tmp/out

=================================================================
Request: 
Verb.READ | penta.gone | 1337haxorsRc00l | /tmp/out | 

Response: 
ERROR 
=================================================================
Request: 
Verb.WRITE | penta.gone | 1337haxorsRc00l | OMG how are you? | 

Response: 
OMG how are you? /tmp/out

=================================================================
Request: 
Verb.READ | penta.gone | 1337haxorsRc00l | /tmp/out | 

Response: 
OMG how are you? 

=================================================================
Request: 
Verb.READ | penta.gone | 1337haxorsRc00l | whoami | 

Response: 
ERROR 
=================================================================
Request: 
Verb.READ | penta.gone | 1337haxorsRc00l | /tmp/out; whoami | 

Response: 
OMG how are you? 
hulto

=================================================================
Request: 
Verb.READ | penta.gone | 1337haxorsRc00l | /tmp/data | 

Response: 
RITSEC{Th1s_is_My_Pa5sw0rd_Th3re_are_Many_like_it_bu7_thi5_1_is_mine}

=================================================================
Request: 
Verb.WRITE | penta.gone | 1337haxorsRc00l | y0l0 man | 

Response: 
y0l0 man /tmp/out

=================================================================
Request: 
Verb.READ | penta.gone | 1337haxorsRc00l | /tmp/out | 

Response: 
y0l0 man

=================================================================
```
