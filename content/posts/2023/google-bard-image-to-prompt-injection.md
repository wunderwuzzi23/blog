---
title: "Image to Prompt Injection with Google Bard"
date: 2023-07-14T09:00:00-07:00
draft: true
tags: [
     "aiml", "machine learning","ai injections"
    ]

twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "Image to Prompt Injection with Google Bard"
  description: "Analyzing images with AI can lead to prompt injection"
  image: "https://embracethered.com/blog/images/2023/aiinjection-image-joke-bard-response.png"
---

A prompt injection scenario that I, and others, have been wondering about in the past, is the potential risk associated with chatbots being able to analyze images. 

**Could this ability open up the way for Indirect Prompt Injection attacks?**

Recently, Google added the ability to uploading and analyze images with Bard. And it turns out that it is indeed possible to add instructions to an image, and have the Bard follow those instructions.

Here is a demonstration picture doing a Rickroll:

[![AI Prompt Injection - Rickroll](/blog/images/2023/aiinjection-image-joke.png)](/blog/images/2023/aiinjection-image-joke.png)

And this is the result after asking Bard to describe the image:

[![AI Prompt Injection - Rickroll](/blog/images/2023/aiinjection-image-joke-bard-response.png)](/blog/images/2023/aiinjection-image-joke-bard-response.png)

It will be interesting to explore how well text can be hidden on an image to still cause an injection, and if there are other places like metadata where text is extracted.

## References

* Base images (bot and banana pictures) generated with Bing Chat (then modified by me)
* Initial tweet showing proof-of-concept

{{< twitter wunderwuzzi23 1679676160341581824 >}}


