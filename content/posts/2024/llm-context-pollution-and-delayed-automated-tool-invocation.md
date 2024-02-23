---
title: "Google Gemini: Planting Instructions For Delayed Automatic Tool Invocation"
date: 2024-02-22T22:00:06-08:00
draft: true
tags: [
     "aiml", "machine learning","ai injections","ttp"
    ]
twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "Google Gemini (now Gemini) - Planting Instructions For Delayed Automatic Tool Invocation"
  description: "Attackers can pollute the prompt context of large language model applications and invoke tools, which otherwise might not be accessible."
  image: "https://embracethered.com/blog/images/2024/llm-planting-instructions.png"
---

Last November, while testing `Google Bard` (now called `Gemini`) for vulnerabilities, I had a couple of interesting observations when it comes to automatic tool invocation. 

## Confused Deputy - Automatic Tool Invocation

First, what do I mean by this... "automatic tool invocation"...

Consider the following scenario: An attacker sends a malicious email to a user containing instructions to call an external tool. Google named these tools `Extensions`.

When the user analyzes the email with an LLM, it interprets the instructions and calls the external tool, leading to a kind of `request forgery` or maybe better called **automatic tool invocation**.

It's the `Confused Deputy` problem that we first discussed and demonstrated with [ChatGPT and Plugins (and now AI Actions)](/blog/posts/2023/chatgpt-webpilot-data-exfil-via-markdown-injection/), here with [Zapier Plugin](https://embracethered.com/blog/posts/2023/chatgpt-cross-plugin-request-forgery-and-prompt-injection./) and with [Chat with Code](https://embracethered.com/blog/posts/2023/chatgpt-plugin-vulns-chat-with-code/).

The same attack idea applies here with Google Gemini!

## Google's Mitigations

What I discovered is that it's possible to invoke Extensions, such as `Flights`, `Hotels` and also the `YouTube Extension` during an attack (for example when analyzing an image).

**BUT, these Extensions do not give access to personal information, such as emails or documents.**

To gain access to the user's data the `Workspace Extension` is used. However, the `Workspace Extension` is not automatically invoked.

Take a look, you can see how it invokes four extensions in one shot:

[![many extenions invoked by Gemini](/blog/images/2023/bard-many-extensions.png)](/blog/images/2023/bard-many-extensions.png)


Observe how the Flights, Hotels, and other Extensions got invoked, but the `Workspace Extension` was not called.

**This means Google implemented a special kind of mitigation. If untrusted data enters the prompt context via accessing untrusted data (indirect prompt injection), then Google Gemini will not invoke all tools in the same conversation turn!**

### Threat Model

It seems Google identified this threat and prevents an adversary issuing commands and bringing the user's `Google Docs` or `Gmail` into the chat context during an attack. The `Workspace Extension` is not invoked in that situation.

The reason this would be a security issue is because:

1. Data exfiltration might subsequently be possible (like image markdown rendering) - [and you might remember a past issue Google fixed relatd to data exfil](/blog/content/2023/google-bard-data-exfiltration.md)
2. Future `Extensions` might post/write content, significantly increasing the severity of this attack technique.

The concept of `Human in the Loop` is an important mitigation for anything that automatically takes action on behalf of the user. Especially because there is no reliable fix for prompt injection as of today.

So, while hacking along I had an interesting idea...

## Planting Instructions To Invoke Tools

What if the adversary "pollutes" the chat context during the prompt injection attack and plants instructions that will trigger the invocation of the `Workspace Extension` at a later stage? 

The idea to plant instructions or special trigger commands is not new, and a rather obvious attack technique. However, what is new here was the question if such planted instructions could lead to an automatic (and unwanted) invocation of the `Workspace Extension` at a **later point without the user explicitly and knowingly authorizing the action.**

## Attack Overview 

1. Adversary creates an email with embedded instructions and sends it to victim. The instructions state a future task, that is triggered with the next user request, or trigger keywords. 
2. The victim interacts/summarizes the email via Google Gemini.
3. The email contains a "prompt injection" and pollutes the chat context, this might or might not be noticed by the victim.
4. When the victim asks the next question, the instructions from the attacker are triggered, invoking the `Workspace Extension`.

Here is the example email I used, notice the highlighted instructions:

[![Email that pollutes the context with instructions](/blog/images/2023/google-bard-context-pollution-email.png)](/blog/images/2023/google-bard-context-pollution-email.png)

The goal was to retrieve the text from this Google Doc:

[![cats](/blog/images/2023/google-bard-context-pollution-document2.png)](/blog/images/2023/google-bard-context-pollution-document2.png)

Now, when using Google Gemini to retrieve the email the question was what would happen...

{{< youtube qYMt9QJFzmI >}}

<br>󠁎󠁩󠁣󠁥
<br> 
<br>

**Note:** Even though this proof-of-concept worked, the exploit was a bit flaky, but as more capabable models got released I think it improved.

## Mitigation Ideas

Google's thought process around not invoking certain tools when untrusted data entered the chat context is an interesting approach that other vendors have not yet adopted. However, it can be bypassed as shown in this post and because of the lack of fixes for indirect prompt injection we have to stay alert.

Currently the security impact for Gemini is limited, since Google fixed the [Data Exfiltration angle via Image Markdown rendering](/blog/content/2023/google-bard-data-exfiltration.md). I did report this behavior and bypass to Google last November to raise awareness. It's still unclear how Google plans to address it, but they confirmed tracking the issue.

If there is a new data exfiltration vulnerability lurking somewhere, or a new Extension with some kind of "write" capabilities, then the impact combined with this technique would be high.

## Conclusion

Polluting the chat context and planting instructions is a powerful way for an adversary to persist.

The interesting new discovery here is that an LLM application might prevent the automatic invocation of a tool during the conversation turn the untrusted data entered the chat. But the LLM application might happily invoke such tools in subsequent conversations turns. This can be exploited by an adversary by planting instructions that execute at a later time.

Hope this was interesting and helpful.

Cheers.

## References

* [Google Gemini: Prompt Injection to Data Exfiltration](/blog/content/2023/google-bard-data-exfiltration.md)
* [ChatGPT and Plugins (and now AI Actions)](/blog/posts/2023/chatgpt-webpilot-data-exfil-via-markdown-injection/)
* [ChatGPT Prompt Injection via Email To Data Exfil via Zapier Plugin](https://embracethered.com/blog/posts/2023/chatgpt-cross-plugin-request-forgery-and-prompt-injection./)
* [Chat with Code Plugin Vulnerability](https://embracethered.com/blog/posts/2023/chatgpt-plugin-vulns-chat-with-code/)

