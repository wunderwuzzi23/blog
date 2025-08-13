---
title: "Google Jules: Vulnerable Vulnerable To Multiple Data Exfiltration Issues"
date: 2025-08-13T11:20:58-07:00  
draft: true  
tags: ["llm", "agents", "month of ai bugs"]
twitter:  
  card: "summary_large_image"  
  site: "@wunderwuzzi23"  
  creator: "@wunderwuzzi23"  
  title: "Google Jules: Coding Agent is vulnerable to data exfiltration via markdown image rendering"  
  description: "Jules is vulnerable to Prompt Injection and can be exploited to leak sensitive source code, environment variables and other information on the host"  
  image: "https://embracethered.com/blog/images/2025/TODO"  
---

{{< raw_html >}}
<a id="top_ref"></a>
{{< /raw_html >}}

This post explores data exfiltration attacks in Google Jules, an asynchronous coding agent. This is the first of three posts that will highlight my research on Google Jules in May 2025. All information provided was also shared with Google at that time.

This first post will focus on data exfiltration, the [lethal trifecta](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/).

[![vscode episode 13](/blog/images/2025/episode13-yt.png)](/blog/images/2025/episode13-yt.png)

But let's first talk about Jules' system prompt.

## Jules' System Prompt and Multiple Agents

To grab the system prompt I just asked it to write it into a file.

[![jules system prompt](/blog/images/2025/jules-system-prompt.png)](/blog/images/2025/jules-system-prompt.png)

What was interesting is that after few tests, I noticed that it seems there are multiple agents involved. The [main Jules agent](](https://github.com/wunderwuzzi23/scratch/blob/master/system_prompts/jules-main-agent-2025-05-22.md)) proposes the plan and then hands off subtasks to worker agents. The [worker agents]((https://github.com/wunderwuzzi23/scratch/blob/master/system_prompts/jules_worker_tools_2025-05-31.txt)) have different tools and capabilities. 

I'm still not 100% sure if I got the two system prompts right, but it doesn't matter much the retrieved info allowed me to understand what tools each agent has available. And Jules has quite a lot of tools to it's disposal.

So, let's focus on the core of this post, data exfiltration avenues.


## Jules' Data Exfiltration Vectors

There were a couple of different data exfiltration vectors identified with Jules that an attacker can leverage during prompt injection. The first two I'm describing in this post and the third one I'll describe more tomorrow, as it goes beyond leaking data. 

**Exfiltration vectors are:**  
* Image Rendering via Markdown Syntax
* `view_text_website` tool   
* Generic RCE with unrestricted Internet access - we will talk more about that in tomorrow's post!

A realistic attack avenue are GitHub issues that might contain instructions that trick the AI to initiate attacks. Since June one can [just tag a GitHub issue with `Jules` and the agent will start working on it](https://jules.google/docs/changelog/#jules-now-listens-to-github-issues), but as you know prompt injection can also come from other places.

Let's discuss the various scenarios in detail.

### Exfiltration Vector 1:  Markdown Image Rendering

The first test was to have Jules load a specific webpage to simulate a prompt injection. I have this page here up for a long time https://wuzzi.net/posts/image-backup-how-to/.

[![jules system prompt](/blog/images/2025/jules-wuzzi-website.png)](/blog/images/2025/jules-wuzzi-website.png)

Then I asked Jules to check it out...

```
view_text_website("https://wuzzi.net/posts/image-backup-how-to/")
```

Jules follows the instructions, and proposes a plan, and it also **auto-approves** the plan after some time. Initially this was a 30 second countdown, and has later been increased to two minutes last time I checked.

The result was that it took information from an earlier message, appended it to the target attacker controlled URL and then Jules issued the web request to load the image. 

[![jules system prompt leak](/blog/images/2025/jules-prompt-injection-leak.png)](/blog/images/2025/jules-prompt-injection-leak.png)

This leaks information to the attacker. You can find the prompt injection payload on the website directly by the way, if you are curious to see the text on the webpage. 

This zero-click image rendering vulnerability is quite common and we have seen this in dozens of AI applications over the last 2+ years.

So, next, let's look at something different!

### Exfiltration Vector 2: Calling the view_text_website Tool

More novel, although increasingly common, and just as dangerous, is the tool Jules uses to load and extract text from web sites. It can also be invoked to send data to an attacker controlled URL.

This is possible without running an OS command. An attacker can just invoke this tool during prompt injection to read any information from the Jules machine and then exfiltrate it using the tool.

Attacker plants prompt injection payload, e.g. in a GitHub issue, that contain instructions to read data from the Jules machine and then invoke the `view_text_website` tool to exfiltrate the data.

I placed the attack payload into a GitHub issue with this as the main description:

[![data exfil e2e](/blog/images/2025/jules-webtool-leakage-e2e.png)](/blog/images/2025/jules-webtool-leakage-e2e.png)

Here you can see the nefarious GitHub issue:

[![jules web tool leakage prompt injection payload](/blog/images/2025/jules-webtool-prompt-injection-payload.png)](/blog/images/2025/jules-webtool-prompt-injection-payload.png)

The prompt injection text is also in the [Appendix](#appendix) if you are curious.

Once Jules navigates to this Github issue, it gets hijacked and the attacker controlled instructions are followed. It reads a source code file and leaks the entire file to the attacker's server via the `view_text_website` tool.

### Human in the Loop Bypass

As mentioned, Jules normally first proposes a plan which after the user approves is handed off to worker agents to go implement the plan. 

The worker agent has more tools and capabilities, such as for instance running bash commands. 

**What was so unique in this case is that the attack completes, before Jules proposes a plan. All actions occur in the main Jules agent (not requiring capabilities of the worker agent), or approval of a plan.**

And while writing this, I just realized they same must be true, for the image rendering described earlier!

The result is an end-to-end exploit that grabs another file off the machine and sends the entire contents of the file to the attacker's server.

This was the second path for data exfiltration. Check out the post tomorrow for the third one, where we hijack Jules and trick it to run malware, which turns it into a ZombAI.

## The Lethal Trifecta

Recently Simon Willison gave this data leakage pattern a great name by the way, he called it the [lethal trifecta](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/). I really like that, and it inspired me to come up with the AI Kill Chain: prompt injection -> confused deputy -> automatic tool invocation.

## Recommendations and Mitigations

There are a couple of improvements that can be made to mitigate such issues:
* For image rendering a CSP has been shown to be effective to only load resources from trusted locations and consider additional mitigations (e.g not rendering images at all)  
* For the `view_text_website` create a configurable allowlist of domains that Jules can access, ask for human approval, or highlight all external communication explicitly in the plan before connecting, do not connect to other domains.

These vulnerabilities were disclosed to Google on May 21, 2025. 

Recently Google Jules went out of beta, and is now generally available. However the security vulnerabilities have not been mitigated as part of general availability release as far as I can tell.

## Conclusion

Google Jules is not in a sandbox (e.g. Content Security Policy, outbound network access) which means Jules is capable of leaking any information accessible to it to third-party servers.

## References

* [Google Jules](https://jules.google.com)
* [Month of AI Bugs 2025](https://monthofaibugs.com)
* [Lethal trifecta](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/)
* [Jules Now Listens to GitHub Issues](https://jules.google/docs/changelog/#jules-now-listens-to-github-issues).

