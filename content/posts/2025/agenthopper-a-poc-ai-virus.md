---
title: "AgentHopper: An AI Virus Research Project"  
date: 2025-08-29T20:20:58-07:00  
draft: true  
tags: ["llm", "agents", "month of ai bugs"]
twitter:  
  card: "summary_large_image"  
  site: "@wunderwuzzi23"  
  creator: "@wunderwuzzi23"  
  title: "AgentHopper: An AI Virus Research Project"  
  description: "AgentHopper: A proof-of-concept AI Virus"  
  image: "https://embracethered.com/blog/images/2025/episode29-yt.png"  
---

{{< raw_html >}}
<a id="top_ref"></a>
{{< /raw_html >}}

As part of the Month of AI Bugs, serious vulnerabilities that allow remote code execution via indirect prompt injection were discovered. There was a period of a few weeks where multiple arbitrary code execution vulnerabilities existed in popular agents, like GitHub Copilot, Amazon Q, AWS Kiro,...

During that time I was wondering if it would be possible to write an AI virus. 

[![Episode Final Episode 29](/blog/images/2025/episode29-yt.png)](/blog/images/2025/episode29-yt.png)

Hence the idea of AgentHopper was born. This post is purely for educational purposes, and make sure to check the mitigations section at the end on tips to mitigate similar threats.

**All vulnerabilities mentioned in this research were responsibly disclosed and have been patched by the respective vendors. This post is shared for educational and awareness purposes to help improve security in AI-based developer tools. Check out the mitigations section for useful tips as well.**

