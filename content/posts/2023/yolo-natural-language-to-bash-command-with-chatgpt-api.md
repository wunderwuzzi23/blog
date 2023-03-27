---
title: "Yolo: Natural Language to Shell Commands with ChatGPT API"
date: 2023-03-05T17:31:58-08:00
draft: true
tags: [
        "aiml", "programming"
    ]

twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "Yolo: Natural Language to Shell Commands With ChatGPT API"
  description:  "Yolo takes your question and translates it into shell commands (such as bash) and runs them yolo style! :)"
  image: "https://embracethered.com/blog/images/2023/yolo-blog.png"
---

Once in a while I go build some fun new tools to adopt new tech. Just last week OpenAI made their `gpt-3.5-turbo` model accessible via API endpoints. 

**Update: The latest version also supports GPT-4.**

So, I thought it's time to start building a tool to leverage it.

## What is yolo?

Do you know those moments when you can't remember a shell command, or some arguments to it? How do you pipe all errors to /dev/null again? Things along those lines. This is where yolo comes to the rescue.

**You live only once!** 

![Yolo Gif Animation](/blog/images/2023/yolo-shell-anim-gif.gif)

Yolo translates your question to a shell command (like `bash`, `zsh` or `PowerShell` on Windows), and then executes the command. By default, it won't right away execute the command (but that can be overwritten, if you desire the true yolo-style).

This project was created for fun, but it's actually kind of useful.

I started building yolo some time ago, and now upgraded it from the GPT-3 completion API to the brand new [ChatCompletion](https://platform.openai.com/docs/guides/chat) (ChatGPT) API.


## How to install and use yourself

The code is here [yolo-ai-cmdbot Github repo](https://github.com/wunderwuzzi23/yolo-ai-cmdbot). 

Easy install, don't forget to set your OpenAI API key.

I mostly use it on Linux and some macOS, Windows is less tested. 

## Detailed walk-through and demonstration

There is also a [YouTube Video](https://youtu.be/g6rvHWpx_Go) that shows it in action: 
[![Video](/blog/images/2023/yolo-thumbnail-small.png)](https://youtu.be/g6rvHWpx_Go)


Happy hacking!


## References

* Github Repo for yolo: https://github.com/wunderwuzzi23/yolo-ai-cmdbot
* ChatCompleation API documentation: https://platform.openai.com/docs/guides/chat