---
title: AsisCTF 2018 - Trashy or Classy
layout: post
date: '2018-05-02 12:19:00'
description: Writeup of Trashy or Classy
image: https://i.imgur.com/UzrndnM.png
tags:
- ctf
- writeup
author: shrimpgo
---

### "Trashy or Classy" (500)

Description:
_Don't be_ [Trashy](https://asisctf.com/tasks/Trashy_Or_Classy_1afb5f5911a97860e181722b55dae50bb765285cd8dcbb38837d1a1094e53444). _Try being Classy!!_

---
Downloaded the file and used de `file` command to check it's signature:

```
$ file Trashy_Or_Classy_1afb5f5911a97860e181722b55dae50bb765285cd8dcbb38837d1a1094e53444
Trashy_Or_Classy_1afb5f5911a97860e181722b55dae50bb765285cd8dcbb38837d1a1094e53444: XZ compress data
```

Decompressed it using `tar` and got a file named `Trashy_Or_Classy`. Using `file` command again we get that it's a tcpdump file (pcap). I'll use wireshark for this search. At first glance, I noticed it has a lot of HTTP requests, so I filtered only this type of packet:

![alt text](https://i.imgur.com/HeAPGN8.png)

In a deeper analysis, I figured out HTTP requests brute-forcing a host's (167.99.233.88) directory. So, I've extracted all of these requests to understand what happened here (File > Export Objects > HTTP...):

![alt text](https://i.imgur.com/UzrndnM.png)

Saved all of those files in new dir called `trash`. Let's dig in!

```
$ cd trash
$ for i in $(ls); do echo -n "$i --> "; cat $i; echo ;done | less
```

![alt text](https://i.imgur.com/4pJUUB6.png)

There is a lot of `404 Not Found`! That means this really is a directory brute-force attempt! Let's check for HTTP responses that are not 404 (200, 403 etc.). Digging more, I found a `401 Unauthorized` for a private dir:

![alt text](https://i.imgur.com/JSoe3jx.png)

Cool! There is a lot of of it. That means the attacker was throwing an authentication brute-force attack on this dir. Let's see if he had success:

![alt text](https://i.imgur.com/CJcEIvl.png)

Yeah, he did it! I think these 2 items had been downloaded by him. Checking this out:

![alt text](https://i.imgur.com/jHQ4EEw.png)

![alt text](https://i.imgur.com/1q0EV0K.png)

The download of the first file (flag.caidx) was successful, but the other one (flag.castr) was forbidden (it is a directory). I don't know what kind of extension `caidx` is and, making a little Google (dumb's father) research, I've found that the `caidx` extension belongs to [casync - Content Addressable Data Synchronizer](https://github.com/systemd/casync) and this is the explanation what it is exactly for:

> What is this?

> 1. A combination of the rsync algorithm and content-addressable storage
> 2. An efficient way to store and retrieve multiple related versions of large file systems or directory trees
> 3. An efficient way to deliver and update OS, VM, IoT and container images over the Internet in an HTTP and CDN friendly way
> 4. An efficient backup system

> File Suffixes
>
> 1. catar → archive containing a directory tree (like "tar")
> 2. caidx → index file referring to a directory tree (i.e. a .catar file)
> 3. caibx → index file referring to a blob (i.e. any other file)
> 4. castr → chunk store directory (where we store chunks under their hashes)
> 5. cacnk → a compressed chunk in a chunk store (i.e. one of the files stored below a .castr directory)

> Operation on archive index files

> (...)
> \# casync mount --store=/var/lib/backup.castr /home/lennart.caidx /home/lennart
> (...)

Therefore, I need a directory which the file points to, but I only have `flag.caidx`. I don't have permission to access `/private/flag.castr`, only server has. Reading more about _casync_, there is an useful command:

> Operations involving the web

> (...)
> \# casync mount --seed=/home/lennart http://www.foobar.com/lennart.caidx /home/lennart2
> (...)

The command above gives me access to mount the directory remotely, but I need the right credentials to do it. Going back to the pcap file, I'll look for a succesful access to a private directory and, when I find it, I'll use the wireshark's `Follow HTTP Stream...`:

![alt text](https://i.imgur.com/u7L3HaO.png)

Holy shit! They're using _Digest Authentication_! OK! How is it work? At [Wikipedia](https://en.wikipedia.org/wiki/Digest_access_authentication):

> (...)
> RFC 2069 was later replaced by RFC 2617 (HTTP Authentication: Basic and Digest Access Authentication). RFC 2617 introduced a number of optional security enhancements to digest authentication; "quality of protection" (qop), nonce counter incremented by client, and a client-generated random nonce. These enhancements are designed to protect against, for example, chosen-plaintext attack cryptanalysis.
 
> If the algorithm directive's value is "MD5" or unspecified, then HA1 is
 
>     HA1=MD5(username:realm:password)
 
> If the algorithm directive's value is "MD5-sess", then HA1 is
 
>     HA1=MD5(MD5(username:realm:password):nonce:cnonce)
 
> If the qop directive's value is "auth" or is unspecified, then HA2 is
 
>     HA2=MD5(method:digestURI)
 
> If the qop directive's value is "auth-int", then HA2 is
 
>     HA2=MD5(method:digestURI:MD5(entityBody))
 
> If the qop directive's value is "auth" or "auth-int", then compute the response as follows:
 
>     response=MD5(HA1:nonce:nonceCount:cnonce:qop:HA2)
 
> If the qop directive is unspecified, then compute the response as follows:
 
>     response=MD5(HA1:nonce:HA2)
> (...)

In this case, `HA1` is `MD5` and `HA2` is `qop=auth`. Made a brute-force code in python with _Rockyou_ wordlist:

```python
import hashlib

username = "admin"
password = open('rockyou.txt', 'r').read().split('\n')
nonce = "dUASPttqBQA=7f98746b6b66730448ee30eb2cd54d36d5b9ec0c"
nc="00000001"
cnonce="edba216c81ec879e"
realm = "Private Area"
uri = "/private/"
qop = "auth"
for i in password:
    print("Testing "+i+"...")
    hash1 = hashlib.md5(username+':'+realm+':'+i).hexdigest()
    hash2 = hashlib.md5("GET:"+uri).hexdigest()
    response = hashlib.md5(hash1+':'+nonce+':'+nc+':'+cnonce+':'+qop+':'+hash2).hexdigest()
    if response == "3823c96259b479bfa6737761e0f5f1ee":
        print("Password is "+i)
        break
```

The credentials for private access is `admin:rainbow`. Installing casync:

```
# apt install sphinx-common libudev-dev libfuse-dev libzstd-dev libcurl4-openssl-dev liblzma-dev meson
$ git clone https://github.com/systemd/casync.git
$ cd casync
$ ./mkosi.build #this command will try to install binaries, but it doesn't have sudo
```

Testing:

```
# cd /usr/lib/casync/protocols/
# ln -sf /pathexample/casync/build/casync-http .
$ cd /pathexample/casync/build
$ sudo ./casync mount --store=http://admin:rainbow@167.99.233.88/private/flag.castr http://admin:rainbow@167.99.233.88/private/flag.caidx /mnt -v
Acquiring http://admin:rainbow@167.99.233.88/private/flag.caidx...
HTTP server failure 401 while requesting http://admin:rainbow@167.99.233.88/private/flag.caidx
Failed to run synchronizer: Bad message
```

WTF! Why is it happening? I've tried so many times... I had an idea: capturing packet using tcpdump and see what is going on.

![alt text](https://i.imgur.com/D4w1Myv.png)

Oh, boy, casync doesn't support Digest Authentication! If I only had it's source code, then I could try patching it! Casync has a binary file called `casync-http`, responsible to making every remote connection. Analyzing this code, I found out it uses libcurl and supports HTTP, HTTPS, FTP and SSH protocols, but no mention of the auth type. I had to read [libcurl's help](https://curl.haxx.se/libcurl/c/CURLOPT_HTTPAUTH.html) and, with a big effort from Alisson Bezerra teaching me how to patch this source code, I've included this code below (after line 275) in `src/casync-http.c`:

```C
if (arg_verbose) { //line 275
		curl_easy_setopt(curl, CURLOPT_HTTPAUTH, CURLAUTH_DIGEST); //added by me
		curl_easy_setopt(curl, CURLOPT_USERPWD, "admin:rainbow"); //added by me
                log_error("Acquiring %s...", url); //don't change this
	} //closing if statement
```

I deleted old build directory, changed the source code, built and ran it again:

```
$ cd ../build
$ sudo ./casync mount --store=http://167.99.233.88/private/flag.castr http://167.99.233.88/private/flag.caidx /mnt -v
Acquiring http://167.99.233.88/private/flag.caidx...
Acquiring http://167.99.233.88/private/flag.castr/caf4/caf4408bde20bf1a2d797286b1ad360019daa59b53e55469935c6a8443c69770.cacnk...
Acquiring http://167.99.233.88/private/flag.castr/b943/b94307380cddabe9831f56f445f26c0d836b011d3cff27b9814b0cb0524718e5.cacnk...
Acquiring http://167.99.233.88/private/flag.castr/4ace/4ace69b7c210ddb7e675a0183a88063a5d35dcf26aa5e0050c25dde35e0c2c07.cacnk...
Acquiring http://167.99.233.88/private/flag.castr/383b/383bd2a5467300dbcb4ffeaa9503f1b2df0795671995e5ce0a707436c0b47ba0.cacnk...
Acquiring http://167.99.233.88/private/flag.castr/7d72/7d722208c35583369d05d1332d2ed8ec547be58c00a2d64aa5687e66adcc58ea.cacnk...
Acquiring http://167.99.233.88/private/flag.castr/fa20/fa2027982d21e906f5c87ddb608f68805fe2ac1efca8892d5560da05472056b6.cacnk...
...
```

Now it works! I watched all of mounting process (it took so long!) and when it stopped, I looked in `/mnt` directory and there is a PNG file (flag.png). Opening this file, there was the flag:

![alt text](https://i.imgur.com/3WfuOUj.png)
