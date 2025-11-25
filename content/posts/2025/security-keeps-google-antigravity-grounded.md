---
title: "Antigravity Grounded! Security Vulnerabilities in Google's Latest IDE"  
date: 2025-11-25T06:00:58-07:00  
draft: true  
tags: ["llm", "data exfiltration","rce"]
twitter:  
  card: "summary_large_image"  
  site: "@wunderwuzzi23"  
  creator: "@wunderwuzzi23"  
  title: "Antigravity Grounded! Security Vulnerabilities in Google's Latest IDE"  
  description: "Security Vulnerabilities Keep Google's Antigravity Grounded"
  image: "https://embracethered.com/blog/images/2025/antigravity-grounded2.png"  
---

Last week Google released an IDE called Antigravity. It's basically the outcome of the Windsurf licensing deal from a few months ago, where [Google paid some $2.4 billion for a non-exclusive license to the code](https://www.reuters.com/business/google-hires-windsurf-ceo-researchers-advance-ai-ambitions-2025-07-11/).

Because it's based on Windsurf, I was curious if vulnerabilities that I reported to Windsurf back in May 2025, long before the deal, would have been addressed in the Antigravity IDE. See [Month of AI Bugs](https://embracethered.com/blog/posts/2025/wrapping-up-month-of-ai-bugs/) for some detailed write-ups.

**The short answer is no.**

[![system prompt](/blog/images/2025/antigravity-grounded2.png)](/blog/images/2025/antigravity-grounded2.png)

In this post we will walk through five security vulnerabilities that I reported to Google last week, including data exfiltration vulnerabilities, and even remote code execution via indirect prompt injection. As an outsider, it's unclear why these known vulnerabilities are in the product, but after researchers started reporting issues last Tuesday, Google started documenting them publicly [here](https://bughunters.google.com/learn/invalid-reports/google-products/4655949258227712/antigravity-known-issues) also. My personal guess is that the Google security team was caught a bit off guard by Antigravity shipping...

Although these vulnerabilities are straightforward to exploit, I will not include the exploit payloads verbatim at this point. The main goal is to raise awareness, and provide a practical mitigations steps as well.

## Overview

As this is a bit of a lengthy post, I'm including a quick index table.

