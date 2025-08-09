---
title: "OpenHands and the Lethal Trifecta: How Prompt Injection Can Leak Access Tokens"  
date: 2025-08-09T03:00:58-07:00  
draft: true  
tags: ["llm", "agents", "month of ai bugs"]
twitter:  
  card: "summary_large_image"  
  site: "@wunderwuzzi23"  
  creator: "@wunderwuzzi23"  
  title: "OpenHands and the Lethal Trifecta: Leaking Your Agent's Secrets"  
  description: "OpenHands Coding Agent Data Exfiltration Threats"  
  image: "https://embracethered.com/blog/images/2025/episode9-yt.png"  
---

{{< raw_html >}}
<a id="top_ref"></a>
{{< /raw_html >}}

Another day, another AI data exfiltration exploit. Today we talk about [OpenHands](https://github.com/All-Hands-AI/OpenHands/), formerly referred to as OpenDevin. It's created by All-Hands AI.  

The OpenHands agent renders images in chat, which enables zero-click data exfiltration.

Simon Willison recently gave this data exfiltration attack pattern a great name: [Lethal Trifecta](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/).

[![OpenHands - Lethal Trifecta Data Exfiltration](/blog/images/2025/episode9-yt.png)](/blog/images/2025/episode9-yt.png)

We discuss this specific image based attack technique frequently. Sometimes a message must be repeated multiple times to raise awareness and become mainstream knowledge. 

It is one of the most common AI application security vulnerabilities, a larger section of the [Trust No AI - Prompt Injection Along the CIA Security Triad](https://arxiv.org/abs/2412.06090) is dedicated to it as well.


## A Prompt Injection To Leak Your GitHub Token! 

A prompt injection payload on a website, in source code or uploaded document can contain malicious instructions to force rendering of an image via Markdown syntax.

**An attacker can append information to the URL, leading to data exfiltration.**

In particular this demonstration exploit shows how a prompt injection attack exfiltrated the `GITHUB_TOKEN` inside the OpenHands container/VM. 

## Model Refuses To Exfiltrate the GitHub Token

Initial attempts to directly access the GitHub token were refused because the model didn't like the idea of access and sending the token to the third-party server. 

[![openhands refusal 1](/blog/images/2025/all-hands-github-token-refusal-data-exfil.png)](/blog/images/2025/all-hands-github-token-refusal-data-exfil.png)

However, the reason I'm writing these posts is to make it clear that the model cannot be trusted and that there are ways to bypass such refusals.

## Bypassing the Refusal via Basic Pattern Matching

So with a little bit of trickery, I found this reliable way to exfiltrate the token. There are some interesting lessons to be learned. The prompt injection looks like this:

[![OpenHands - Prompt Injection Env](/blog/images/2025/all-hands-github-token-success-pi.png)](/blog/images/2025/all-hands-github-token-success-pi.png)

The trick was to only look for a subpattern, like  `hp_`, which is a matching pattern for a classic GitHub personal access token. The pattern match for `ghp_` was refused a few times.

Once that was bypassed though, another roadblock showed up...

## Secrets Redacted In Output - How To Bypass?

When `OpenHands` identifies the `GITHUB_TOKEN` as secret it will not render it in clear text:

[![openhands refusal 2 - does not render the github token](/blog/images/2025/all-hands-redacted-small.png)](/blog/images/2025/all-hands-github-token-hidden.png)

Unfortunately a bypass for something like this is usually trivial using Base64 or other custom encodings.

## Walkthrough 

This is the prompt injection payload PoC coming from a webpage:

[![github token leak](/blog/images/2025/all-hands-github-token-success-pi.png)](/blog/images/2025/all-hands-github-token-success-pi.png)

If `OpenHands` encounters such a payload it will interpret it as instructions and execute them:
 [![github token leak](/blog/images/2025/all-hands-github-token-exfil-via-image-sm.png)](/blog/images/2025/all-hands-github-token-exfil-via-image.png)

This leads to data exfiltration of the `GITHUB_TOKEN` to the third-party server.

## Impact

The `GITHUB_TOKEN`, or any other information present in the chat history, memory or inside the OpenHands host is subject to data exfiltration via this zero-click channel.

This is the web request the server received:
[![webserver log github token leak](/blog/images/2025/all-hands-web-server-log-github-token.png)](/blog/images/2025/all-hands-web-server-log-github-token.png)

As you can see, the server received the Base64-encoded token. 

Most vendors classify such vulnerabilities with High severity, and Microsoft marks such zero-click data leaks even Critical in their [AI Bug Bar](https://www.microsoft.com/en-us/msrc/aibugbar).

Scary stuff.

## Responsible Disclosure

* March 13th, 2025: Disclosed to All-Hands AI via the GitHub [security tab](https://github.com/All-Hands-AI/OpenHands/security) of the project. 

* March 20th, 2025: Follow up, no response.

* March 30th, 2025: Vulnerability not triaged. Hence, I created a [public ticket highlighting that security issues are not being triaged](https://github.com/All-Hands-AI/OpenHands/issues/7594). In that ticket it was confirmed that someone would look at the security report. 

* March 31st, 2025: I wanted to disclose this vulnerability (and fix) at the [Agentic AI Summit](https://zenity.io/resources/events/ai-agent-security-summit-2025-on-demand) as part of the keynote, but I removed as no fix was available.

* June 18th, 2025: Inquiry since the 90+ day period for responsible disclosure passed. No reply.

* July 10th, 2025: Additional inquiries and informed All-Hands AI of disclosure in August.

* August 9th, 2025: 148 days after private disclosure. To follow industry best-practices for responsible disclosure this vulnerability is now shared publicly to ensure users can take steps to protect themselves and make informed risk decisions. 

Interesting coincidence (or it might be related to an out-of-band email that I sent yesterday), but today when publishing this post I received a response. So, hopefully we will see improvements soon.

## Mitigation and Recommendations

The fix is to not render images from untrusted domains. For instance a `Content-Security-Policy` to only loads images from trusted domains could effectively mitigate this risk.

Alternatively, some vendors decided to show a pop-up including the full URL to the user for confirmation before navigating off-domain (this includes clickable hyperlinks as well by the way).

## Conclusion

This post showed how agentic systems commonly have the same AI application security vulnerabilities that we have discussed quite often over the last two years.

Some vendors of agentic systems over-rely on model behavior to enforce security, which is often trivial to bypass if there is an attacker in the loop via indirect prompt injection. To enforce security boundaries, the focus has to shift to what can be controlled.

Stay safe.

## References

* [Trust No AI - Prompt Injection Along the CIA Security Triad](https://arxiv.org/abs/2412.06090)
* [All-Hands AI - OpenHands](https://github.com/All-Hands-AI/OpenHands/)  
* [Simon Willison - The lethal trifecta](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/)  
* [All-Hands - Cloud Service](https://www.all-hands.dev/)  
* [Month of AI Bugs 2025](https://monthofaibugs.com)
* [Zenity - Agentic AI Summit](https://zenity.io/resources/events/ai-agent-security-summit-2025-on-demand)


