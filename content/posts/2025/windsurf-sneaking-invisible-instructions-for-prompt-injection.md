---
title: "Sneaking Invisible Instructions by Developers in Windsurf"  
date: 2025-08-23T16:20:58-07:00  
draft: true  
tags: ["llm", "agents", "month of ai bugs"] 
twitter:  
  card: "summary_large_image"  
  site: "@wunderwuzzi23"  
  creator: "@wunderwuzzi23"  
  title: "Sneaking Invisible Instructions by Developers in Windsurf"  
  description: "A vulnerability in Windsurf Cascade allows malicious instructions to be hidden from developers but followed by the AI, leading to potential data exfiltration. Learn how this 'invisible' attack works."  
  image: "https://embracethered.com/blog/images/2025/episode23-yt.png"  
---

{{< raw_html >}}
<a id="top_ref"></a>
{{< /raw_html >}}

Imagine a malicious instruction hidden in plain sight, invisible to you but not to the AI. This is a vulnerability discovered in Windsurf Cascade, it follows invisible instructions. This means there can be instructions in a file or result of a tool call that the developer cannot see, but the LLM does. 


[![Episode 23](/blog/images/2025/episode23-yt.png)](/blog/images/2025/episode23-yt.png)

Some LLMs interpret invisible Unicode Tag characters as instructions, which can lead to hidden prompt injection.

As far as I can tell the Windsurf `SWE-1` model can also "see" these invisible characters, but the `SWE-1` is not yet capable of interpreting them as instructions. As capabilities improve, so will the risk and impact of hidden prompt injections.

However, Windsurf supports models that reliably interpret these invisible instructions.

## Demonstration 🧪

Here is a simple demo that shows an empty appearing file, but it actually contains hidden instructions. The model used is Claude Sonnet 3.7:

[![Windsurf-Invisible-Tool-Invocation](/blog/images/2025/windsurf-mini-demo-invisible-prompt-injection.png)](/blog/images/2025/windsurf-mini-demo-invisible-prompt-injection.png)

Windsurf Cascade sees these invisible characters as instructions and **invokes the web tool.**

If you pay close attention to the screenshot, you can actually see that the tiny preview pane in Windsurf shows the visible text! 

As we have seen with [previous posts](/blog/posts/2025/windsurf-data-exfiltration-vulnerabilities/), automatic tool invocation like this can lead to information disclosure, data exfiltration, and worse... And leveraging hidden instructions allows a potential attack to remain undetected by the user.

Compared to the impact of automatic tool invocation, invisible instructions are a lower severity issue. 

**But the key point is: invisible instructions can invoke tools.**

## Additional Background Information

This behavior with LLMs was [discovered](https://x.com/goodside/status/1745511940351287394) by Riley Goodside with ChatGPT and was fixed by OpenAI. Since then, I have written about it quite frequently, [going back to January 2024](/blog/posts/2024/hiding-and-finding-text-with-unicode-tags/). I also built the [ASCII Smuggler](/blog/ascii-smuggler.html) to help with testing to show actual impact on system and application security.

## Mitigation

I suggest making the characters visible in the UI, so that it's clear that there is some hidden information.

Many other apps and agents decided to remove invisible Unicode Tag characters entirely before and after inference. That is probably the most practical mitigation.

Many coding agents, like [Amp](/blog/posts/2025/amp-code-fixed-invisible-prompt-injection/) and [Amazon Q Developer for VS Code](/blog/posts/2025/amazon-q-developer-interprets-hidden-instructions/) mitigate this at the application level, which is an effective mitigation for models that support invisible instructions. Many other agents remain susceptible to such attacks.

## Responsible Disclosure

This information was disclosed to Windsurf May 30, 2025 and receipt acknowledged by Windsurf a few days later. However, all further inquiries remained unanswered, hence disclosing this publicly seems the best approach to raise awareness and educate users.

**Update: After starting the disclosure process as part of Month of AI Bugs, I was contacted by Windsurf, and the team is looking to implement fixes and mitigations for the disclosed vulnerabilities.**

## Conclusion

Invisible instructions via Unicode Tag characters can be a sneaky way to remain undetected as an adversary. We have seen other coding agents mitigate this at the application level. If you are building exclusively on top of OpenAI models, you should be good, as OpenAI is generally mitigating this threat at the model/API level.

However, there are plenty of apps and models (Google, Anthropic, Grok, Windsurf Cascade,...) that remain susceptible to this behavior. 

If you build an app on top of a model which has this behavior, then consider taking extra steps to protect or inform end-users of invisible text being present.

## References

* [Month of AI Bugs 2025](https://monthofaibugs.com)
* [ASCII Smuggler tool](/blog/ascii-smuggler.html)
* [Riley Goodside - Initial Discovery](https://x.com/goodside/status/1745511940351287394)
