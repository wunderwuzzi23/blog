---
title: "Video: Data Exfiltration Vulnerabilities in LLM apps (Bing Chat, ChatGPT, Claude)"
date: 2023-08-28T10:00:51-07:00
tags: [
     "aiml", "machine learning","prompt injection","video", "exfil"
    ]
twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "Video: Data Exfiltration Vulnerabilities in LLM apps (Bing Chat, ChatGPT, Claude)"
  description: "This video highlights various data exfiltration vulnerabilities I discovered and responsibly disclosed to Microsoft, Anthropic, ChatGPT and Plugin Developers. It also highlights the response and fixes of various vendors - with surprisingly different outcomes."
  image: "https://embracethered.com/blog/images/2023/llm-data-exfil-thumbnail.png"
---

This video highlights the various data exfiltration vulnerabilities I discovered and responsibly disclosed to Microsoft, Anthropic, ChatGPT and Plugin Developers. 

It also briefly discusses mitigations various vendors put in place (and triage decisions).

{{< youtube L_1plTXF-FE >}}

&nbsp;

Thanks to MSRC, Anthropic and Zapier for addressing vulnerabilities to help protect their users.

Let's hope it inspires OpenAI to mitigate the image markdown injection issue finally as well. It's rated as a CVSS High scored vulnerability basically and was first reported to them on April, 9th 2023 - the triage decision was "won't fix".


## References

Detailed write up of each scenario, bug report and response:

* [Microsoft - Bing Chat (fixed)](/blog/posts/2023/bing-chat-data-exfiltration-poc-and-fix/)
* [Anthropic - Claude (fixed)](/blog/posts/2023/anthropic-fixes-claude-data-exfiltration-via-images/)
* [Plugin Vendor Email Exfil (fixed)](https://embracethered.com/blog/posts/2023/chatgpt-cross-plugin-request-forgery-and-prompt-injection./)
* [OpenAI - ChatGPT (won't fix)](/blog/posts/2023/chatgpt-webpilot-data-exfil-via-markdown-injection/)


