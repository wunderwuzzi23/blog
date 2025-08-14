---
title: "Jules Zombie Agent: From Prompt Injection to Remote Control" 
date: 2025-08-14T4:20:58-07:00  
draft: true  
tags: ["llm", "agents", "month of ai bugs"] 
twitter:  
  card: "summary_large_image"  
  site: "@wunderwuzzi23"  
  creator: "@wunderwuzzi23"  
  title: "Jules Zombie Agent: From Prompt Injection to Remote Control (ZombAI)"  
  description: "Jules is vulnerable to Prompt Injection and can be exploited to leak sensitive source code, environment variables and achieve remote command & control by joining a botnet."  
  image: "https://embracethered.com/blog/images/2025/episode14-yt.png"  
---

{{< raw_html >}}
<a id="top_ref"></a>
{{< /raw_html >}}

In the [previous post](/blog/posts/2025/google-jules-vulnerable-to-data-exfiltration-issues/), we explored two data exfiltration vectors that Jules is vulnerable to and that can be exploited via prompt injection. This post takes it further by demonstrating how Jules can be convinced to download malware and join a remote command & control server. 

[![vscode episode 14](/blog/images/2025/episode14-yt.png)](/blog/images/2025/episode14-yt.png)

The information in this post was shared with Google in May 2025.

## Remote Command & Control - Proof Of Concept

The basic attack chain follows the classic AI Kill Chain: 

- Prompt Injection 💉
- Confused Deputy 🤷‍♂️
- Automatic Tool Invocation 🔧

From the worker agent's [system prompt](https://github.com/wunderwuzzi23/scratch/blob/master/system_prompts/jules_worker_tools_2025-05-31.txt) we know there is a tool called `run_in_bash_session`. This tool allows the agent to execute arbitrary operating system commands.

With this knowledge, we can craft a prompt injection payload that targets that tool, uses it to download malware, and execute it.

### What About Network Connectivity?  
Turns out Jules has unrestricted outbound Internet connectivity. 

### Prompt Injection Embedded in GitHub Issue

Prompt injection instructions that originate from GitHub issues seem quite plausible, although not the only place an indirect prompt injection can occur. Recently, a new feature was added to Jules [where one can tag a GitHub issue](https://jules.google/docs/changelog/#jules-now-listens-to-github-issues) with `jules` and the agent will start working on it.

For the C2 server, I used Sliver, and created a basic implant for Jules to download.

The following screenshot shows the prompt injection payload that hijacked Jules and installed the malware:

[![jules prompt injection payload c2](/blog/images/2025/jules-github-issue-c2.png)](/blog/images/2025/jules-github-issue-c2.png)

As you can see, such attacks are surprisingly straightforward.

The result can be seen in this screenshot:

[![jules e2e zombai](/blog/images/2025/jules-c2-e2e.png)](/blog/images/2025/jules-c2-e2e.png)

You can see the initial prompt asking Jules to explore a GitHub issue, which then creates a plan that contains malicious actions. These actions eventually get auto-approved, leading to the download of the Sliver malware binary and a full-system compromise. 

**The attacker can now remote control Jules' dev box, it's now a ZombAI.**

But the attack is so obvious, no one would task Jules with such a GitHub issue you might say. Well, I would challenge that already, but wait for tomorrow's post where we'll add a surprising and scary twist to it!

## Plan Approval Happens Only at Session Start

An interesting edge case is that Jules only proposes a plan at the very beginning of a session. If tasks are created later in the same conversation, tools are invoked automatically without re-triggering the planning and confirmation step.

[![jules e2e combined no plan approval](/blog/images/2025/jules-joins-c2-combined.png)](/blog/images/2025/jules-joins-c2-combined.png)


Also, the initially proposed plan is auto-approved after a certain amount of time, it was initially 20 seconds and then was changed to 120 seconds last time I checked.

## Recommendations and Mitigation

* **External Websites and GitHub Issues**: Be careful when directly tasking Jules to work with untrusted data (e.g. GitHub issues that are not from trusted sources, or websites with documentation that does not belong to the organization, etc.)
* **Private Code and Secrets**: It's probably also good advice, for now at least, to not have Jules work on private, important, source code or give it access to production-level secrets, or anything that could enable an adversary to perform lateral movement.
* **Anti-virus and EDR**: An interesting realization I had when looking at Devin, ChatGPT Codex, and then Jules is that, as enterprises adopt coding agents, we need to explore how to deploy monitoring and detection tools on these systems. This would enable security teams to monitor and understand potentially malicious behavior and actions taken by agents.
* **Outbound Access Control**: Similar to what ChatGPT Codex has enabled by default, I recommend to not allow arbitrary Internet access by default. Instead, allow the configuration to be enabled when needed. This seems like a sensible baseline, then systems can be configured with fine-grained outbound access as needed.

## Conclusion

Today's post demonstrated how coding agents like Jules can be weaponized to run malware via indirect prompt injection. With unrestricted Internet access and shell execution, an attack could have a significant impact. I mentioned this before, but asynchronous agents with access to private data or source code are basically similar to a malicious insider threat - which is a known risk. However, proliferation, scale and velocity of attacks could differ quite a bit.

Tomorrow, we'll look at how such prompt injection attacks can be stealthy and invisible, and how Google Jules will still get hijacked.

## References

* [Month of AI Bugs 2025](https://monthofaibugs.com)
* [Google Jules](https://jules.google.com)
* [Jules Now Listens to GitHub Issues](https://jules.google/docs/changelog/#jules-now-listens-to-github-issues)
* [Bishop Fox - Sliver](https://github.com/BishopFox/sliver)
