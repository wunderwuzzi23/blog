---
title: "I Spent $500 To Test Devin AI For Prompt Injection So That You Don't Have To"  
date: 2025-08-06T01:01:58-07:00  
draft: true  
tags: ["llm", "agents", "month of ai bugs"]
twitter:  
  card: "summary_large_image"  
  site: "@wunderwuzzi23"  
  creator: "@wunderwuzzi23"  
  title: "I Spent $500 To Test Devin AI For Prompt Injection So That You Don't Have To"  
  description: "I Paid $500 to test Devin AI for security vulnerabilities in April 2025. When processing untrusted data Devin can be hijacked to run remote code (RCE) and connect to an attacker's command and control system (ZombAI)."  
  image: "https://embracethered.com/blog/images/2025/episode6-yt.png"  
---

{{< raw_html >}}
<a id="top_ref"></a>
{{< /raw_html >}}


Today we cover [Devin AI](https://cognition.ai/blog/introducing-devin) from Cognition, the first AI Software Engineer. 

[![Devin Title Image Month of AI Bugs Episode 6](/blog/images/2025/episode6-yt.png)](/blog/images/2025/episode6-yt.png)

We will cover Devin proof-of-concept exploits in multiple posts over the next few days. In this first post, we show how a prompt injection payload hosted on a website leads to a full compromise of Devin's DevBox.

## GitHub Issue To Remote Code Execution

By planting instructions on a website or GitHub issue that Devin processes, it can be tricked to download malware and launch it. This leads to full system compromise and turns Devin into a remote-controlled ZombAI. Any exposed secrets can then be leveraged to perform lateral movement, or other post-exploitation steps.

Read on to learn all the details behind this research.

## Investing $500 To Shine Light On Vulnerabilities 

First, I had to invest $500 to get access to Devin for 30 days.

[![devin-spaiware-500](/blog/images/2025/devin-subscription.png)](/blog/images/2025/devin-subscription.png) 

For reference, Cognition actually now offers a cheaper plan.

## Command And Control Infrastructure 
First, I had to set up a few infrastructure components for the C2 server. Luckily this is something I typically have handy, so it was quite quick to set up.

### Sliver Server
As a command & control system I used Sliver from Bishop Fox. I hosted a Sliver server and generated a Linux malware binary with Sliver that will call back and allow remote control of any host that launches the binary.

### Hosting the Prompt Injection Payload
Next, this GitHub issue was used in the exploit demonstration:
[![devin-spaiware-malware-prompt-injection](/blog/images/2025/devin-malware-github-issue-works.png)](/blog/images/2025/devin-malware-github-issue-works.png)

The key trick here is to have Devin navigate off domain (e.g. away from GitHub to the attacker's page). This is not a requirement, but it appears to be a lot more reliable that way, and better simulates real-world risk. But I observed it followed more complex instructions directly from GitHub also.

## Attack Sequence 
The scenario is that we have a GitHub issue that Devin is tasked to look into. 

After Devin starts exploring the GitHub issue, it notices the text of the support tool. Since it is mentioned that the tool could help with debugging the issue, Devin navigates off domain to the attacker-controlled website.
[![devin-spaiware-malware-support-tool](/blog/images/2025/devin-download-support-tool.png)](/blog/images/2025/devin-download-support-tool.png)

Here we go, now Devin has reached the attacker's website!

### **Agents Love Clicking Links!**

This is one of the key observations I had when testing various agents by the way, **they like clicking links!** And once you get them off domain to an attacker-controlled site things get quite easy with prompt injection payloads. Attacks often work at first try basically. 

Devin, again, follows instructions on a website and clicks the link, which initiates the file download.

Next, it switched to a Terminal to inspect and try to run the binary:  
[![devin-spaiware-malware-permission-issue](/blog/images/2025/devin-spaiware-permission-denied.png)](/blog/images/2025/devin-spaiware-permission-denied.png)

As you can see, Devin received a permission denied error. But of course that doesn't stop Devin. Again, it opened another Terminal, added the execute permission and executed the malware binary again:

[![devin-spaiware-malware-support-tool](/blog/images/2025/devin-execute-permission.png)](/blog/images/2025/devin-sliver-run-command.png)

If you open the screenshot in full screen you can see the entire Devin session.

This now worked and we got a callback on the Sliver server!

[![devin-spaiware-malware-support-tool](/blog/images/2025/devin-joins-sliver.png)](/blog/images/2025/devin-joins-sliver.png)

And we can drop into a remote shell and, of course, snoop around Devin's machine.

[![devin-spaiware-malware-support-tool](/blog/images/2025/devin-sliver-access-aws-key.png)](/blog/images/2025/devin-sliver-access-aws-key.png)

More about exfiltrating secrets and information in the next post. So, stay tuned because there are more security issues lurking in Devin.

## Video Demonstration

Check out this demo video for more details and also a couple of variations around the attack chain and learnings

**Note:** I will package all three Devin posts into a single video that I'll release with the third post about Devin (planned for Friday this week), then I'll update this section here.

If you find the video interesting, please subscribe and share! 

##  Setting Up Auto-Execute Commands With Sliver

One observation I had in early exploits was that Devin would disconnect and cancel running the binary pretty quickly. 

In order to work around that and maintain persistence I set up a `reaction` event for `session-connected` which sends a few commands down to the ZombAI right away. 

This demonstrates that even if Devin cancels the command, secrets can be exfiltrated within a few milliseconds and an attacker can lay down additional persistence to maintain access. 

Since many readers may not have a red teaming background, I figured to throw in some interesting behind the scenes info as well.

## Sending Tasks To Devin Via Slack

It's also possible to interact with Devin directly from Slack, which makes it an entirely unsupervised interaction. Here is a very similar attack, where one user tasked Devin to investigate and research a website, but while doing that, Devin is compromised.

[![devin-spaiware-via-slack](/blog/images/2025/devin-slack-devin-ui.png)](/blog/images/2025/devin-slack-devin-ui.png)

**This shows the danger of unsupervised AI agents with unrestricted access to a large amount of tools.**

## Responsible Disclosure

This vulnerability was reported to Cognition on April 6th, 2025 and acknowledged a few days later. Follow-up queries around fix timelines and status, or coordinated disclosure remain unanswered after 120+ days.

The creation of powerful agents is pretty straightforward, and the value is unlocked by giving an agent access to data and tools. However, some systems are not designed with security in mind at all. Especially novel threats like indirect prompt injection are either not understood or ignored by a few vendors.

Hence, this information is now released publicly according to responsible disclosure best practices, so that users can protect themselves.

## Recommendations

The following recommendations were provided to Cognition as part of disclosing the vulnerabilities in April.

* Do not depend on model-behavior, in-chat confirmations or prompting for secure tool invocation, **but develop an out-of-band validation step** to allow the user to approve sensitive operations  
* Be transparent that users should not join Devin to an enterprise or VPN/private network.
* Highlight that any secret on a Devin box can be compromised by an adversary via a prompt injection attack from untrusted data, or Devin, if it decides, can easily leak them to remote hosts by itself.
* Prevent outbound connections by default  
* Lack of EDR and AV on hosts (advise customers to leverage endpoint protection)  
* Develop a [prompt injection monitor](https://embracethered.com/blog/posts/2025/chatgpt-operator-prompt-injection-exploits/) that could kick in when Devin starts browsing to untrusted domains or downloads binaries. 

## How To Mitigate Issues as a User?

* Understand the risks of prompt injection and the untrustworthiness of LLM output
* Be aware that any secret or private code Devin has access to can be leaked at will to third-party systems by the AI, or an attacker via indirect prompt injection  
* Monitor what Devin does 
* Do not give Devin access to sensitive data  
* Do not give Devin access to important infrastructure or join it to a corporate network without fully understanding the risks

## Conclusion

During a prompt injection attack from untrusted data, an attacker can gain access to all secrets and keys on the devin-box, and then perform lateral movement to gain access to other hosts in an organization or gain access to cloud infrastructure, etc.

**There seem to be two approaches when building coding agents:**

* The first is that some companies start with some core pieces like reviewing code, writing functions, and slowly expand capabilities, all based on the principle of human in the loop. A great example of this approach is for instance Anthropic's Claude Code. 

* The second approach is to just give an agent access to everything possible and hope for the best. 

The better approach certainly appears to start small and get the core pieces right first. 

**Many vendors of agentic systems over-rely on the model doing the right thing.**

This post showed how modern agentic systems commonly have fundamental design weaknesses that can easily lead to full system compromise, which makes them unsuitable for enterprise adoption. 

Specifically, agents are naive, and when they process untrusted data, like text on a website, can lead to remote code execution, including compromise of all compute resources, sensitive information and API keys on the host.

## References

* [Devin AI](https://devin.ai/)  
* [Cognition announces Devin](https://cognition.ai/blog/introducing-devin)   
* [Devin System Prompt](https://github.com/wunderwuzzi23/scratch/blob/master/system_prompts/devin-2025-04-10.md)
* [Month of AI Bugs 2025](https://monthofaibugs.com)
* [Operator and improvements OpenAI made](https://embracethered.com/blog/posts/2025/chatgpt-operator-prompt-injection-exploits/)