* [Antigravity System Prompt](#antigravity-system-prompt)
* [Issue #1: Remote Command Execution via Indirect Prompt Injection (Auto-Execute Bypasses)](#issue-1-remote-command-execution)
* [Issue #2: Antigravity Follows Hidden Instructions](#issue-2-antigravity-follows-hidden-instructions)
* [Issue #3: Lack of Human in the Loop for MCP Tool Invocations](#issue-3-lack-of-human-in-the-loop-for-mcp)
* [Issue #4: Data Exfiltration via read_url_content tool](#issue-4-data-exfiltration-via-read_url_content)
* [Issue #5: Data Exfiltration via image rendering](#issue-5-data-exfiltration-via-image-rendering)
* [Recommendations and Mitigations](#recommendations-and-mitigations)

For all reports I created fresh, reliable exploit payloads and demo videos. 

If you prefer to watch a video with details and demos:
{{< youtube 3gHx1gvDnZU >}}

There are also five additional issues, which I have not previously discussed. I'll share details on those as fixes arrive, issues are won't fixed, or as responsible disclosure deadlines pass.

Let's take a look.


## Antigravity System Prompt

For reference, you can find the [system instructions](https://github.com/wunderwuzzi23/scratch/blob/master/system_prompts/antigravity-ide-2025-11-24.txt) from my session here.

[![system prompt](/blog/images/2025/antigravity-system-prompt-small.png)](/blog/images/2025/antigravity-system-prompt.png)

Note: I had two MCP servers attached, which you can see on the bottom of the prompt, and hence the tools sections contains all the tools from those MCP servers.

I also want to give a shout out to [p1njc70r](https://x.com/p1njc70r/status/1990919996265148701?s=20) who was the first to share the system prompt publicly to my knowledge.

Let's get started.

## Issue #1: Remote Command Execution

The Antigravity IDE by default is set to execute Terminal commands via the `run_command` tool by the discretion of the AI. The default setting is for the AI to decide if a command is "safe" to execute or not, without human in the loop. That's like rolling a dice in a way. 

But that also means we have to run some actually malicious commands as proof. Because even if exploited via indirect prompt inject, if an attack runs `calc.exe`, that will be fine, because it's not doing something malicious.

However, as soon as an attacker tries to load a remote script (e.g. by using `curl`) many models will refuse the request, including Claude and Gemini 3. But as you know if you follow my work, **such refusals are coming from the model, and hence are suggestions, and not a security boundary.**

It required a few tricks to bypass the model's behavior and have it run arbitrary remote code that is downloaded via curl then piped to bash, but I have working exploits for both Gemini 3 and Claude Sonnet 4.5 within Antigravity now:

[![antigravity rce with invisible instructions](/blog/images/2025/antigravity-arbitrary-command-execution-redacted.png)](/blog/images/2025/antigravity-arbitrary-command-execution-redacted.png)

In the above screenshot you can see a source code file that contains instructions that hijack Gemini 3 to download a remote script and run it via bash (which then launches a calculator). This shows that we achieve arbitrary code execution via the remote script.

**This demo shows how Antigravity over-relies on the output of the LLM, in an attempt to enforce security.**

*In case you are wondering why I call this RCE, it's because it's indirect prompt injection and issue #3 will highlight this even better*

But it already better, with the second issue... hidden instructions via Unicode Tag characters.

## Issue #2: Antigravity Follows Hidden Instructions

Gemini models are very good at interpreting invisible instructions, and Gemini 3 is exceptional at it.

**This now also impacts the new Antigravity IDE:** 

An attacker can hide instructions in code or other data source (invisible to users in the UI), and when Antigravity sends it to Gemini it will follow the hidden instructions.

**This will increase the likelihood of successful attacks hiding in plain sight.**

### Demo: Arbitrary Command Execution with Hidden Instructions 

I created a demonstration file that contains invisible instructions to print a certain text and invoke the `run_command` to download malware and run it. For creating/decoding hidden Unicode Tag instructions I use [ASCII Smuggler](https://embracethered.com/blog/ascii-smuggler.html).

Here is the result when the file enters the chat context:

[![antigravity rce with invisible instructions](/blog/images/2025/antigravity-zombai-invisible-instructions-e2e-demo-screenshot.png)](/blog/images/2025/antigravity-zombai-invisible-instructions-e2e-demo-screenshot.png)

**This is scary. Code reviews will not catch this!**

This weakness, which I reported first back in the Bard days has not been addressed at the model or API level. Hence, all applications built on top of Gemini models inherit this behavior. As we have [shown with Google Jules in the past](/blog/posts/2025/google-jules-invisible-prompt-injection/) this applies to all Google products that use Gemini.

The implications are getting worse. As predicted two years back, the models are getting better, and in the case with Gemini 3 Fast it seems to often bypass guardrails as well.

But, wait... there is more!

## Issue #3: Lack of Human in the Loop for MCP 

When invoking tools of an MCP server, the Antigravity IDE is missing an important basic security control: It does not have a human in the loop feature, at all. 

This means an indirect prompt injection attack or hallucinations can invoke any MCP tool once it's been added.

**And here is the kicker!** 
An attacker can use invisible Unicode Tag characters as instructions here too, either hiding them in source code or also coming in via MCP tools calls. 

Here is an example that shows hidden instructions being inside a Linear ticket.

[![MCP Invisible Instructions](/blog/images/2025/antigravity-mcp-exploit-invisible-instructions-rce.png)](/blog/images/2025/antigravity-mcp-exploit-invisible-instructions-rce.png)

When the developer brings the ticket into the chat context via an MCP tool call, the remote instructions are passed through to the LLM which leads again to full compromise of the developer's workstation (classic RCE).

Again, this increases the likelihood of attacks staying unnoticed by developers.

Depending on the capabilities of the tool, this allows the AI to exfiltrate data, code execution, data manipulation or deletion, etc., all without the developer's consent. 

An interesting feature Microsoft recently added to GitHub Copilot is that the result of an MCP tool is actually displayed to the developer and the developer can decide to include the data or not in the prompt context.

### Current Mitigating Factors For MCP and Recommendations

It is possible to disable individual tools, which is good. But there is no secure way to enable dangerous tools at the moment.

A possible trade off could be to have `readOnly` tools auto-approve, but require HITL for all tools with an annotation of `destructiveHint` or `openWorldHint`. But even `readOnly` tools can have side-effects, like data leakage by the way, so automatic tool invocation is often exploitable. 

Hence, best is to allow customers and enterprises to configure settings according to their own risk appetite.

## Issue #4: Data Exfiltration via `read_url_content`

The Antigravity IDE is vulnerable to multiple data exfiltration issues. The ones I'm describing are again vulnerabilities that were inherited from Windsurf and are known since at least May 2025.

The issue here is the `read_url_content` tool, which can be invoked without human in the loop during an indirect prompt injection attack.

[![antigravity read_url_content](/blog/images/2025/antigravity-exploit-read_url_content-redacted.png)](/blog/images/2025/antigravity-exploit-read_url_content-redacted.png)

The exploit demo calls the `read_file` tool first read the `.env` file, then sends contents of the file to the attacker using the `read_url_content` tool.

Also, please note that the attack payload does not have to be coming from the source code file. I see people often misunderstanding the demos. It can be part of a response from a tool call as shown with the Linear ticket before, or the model can decide to do it (e.g. backdoored).

## Issue #5: Data Exfiltration via Image Rendering

Similarly, the AI can also leak data by rendering html images using markdown syntax.
I was able to repro a data exfiltration example from Windsurf quickly by planting a prompt injection demo exploit into a `.c` file, and then have Antigravity explain that file.

The result is that it invokes the `read_file` tool to read the developers `.env` file and leaks the secrets to the third-party server via an http request that loads the image.

[![antigravity data exfil via images](/blog/images/2025/antigravity-render2-redacted.png)](/blog/images/2025/antigravity-render2-redacted.png)

I'm not the only one pointing this out. [p1njc70r](https://x.com/p1njc70r/) has also reported this and got a similar response from Google.

## Video Walkthrough 

Here is a video that walks through all the scenarios and demos:

{{< youtube 3gHx1gvDnZU >}}

Hope it's educational and useful.

## Recommendations and Mitigations

There are a couple recommendations that can help mitigate the situation.

* Be careful when enabling MCP servers and disable dangerous tools.
* For the Antigravity team adding Human in the Loop controls by default seems like the best solution for MCP servers, and requiring it for destructive or consequential tools
* Building CI/CD tooling to uncover hidden Unicode Tags programmatically could be helpful. Even reviewing code manually for malicious instructions carefully does not help to mitigate prompt injection attacks that leverage hidden Unicode Tags
* Developers should consider alternative IDEs until these issues are addressed
* Disable "Auto-Execute" (the default) and use manual approvals, and carefully enable only commands that you would entrust the AI via "Allow List Terminal Commands"
* If your organization uses this IDE widely, it might be a good idea to run a Red or Purple Team exercise to identify detection and monitoring opportunities
* Be ready to hit the **Stop** button! lol
* High thinking mode and planning are more resilient to adversarial misalignment (e.g. prompt injection) - but it's not a silver bullet!

Let's see how Antigravity will evolve over the next few weeks. Technically, these are relatively simple bug fixes to have an out-of-box secure experience and it will be interesting to see if these are a priority or not. 

## Conclusion

In this post we revisited common vulnerabilities that we discussed in Windsurf and the Month of AI bugs in August before. Unfortunately, Google's latest Antigravity IDE is vulnerable to all the same issues as Windsurf, first disclosed to the team back then in May 2025.

The issues described in this post are not the only ones I'm aware of. There are 5 more security issues reported, and then I stopped looking for now to see how things are triaged and being handled.

Overall, coding agents are a game changer, but we also have to take security seriously. There are currently other, more mature, coding agents out there with better security and patch history, including Claude Code, GitHub Copilot, Cursor, Codex, Google's own Gemini CLI,... and some also support Gemini 3 already. 

Keep an eye out for Antigravity CVEs.

Stay safe.

## References

* [Google paid $2.4 billion non-exclusive license](https://www.reuters.com/business/google-hires-windsurf-ceo-researchers-advance-ai-ambitions-2025-07-11/)
* [p1njc70r on X](https://x.com/p1njc70r/status/1990919996265148701?s=20)
* [ASCII Smuggler](https://embracethered.com/blog/ascii-smuggler.html)
* [Month of AI Bugs August 2025](https://monthofaibugs.com)
* [Antigravity Known Issues](https://bughunters.google.com/learn/invalid-reports/google-products/4655949258227712/antigravity-known-issues)
* [Google Jules is vulnerable to invisible prompt injection](https://embracethered.com/blog/posts/2025/google-jules-invisible-prompt-injection/)


## Appendix

Google documented [known-issues](https://bughunters.google.com/learn/invalid-reports/google-products/4655949258227712/antigravity-known-issues) and statement that team is working on fixes:
![known issues](/blog/images/2025/known-issues.png)