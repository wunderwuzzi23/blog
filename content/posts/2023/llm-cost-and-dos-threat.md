---
title: "LLM Apps: Don't Get Stuck in an Infinite Loop! 💵💰"
date: 2023-09-16T00:00:00-07:00
draft: true
tags: [
     "aiml", "machine learning","prompt injection","conference"
    ]
twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "Don't Get Stuck in a Costly Infinite Loop! 💵💰"
  description: "Securing LLM Apps Against Costly Attacks.💵💰 Amongst threats to consider are LLM apps and tools that might get stuck in an infinite loop."
  image: "https://embracethered.com/blog/images/2023/llm-plugin-loop-2.png"
---

What happens if an attacker calls an LLM tool or plugin recursively during an Indirect Prompt Injection? Could this be an issue and drive up costs, or DoS a system?

I tried it with ChatGPT, and it indeed works and the Chatbot enters a loop! 😊

[![llm-dos-loop](/blog/images/2023/llm-plugin-loop-2.png)](/blog/images/2023/llm-plugin-loop-2.png)

**However, for ChatGPT users this isn't really a threat, because:**
1. It's subscription based, so OpenAI would pay the bill.
2. There seems to be a call limit of 10 times in a single conversation turn (I tried a few times).
3. Lastly, one can click "Stop Generating" if the loop keeps ongoing.
 
**BUT**

Other applications might be vulnerable to this threat, especially if there is backend automation service consuming untrusted data and calling tools.

**Things could become costly quickly!**

[@wunderwuzzi23](https://twitter.com/wunderwuzzi23)

Here is a short video:


{{< youtube HjHWN7kGBC8 >}}
