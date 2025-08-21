---
title: "Hijacking Windsurf: How Prompt Injection Leaks Developer Secrets"  
date: 2025-08-21T02:20:58-07:00  
draft: true  
tags: ["llm", "agents", "month of ai bugs"] 
twitter:  
  card: "summary_large_image"  
  site: "@wunderwuzzi23"  
  creator: "@wunderwuzzi23"  
  title: "Hijacking Windsurf: How Prompt Injection Leaks Developer Secrets"  
  description: "Windsurf is vulnerable to indirect prompt injection and can be exploited to leak sensitive source code, environment variables and other information on the host"  
  image: "https://embracethered.com/blog/images/2025/episode21-yt.png"  
---

{{< raw_html >}}
<a id="top_ref"></a>
{{< /raw_html >}}

This is the first post in a series exploring security vulnerabilities in Windsurf. If you are unfamiliar with Windsurf, it is a fork of VS Code and the coding agent is called [Windsurf Cascade](https://windsurf.com/cascade).

The attack vectors we will explore today allow an adversary during an indirect prompt injection to exfiltrate data from the developer's machine.

These vulnerabilities are a great example of Simon Willison's [lethal trifecta](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/) pattern.

[![Episode 21](/blog/images/2025/episode21-yt.png)](/blog/images/2025/episode21-yt.png)

Overall, the security vulnerability reporting experience with Windsurf has not been great. All findings were responsibly disclosed on May 30, 2025, and receeipt was acknowledged a few days after. However, all further inquiries regarding bug status or fixes remain unanswered. The recent business disruptions and [departure of CEO and core team members](https://techcrunch.com/2025/07/11/windsurfs-ceo-goes-to-google-openais-acquisition-falls-apart/) certainly put Windsurf in the news.

Let's explore this in detail.

## Windsurf System Prompt

When looking at a new system, I always take a peek at the system prompt first. I’m looking for tools that might be able to be invoked during a prompt injection attack. Sometimes there are also other interesting tidbits present.

Here is a snipet of the system prompt:

[![Windsurf System Prompt Snippet](/blog/images/2025/windsurf-system-prompt-snippet.png)](/blog/images/2025/windsurf-system-prompt-snippet.png)

You can find the [Windsurf Cascade system prompt here](https://github.com/wunderwuzzi23/scratch/blob/master/system_prompts/windsurf_2025-05-30.txt).

One thing that stood out to me right away was the `read_url_content` tool.

## Attack Vector 1: Tools as Data Exfiltration Vectors

As the name `read_url_content` suggests, this tool allows Windsurf to connect to a website and read data from it. However, a reading capability can also serve as a data exfiltration channel as part of the outbound HTTP request. And this tool does not require user approval, so it can be successfully invoked by an adversary during a prompt injection attack.

As a result, an attacker can exploit this during an indirect prompt injection attack - without requiring user confirmation.

### Exploit Demonstration

The prompt injection exploit for this demo is stored in a source code file. This represents one plausible attack vector, but certainly not the only one! 

Now, when the developer analyzes the file with Windsurf Cascade it will attack and exploit the AI agent and exfiltrate the contents of the `.env` file using the `read_url_content` tool. 

[![Screenshot showing data exfiltration via the read_url_content tool](/blog/images/2025/windsurf-data-leakage-read_url_content-tool-redacted.png)](/blog/images/2025/windsurf-data-leakage-read_url_content-tool-redacted.png)

The above screenshot shows how by simply analyzing a file with Windsurf, the text within it can hijack Windsurf Cascade to exfiltrate sensitive information from the developer's workstation.

### Video Demonstration Proof-of-Concept

This video shows how such an exploit chain looks end-to-end:

{{< youtube _Mb9n7frGAs >}}


### Prompt Injection Payload

This is the text in the beginning of the source code file:

```  
<redacted for now, as this is not fixed> 
```

With this simple proof-of-concept payload, the AI agent is hijacked to exfiltrate sensitive environment variables and other information as it analyzes the file, all without requiring user approval

## Attack Vector 2: Image Rendering

The second attack vector is quite similar to a vulnerability I found in GitHub Copilot last year, which Microsoft has since fixed. You can read up details [here](https://embracethered.com/blog/posts/2024/github-copilot-chat-prompt-injection-data-exfiltration/). 

Image rendering from untrusted domains is actually one of the most common AI application security vulnerabilities to be aware of. We have seen dozens of examples of this vulnerability over the last two+ years.

### Exploit Demonstration

As we have shown with many similar attacks over the last 2+ years this does not require a human in the loop and leads to data leakage when an application renders images from untrusted domains.

To keep things short in this post, the following screenshot shows the end to end exploit chain in a single screenshot:

[![Windsurf-read_url_content](/blog/images/2025/windsurf-data-exfil-image-markdown-e2e-redacted.png)](/blog/images/2025/windsurf-data-exfil-image-markdown-e2e-redacted.png)

This shows how Windsurf is hijacked and exfiltrates sensitive information from the developer's workstation to a third-party server.

### Video Demonstration Proof-of-Concept

{{< youtube lTkiCe3uhEY >}}



### Prompt Injection Payload

This is the text in the beginning of the source code file:

```  
<redacted for now, as this is not fixed>
```

That’s all that was needed to hijack Windsurf Cascade. 

## Responsible Disclosure

These vulnerabilities were disclosed to Windsurf on May 30, 2025 and receipt acknowledged by Windsurf a few days later. However, all further inquiries around triage, bug status or fixes remain unanswered.

Hence, disclosing this publicly after ~3 months seems the best approach to raise awareness and educate customers and users about these high-severity vulnerabilities. 

## Mitigations

- Require a human-in-the-loop before invoking the `read_url_content` tool with untrusted servers 
- Consider an allow-list of trusted domains that can be read from securely without user approval
- Do not render images/hyperlinks to untrusted domains. VS Code has an allow list of trusted domains, so it might be best to integrate with that feature
- Also, do not automatically navigate to clickable hyperlinks (e.g. phishing attacks)
- If you are a customer of Windsurf, I suggest reaching out to your account manager to inquiry about these high-severity vulnerabilites to raise awareness and get them fixed

## Conclusion

Windsurf Cascade is vulnerable to prompt injection and since there is no deterministic solution to fix prompt injection itself, security has to be enforced downstream of LLM output.

As demonstrated, weaknesses in tool invocation and image rendering allow a third-party attacker to embed malicious instructions into source code, websites, or even through RAG poisoning. When Cascade analyzes such content, it becomes a “confused deputy,” potentially leading to data exfiltration.

## References

* [Windsurf System Prompt](https://github.com/wunderwuzzi23/scratch/blob/master/system_prompts/windsurf_2025-05-30.txt)  
* [Simon Willison - The lethal trifecta](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/)
* [Month of AI Bugs 2025](https://monthofaibugs.com)
* [GitHub Copilot Data Exfiltration Fix](https://embracethered.com/blog/posts/2024/github-copilot-chat-prompt-injection-data-exfiltration/)  
* [Windsurf's CEO goes to Google; OpenAI’s acquisition falls apart](https://techcrunch.com/2025/07/11/windsurfs-ceo-goes-to-google-openais-acquisition-falls-apart/)