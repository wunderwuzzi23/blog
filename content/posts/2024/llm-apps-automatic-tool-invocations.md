---
title: "Automatic Tool Invocation when Browsing with ChatGPT - Threats and Mitigations"
date: 2024-05-28T20:57:38-07:00
draft: true
tags: [
     "ai", "testing", "machine learning", "ai injection", "chatgpt", "ttp"
    ]

twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "Automatic Tool Invocation when Browsing with ChatGPT - Threats and Mitigations"
  description: "Automatic Tool Invocation when Browsing with ChatGPT remains a risk to be aware of. Unfortunately it is possible to invoke the tools like memory or DALLE directly during prompt injection."
  image: "https://embracethered.com/blog/images/2024/chatgpt-ati-thumbnail.png"
---

In the [previous post](/blog/posts/2024/chatgpt-hacking-memories/) we demonstrated how instructions embedded in untrusted data can invoke ChatGPT's memory tool. The examples we looked at included `Uploaded Files`, `Connected Apps` and also the `Browsing` tool.

[![image tool invocation](/blog/images/2024/chatgpt-ati2.png)](/blog/images/2024//chatgpt-ati2.png)

When it came to the browsing tool we observed that mitigations were put in place and older demo exploits did not work anymore. After chatting with other security researchers, I learned that they had observed the same. 

**However, with some minor prompting tricks mitigations are bypassed and we, again, can demonstrate automatic tool invocation when browsing the Internet with ChatGPT!** 

This follow up post (as promised) shares details on the status of prompt injection and automated tool invocation when ChatGPT surfs the Internet.

## Automatic Tool Invocation when Browsing

Although this post focuses on exploits with OpenAI's ChatGPT, such security vulnerabilities are present in other LLM applications and agents.

The exploit chain is also reflected in the [OWASP Top 10 for LLMs](https://owasp.org/www-project-top-10-for-large-language-model-applications/assets/PDF/OWASP-Top-10-for-LLMs-2023-v1_1.pdf) in particular concerning these threats: 
* `LLM01: Prompt Injection`, 
* `LLM07: Insecure Plugin Design` and 
* `LLM08: Excessive Agency`.

### Try For Yourself!

If you want you can also try it yourself, all that is needed is ChatGPT 4o.

**1. Invoking Image Creation with DALLE**

A fun demo is having instructions on a website invoke DALLE! Just paste this URL into a new chat and see what happens: `https://wuzzi.net/create.txt`.

[![image creation](/blog/images/2024/chatgpt-invoke-dalle2.png)](/blog/images/2024//chatgpt-invoke-dalle2.png)

Look! ChatGPT just created an image!

If you want to see something even more interesting, the next example will invoke DALLE and create an image based on **your stored memories**: `https://wuzzi.net/8create.txt`

*Use with caution as personal information might be placed into the image by ChatGPT, alternatively just watch the video further below for a demo.*

Cool, right? This works because all the data from the website, the user's written text, and the initial prompt (system prompt), including the memories, are in the same trust boundary.

Speaking of the memory tool, let's look at that next.

**2. Invoke the Memory Tool**

We can also invoke the memory tool by placing specific instructions on a website:

[![add and remove memories](/blog/images/2024/chatgpt-invoke-tools-annotated.png)](/blog/images/2024/chatgpt-invoke-tools-annotated.png)

**Addn memories:** `https://wuzzi.net/c/add.txt`

*Note: I noticed that above might not work if there are already memories present, you can also try `https://wuzzi.net/c/do2-2.txt` which is more reliable when there are already memories stored.*

**Delete memories:** This URL will delete your memories: `https://wuzzi.net/c/rm.txt`

At the moment these are quite reliable and work close to 100%.

### Prompt Injections Instructions

Let's review the contents of the websites, which trick ChatGPT into calling tools. The instructions are not at all using any form of sophisticated approaches (like via special tokens or automated testing and transfer attacks).

The first one is the creation of an image based on memories:

[![prompt injection payload](/blog/images/2024/chatgpt-invoke-prompt-injection.png)](/blog/images/2024/chatgpt-invoke-prompt-injection.png)

This one, which I really like, is for clearing of memories:

[![add and remove memories](/blog/images/2024/chatgpt-prompt-bypass.png)](/blog/images/2024/chatgpt-prompt-bypass.png)

Both are great examples of how levels of indirection easily "trick" language models into following instructions. LLMs are naive and gullible. The second demo adds a twist with stating a challenge.

### Video Demonstration

This video shows the three POCs in a quick run through:

{{< youtube j72rT-QAjuk >}}

The three examples are:

1. Website instructions **add new long term memories** 🧠
2. Instructions on website **invoke DALLE** to create an image based on stored memories 
3. Instructions from website **erase all your precious memories** 🤯

If you followed along with the ChatGPT security journey, you might wonder about plugins...

### So, what about plugins?

**Plugins are gone now!** 

The replacement for plugins are custom GPTs with `AI Actions`.

### What about AI Actions?

Custom GPTs can have `AI Actions` and those can be vulnerable as well!

The good news, there is a [mitigation](https://platform.openai.com/docs/actions/getting-started) with the `x-openai-isConsequential` flag!

[![is-consequential](/blog/images/2024/chatgpt-matrix-is-consequential-true.png)](/blog/images/2024/chatgpt-matrix-is-consequential-true.png)

As you can see from the screenshot, it still lacks a great user experience, like better visualization to understand what the action is about to do, but it is a mitigation and puts the user in control.

## Mitigations and Recommendations

It is surprising that OpenAI has not entirely tackled the problem of tool invocation with their built-in tools, especially when the memory feature was added. As highlighted, it was observed that exploitation is slightly more difficult, but what seems needed is an actual security control that prevents uncontrolled tool usage.

A year ago I brought this up, and most suggestions remain the same and OpenAI implemented some basic mitigations as suggested (like the introduction of the `x-openai-isConsequential` flag):

* Do not automatically invoke sensitive or powerful tools once untrusted data is in the context.
* Establishing basic security controls such as integrity levels, tagging of data and taint tracking to help track untrusted data and highlight the integrity level/trustworthiness of a chat.
* There is a remaining risk that even with integrity levels and taint tracking hallucinations might randomly invoke a tool!
* For the browsing tool specifically, it might also help to not directly connect to websites but leverage Bing's index/condensed information -> fewer degrees of freedom.
* Creation and documentation of trust boundaries help understand the security contract, and if a discovered behavior is an accepted risk by a vendor (that needs to be communicated to users) or a an unintended flaw in the system.
* "Content Security Policy" frameworks could allow better configuration and definitions for AI systems, for instances which sources/domains are trusted and which ones are not.
* The usage of additional LLMs (and other content moderation systems) to validate if the returned text still fulfills the initial objectives. This is actually something OpenAI might be doing now, since some mitigations were observed recently.
* A bit of a segue, Microsoft has [Azure Purview](https://techcommunity.microsoft.com/t5/azure-architecture-blog/security-best-practices-for-genai-applications-openai-in-azure/ba-p/4027885) which could allow them to have solid mitigations down the road by tracking what kind of data enters a chat!

Finally, one thing to keep in mind... intuation tells me, that as capabilities of language models increase, so will the bypass capabilities.

## Conclusion

Prompt injection and automatic tool invocation remain threats to be aware of and mitigate in LLM applications. In this post we [continued from the previous post](/blog/posts/2024/chatgpt-hacking-memories/) and explored how newly added features (like memory) can be exploited using these techniques via the browsing tool, and we observed that actual security controls are still lacking behind.

## References

* [Hacking Memories with Prompt Injection](/blog/posts/2024/chatgpt-hacking-memories/)
* [OpenAI Documentation - Is Consquential Flag](https://platform.openai.com/docs/actions/getting-started)
* [Simon Willison - Confused Deputy Attacks](https://simonwillison.net/2023/Apr/25/dual-llm-pattern/#confused-deputy-attacks)
* [Azure Purview](https://techcommunity.microsoft.com/t5/azure-architecture-blog/security-best-practices-for-genai-applications-openai-in-azure/ba-p/4027885)
* Images and Thumbnails created with DALLE and manually edited
