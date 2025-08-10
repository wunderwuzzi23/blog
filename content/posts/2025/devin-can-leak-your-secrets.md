---
title: "How Devin AI Can Leak Your Secrets via Multiple Means"  
description: ""  
date: 2025-08-07T08:20:58-07:00  
draft: true  
tags: ["llm", "agents", "month of ai bugs"]
twitter:  
  card: "summary_large_image"  
  site: "@wunderwuzzi23"  
  creator: "@wunderwuzzi23"  
  title: "How Devin AI Can Leak Your Secrets via Multiple Means"  
  description: "Data gone, oops."  
  image: "https://embracethered.com/blog/images/2025/episode7-yt.png"  
---

{{< raw_html >}}
<a id="top_ref"></a>
{{< /raw_html >}}

In this post we show how an attacker can make Devin send sensitive information to third-party servers, via multiple means. This post assumes that you read the [first post](/blog/posts/2025/devin-i-spent-usd500-to-hack-devin/) about Devin as well. 

[![Devin Title Image Month of AI Bugs Episode 6](/blog/images/2025/episode7-yt.png)](/blog/images/2025/episode7-yt.png)

But here is a quick recap: During an indirect prompt injection Devin can be tricked into download malware and extract sensitive information on the machine. But there is more...

Let's explore how Devin can leak sensitive information and send it to a third-party server.

## Secrets Management in Devin

Devin has a built-in secrets management platform. Users can define secrets and those will become available during runtime as environment variables.

[![devin-secrets](/blog/images/2025/devin-store-secret.png)](/blog/images/2025/devin-store-secret.png)

For these exploit demos these secrets are the target of our exfiltration efforts. Of course, an attacker can target any other information present on Devin's box or anything Devin has access to.

## Data Exfiltration In AI Systems

An important step when reviewing an AI systems for security vulnerabilities is to analyze the system prompt, in particular, the tools an AI has access to. 

