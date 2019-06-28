---
title: NeverLAN - Cookie Monster
layout: post
date: '2018-02-27 12:19:00'
description: Writeup of Cookie Monster
image: '/assets/images/logo.png'
tags:
- ctf
- writeup
author: Kauê Doretto
---

### "cookie_monster" (50)

Description:
> [http://neverlanctf-challenges-elb-2146429546.us-west-2.elb.amazonaws.com:14098/](http://neverlanctf-challenges-elb-2146429546.us-west-2.elb.amazonaws.com:14098/){:target="_blank" rel="noopenner noreferrer"}


---
1.A little recon in case you don't know who is "cookie monster"
+ Google search "cookie monster"
+ It's a character from Sesame Street

2.The description's URL gives you only this string
> He's my favorite Red guy
+ Google search "cookie monster best friend red"
+ It's a character named Elmo

3.Open the browser's web developer tools and check the request headers, in the "network" tab

4.Notice the cookie
> Red_Guy's_name=NameGoesHere

5.Send the request with his name
```
$ curl -v --cookie "Red_Guy's_name=Elmo" http://neverlanctf-challenges-elb-2146429546.us-west-2.elb.amazonaws.com:14098/
*   Trying 52.10.167.210...
* TCP_NODELAY set
* Connected to neverlanctf-challenges-elb-2146429546.us-west-2.elb.amazonaws.com (52.10.167.210) port 14098 (#0)
> GET / HTTP/1.1
> Host: neverlanctf-challenges-elb-2146429546.us-west-2.elb.amazonaws.com:14098
> User-Agent: curl/7.55.1
> Accept: */*
> Cookie: Red_Guy's_name=Elmo
> 
< HTTP/1.1 200 OK
< Content-Type: text/html; charset=UTF-8
< Date: Tue, 27 Feb 2018 03:30:44 GMT
< Server: nginx
< X-Powered-By: PHP/7.1.12
< Content-Length: 160
< Connection: keep-alive
< 
<!DOCTYPE html>
<html>
    <head>
        <title>Cookie_monster</title>
    </head>
    <body>
    <p>You got it! **flag{C00kies_4r3_the_b3st}**</p></body>
</html>
* Connection #0 to host neverlanctf-challenges-elb-2146429546.us-west-2.elb.amazonaws.com left intact
```
