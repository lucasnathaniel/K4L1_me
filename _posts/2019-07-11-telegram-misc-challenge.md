---
title: CyBRICS CTF Quals 2019 - Telegram
layout: post 
date: '2019-07-25 18:30:00'
description: writeup for misc Telegram
tags:
- ctf
- writeup
- stego
- misc
- infosec
---

## Telegram(misc) - CyBRICS Quals 2019

>Hi everyone! C:
>This challenge was funny, I solved with my team

## Description
> Author: Alexander Menshchikov (n0str)
>
>This Telegram bot really loves live face-to-face communication! And it also seems to have some covert channel!
>
>@cybrics_facetoface_bot
{:.blockquote}

## The ~~*Jurandir*~~ Solver

So basically this is a Telegram bot that sends and receives TVN (Telegram Video Note). You can simply upload this type of video using your mobile phone or Telegram API. The bot does not receive normal or larger videos than 150 KB TVN.

When we sent a small TVN, the bot checks to see if the file is correct, but does not recognize a `value secrets sign`.

BTW, let's check out these bot videos... So that you can inspect the page and see the thumbnail, there is some corner information you can download by copying the video while playing, opening the tab and saving.

![](https://i.imgur.com/xduj8XN.png)


The text in the video is: `send video note to the bot with four green circles in the corners like the one above`.

So, let's do it :D

We can use [Gimp](https://www.gimp.org/) to create a `png` file with each green(0, 255, 0) circle in the corners and [OpenShot](https://www.openshot.org/) to mount.

To create a TVN, we can use the [Telegram API](https://telegrambots.github.io/book/2/send-msg/video-video_note-msg.html), written in C#. Following the documentation we get this and this is the script below:


```cs
using System;
using System.Threading;
using Telegram.Bot;
using Telegram.Bot.Args;

namespace Awesome {
  class Program {
    static ITelegramBotClient botClient;

    static void Main() {
      botClient = new TelegramBotClient("token");

      var me = botClient.GetMeAsync().Result;
      Console.WriteLine(
        $"Hello, World! I am user {me.Id} and my name is {me.FirstName}."
      );

      botClient.OnMessage += Bot_OnMessage;
      botClient.StartReceiving();
      Thread.Sleep(int.MaxValue);
    }

    static async void Bot_OnMessage(object sender, MessageEventArgs e) {
      if (e.Message.Text != null)
      {
        Console.WriteLine($"Received a text message in chat {e.Message.Chat.Id}.");

        await botClient.SendTextMessageAsync(
          chatId: e.Message.Chat,
          text:   "You said:\n" + e.Message.Text
        );
        using (var stream = System.IO.File.OpenRead("video.mp4")) {
          await botClient.SendVideoNoteAsync(
            chatId: e.Message.Chat,
            videoNote: stream,
            duration: 47,
            length: 360
          );
        }
      }
    }
  }
}
```

This will create a bot that mounts and sends the video, you can get the token with @BotFather.

After that, just run the script, access your bot, send anything to it, receive the video and forward to @cybrics_facetoface_bot, and get the flag :D

![](https://i.imgur.com/wBYXVDt.png)

`Flag: cybrics{1_H47e_V1de0_n07e2}`

## Reference

* [Telegram Misc Challenge](https://lucasnathaniel.github.io/telegram-misc-challenge/)
