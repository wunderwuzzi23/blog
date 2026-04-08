---
title: "Given Enough Agents, All Bugs Become Shallow"  
date: 2026-04-07T23:58:58-07:00  
draft: true  
tags: ["llm", "red", "blue"]
twitter:  
  card: "summary_large_image"  
  site: "@wunderwuzzi23"  
  creator: "@wunderwuzzi23"  
  title: "Given Enough Agents, All Bugs Become Shallow" 
  description: "Thoughts on Situational Awareness, Anthropic Mythos and the challenges and opportunities ahead"
  image: "https://embracethered.com/blog/images/2026/agents-and-bugs.png"  
---

A few months ago I had this realization that agents have become really good at identifying bugs in code, especially security vulnerabilities. They are relentless in analyzing code and you can spin up multiple of them to go through source code quickly.

{{< x user="wunderwuzzi23" id="2021046801630101595" >}}

It is an emerging capability that many security researchers and bug bounty hunters have observed over the last few months. 

Gadi Evron [posted](https://www.linkedin.com/posts/gadievron_the-ai-vulnerability-cataclysm-is-coming-activity-7366486915878924288-iPIZ) about the upcoming **AI Vulnerability Cataclysm** last year to help raise awareness.

## Increases in Security Capabilities
 
Today Anthropic announced `Mythos - Preview`, and according to their announcement the model is very capable in performing cyber security research and attacks. It's so powerful that access to the model is gated to a handful of companies.

The information shared by Anthropic highlights a significant increase in capabilities, most notable agents powered by the model can identify and exploit previously unknown vulnerabilities. 

According to Anthropic's post Opus 4.6 succeeded at exploiting Firefox JS engine vulns only twice out of hundreds of attempts, but `Mythos Preview` succeeded 181 times.

**That is a significant improvement.**

Even though a footnote in Anthropic's post highlights that this using a dedicated testing harness without the Firefox sandbox or other defense-in-depth mitigations, it's a significant progression.

![Agents And Bugs](/blog/images/2026/agents-and-bugs.png)

Anthropic will not release Mythos publicly (for now), it's available as a gated preview only.

## Finding Security Bugs

Specifically, the [red team post](https://red.anthropic.com/2026/mythos-preview/) highlights details of some of the identified vulnerabilities. Like a 27-year-old OpenBSD SACK bug, a 16-year-old FFmpeg H.264 bug, and the 17-year-old FreeBSD NFS RCE. So, there are some quite severe and interesting findings.

It will be interesting to see what kind of vulnerabilities it finds in closed source software, and hopefully Microsoft and others share outcomes of their efforts as well.

## Building Exploits for Newly Patched Vulnerabilities

When companies release security patches for their products, these patches get immediately reverse engineered by security researchers to identify the root cause of the patched issue, with the goal to build an exploit.

We have seen researchers use LLMs for that task, and Anthropic highlights that they investigated this capability also. And it's scary, because according to their data points Mythos often succeeds.

Current LLMs can autonomously write exploits for published patches (CVEs), which allows threat actors who would have access to such capabilities to have working exploits quickly.

**The window between disclosure and exploitation was narrowing already, but it might soon collapse.** 

And some companies hide patches in larger updates. So the next logical step will be to review and diff all software updates, not just CVEs and patches. LLMs make such tasks feasible.

## Democratizing Offensive Security

An important part in Anthropic's post is the fact that it allows non-security people to find and exploit systems. 

> Engineers at Anthropic with no formal security training have asked Mythos Preview to find remote code execution vulnerabilities overnight, and woken up the following morning to a complete, working exploit.

I'm wondering if the bar for performing successful cyberattacks will drop and become more objective driven "Go ransom company X", rather than having to find exploits or buy them.

It's plausible that adversaries have found and exploited some of the newly discovered vulnerabilities already in the past. So, these new LLM powered capabilities are a very good thing for securing systems in the long run.

## Deploying Patches Remains the Big Challenge

**The biggest challenge for organizations is typically to get vendor fixes deployed.**

Even when patches are available, it does not mean that the problem is gone. 

Organizations often take a very long time to apply patches, and at times no patch management is present at all. This is where the industry needs to adjust, and new innovations could help.

## Project Glasswing

To try to raise awareness and tackle some of these challenges Anthropic announced **Project Glasswing**.

> Project Glasswing is an initiative to secure the world’s most critical software for the AI era

It's an effort that multiple large companies, such as AWS, Cisco, Microsoft, Linux Foundation, CrowdStrike, Palo Alto Networks, NVIDIA,.. joined. 

Anthropic also donates up to $100M credits + $4M to Open Source Software. 

This means a lot of good things are happening already to help identify and patch the "low-hanging" LLM fruits (difficult for humans, but easy for LLMs). As the testing harnesses, approaches and capabilities improve, and reruns happen more vulnerabilities will be discovered. 

Remember LLMs are non-deterministic. This means running an analysis a few times will not yield all the same bugs a given LLM can find. 

## Isolation Concerns For Testing

As systems become more capable, Anthropic realized that current security benchmarks are maxed out, and hence capabilities are evaluated against real-world targets in isolated environments.

This raises an obvious concern, especially if further step-function improvements occur. How will AI labs, researchers and the industry at large properly isolate such systems, when they can potentially identify flaws in that isolation boundary, and at the same time the systems are explicitly tasked to identify issues and build exploits.

Anthropic [does mention](https://www-cdn.anthropic.com/8b8380204f74670be75e81c820ca8dda846ab289.pdf) that they "arranged a 24-hour period of internal alignment review" before proceeding with wide range internal usage on February, 24.

A fun anecdote from my side. In my testing harness, Opus 4.6. figured out a way to `sudo`, and install `gdb` and other tools which it seemed would be useful for debugging a buffer overflow that it found while testing the target.

As a side note, in the model card it also shows that Mythos shows improved resistance to indirect prompt injection, which matters as these agents interact with untrusted inputs.

## The Rise of Open Weight Models 

Open weight (sometimes generically called open source) models are usually not that far behind the frontier labs. So it's likely that 6-12 months from now comparable capabilities will be available to anyone in the world.

The next 12-18 months might be rough, as offensive capabilities arrive before patches.

This was all predicted by the Situational Awareness [paper](https://situational-awareness.ai/), from former OpenAI employee Leopold Aschenbrenner. He said there is a scary time in between where attacker will have the upside, and it seems we are entering this period.

## Conclusion

With all of this actually comes a great opportunity, which is that many of the core projects that run the Internet and computing systems are going through a security push at the moment.

**This is extremely good.**

Overall, by what Anthropic shared, it's clear that things are accelerating, and that the industry has to come together. It's a great opportunity to improve the security posture quickly. 

There are big challenges around deployment of patches and that these capabilities are also used by threat actors. The one thing organizations really have to figure out is quicker roll-out of patches. 

**A monthly patch Tuesday might soon not cut it anymore if you want to stay protected.** 

I'm quite curious how large vendors of client-side enterprise software will react to this new reality that is approaching fast.

In case you were not familiar with the title pun of this post, it's about [Linus's law](https://en.wikipedia.org/wiki/Linus%27s_law) that states "Given enough eyeballs, all bugs become shallow."

Let's hope that the industry figures out how to deploy patches quickly without breaking too much.

## References

* [Claude Mythos - System Card](https://www-cdn.anthropic.com/8b8380204f74670be75e81c820ca8dda846ab289.pdf)
* [Glasswing](https://www.anthropic.com/project/glasswing)
* [Mythos Preview](https://red.anthropic.com/2026/mythos-preview/)
* [Linus' law](https://en.wikipedia.org/wiki/Linus%27s_law)
* [Situational Awareness](https://situational-awareness.ai/)
