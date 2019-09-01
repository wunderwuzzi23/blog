---
title: "Lyrebird - Hack the hacker (and take a picture)"
date: 2019-05-21T21:48:29-07:00
draft: true
tags: [
        "pentesting",
        "fun",
        "code",
        "tool"
    ]
description: "Series of posts for Password Spraying Technques"
meta_title: "Bash and password spraying"
---

The idea for Lyrebird came from observing that sometimes when someone forgets lock their workstation, someone else might mess with their computer. Since I wanted to learn more on how to program a webcam and take pictures - I figured why not create a little tool that takes a screenshot and uses the webcam to take pictures of anyone that interacts with the computer while I'm gone.

The way this work is simple, start Lyrebird. It will take a screenshot of the current desktop and then enter its observation mode.

**Note:** the machine is really not locked at this point. So this is not secure in anyway and you should not leave your workstation unattended without locking.

But anyhow, so then you can sneak away and when a malicous hacker in the office comes along and wants to interact with the desktop (the screenshot capture of it) Lyrebird will kick in and take a picture of the mysterious hacker, and then lock the workstation. 

I tried this a few times and it was really fun to watch! haha.

Have fun! This should work on both Mac and Windows. But there might be bugs, it was just a quick prototype.

Again, this is pure experimentation and not meant to provide any actual security besides some educational "hack the hacker' value in a controlled environment. 

Ideally this would be implemented as a screensaver, but it's just a quick prototype to hack the hacker. :)

So, always lock your workstation. :)

Code is here:
[Lyrebird Source on Github](https://github.com/wunderwuzzi23/Lyrebird)
