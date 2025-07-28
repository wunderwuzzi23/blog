---
title: "The Month of AI Bugs 2025"
date: 2025-07-28T10:20:58-07:00
draft: true
tags: ["llm", "agents", "month of ai bugs"]
twitter:
  card: summary_large_image
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "Month of AI Bugs 2025 - Announcements"
  description: "August 2025 will be the month of Agentic ProbLLMs and AI Bugs. Fresh posts nearly every day."
  image: "https://embracethered.com/blog/images/2025/moaib-tn.png"
---

This year I spent a lot of time reviewing, exploiting and working with vendors to fix vulnerabilities in agentic AI systems.

As a result, I'm excited to announce the **[Month of AI Bugs 2025!](https://monthofaibugs.com)**

[![Month Of AI Bugs Logo](/blog/images/2025/moaib-tn.png)](/blog/images/2025/moaib-tn.png)


## Goal Of The Initiative

The main purpose of the Month of AI Bugs is to raise awareness about novel security vulnerabilities in agentic systems, primarily focusing on AI coding agents. Posts will cover both simple and advanced, sometimes even mind-boggling exploits. 

There will be some AI security vulnerabilities and attack chains that have probably not been demonstrated elsewhere before. Exploits will include some scary ones, that allow an adversary via prompt injection (or the model for FWIW) to compromise the developer's host without the developer's consent.

## Responsible Disclosure

Every AI coding agent that was reviewed had vulnerabilities that were responsibly disclosed to the corresponding vendor. 

**Many vendors fixed vulnerabilities quickly - some even within days. Others ignored reports for months and some issues remain unmitigated as of this writing, and a few vendors entirely stopped responding to inquiries.**

Some vendors appear unsure how to address novel AI threats, such as prompt injection, untrustworthy LLM output or automatic tool invocation. That is alarming, because at the same time they continue to add more risky capabilities, rather than mitigating existing vulnerabilities.

## Cadence and Scope

Throughout the month of August I will be sharing more than 20 blog posts of vulnerabilities in popular agentic systems. The focus is primarily on coding agents, but there will also be a few other interesting posts.

**We will uncover security vulnerabilities in systems such as ChatGPT, ChatGPT Codex, Anthropic Claude Code, Google's Jules, Amazon Q Developer, GitHub Copilot Agent Mode, AmpCode, Manus, OpenHands, Devin, Windsurf, Cursor and more!**

Posts will be published on this blog, and I also created a neat website at [Month of AI Bugs](https://monthofaibugs.com) featuring all posts in an easy-to-consume format.

The philosophy of this blog is **"learn the hacks, stop the attacks"**, and when it comes to novel AI security challenges, like prompt injection, a lot more awareness, knowledge-sharing and transparency around exploit techniques, risks, possible mitigations and best practices is needed.

## Raising the AI Application Security Bar

If you are interested in AI Application Security, I suggest reading most of the posts - even if a vulnerability might seem like something you heard of before.

Each post should bring something new to the table. Some posts will demonstrate how vulnerabilities can be chained together to achieve objectives.

I'll try to publish posts every weekday, with occasional weekend posts. 

I spent many weekends and evenings over the past months working on this initiative with the hope that it will raise the bar on learning about AI Application Security and enable developers, testers and users to spot common mistakes quickly and have effective mitigation strategies and discussions before deployment.

Also, it hopefully will help provide insights to quickly cut through AI marketing claims that promise systems which are not yet possible to build securely.

We need a lot more transparency in the AI security space, and we need a lot more testers evaluating and finding issues in AI systems, and users asking vendors basic questions, such as "can this even be secure?"

## Conferences in August

Furthermore, I will be speaking at a couple of conferences in August where I will discuss exploit techniques and mitigations:
1. [ACL - LLMSEC 2025 (Keynote): Trust No AI - Prompt Injection Along the CIA Security Triad](https://sig.llmsecurity.net/workshop/#trust-no-ai---prompt-injection-along-the-cia-security-triad)
2. [HITCON 2025: Agentic ProbLLMs: Exploiting Computer-Use and Coding Agents](https://hitcon.org/2025/en-US/agenda/b159e10f-0f1f-45fd-86f3-efe65c912c0c/)
3. [Out Of The Box: Agentic ProbLLMs: Exploiting Computer-Use and Coding Agents](https://ootb.net/talks/agentic-probllms)

If you have the opportunity to catch both HITCON and OOTB talks, I think you can attend both. Although they are similar, exploits that will be demonstrated (including full remote code execution via prompt injection in highly popular IDEs) will actually cover some different techniques and IDEs in each presentation.

## Some Additional Background

Interestingly, I first had this idea to publish a series of "Month of AI Bugs" posts a bit over two years ago when ChatGPT introduced plugins. Back then we already saw an entire ecosystem of vulnerable plugins and integrations being created, seemingly over night. 

The situation today is similar, but risks are elevated. With prompt injection, untrustworthy model output, and automatic tool invocation the primary defense often lies with "human approval" for consequential actions.

MCP and its ecosystem took the previously narrow plugin approach and turned it into an entire dynamic ecosystem that promotes insecure system design.

## Contribute Ethically - Your Work Matters!

As always, all the content provided on this blog is for educational purposes to help raise the bar and improve the security posture of systems. If you encounter a security vulnerability, then report it responsibly to the corresponding vendor. 

Many companies offer bug bounty programs to enable and reward impactful, ethical research.

## Happy Hacking!