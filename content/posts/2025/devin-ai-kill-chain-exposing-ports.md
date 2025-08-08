---
title: "AI Kill Chain in Action: Devin AI Exposes Ports to the Internet with Prompt Injection"  
date: 2025-08-08T00:02:58-07:00  
draft: true  
tags: ["llm", "agents", "month of ai bugs"]
twitter:  
  card: "summary_large_image"  
  site: "@wunderwuzzi23"  
  creator: "@wunderwuzzi23"  
  title: "Devin: Exposing Ports To The Internet With Prompt Injection"  
  description: "AI Kill Chain in Action: Devin AI Exposes Ports to the Internet with Prompt Injection"  
  image: "https://embracethered.com/blog/images/2025/episode8-yt.png"  
---

{{< raw_html >}}
<a id="top_ref"></a>
{{< /raw_html >}}


Today let's explore Devin's system prompt a bit more. Specifically, an interesing tool that I discovered when reading through it. 

Hidden in Devin’s capabilities is a tool that can open any local port to the public Internet. That means, with the right indirect prompt injection nudge, Devin can be tricked into publishing sensitive files or services for anyone to access.

[![Devin Title Expose Port AI Kill Chain - Episode 8](/blog/images/2025/episode8-yt.png)](/blog/images/2025/episode8-yt.png)

This tunneling feature can be invoked without a human in the loop.

## What is the `expose_port` tool?

Here is the tool definition for `expose_port` as seen in [Devin's system prompt](https://github.com/wunderwuzzi23/scratch/blob/master/system_prompts/devin-2025-04-10.md):

```
Exposes a local port to the internet and returns a public URL. Use this command to let 
the user test and give feedback for frontends if they don't want to test through your 
built-in browser. Make sure that apps you expose don't access any local backends.
```

So the tool allows Devin to expose its Devbox on the Internet. The idea appears to be for development and maybe testing, so that Devin can create a website and make it externally accessible.

But, the immediate thought I had when seeing this was that an indirect prompt injection attack can expose a random port of Devin's machine to the Internet. This will allows exposing sensitive information and creating backdoor access quite easily.

## AI Kill Chain - Prompt Injection To Host Exposure

**The AI Kill Chain consists of the three typical steps:**
- Prompt Injection 💉
- Confused Deputy 🤷‍♂️
- Automatic Tool Invocation 🔧

**Here are the details on how such an attack chain works specifically in this case:**

1. Adversary hosts a malicious prompt injection payload on a website   
2. Devin navigates to the website during its research  
3. The website takes control of Devin (indirect prompt injection) and  
	a. Creates a Python web server exposing all files on port `8000`  
	b. Invokes the `expose port` tool to share the port on the Internet. `expose_port` returns a public URL ending in `.devinapps.com`
	c. The exploit then exfiltrates that URL and sends it to the attacker  
4. Attacker can access the app over the Internet and view the files

## Building A Proof-of-Concept Demonstration

I developed a PoC that hijacks Devin to invoke the “expose port” feature, showing how  
an adversary can perform a prompt injection attack via untrusted data from a website.

Consider a scenario where you ask Devin to look at websites or investigate a GitHub issue. You can just task Devin via Slack actually, and afterwards it works entirely unsupervised.

[![devin-system-prompt-expose-port](/blog/images/2025/devin-tunnel-slack.png)](/blog/images/2025/devin-tunnel-slack.png)

A key concern arises when Devin interacts with potentially malicious websites that contains prompt injection instructions.

Turns out initially Devin refused these requests, but one trick I learned over the last few months researching browsing and computer use agents is that you can bounce them often off multiple domains or stages and slowly steer them into the "right" direction. 

So, in order to make it work, I built a multi-stage attack that separates the malicious actions in a way that makes them less obvious, and Devin executed the instructions without objection.

### **Stage 1:** Creates webserver to expose filesystem and then redirects to the second webpage with additional instructions

Here is the first stage prompt injection. This is the first stage of the multi-page attack:

[![devin-system-prompt-expose-port](/blog/images/2025/devin-tunnel-stage-1.png)](/blog/images/2025/devin-tunnel-stage-1.png)

### **Stage 2:**  Exposes the port and leaks the URL to access the web application

This second stage uses the image markdown vulnerability we described yesterday to leak the publicly created endpoint to the third-party:

[![devin-system-prompt-expose-port](/blog/images/2025/devin-tunnel-stage-2.png)](/blog/images/2025/devin-tunnel-stage-2.png)

Here is the second stage prompt injection:
[![devin-system-prompt-expose-port](/blog/images/2025/devin-expose-port.png)](/blog/images/2025/devin-expose-port.png)

Now the public `.devinapps.com` link to the internal server is now leaked to the attacker’s web server.
[![devin-system-prompt-expose-port](/blog/images/2025/devin-tunnel-server-log-including-proxylink.png)](/blog/images/2025/devin-tunnel-server-log-including-proxylink.png)

### **Voila:** Now the application can be accessed over the Internet! 

Using that leaked URL the attacker now has access to the files on the server over the Internet:  
[![devin-system-prompt-expose-port](/blog/images/2025/devin-tunnel-exposed-proof.png)](/blog/images/2025/devin-tunnel-exposed-proof.png)

As a result, Devin created the Python web server and exposed the file system on the Internet!

## Do Attacks Require Prompt Injection?

An often-overlooked aspect is that even without prompt injection, Devin or any other AI agent for that matter could decide to take malicious actions due to backdoors or other latent behavior present in the model itself.

## Responsible Disclosure and Recommendations

These vulnerabilities were reported to Cognition on April 6th, 2025 and receipt acknowledged a few days later. Follow-up queries around fix timelines and status have remained unanswered.

According to responsible disclosure guidelines (it has been 120+ days since the vendor was notified) this information is now publicly shared to raise awareness and empower users to protect themselves.

A specific recommendation for sensitive tool invocations was to not depend on model-behavior, in-chat confirmations or prompting for secure tool invocation, but develop an out-of-band validation step to allow the user to approve sensitive operations, like exposing a port on the Internet.

Please see the previous post for details of other recommendations provided to Cognition.

## Conclusion

In this post we highlighted the importance of reviewing the system prompt for tools that allow further elevation of privilege. The classic AI Kill Chain steps of prompt injection, confused deputy and automatic tool invocation. 

Devin executes consequential tool commands, such as exposing a port, without seeking human verification. No additional mitigations were observed (e.g. enforcing IP firewall rules or similar).

Such design pattern can have dangerous consequences for autonomous systems as can be seen with this example.

Stay safe.

## References

* [Devin AI](https://devin.ai/)  
* [Cognition announces Devin](https://cognition.ai/blog/introducing-devin)   
* [Devin System Prompt](https://github.com/wunderwuzzi23/scratch/blob/master/system_prompts/devin-2025-04-10.md)
* [Month of AI Bugs 2025](https://monthofaibugs.com)

