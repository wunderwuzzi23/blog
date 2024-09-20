---
title: "Spyware Injection Into Your ChatGPT's Long-Term Memory (SpAIware)"
date: 2024-09-20T11:02:36-07:00
draft: true
tags: [
    "threats", "ttp", "chatgpt", "ai injection", "llm", "memories"
    ]

twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "Spyware Injection Into Your ChatGPT's Long-Term Memory (SpAIware)"
  description: "The ChatGPT iOS and macOS versions were vulnerable to persistent data exfiltration. This is the story behind finding the issue and getting it fixed."
  image: "https://embracethered.com/blog/images/2024/chatgpt-persistent-data-exfiltration.png"
---

This post explains an attack chain for the ChatGPT macOS application. Through prompt injection from untrusted data, attackers could insert long-term persistent spyware into ChatGPT's memory. This led to continuous data exfiltration of any information the user typed or responses received by ChatGPT, including any future chat sessions.

[![Thumbnail Memory Persistence](/blog/images/2024/chatgpt-persistent-data-exfiltration.png)](/blog/images/2024/chatgpt-persistent-data-exfiltration.png)
<br>
OpenAI released a fix for the macOS app last week. Ensure your app is updated to the latest version.

Let's look at this `spAIware` in detail.

## Background Information

OpenAI implemented a [mitigation for a common data exfiltration vector end of last year](https://embracethered.com/blog/posts/2023/openai-data-exfiltration-first-mitigations-implemented/) via a call to an API named `url_safe`. This API informs the client if it is safe to display a URL or an image to the user or not. And it mitigates many attacks where prompt injection attempts to render images from third-party servers to use the URL as a data exfiltration channel.

As highlighted [back then](https://embracethered.com/blog/posts/2023/openai-data-exfiltration-first-mitigations-implemented/), the iOS application remained vulnerable. This was because the security check (`url_safe`) was performed client-side. Unfortunately, and as warned in my post last December, new clients (both macOS and Android) shipped with the same vulnerability this year. 

**That's bad, but it gets a lot more interesting!**

A recent feature addition in ChatGPT increased the severity of the vulnerability, specifically - **OpenAI added "Memories" to ChatGPT!**

## Hacking Memories to Store Malicious Instructions

When the Memory feature released we discussed [how an attacker can inject fake long-term memories into ChatGPT by invoking the memory tool](/blog/posts/2024/chatgpt-hacking-memories/), and how such memories persist into future chat conversations.

[![Memory Updated](/blog/images/2024/chatgpt-memory-updated.png)](/blog/posts/2024/chatgpt-hacking-memories/)

This happens through prompt injection from third-party websites.

What I described back then was focused on misinformation, scams and generally have ChatGPT respond with attacker-controlled information. 

## Persisting Data Exfiltration Instructions in ChatGPT's Memory

What was not shared at the time was the combination of injecting memories that contain spyware instructions that steal the user's information. 

Since the malicious instructions are stored in ChatGPT's memory, all new conversation going forward will contain the attackers instructions and continuously send all chat conversation messages, and replies, to the attacker.

So, the data exfiltration vulnerability became a lot more dangerous as it now spawns across chat conversations.

### The Data Exfiltration Technique 

The exfiltration technique is not new, and we have discussed it many times on this blog. The idea is to render an image to an attacker controlled server, and asking ChatGPT to include the user's data as query parameter. 

Let's look at it in action.

## End-to-End Exploit Demonstration

Here is an end to end demonstration that shows how a prompt injection from a website persists spyware instructions in ChatGPT's memory to continuously exfiltrate everything the users types going forward:

{{< youtube zb0q5AW5ns8>}}
<br>

The image rendered in the video is invisible for stealth. Pretty scary.

## Step by Step Explanation

First, the user analyzes an untrusted document or navigates to an untrusted website.

[![Injection into Memory](/blog/images/2024/chatgpt-exfil-inject.png)](/blog/images/2024/chatgpt-exfil-inject.png)

The website contains instructions that take control of ChatGPT and insert the malicious data exfiltration memory that will send all future chat data to the attacker going forward.

[![Updated Memories](/blog/images/2024/chatgpt-macos-persisten3.png)](/blog/images/2024/chatgpt-macos-persisten3.png)

The user continues to interact with ChatGPT but all information is sent to the attacker:

[![Showing Exfil occurring - including image](/blog/images/2024/chatgpt-macos-persistent2-small.png)](/blog/images/2024/chatgpt-macos-persistent2.png)

The server retrieves all the information going forward:
[![Server Log](/blog/images/2024/chatgpt-macos-persistent-serverlog.png)](/blog/images/2024/chatgpt-macos-persistent-serverlog.png)

This is the prompt injection that was hosted on the website (note, that sometime in July this payload stopped working, but I found a bypass, which is the one that was used in the video):

[![Prompt Injection hosted on Website](/blog/images/2024/ChatGPT-macos-persistent-exfil1.png)](/blog/images/2024/ChatGPT-macos-persistent-exfil1.png)

The video also shows how the image does not have to be visible - the pictures here are mostly to demonstrate it better in the screenshots. So I recommend watching the video to best understand end to end exploit.

## Is url_safe a holistic fix?

The `url_safe` feature still allows for some information to be leaked. Some bypasses are described in the post from [last December](/blog/posts/2023/openai-data-exfiltration-first-mitigations-implemented/) when `url_safe` was introduced, and Gregory Schwartzman wrote a [paper](https://arxiv.org/pdf/2406.00199) exploring this in detail.

## Is Hacking Memories via Prompt Injection Fixed?

No. To be clear: A website or untrusted document can still invoke the memory tool to store arbitrary memories. The vulnerability that was mitigated is the exfiltration vector, to prevent sending messages to a third-party server via image rendering.

**ChatGPT users should regularly review the memories the system stores about them, for suspicious or incorrect ones and clean them up.**

Furthermore, I recommend taking a look at the [detailed OpenAI documentation](https://openai.com/index/memory-and-new-controls-for-chatgpt/) on how to manage memories, delete them, or entirely turn off the feature. It is also possible to start temporary chats that do not use memory.

## Conclusion

This attack chain was quite interesting to put together, and demonstrates the dangers of having long-term memory being automatically added to a system, both from a misinformation/scam point of view, but also regarding continuous communication with attacker controlled servers. The proof-of-concept demonstrated that this can lead to persistent data exfiltration, and technically also to establish a command and control channel to update instructions.

Make sure you are running the latest version of your ChatGPT apps, and review your ChatGPTs memories regularly.

Thanks to OpenAI for mitigating this vulnerability.

Cheers and stay safe.

## Timeline

* April, 2023: Data exfiltration attack vector via image rendering reported to OpenAI
* December, 2023: Partial fix via `url_safe` implemented by vendor - but, only for the web application. Other clients, like iOS remained vulnerable.
* May, 2024: Reported how its possible to inject fake memories into ChatGPT via prompt injection. This ticket was closed as a model safety issue, not a security concern.
* June, 2024: Reported an end-to-end exploit demo as seen in the video, leading to persistent data exfiltration beyond a single chat session.
* September, 2024: OpenAI fixes the vulnerability in ChatGPT version 1.2024.247
* September, 20 2024: Disclosure at [BSides Vancouver Island 2024](https://www.bsidesvi.com/bsidesvi24)


## References

* [OpenAI starts tackling data exfiltration](/blog/posts/2023/openai-data-exfiltration-first-mitigations-implemented/)
* [ChatGPT: Hacking Memories with Prompt Injection](/blog/posts/2024/chatgpt-hacking-memories/)
* [Exfiltration of personal information from ChatGPT via prompt injection](https://arxiv.org/pdf/2406.00199)
* [Sorry, ChatGPT Is Under Maintenance: Persistent Denial of Service through Prompt Injection and Memory Attacks](/blog/posts/2024/chatgpt-persistent-denial-of-service/)
* [Memory and new controls for ChatGPT](https://openai.com/index/memory-and-new-controls-for-chatgpt/)

## Appendix

### Initial POC Video

An earlier demo that does not attempt to hide the image that is being rendered:

{{< youtube YHW-UMxsais>}}

### Latest Prompt Injection Payload that was used

[![Prompt PI Bypass](/blog/images/2024/chatgpt-persist-pi-bypass.png)](/blog/images/2024/chatgpt-persist-pi-bypass.png)
