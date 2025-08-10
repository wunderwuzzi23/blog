---
title: "OpenHands ZombAI Exploit: Prompt Injection To Remote Code Execution"  
date: 2025-08-10T04:20:58-07:00  
draft: true  
tags: ["llm", "agents", "month of ai bugs"]
twitter:  
  card: "summary_large_image"  
  site: "@wunderwuzzi23"  
  creator: "@wunderwuzzi23"  
  title: "OpenHands ZombAI Exploit: Prompt Injection To Remote Code Execution"  
  description: "When processing untrusted data OpenHands can be hijacked to run remote code (RCE) and connect to an attacker's command and control system"  
  image: "https://embracethered.com/blog/images/2025/episode10-yt.png"  
---

{{< raw_html >}}
<a id="top_ref"></a>
{{< /raw_html >}}

Today we have another post about [OpenHands](https://github.com/All-Hands-AI/OpenHands) from All Hands AI. It is a popular agent, initially named "OpenDevin", and recently the company also provides a [cloud-based service](https://www.all-hands.dev/). Which is all pretty cool and exciting.

## Prompt Injection to Full System Compromise

However, as you know, LLM powered apps and agents are vulnerable to prompt injection.  That also applies to OpenHands and it can be hijacked by untrusted data, e.g. from a website. That impacts `Confidentiality`, `Integrity`, and `Availability` of the system.

## Turning OpenHands into a ZombAI

The working exploit was shared with the All-Hands team, and turns an OpenHands instance into a ZombAI. This is performed by hijacking the agent via prompt injection and then tricking it to download malware, run it and join an attacker-controlled C2 (Command & Control) server.

The exploit payload was actually the exact same as described in the [blog post about Anthropic Claude Computer-Use Agent](https://embracethered.com/blog/posts/2024/claude-computer-use-c2-the-zombais-are-coming/).

## Prompt Injection Payload

The prompt injection attack can be as simple as this information on a website or inside a GitHub issue, or other untrusted data the agent processes. To keep it simple I [used](/blog/posts/2024/claude-computer-use-c2-the-zombais-are-coming/) the same demo that I had used for Anthropic Claude Computer-Use and it just worked. The only thing I changed was to swap out the binary that gets executed.

[![spaiware website](/blog/images/2025/computer-download-this.png)](/blog/images/2025/computer-download-this.png)

Here is a screenshot, showing how OpenHands downloaded the malware to the host and executed it, which led to the system compromise and remote control (ZombAI). All this happened just because the agent visited a website and started following the malicious instructions on the website.

[![all hands spaiware browsing](/blog/images/2025/all-hands-spaiware-browsing.png)](/blog/images/2025/all-hands-spaiware-browsing.png) 

And here is the C2 Server showing the connection of the ZombAI:

[![all hands spaiware browsing](/blog/images/2025/all-hands-sliver-connect.png)](/blog/images/2025/all-hands-sliver-connect.png) 


## Initial Task That OpenHands Was Given

Since the screenshot above doesn't show the initial prompt, let me put it here for reference:

[![load website](/blog/images/2025/all-hands-check-out-site.png)](/blog/images/2025/all-hands-check-out-site.png) 

As you can see, it just prompts the agent to visit a website, not to follow any instructions. This is to simulate such an attack. 

Coding agents often decide to look up additional information on the fly from websites, and this is a likely attack avenue. Another one is host attack payloads in GitHub issues that the agent processes. However, those are not the only way malicious instructions can make it into the context.

A key tactic for exploits is to lure the agent to a website, and I noticed that many agents really like following and clicking on links! Hence, if they end up at an attacker-controlled site (e.g. via search or a benign looking hyperlink in a document), chances are high that a prompt injection exploit will succeed.

## Video Demonstration of AI ClickFix

I was curious about trying out the AI ClickFix attack as well. If you are unfamiliar, then read up on this novel technique [here](/blog/posts/2025/ai-clickfix-ttp-claude/). To my surprise it worked end to end at first try, check out this brief and simplified demo:

{{< youtube QlwOUQnUUvM >}}

OpenHands followed the AI ClickFix instructions, copied the shell commands and pasted them into the terminal tool...

## Mitigations - Running In A Sandbox

For the local version of OpenHands, the guidance is to run it in a sandbox and it's critical to follow those setup instructions. You can find them [here](https://github.com/All-Hands-AI/OpenHands?tab=readme-ov-file#-running-openhands-locally). That ensures exploits are limited to what is available inside the container. Still not great, but that provides some damage control. One can also control outbound network connections via firewall settings.

However, be careful sharing or using any sensitive API keys, code or data inside the container. This container sandbox does not prevent the image markdown data leakage [we described yesterday](blog/posts/2025/openhands-the-lethal-trifecta-strikes-again/), where your host is the proxy of the request.

## Recommended Mitigation

The following mitigations come to mind.

Mitigations the vendor could implement:

* Update documentation to highlight and inform users about prompt injection attacks. 
* Consider implementing mitigations similar to what `OpenAI Operator` started doing with their Prompt Injection Monitor
* Provide guidance to customers to leverage endpoint protection, monitoring solutions and collect log files - as with other systems.
* Consider adding capabilities to control outbound network access
* It would also be useful to add a feature that stops and asks the user for permission before running commands in the Terminal or browsing across websites. 

Mitigations for users:
* When running the local version of OpenHands use a sandbox (already provided in guidance), and as users consider using firewall rules to control outbound network connections to limit access to trusted domains.
* Do use OpenHands with caution, limit what data, code or secrets you give it access to. 
* Agents are potential malicious insiders, treat these systems as such

If you are curious why I mentioned ChatGPT Operator, refer to this post on design and also [improvements OpenAI made to Operator](https://embracethered.com/blog/posts/2025/chatgpt-operator-prompt-injection-exploits/). Although not a 100% mitigation in Operator, it demonstrates the seriousness of such exploits and the need to be transparent with users around capabilities and security contract of such agentic systems. 

## Responsible Disclosure

* March 13th, 2025: Disclosed to All-Hands AI via the GitHub [security tab](https://github.com/All-Hands-AI/OpenHands/security) of the project. 
* March 20th, 2025: Follow up, no response.
* March 30th, 2025: Vulnerability not triaged. Followed up and it was acknowledged.
* June 18th, 2025: Inquiry since the 90+ day period for responsible disclosure passed. No reply.
* July 10th, 2025: Additional inquiries and informed All-Hands AI of disclosure in August.
* August 10th, 2025: 140+ days after private disclosure. To follow industry best-practices for responsible disclosure this vulnerability is now shared publicly to ensure users can take steps to protect themselves and make informed risk decisions. 

## Conclusion

This post showed how modern agentic systems, in this case OpenHands, with unrestricted access to many different tools have fundamental design weaknesses that can easily lead to full system compromise.

Specifically, agents are naive systems, and untrusted data on for instance a website, can lead to remote code execution and full system compromise, including compromise of all compute resources, sensitive information and API keys on the host.

A lot of the risks are assumed by end users, requiring them to isolate execution, deploy network security controls, implement EDR and monitoring. Similar to other systems in an enterprise network, so there is a chance to catch a rogue agent.

# References

* [OpenHands - Running OpenHands Locally](https://github.com/All-Hands-AI/OpenHands?tab=readme-ov-file#-running-openhands-locally)
* [All Hands AI - OpenHands](https://github.com/All-Hands-AI/OpenHands/)  
* [Operator and improvements OpenAI made](https://embracethered.com/blog/posts/2025/chatgpt-operator-prompt-injection-exploits/)  
* [All Hands - Cloud Service](https://www.all-hands.dev/)
* [Month of AI Bugs 2025](https://monthofaibugs.com)
