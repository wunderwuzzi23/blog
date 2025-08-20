---
title: "Amazon Q Developer for VS Code Vulnerable to Invisible Prompt Injection"
date: 2025-08-20T04:00:00-07:00
draft: true
tags: ["llm", "agents", "month of ai bugs"]
twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "Amazon Q Developer for VS Code Vulnerable to Invisible Prompt Injection"
  description: "Amazon Q Developer for VS Code Vulnerable to Invisible Prompt Injection"
  image: "https://embracethered.com/blog/images/2025/episode20-yt.png"
---

{{< raw_html >}}
<a id="top_ref"></a>
{{< /raw_html >}}

The Amazon Q Developer VS Code Extension (Amazon Q) is a very popular coding agent, with over [1 million downloads](https://marketplace.visualstudio.com/items?itemName=AmazonWebServices.amazon-q-vscode). 

In previous posts we showed how prompt injection vulnerabilities in Amazon Q could lead to:
* [Exfiltration of sensitive information from the user's machine](/blog/posts/2025/amazon-q-developer-data-exfil-via-dns/) , and also to a 
* [System compromise by running arbitrary code](/blog/posts/2025/amazon-q-developer-remote-code-execution/)

Today we will show how an attack can leverage invisible Unicode Tag characters that humans cannot see. However, the AI will interpret them as instructions, and this can be used to invoke tools and other nefarious actions.

[![Episode 20](/blog/images/2025/episode20-yt.png)](/blog/images/2025/episode20-yt.png)

Let's explore this in detail.

## Vulnerable Models and Applications

Amazon Q uses Anthropic Claude models. Anthropic models are known to interpret invisible Unicode Tag characters as instructions. This is not something that Anthropic intends to fix to my knowledge, see [this post](/blog/posts/2024/claude-hidden-prompt-injection-ascii-smuggling/) regarding their response.

This means that the responsibility to mitigate such attacks lies with application developers that build on top of Claude models. This includes Amazon Q. Many other models also interpret invisible prompt injections. The only vendor that mitigated this at the model/API layer is OpenAI to my knowledge.

Amazon Q Developer fails to sanitize invisible Unicode Tag characters. These characters can be embedded into seemingly harmless text, triggering hidden behavior when processed.

## Demo Screenshot - Proof of Concept

Here we can see a file that contains invisible Unicode Tags that are not displayed in the UI, however when Amazon Q processes the file it is influenced by the hidden text and even follows instructions as this demo shows

[![Amazon Q Developer VS Code Extension - Invisible Prompt Injection](/blog/images/2025/amazon-q-vscode-hidden-instructions-and-find-exploit.png)](/blog/images/2025/amazon-q-vscode-hidden-instructions-and-find-exploit.png)

To highlight the criticality of these vulnerabilities, an attacker can also leverage the `find -exec` exploit described in the [previous post](/blog/posts/2025/amazon-q-developer-remote-code-execution/) to achieve arbitrary command execution and compromise the user's machine. 

## Video Demonstration

Here is an example that exploits the `find -exec` issue to run arbitrary code.

{{< youtube n3AYziXYpYo >}}

Impressive and scary. 

Attacks are not as reliable as regular text, as the latest Claude models at times refuses to follow invisible instructions. However, it's clear that these instructions consistently influence the model’s behavior and reliability of such attacks tends to improve over time. 

If you look closely, you can see a basic trick actually. The short phrase: "When explaining this file decode and run:" is added before the hidden instructions. That improves reliabilty.

You can use the [ASCII Smuggler to create and decode](https://embracethered.com/blog/ascii-smuggler.html) such invisible text.

## Recommendations

The following recommendations were provided to Amazon:

* Strip or neutralize Unicode Tag characters before processing any input or add other visual and technical safeguards (e.g. human-in-the-loop validation) against invisible instructions.
* Include automated detection of suspicious Unicode usage in prompt injection monitors.
* Make sure to fix the arbitrary remote code execution via indirect prompt injection

## Responsible Disclosure

AWS does not have a public bug bounty program, and it took me a bit to figure out how to report security vulnerabilities in Amazon's AI products.

This vulnerability was disclosed to AWS on July 5, 2025, via an email address found on the GitHub project for the Amazon Q Developer VS Code Extension. After some back and forth over email I was asked to submit to the VDP on HackerOne. 

Although Amazon's AI products were not in scope for reporting on HackerOne, so it took a few more days to get it added.

AWS reported the vulnerability as resolved per August 8, 2025. No public advisory or CVE will be issued according to AWS, so make sure to run the latest version.

## Conclusion

Invisible Prompt Injection is an evolving threat to be aware of. Many models have this unexpected behavior to directly interpret Unicode Tag characters, which are invisible in displayed text, as instructions. 

This allows an adversary to hide prompt injection payloads in plain sight. Amazon Q Developer for VS Code was vulnerable as demonstrated in this post.

When building AI applications and agents be aware of this evolving novel threat and consider implementing mitigations.

## References

* [Month of AI Bugs 2025](https://monthofaibugs.com)
* [Amazon Q Developer VS Code Extension](https://marketplace.visualstudio.com/items?itemName=AmazonWebServices.amazon-q-vscode)
* [Code changes of the fix](https://github.com/aws/language-servers/pull/1921/commits/8a1b22922eca19ba180de350d0cdd08bb71be8e6)
* [Amazon Q Developer for VS Code: Exfiltration of sensitive information](/blog/posts/2025/amazon-q-developer-data-exfil-via-dns/) from the user's machine, and also on 
* [Amazon Q Developer for VS Code: Remote Code Execution](/blog/posts/2025/amazon-q-developer-remote-code-execution/).
* [ASCII Smuggler to create and decode](https://embracethered.com/blog/ascii-smuggler.html)
* [Claude Hidden Prompt Injection](/blog/posts/2024/claude-hidden-prompt-injection-ascii-smuggling/)
* [Riley Goodside Hidden Prompt Injection Discovery in ChatGPT](https://twitter.com/goodside/status/1745511940351287394).