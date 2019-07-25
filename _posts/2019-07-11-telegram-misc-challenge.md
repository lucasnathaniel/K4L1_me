---
title: Telegram Misc Challenge
layout: post 
date: '2019-07-25 18:30:00'
description: writeup for misc of CyBRICS Quals CTF 2019
tags:
- ctf
- writeup
- stego
- miscc
- infosec
---

## Telegram(misc) - CyBRICS Quals 2019

>Hi everyone! C:
>This challenge was funny, I solved with my team

## Description

Author: Alexander Menshchikov (n0str)

This Telegram bot really loves live face-to-face communication! And it also seems to have some covert channel!

@cybrics_facetoface_bot

## The ~~*Jurandir*~~ Solver

So, basically this is a Telegram bot that send and receives TVN(Telegram Video Note). You can just send this video type using the cellphone or telegram API, the bot dont receives normal videos or more than 150Kb TVN.

When we sent a small TVN, the bot check that the file is corret but don't recognize a "value secrets sign".

Btw, let's check these bot videos... for that you can inspect the page and see the thumbnail, there is some information on corners, you can download copying the video location while playing, opening on tab and saving.

![](https://i.imgur.com/xduj8XN.png)


The text on video is: "send video note to the bot with four green circles in the corners like the one above".

So, let's do it :D

We can use [Gimp](https://www.gimp.org/) to create a png file with each green(0, 255, 0) circle on corners, and [OpenShot](https://www.openshot.org/pt/) to mount.

To create a TVN, we can use [Telegram API](https://telegrambots.github.io/book/2/send-msg/video-video_note-msg.html), written in C#. Following the documentation we got that and this is the scrpit below.


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

This will create a bot that mount and send the video, you can get the token wih @BotFather.

After that, just run the script, acess your bot, send anything for him, receive the video and forward to @cybrics_facetoface_bot, and got the flag :D

![](https://i.imgur.com/wBYXVDt.png)

>Flag: cybrics{1_H47e_V1de0_n07e2}


By: Lucas ~K4L1~ Nathaniel | FireShell