Interestingly, recently we have seen [multiple real-world cases](https://github.com/aws/aws-toolkit-vscode/security/advisories/GHSA-7g7f-ff96-5gcw) where adversaries [target and exploit coding agents](https://www.stepsecurity.io/blog/supply-chain-security-alert-popular-nx-build-system-package-compromised-with-data-stealing-malware), including pushing information up to GitHub. 

## AgentHopper - An AI Virus Research Project

The idea was to have one prompt injection payload that would operate across agents and exploit them accordingly. This was to demonstrate that conditional prompt injection can be leveraged as a powerful mechanism to target specific agents. 

Let's first briefly revisit the vulnerabilities that vendors fixed which enabled AgentHopper.

## The Vulnerabilities and the Proof-of-concept

During the Month of AI Bugs we described a couple of arbitrary code execution vulnerabilities that vendors addressed:
* [GitHub Copilot: Remote Code Execution via Prompt Injection (CVE-2025-53773)](/blog/posts/2025/github-copilot-remote-code-execution-via-prompt-injection/)
* [Amp Code: Arbitrary Command Execution via Prompt Injection Fixed](/blog/posts/2025/amp-agents-that-modify-system-configuration-and-escape/)
* [Amazon Q Developer: Remote Code Execution with Prompt Injection](/blog/posts/2025/amazon-q-developer-remote-code-execution/)
* [AWS Kiro: Arbitrary Code Execution via Indirect Prompt Injection](/blog/posts/2025/aws-kiro-aribtrary-command-execution-with-indirect-prompt-injection/)

When discovering and responsibly disclosing them to vendors, I was wondering if it would be possible to build a prompt injection payload that exploits them all. 

A payload that can propagate, like a good old fashion computer virus. 

Back when I had a Commodore Amiga, viruses actually got transferred via floppy disks. When inserting an infected floppy disk, the virus would move to memory and wait for any new floppy disk to be inserted. If a new floppy was inserted, then the virus would attach itself to that floppy, and so on.

All vulnerabilities have been addressed by vendors by the way.

## The Infection Model: How AgentHopper Spreads

So, how does this thing actually move? The basic concept for AgentHopper's propagation is deceptively simple and leverages common developer workflows.

This diagram is sort of a high level architectural overview:
[![AgentHopper - Concept](/blog/images/2025/agenthopper-concept.png)](/blog/images/2025/agenthopper-concept.png)

Here's the breakdown:

* **Initial Infection**: An adversary compromises a developer's agent (Agent 1) through an initial indirect prompt injection or otherwise. This is the entry point.

* **Payload Distribution**: Once Agent 1 is compromised, it downloads the AgentHopper binary and executes it. AgentHopper then scans for Git repositories (e.g., Repo 1, Repo 2), injects them with the universal prompt injection payload, commits the changes and does a `git push --force` back to GitHub. The virus basically finds new repositories, which serve as host.

* **The Hop**: Now, imagine another developer, using another coding agent (Agent 2), pulls the now-infected code from GitHub (say, from Repo 2). When Agent 2 processes or interacts with this code, the prompt injection is triggered.

* **Secondary Infection & Propagation**: Agent 2 becomes compromised, downloads and executes AgentHopper, and the cycle continues, infecting its own repositories (Repo 2, Repo 3) and pushing those changes.

This mechanism highlights how easily a well-crafted prompt injection can propagate across different systems and agents. 

After a successful hope, the target source code in GitHub would look like this:

[![Virus Payload](/blog/images/2025/agenthopper-after.png)](/blog/images/2025/agenthopper-after.png)

As you can see it inserted the prompt at the beginning of the file.

**Safety Switch:** To avoid accidents and to make sure it doesn't compromise repos in my environment that I didn't want to be backdoored, I added a safety switch that asks for approval for each repo. 

## What are Conditional Prompt Injections?

I first [described the idea of conditional instructions](blog/posts/2024/whoami-conditional-prompt-injection-instructions/) for prompt injection exploit development a bit over a year ago with Microsoft Copilot, where we sent phishing emails that targeted different users with different attack payloads.

The idea is that we leverage information from the system prompt (like a user's name, job title) to branch off with conditional instructions targeted specifically for that individual.

[![Conditional Prompt Injection - AgentHopper](/blog/images/2025/agenthopper-conditional-pi.png)](/blog/images/2025/agenthopper-conditional-pi.png)

AgentHopper works similarly. To demonstrate that a prompt injection payload can be universal, and target different coding agents with different exploits to achieve arbitrary code execution.

## Attack Paths and Exploits

All of these vulnerabilities have now been patched.

1. **GitHub Copilot:** Write to the agents the `settings.json` file to put Copilot into YOLO mode, then download and run AgentHopper
2. **Amp Code:** Write to the agent's configuration file and add a fake MCP server which is a python program that downloads AgentHopper and runs it
3. **Amazon Q Developer:** Execute the `find` bash command with the `-exec` argument to download AgentHopper and execute it
4. **AWS Kiro:** Write to the agent's configuration file to allowlist all bash commands, then download and run AgentHopper

## Agents That Can Modify Their Own Configuration

As I described when I first discovered these vulnerabilities where an agent can modify its own configuration to escape, it sounded like science fiction, but it turns out it's actually a common design flaw that can easily be mitigated and even be caught early on during threat modeling.

## Video Demonstration

Stop by again tomorrow for a long-form video.

## Mitigations & Recommendations For Developers!

When developing this, it gave me a bit the chills, because it was seemingly quite easy to vibe code such malware. Scary simple.

But it also highlights a couple of things that are important for user/developer to consider, because we can protect ourselves from such attacks:
* Make sure you **have a passphrase** on your SSH and signing keys!
* **Enable branch protection**
* **Principle of Least Privilege**: This demo exploited vulnerabilities which all have been patched by vendors. However users might configure agents in a way that gives them too many privileges (e.g. allowlisting dangerous commands etc.), or running agents entirely in YOLO mode.
* If a coding agent has **sandbox capabilities**, then leverage them
* EDR vendors and GitHub hopefully considers such threats, as they can detect widespread infections quickly and mitigate it

## Conclusion

Given the [recent malware cases](https://www.stepsecurity.io/blog/supply-chain-security-alert-popular-nx-build-system-package-compromised-with-data-stealing-malware)  and [exploits leveraging coding agents](https://github.com/aws/aws-toolkit-vscode/security/advisories/GHSA-7g7f-ff96-5gcw), AI-driven malware and prompt payloads will likely become more prevalent in the near future.

AgentHopper is a good reminder to make sure you have branch protection on, and that you use a passphrase on your ssh and signing keys to prevent malware automatically pushing changes to GitHub, or other places.

## Thank you.

This is the last post of the Month of AI Bugs. 

If you read many of my posts I want to thank you very much. I really do hope that my work can inspire more people to get into security testing, especially now with the rapid advent of AI and agents, we need a lot more people to hold vendors accountable and ask basic questions around the rapid and wide deployment of AI systems that have fundamental design weaknesses.

With that said, my posting schedule will go back to a less frequent cadence.

Wish you all well, and happy hacking!

## References

* [Month of AI Bugs 2025](https://monthofaibugs.com)
* [Amazon Q - Malicious script injected into Amazon Q Developer for VS Code Extension](https://github.com/aws/aws-toolkit-vscode/security/advisories/GHSA-7g7f-ff96-5gcw)
* [Step Security - Supply chain alert in popular nx build system package](https://www.stepsecurity.io/blog/supply-chain-security-alert-popular-nx-build-system-package-compromised-with-data-stealing-malware])