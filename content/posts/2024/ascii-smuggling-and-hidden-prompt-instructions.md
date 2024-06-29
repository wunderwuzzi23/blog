---
title: "Video: ASCII Smuggling and Hidden Prompt Instructions"
date: 2024-02-12T17:11:48-08:00
draft: true
tags: [
     "aiml", "machine learning", "threats", "bugbounty", "llm", "video"
    ]

twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "Video: ASCII Smuggling and Hidden Prompt Instructions"
  description: "ASCII Smuggling - Crafting Invisible Text and Decoding Hidden Secrets (with LLMs)"
  image: "https://embracethered.com/blog/images/2024/ascii-smuggler-qr.png"
---


A couple of weeks ago hidden prompt injections were discovered and [we covered it at the time](/blog/posts/2024/hiding-and-finding-text-with-unicode-tags/).

This video explains it in more detail, and also highlights implications beyond hiding instructions, including what I call `ASCII Smuggling`. This is the usage of [Unicode Tags Block characters](https://en.wikipedia.org/wiki/Tags_(Unicode_block)) to both craft and deciper hidden messages in plain sight.

{{< youtube 7z8weQnEbsc >}}
<br>
<br>

Using Unicode encoding to bypass security features or execute code (XSS, SSRF,..) has been in use for a while, however this new TTP enables more sophisticated attack scenarios. 

This is because Unicode Tags Code Point mirror the entire ASCII set, but are often not visible in UI elements. For Large Language Models this is interesting because LLMs often interpret this hidden text as ASCII, and they can also craft such hidden text when replying to user queries.

### Take-aways

Couple of things come to mind and I touch on those in the video in more detail:

* Test your own LLM apps for this new attack vector
* As developer a possible mitigations is to remove Unicode Tags Block text on the way in and out
* Consider implications beyond LLM applications and Chatbots
<br>
<br>

### The ASCII Smuggler Tool is here by the way

[![ASCII Smuggler Tool](/blog/images/2024/ascii-smuggler-qr-clean.png)](/blog/ascii-smuggler.html)

