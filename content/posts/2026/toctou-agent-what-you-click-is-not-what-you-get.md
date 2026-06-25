---
title: "Computer-Use and TOCTOU: What You Click Is Not What You Get!"
date: 2026-06-25T05:20:58-07:00
draft: true
tags: ["llm", "agents"]
twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "Computer-Use and TOCTOU: What You Click Is Not What You Get!"
  description: ""
  image: "https://embracethered.com/blog/images/2026/toctou-agents.png"
---

Last year, Jun Kokatsu disclosed an [interesting vulnerability](https://github.com/google/security-research/security/advisories/GHSA-mp56-7vrw-qxvf) with ChatGPT Operator by exploiting a race condition. I was wondering if I could reproduce this attack chain, and this post describes the results of that research.

I had this post drafted for months, and yesterday at the [Real-world AI security conference](https://seclab.stanford.edu/RealWorldAIsec/) I included a video demo of this attack in my talk and that reminded me that I should finally publish this.

[![toctou](/blog/images/2026/toctou-agents.png)](/blog/images/2026/toctou-agents.png)

I will discuss `Claude Computer-Use` specifically in this post. This research happened last October, and I just never got around sharing it.

## What Is a TOCTOU Attack?

`TOCTOU` stands for **time-of-check to time-of-use** and describes a kind of [race condition](https://en.wikipedia.org/wiki/Time-of-check_to_time-of-use) in security.

## How Do Computer-Use Agents Work?

Computer-Use Agents take screenshots, and the AI processes the screenshots to determine the next steps. Once the Agent completes the reasoning process, it will take an action, like entering data or clicking a button or link.

This creates a classic `TOCTOU` situation, where the computer screen could have changed while the LLM inference was happening.

**That means that the action the AI Agent takes can happen on a different object than intended!**

### Basic Test To Trick an Agent

I ran a quick experiment with a single page that has an **OKAY** button, that I swapped out with an **ENTER THE MATRIX** button after 2 seconds.

[![email draft](/blog/images/2026/toctou-basic.png)](/blog/images/2026/toctou-basic.png)

To my surprise both Claude Computer-Use and also ChatGPT Agent both fell for this at the first time, and it repro'd basically every single time.

Knowing that, it was time to craft something more malicious.

## Let's Trick the AI To Send An Email

Outlook allows you to draft an email with a single URL.

The only thing needed is to click that link and you have a fully drafted email open, and all you need todo is click send!

Let me show you what I mean:

```
https://outlook.office.com/mail/deeplink/compose?to=REDACT@wunderwuzzi.net&subject=Trust%20No%20AI&body=This%20email%20was%20sent%20by%20your%20friendly%20AI%20Agent%20%0a%0dTrust%20No%20AI
```

Note: For the Outlook consumer version the domain is different but it works the same.

If you click that link it will open Outlook, like this:

[![email draft](/blog/images/2026/toctou-email-draft.png)](/blog/images/2026/toctou-email-draft.png)

I noticed this feature a few years back and it seemed that it could be used malicously. I just didn't know how back then... Now, I know. AI! 

## Crafting The Attack

So, all we need to do is have an attacker controlled phishing page, which shows a "Click here to continue" button. And then we place that button exactly at the same location as the "Send" button in Outlook.

This is the page I came up with:

[![email draft](/blog/images/2026/toctou-send-pi.png)](/blog/images/2026/toctou-send-pi.png)

**The important part is that the "Continue" button is at the exact same location as the "Send" button in the draft! The Agent will think it clicks the Continue button, but in reality it clicks "Send" to send an arbitrary email.**

There is one trick that I had to come up with.

The load of the Outlook page takes long and the Agent is taking new screenshots which prevents the exploit from working. This means I had to find a way to wait for 4-5 seconds until the Outlook page with the mail draft is loaded.

To achieve that, the prompt injection asked Claude to calculate `1+1` using bash. This means there is an additional bash command run, which takes enough time to successfully load the Outlook page that we want to have the AI click "Send" on!

## Video Demonstration

Here is an end-to-end demonstration to show this TOCTOU scenario:
{{< youtube i16jHwD1tIQ >}}


## Disclosure

I reported this to Anthropic last October, and it was acknowledged and highlighted that they were already tracking this risk, and that Computer-Use agent, back then, was still a preview feature, see the [security considerations](https://platform.claude.com/docs/en/agents-and-tools/tool-use/computer-use-tool#security-considerations) section.

When Anthropic shipped Cowork with Computer-Use a few months ago, Felix Rieseberg (one of the developers) said in the [announcement](https://x.com/felixrieseberg/status/2036476516960539118) that "We also ensure that pixels haven't changed before action".

[![felix announcement](/blog/images/2026/felix-tweet.png)](/blog/images/2026/felix-tweet.png)

So, this was addressed by Anthropic. Additionally, I also reported this TOCTOU threat to a couple of other vendors last year that I found vulnerable.

## Recommendation

The solution for this is simple: Ensure that the UI hasn't changed before taking an action, e.g take a snapshot when reasoning starts and when reasoning concludes check again that nothing significant changed.

**It's a classic old school security problem, that surfaces under novel conditions.**

## Conclusion

The reasoning step (which often takes multiple seconds) offers a quite large window to create a very reliable race condition. This can lead to accidents, where the agent takes actions on outdated pixels, but it can also lead to security exploits, like data exfiltration, modification of settings and configurations.

Happy hacking,
Johann.

## References

* [Jun's Disclosure: OpenAI Operator - Click on arbitrary origin by TOCTOU attack ](https://github.com/google/security-research/security/advisories/GHSA-mp56-7vrw-qxvf)
* [Race Conditions](https://en.wikipedia.org/wiki/Time-of-check_to_time-of-use)
* [Real-world AI Security Conference](https://seclab.stanford.edu/RealWorldAIsec/)