When I [looked at Devin's system prompt](https://github.com/wunderwuzzi23/scratch/blob/master/system_prompts/devin-2025-04-10.md), I noticed the `Browser` and `Shell` tools. These allow a degree of freedom that allows an attacker to achieve data exfiltration.

There are actually multiple 0-click attacks that allow data exfiltration. Also, I did this research in April 2025 and since then additional integrations and tools might have been added.

### **Exfil Vector 1: Using The Shell Tool**

Devin can run terminal commands via the Shell tool defined in the system prompt. 

Using this tool, data exfiltration can be achieved by executing commands such as `curl` or `wget`, or have Devin write a Python script that connects to a third-party server and run it. **Also, since I haven't mentioned it yet, Devin by default has unrestricted access to the Internet.**

Here is an indirect prompt injection demonstration that highlights how this works. We have a GitHub issue that asks to investigate a runtime error:

[![devin-curl prompt injection Curl](/blog/images/2025/devin-promp-injection-curl.png)](/blog/images/2025/devin-promp-injection-curl.png)

Then we ask Devin to investigate the latest open GitHub issue:
[![devin-curl-chatbox visible](/blog/images/2025/devin-curl-chatbox.png)](/blog/images/2025/devin-curl-chatbox.png)

And observe how it is hijacked and follows the nefarious instructions. Here is the full screen version to show all the details:  

[![devin-curl Leaks Data](/blog/images/2025/devin-curl-leak.png)](/blog/images/2025/devin-curl-leak.png)

You can see that it encountered multiple errors, but figures out how to correctly construct the `curl` request to leak the data.

[![devin-curl-tool](/blog/images/2025/devin-curl-webserver.png)](/blog/images/2025/devin-curl-webserver.png)

The final result is that the third-party received the environment variables.

### **Exfil Vector 2: Using the Browsing Tool**

Devin can operate a web browser. One of the obvious ways to leak data is to use a browsing tool by navigating to an attacker-controlled URL with the sensitive data appended. 

Here is how this looks like:

[![devin-browser-prompt-injection-image](/blog/images/2025/devin-browse-tool-image.png)](/blog/images/2025/devin-browse-tool-image.png)

You can see the final step of the attack in the above screenshot, and on the left you can see that it used `env` and `sort` commands, and also created a python script to correctly encode the info before using the browsing tool. Quite fascinating. Make sure to check out the video as well.

### Combination Trickery Tests

Additionally, I was curious if the trick that OpenAI mitigated with [ChatGPT Operator earlier this year after I reported it](https://embracethered.com/blog/posts/2025/chatgpt-operator-prompt-injection-exploits/) would also work here. 

Here is the result:
[![devin-combine-tool](/blog/images/2025/devin-combine-tool-paste.png)](/blog/images/2025/devin-combine-tool-paste.png)

As you can see Devin had no issues pasting Jira and Slack tokens into a third-party website, all because it investigated a GitHub issue with malicious text embedded.

### **Exfil Vector 3: Markdown Image Rendering**

Devin is also vulnerable to rendering images from untrusted domains. **This is one of the most common AI application security vulnerabilities.** By rendering an image in the chat box and appending sensitive data, an attacker can leak the info to third-party servers.

Below is a demo screenshot that shows the attack chain:

[![devin-ascii-smuggling](/blog/images/2025/devin-markdown-exfil-e2e.png)](/blog/images/2025/devin-markdown-exfil-e2e.png)

In this case Devin visited a website, got hacked and leaked environment variables.  
[![devin-ascii-smuggling](/blog/images/2025/devin-base64-decode.png)](/blog/images/2025/devin-base64-decode.png)

Notice the trick for `grep` is used again here, not looking for the exact string, just a pattern. This bypasses refusal requests that were otherwise observed at times.

### **Exfil Vector 4: Hyperlinks and Invisible Unicode Characters Sent To Slack**

The last attack avenue to highlight is that Devin can be connected to Slack. So, you can just give it a task and it will go off work on it, with little to no human oversight.

One trick for data leakage we first described about 2 years ago is "link unfurling", which is a common feature in chatbots to show previews. The attack idea being that we trick Devin to read some sensitive information, like an environment variable, and then have it render a hyperlink back to the Slack thread. 

The attacker can also entirely hide the sensitive data within the UI with invisible Unicode Tag characters (ASCII Smuggling).

[![devin-ascii-smuggling](/blog/images/2025/devin-ascii-smuggle-works.png)](/blog/images/2025/devin-ascii-smuggle-works.png)

However, when testing this it turns out that **Cognition must have disabled link unfurling** when Devin posts, as the information was not automatically leaked! So this is great!

Reviewing how Devin actually created the invisible text was enlightening, because it actually wrote Python code to do so!

[![devin-create-invisible-text-with-python](/blog/images/2025/devin-ascii-smuggle-python-encode-code.png)](/blog/images/2025/devin-ascii-smuggle-python-encode-code.png)

There is still a social engineering attack angle though, because if the user clicks the benign seeming link, which doesn't appear to contain the secret, the browser will URL encode the hidden characters and send them as part of the HTTP request. This is something, I first [demonstrated last year](https://embracethered.com/blog/posts/2024/m365-copilot-prompt-injection-tool-invocation-and-data-exfil-using-ascii-smuggling/) with the Copirate research and Microsoft Copilot that Microsoft addressed by the way.

Here is what happened when I clicked the link that Devin sent to Slack:

[![devin-ascii-smuggling](/blog/images/2025/user-clicks-link-small.png)](/blog/images/2025/devin-user-clicks-slack-link.png)

You can expand the image to see the full screen version.

**This is an important reminder of how the output of LLMs can have side effects that don't surface immediately but surface in downstream systems and processing.** 

## Video Demonstration


**Note:** I will package all three Devin posts into a single video that I'll release with the third post about Devin (planned for this weekend), then I'll update this section here.


## Responsible Disclosure and Recommendations

These vulnerabilities were reported to Cognition on April 6th, 2025 and the report acknowledged a few days later. Multiple follow-up queries around fix timelines and status remained unanswered. 

According to responsible disclosure guidelines (it has been 120+ days since the vendor was notified) this information is now publicly shared to raise awareness and empower users to protect themselves.

One of the key recommendations is to implement security controls to restrict internet access by default, and allow fine-grained control, and disable image markdown and hyperlink rendering to untrusted domains in the chat box.

Please see the [previous post for details of other recommendations](/blog/posts/2025/devin-i-spent-usd500-to-hack-devin/) provided to Cognition.

## Conclusion

During a prompt injection attack from untrusted data, an attacker can gain access to secrets and other data on Devin's Devbox and leak them to an adversary via multiple means.

**Many vendors of agentic systems over-rely on the model doing the right thing.**

Agents are naive. Untrusted data, like text on a website, can take control of the AI and send data to external servers.

Stay safe out there.

## References

* [Devin AI](https://devin.ai/)  
* [Cognition announces Devin](https://cognition.ai/blog/introducing-devin)   
* [Operator and improvements OpenAI made](https://embracethered.com/blog/posts/2025/chatgpt-operator-prompt-injection-exploits/)
* [Devin System Prompt](https://github.com/wunderwuzzi23/scratch/blob/master/system_prompts/devin-2025-04-10.md)
* [Month of AI Bugs 2025](https://monthofaibugs.com)

