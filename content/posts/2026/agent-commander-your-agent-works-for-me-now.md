---
title: "Agent Commander: Promptware-Powered Command and Control"
date: 2026-03-16T20:20:58-07:00
draft: true
tags:
  - llm
  - prompt injection
  - red
  - ttp
twitter:
  card: summary_large_image
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "Agent Commander demonstrates how  personal agents can be tricked to join novel promptware-powered command and control infrastructure to continuously receive tasks and objectives from third-party adversaries."
  image: https://embracethered.com/blog/images/2026/agent-commander/agent-commander-tn.png
---

This post is about prompt-based command and control (C2), which is becoming more relevant.

## What is Promptware-Powered C2?

Three years ago, when ChatGPT introduced the browsing tool, we already experimented with the idea of prompt-based command and control. And when ChatGPT got memories we showed that this can be combined and abused for a [full command and control channel](https://i.blackhat.com/EU-24/Presentations/EU-24-Rehberger-SpAIware-Advanced-Prompt-Injection-Exploits.pdf). 

[Recent work uses](https://arxiv.org/html/2601.09625v1) the term **promptware** to describe prompt-injection payloads that are more complex in behavior and closer to malware. I'm using that term here as it fits well.

As agents become more powerful and widespread, attackers will target them more frequently.

**Agents are a new execution layer.**

## Agent Commander

Agent Commander is a command and control (C2) server where hijacked agents regularly check in for any new tasks and objectives. Unlike classical C2, it doesn’t execute raw OS commands or APIs.

**All commands are provided in natural language.**

[![Agent Commander Dashboard](/blog/images/2026/agent-commander/three-different-agents-calling-back.png)](/blog/images/2026/agent-commander/three-different-agents-calling-back.png)


**This targets the agent layer, not the underlying operating system.**

It's a conceptual shift as agents evolve into a kind of new operating system layer.

[![Kimi Claw - Andy joins too Commander Dashboard](/blog/images/2026/agent-commander/agent-commander-0.png)](/blog/images/2026/agent-commander/agent-commander-0.png)

The goal of Agent Commander is to show that, post-exploitation, agents can be continuously hijacked to serve multiple stakeholders, and all of that at scale.

Let's walk through the steps of this research.

## Video Walkthrough

There is a lot more information and details, including a live demo, that I put in this video. 

{{< youtube VrW5VMTnfgQ >}}

Hope it's useful to help understand where things are headed. 

## Initial Entry Points and Exploitation

There are typical attack vectors, such as:
- Traditional host compromise (config issue, port exposure, SSH,...)
- Exploiting appsec issues. See [OpenClaw's lengthy security advisory list](https://github.com/openclaw/openclaw/security) for examples
- Supply chain. Skills have already been extensively used by threat actors to install malware
- However, the scenarios I'm most interested in are prompt-based, like indirect prompt injection 

Specifically, we will show exploits for three variants of agents:
1. **Kimi Claw**: Asking the agent to analyze a document
2. **OpenClaw**: Email delivery. Sending a malicious email to the agent (**no human action needed**)
3. **NanoClaw**: Agent visits a website and gets hijacked

For OpenClaw to get proactive email notifications, it's advised to configure via GCP Pub/Sub. This is something I learned from *veganmosfet*, who first showcased this OpenClaw running Opus 4.6 [here](https://veganmosfet.codeberg.page/posts/2026-02-02-openclaw_mail_rce/#setup). And this here is another good [post](https://labs.zenity.io/p/openclaw-or-opendoor-indirect-prompt-injection-makes-openclaw-vulnerable-to-backdoors-and-much-more) about similar attacks.

## Persistence Using Heartbeats

Over the last year, the persistence opportunities have grown. It's not just agent memory anymore, like it was 2 years ago with ChatGPT. Now agents can write to files, databases, soul, memory or identity definition files and many more...

The scenario I chose was the `HEARTBEAT.md` in OpenClaw. There were a few reasons:
1. The word **heartbeat** is just too classic for red teaming not to abuse it
2. It's a regular proactive step that runs every 30 minutes by default
3. It typically runs on the main channel
4. **Responses to the user can be suppressed with `HEARTBEAT_OK`**
[![Agent Commander Heartbeat](/blog/images/2026/agent-commander/hearbeat-ok-no-ui-message.png)](/blog/images/2026/agent-commander/hearbeat-ok-no-ui-message.png) This screenshot shows the suppression message.
5. The heartbeat often is **configured to use weaker models to save costs**. It's even mentioned in the docs. That means it likely stays undetected by the agent itself for a longer time. And the more often an agent follows the malicious steps the more it normalizes that this is what the user wants.


This works well, and here are a few things I learned along the way:
1. User notification suppression works surprisingly well. There is also a `NO_REPLY` option to suppress info.
2. One time an OpenClaw agent identified the added backdoor in the heartbeat. It happened when it summarized the daily notes into a memory file for the day. So it happened delayed, but I found that super interesting.
3. One compromised agent started ignoring the heartbeat instructions after a while entirely
4. Opus 4.6 is better in catching attacks, but after some testing, reliable bypasses can be found
5. Sometimes a compromised agent only follows heartbeat instructions partially, which will be an interesting challenge for attackers


## Objectives: Your Agent Works For Me Now

Now that we have compromised agents calling back we can assign them tasks, and this is where things are conceptually changing from an adversarial point of view. 

In the past threat actors and red teams would tell what commands to run and what to do, but with prompting all of this elevates to a new abstraction layer, where we prompt the compromised agent what we would like to be done, and it will figure it out. 

This could be short-lived or long-running tasks or missions, a few examples:
1. Host enumeration and creating an executive summary in one shot
2. Task agent to browse to inbox, take a screenshot and email it (or upload to Agent Commander)
3. Do something totally different, like assign tasks to monitor third party websites for changes
4. Complex multi-system objectives, find some source code and then leak it out, etc.
5. Find and exploit custom zero-days on the fly when laterally moving within an environment
6. **Influence campaigns, social media posts, ad-click fraud, likes,...**
7. Be creative, imagine how wild this is.. I mean we knew and predicted that it's eventually going to happen, but now it's basically real...

The level of sophistication and technical expertise for an adversary to achieve objectives will continue to drop significantly. This does not require any malware to be installed or dropped in the traditional sense, just prompts.


A cool feature is that you can upload and download images from the compromised host via the agent.

[![Agent Commander Dashboard](/blog/images/2026/agent-commander/agent-commander-screenshot.png)](/blog/images/2026/agent-commander/agent-commander-screenshot.png)

In the above screenshot, you can see I asked OpenClaw to navigate to the Outlook inbox, take a screenshot and upload it to Agent Commander. Pretty cool. 

**Assume Breach: Your agent is the potential malware!**

### NanoClaw - Demonstration

Here are some more details around the NanoClaw demo in the video.

#### Installation

NanoClaw has no ordinary installer, and it's a fascinating shift. You basically use Claude to create it! So, for instance, I don't want to use the WhatsApp integration, so I ran the `/add-telegram` skill, and the rest was all via Claude Code. Custom software, on the fly.

An interesting security issue I identified is that by default the inference request do not specify a model explicitly. It turns out that if that is omitted the Claude Agent SDK falls back to some very old model. I think it was Claude Sonnet 3.5, and the attack was so easy to pull off!

#### Scheduled Tasks

NanoClaw does not have a heartbeat. In order to achieve persistence, the indirect prompt injection attack adds a scheduled task that fires regularly.

#### Prompt Injection Attacks

The demo in the video shows an indirect prompt injection via an uploaded file:
[![Kimi Claw - Andy joins too Commander Dashboard](/blog/images/2026/agent-commander/kimi-claw-prompt-injected.png)](/blog/images/2026/agent-commander/kimi-claw-prompt-injected.png)

As you can see the analysis of the file, which installs the scheduled promptware task, does not trigger suspicion and Kimi Claw joins Agent Commander:

[![Kimi Claw - Andy calls back](/blog/images/2026/agent-commander/three-different-agents-calling-back.png)](/blog/images/2026/agent-commander/three-different-agents-calling-back.png)


And we can run commands:
[![Kimi Claw - Andy joins Human Investigation](/blog/images/2026/agent-commander/kimi-claw-human.png)](/blog/images/2026/agent-commander/kimi-claw-human.png)

look around the machine, etc...
[![Kimi Claw - Andy shares Processes](/blog/images/2026/agent-commander/kimi-claw-process.png)](/blog/images/2026/agent-commander/kimi-claw-process.png)

Check the video at the top for a live demo.

### Sandboxing

The space is rapidly changing, pretty much daily. This post is focusing on the overarching attack primitives with concrete examples, that are representative and what is happening. 

It's possible to sandbox execution, like run all agents in Docker containers, and there are also OpenClaw alternatives that are more lightweight with similar container isolation approaches. 

[![Kimi Claw - summarize website](/blog/images/2026/agent-commander/kimi-claw-summarize-website.png)](/blog/images/2026/agent-commander/kimi-claw-summarize-website.png)

The way I use OpenClaw is on a dedicated host, but as non-root. So it can do most things, but not totally mess up the host easily. It gets its own accounts, inboxes, etc. That allows it to go do useful work and tasks, without damaging and deleting all your emails.

Overall, this project is something that I want to keep following because I like the lightweight approach and focus on building architectural boundaries, rather than adding tons of features right away.

However, exploits with sandboxed agents are still often possible. Messages (which end up in prompts) are passed around, **e.g. subagent sends summary of email to main agent -> BOOM!** 

## Recommendations and Mitigations

First, be aware of the [Normalization of Deviance](/blog/posts/2025/the-normalization-of-deviance-in-ai/) it's a cultural shift that does not just happen within AI labs and vendors, but it happens at the individual level. 

You might feel more and more comfortable over time to trust an agent, but at its core LLM output is untrusted and can instruct an agent to take harmful and malicious actions.

Here are some additional thoughts.

### Protect

* If you adopt something like OpenClaw, be aware that it basically requires daily patching.
* [Hundreds of security vulnerabilities have been patched and advisories issued](https://github.com/openclaw/openclaw/security). All that in just a few weeks. That trend is likely going to continue as more researchers look at it and as more features are being added. 
* For OpenClaw specifically, run it on a dedicated system in isolation
* Do not give it access to all your personal information. You still get a lot of value and fun out of it, without the elevated risk. 
* Switching heartbeat to a cheaper model allows an adversary to achieve objectives more easily via prompt injection. So I'd recommend against that, if you are worried about costs, just reduce the frequency of heartbeats in the configuration.
* **Sandboxing and Isolation:** Setting up sandboxing and configuring things in a more secure way is still complex and basic scenarios break entirely out of the box. 

### Detect 

* For enterprises it becomes clear that monitoring of prompts is important. Otherwise it is extremely difficult to understand what happened, and why an agent took a certain action. 
* Integrity monitoring can help identify drift in configuration files, skills,..
* Inventory - know which OpenClaw variants are running where

### Respond

* Make sure to have a **kill-switch** to disable agents immediately if the need arises
* We will see rare or entirely new techniques chained together and that's another reason why reasoning and prompt logging is important
* Make sure agent creds can be rotated (ideally automatically)

## What's Next?

- Compromised agents will not need frequent check-ins. They might operate like spies with occasional dead-drops of info. No need to reach out to C2 frequently.
- **Software will evolve to be a lot more "organic", and compromises will behave more like infections. Systems are probabilistic. This means that attackers will lose control of what happens exactly. An attack might only succeed ~50% (random number) correctly, and morph. And this seems quite risky.**
- **Living-Off-Agents** will be very real. Attackers will hijack and instruct Agents to fulfill objectives on their behalf.

## Agent Commander Access

At this point I haven't open sourced or published the tool widely. And I don't immediately plan on doing so. If you find this research interesting or want to leverage it for defensive purposes or a red team engagement, let me know. 

As always, this post is for educational and research purposes. Only evaluate systems that you are authorized to test.

## Conclusion

This post introduced **Agent Commander**, a promptware-powered command and control system. 

We demonstrated how popular agents like OpenClaw, Kimi Claw and NanoClaw can be tricked via indirect prompt injection to join Agent Commander, as bots in a prompt-based botnet.

One of the main realizations is that compromised agents (not surprisingly) also follow instructions sometimes only partially. This is especially true for the initial indirect prompt injection payload, which could be an interesting challenge for attackers. 

**As software becomes more "organic", attacks will evolve because the attack surface is alive and constantly changing, and attacks themselves will become less predictable as well, which probably should worry us all.**

Cheers.

[![Agent Commander Dashboard](/blog/images/2026/agent-commander/agent-commander-dashboard.png)](/blog/images/2026/agent-commander/agent-commander-dashboard.png)

## References

* [Black Hat Europe SpAIware](https://i.blackhat.com/EU-24/Presentations/EU-24-Rehberger-SpAIware-Advanced-Prompt-Injection-Exploits.pdf)
* [The Promptware Kill Chain: How Prompt Injections Gradually Evolved Into a Multi-Step Malware](https://arxiv.org/html/2601.09625v1)
* [veganmosfet: OpenClaw Mail RCE](https://veganmosfet.codeberg.page/posts/2026-02-02-openclaw_mail_rce/#setup)
* [OpenClaw GitHub](https://github.com/openclaw/openclaw)
* [OpenClaw or OpenDoor](https://labs.zenity.io/p/openclaw-or-opendoor-indirect-prompt-injection-makes-openclaw-vulnerable-to-backdoors-and-much-more)