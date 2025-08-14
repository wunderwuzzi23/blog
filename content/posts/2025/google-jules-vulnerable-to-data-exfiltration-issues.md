---
title: "Google Jules: Vulnerable to Multiple Data Exfiltration Issues"
date: 2025-08-13T15:20:58-07:00  
draft: true  
tags: ["llm", "agents", "month of ai bugs"]
twitter:  
  card: "summary_large_image"  
  site: "@wunderwuzzi23"  
  creator: "@wunderwuzzi23"  
  title: "Google Jules: Vulnerable to Multiple Data Exfiltration Issues"  
  description: "Jules is vulnerable to Prompt Injection and can be exploited to leak sensitive source code, environment variables and other information on the host"  
  image: "https://embracethered.com/blog/images/2025/episode13-yt.png"  
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

What was interesting is that after a few tests, I noticed that it seems there are multiple agents involved. The [main Jules agent](https://github.com/wunderwuzzi23/scratch/blob/master/system_prompts/jules-main-agent-2025-05-22.md) proposes the plan and then hands off subtasks to worker agents. The [worker agents](https://github.com/wunderwuzzi23/scratch/blob/master/system_prompts/jules_worker_tools_2025-05-31.txt) have different tools and capabilities. 

I'm still not 100% sure if I got the two system prompts right, but that didn't matter much, as the retrieved info allowed me to understand what tools each agent has available. Jules has quite a lot of tools at its disposal.

So, let's focus on the core of this post, data exfiltration avenues.

## Jules' Data Exfiltration Vectors

There were a couple of different data exfiltration vectors identified with Jules that an attacker can leverage during prompt injection. The first two I'm describing in this post and the third one I'll describe more tomorrow, as it goes beyond leaking data. 

**Exfiltration vectors are:**  
* Image Rendering via Markdown Syntax
* `view_text_website` tool   
* Generic RCE with unrestricted Internet access - we will talk more about that in tomorrow's post!

A realistic attack avenue are GitHub issues that contain instructions to trick the AI into initiating attacks. Since June one can [tag a GitHub issue with `Jules` and the agent will start working on it](https://jules.google/docs/changelog/#jules-now-listens-to-github-issues) apparently, but as you know prompt injection can also come from other places.

Let's discuss the various scenarios in detail.

### Exfiltration Vector 1:  Markdown Image Rendering

The first test was to have Jules load a specific webpage to simulate a prompt injection. I have this page here up for a long time https://wuzzi.net/posts/image-backup-how-to/.

[![jules system prompt](/blog/images/2025/jules-wuzzi-website.png)](/blog/images/2025/jules-wuzzi-website.png)

Then I asked Jules to check it out by directly using the tool name...

```
view_text_website("https://wuzzi.net/posts/image-backup-how-to/")
```

Jules followed the instructions, proposed a plan, and also **auto-approves** the plan after some time. Initially the auto-approval happened after 30 seconds, but has later been increased to two minutes last time I checked.

The result was that it took information from an earlier in the chat, appended it to the target third-party URL and then Jules issued the web request to load the image. 

[![jules system prompt leak](/blog/images/2025/jules-prompt-injection-leak.png)](/blog/images/2025/jules-prompt-injection-leak.png)

This leaks information to the third-party site.

Zero-click data exfil image rendering vulnerabilities are quite common and we have seen this in dozens of AI apps over the last 2+ years. So, it's somewhat surprising that such vulnerabilities are not yet caught during development. Luckily, this can be fixed quite easily.

So, next, let's look at something different!

### Exfiltration Vector 2: Calling the view_text_website Tool

Increasingly common, and just as dangerous, is tool calling for data exfiltration. Jules uses a tool called `view_text_website` to load and extract text from web sites. It can also be invoked to send data to an attacker controlled URL.

This is possible without running an OS command. An attacker can just invoke this tool during prompt injection to read any information from the Jules machine and then send it to a third-party server.

For instance, someone might plant a prompt injection payload, e.g. in a GitHub issue that contain instructions to read data from the Jules machine and then invoke the `view_text_website` tool to exfiltrate the data.

I placed the attack payload into a GitHub issue with this as the main description:

[![data exfil e2e](/blog/images/2025/jules-webtool-leakage-e2e.png)](/blog/images/2025/jules-webtool-leakage-e2e.png)

Here you can see the nefarious GitHub issue:

[![jules web tool leakage prompt injection payload](/blog/images/2025/jules-webtool-prompt-injection-payload.png)](/blog/images/2025/jules-webtool-prompt-injection-payload.png)

Once Jules navigates to this Github issue, it gets hijacked and the attacker controlled instructions are followed. It reads a source code file and leaks the entire file to the attacker's server via the `view_text_website` tool.

### Human in the Loop Bypass

As mentioned, Jules normally first proposes a plan which after the user approves is handed off to worker agents to implement the plan. This is actually good design I think, and can aid as as a mitigation step. For instance Jules could propose all URLs it will visit upfront.

The worker agent has more tools and capabilities, such as for instance running bash commands. 

**What was so unique in this case is that the attack completes before Jules proposes the plan. All actions occur in the main Jules agent (not requiring capabilities of the worker agent), or approval of a plan.**

And while writing this, I just realized the same must be true, for the image rendering described earlier!

The result is an end-to-end exploit that grabs another file off the machine and sends the entire contents of the file to the attacker's server.

Check out tomorrow's post for the third data exfiltration example, where we hijack Jules and trick it to run arbitrary code, turning it into a ZombAI.

## The Lethal Trifecta

Recently Simon Willison gave this data leakage pattern a great name by the way, he calls it the [lethal trifecta](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/). I really like the term.  It inspired me to come up with the `AI Kill Chain`: prompt injection -> confused deputy -> automatic tool invocation.

## Recommendations and Mitigations

There are a couple of improvements that can be made to mitigate such issues:
* For image rendering a CSP has been shown to mitigate loading resources from trusted locations and consider additional mitigations (e.g not rendering images at all in the chat)  
* For the `view_text_website` create a configurable allowlist of domains (like ChatGPT Codex) that Jules can access, or ask for human approval, and/or 
* Highlight and let the user review all external communication explicitly in the proposed plan, and do not connect to other domains unless a domain is proposed in the plan

As a user, I would recommend to be careful giving sensitive information or secrets to asynchronous coding agents, and with Jules carefully review the plan it proposes. Unless an attack targets the planner agent, there is likely evidence of malicious intent present in the detailed plan it shares upfront.

These vulnerabilities and observations were disclosed to Google on May 21, 2025. 

Recently Google Jules went out of beta, and is now generally available. However, the weaknesses have not been entirely mitigated as far as I can tell. Google asked me to share these write-ups upfront, which I did last week, however by the time of publishing I hadn't yet heard back on latest status - and I will update this section here if I learn more.

## Conclusion

Google Jules is not properly sandboxed by default (e.g. Content Security Policy, outbound network access), which means it is capable of leaking any information accessible to it to third-party servers. This behavior can be forced with an indirect prompt injection attack.

Part of the reason for the Month of AI Bugs series is to help raise awareness of common AI application security vulnerabiliites, but also that I believe the industry needs to start more aggressively tackling security vulnerabilities, as with the advent of offensive AI, the number of identified vulnerabilities is likely to rise significantly, giving attackers the upper hand - at least until defensive catches up.

## References

* [Google Jules](https://jules.google.com)
* [Month of AI Bugs 2025](https://monthofaibugs.com)
* [Lethal trifecta](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/)
* [Jules Now Listens to GitHub Issues](https://jules.google/docs/changelog/#jules-now-listens-to-github-issues).

