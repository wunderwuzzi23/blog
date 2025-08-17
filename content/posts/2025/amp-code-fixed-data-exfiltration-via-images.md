---
title: "Data Exfiltration via Image Rendering Fixed in Amp Code"  
date: 2025-08-17T04:10:58-07:00  
draft: true  
tags: ["llm", "agents", "month of ai bugs"]
twitter:  
  card: "summary_large_image"  
  site: "@wunderwuzzi23"  
  creator: "@wunderwuzzi23"  
  title: "Data Exfiltration via Image Rendering Fixed in Amp Coded"  
  description: "AmpCode is vulnerable to Prompt Injection and it was possible to leak sensitive source code, environment variables and other information on the host"  
  image: "https://embracethered.com/blog/images/2025/episode17-yt.png"  
---

{{< raw_html >}}
<a id="top_ref"></a>
{{< /raw_html >}}

In this post we discuss a vulnerability that was present in Amp Code from Sourcegraph by which an attacker could exploit markdown driven image rendering to exfiltrate sensitive information.

[![Amp Episode 17 Data Exfiltration Fixed](/blog/images/2025/episode17-yt.png)](/blog/images/2025/episode17-yt.png)

This vulnerability is common in AI applications and agents, and it's actually similar to one we discussed last year in GitHub Copilot which [Microsoft fixed](https://embracethered.com/blog/posts/2024/github-copilot-chat-prompt-injection-data-exfiltration/). 

## Exploit Demonstration

For the proof-of-concept I use a pre-existing demo that created a longer time ago. It happened to just work with Amp as well. The prompt injection is hosted on a website which asks the AI to "backup" information to a third-party site by rendering an image and including previous chat data as a query parameter.

[![backup instructions prompt](/blog/images/2025/backupwuzzi-website-small.png)](/blog/images/2025/jules-wuzzi-website.png)

Now, if Amp visits the website to complete a task, the embedded instructions exploit the AI agent, and trick it to send previous information in the chat context to the third-party server.

[![AmpCode info leakage via image rendering](/blog/images/2025/amp_image_render_data_exfil.png)](/blog/images/2025/amp_image_render_data_exfil.png)

As we have shown with many similar attacks over the last 2+ years this does not require a human-in-the-loop and leads to a 0-click data exfiltration scenario.

## Exfiltrating Sensitive Information

This attack not only allows adversaries to exfiltrate data, but also to first pull sensitive information into the chat context (e.g., via file reads) before leaking it.

[![AmpCode sensitive info leakage via image rendering](/blog/images/2025/amp_env_e2e_leak.png)](/blog/images/2025/amp_env_e2e_leak.png)

You might wonder about the "cryptic" way to invoke the grep tool. This is to work around refusals that the underlying model demonstrated when trying to read and leak sensitive information. This trick bypassed that.

In this case the prompt injection is coming from a comment in the code by the way. By simply analyzing that source code file, the text within the file hijacks the AI, brings additional information into the context window, and then leaks the data via image rendering.

## Video Demonstration

The following is a short video demonstration that shows the AI kill chain in action:
{{< youtube KpU8XBFhWSE >}}

Hope this short demo is insightful, please subscribe to the channel if you find my research useful.

## Prompt Injection Payload 

This is the text in the beginning of the source code file:

[![AmpCode sensitive info leakage via image rendering](/blog/images/2025/amp-source-prompt-injection-demo.png)](/blog/images/2025/amp-source-prompt-injection-demo.png)

Above text shows the entire source code file, including the prompt injection, that Amp followed.

Remember that a prompt injection can come from various untrusted data sources. It coming from a comment in source code is just one possible vector, data returned from tool calls is of course also a vector.

## Responsible Disclosure

After reaching out to Sourcegraph on June 14, 2025, the vulnerability was quickly addressed. Make sure to run the latest version. Kudos to the Amp team for addressing this promptly.

## Conclusion

Amp Code was vulnerable to information leakage via a prompt injection attack by being tricked into rendering images via markdown syntax. We highlighted how this could be used to leak the contents of the `.env` file.

This vulnerability class remains one of the most common AI application security issues.

Cheers.

## References

* [GitHub Copilot Data Exfiltration Fix](https://embracethered.com/blog/posts/2024/github-copilot-chat-prompt-injection-data-exfiltration/)  
* [Month of AI Bugs 2025](https://monthofaibugs.com)
