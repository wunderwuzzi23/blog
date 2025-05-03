---
title: "GitHub Copilot Custom Instructions and Risks"
date: 2025-04-06T20:11:43-07:00
draft: true
tags: [
     "aiml", "machine learning", "threats", "llm"
    ]

twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "GitHub Custom Copilot Instructions and Risks"
  description: "Custom Rule Files in Code Editors Can Be Abused By Adversaries "
  image: "https://embracethered.com/blog/images/2025/github-copilot-instructions-tn.png"
---

GitHub Copilot has the capability to be augmented with [custom instructions](https://docs.github.com/en/copilot/customizing-copilot/adding-repository-custom-instructions-for-github-copilot) coming from the current repo, via the `.github/copilot-instructions.md` file.

[![copilot instructions](/blog/images/2025/github-copilot-instructions-tn.png)](/blog/images/2025/github-copilot-instructions-tn.png)

[Pillar Security](https://www.pillar.security/blog/new-vulnerability-in-github-copilot-and-cursor-how-hackers-can-weaponize-code-agents) recently highlighted the risks associated with rules files. Their post discusses custom `Cursor` rules in `./cursor/rules` ending in `.mdc`. 

If you watch the demos, you'll notice that they also have a GitHub Copilot demo which uses the GitHub specific `copilot-instructions.md` file.

**Update: May 1, 2025**
GitHub made a product change and is now highlighting invisible Unicode characters in the Web UI.  In [their announcement](https://github.blog/changelog/2025-05-01-github-now-provides-a-warning-about-hidden-unicode-text/) GitHub is referencing the Pillar Security post and also my post about ASCII Smuggling. Very cool!


## GitHub Copilot Custom Instructions File

I’ve also been experimenting with the `.github/copilot-instructions.md` file a bit recently.

Here’s a demonstration I’ve been using:
[![Github Instructions Contents](/blog/images/2025/copilot-custom-instr.png)](/blog/images/2025/copilot-custom-instr.png)

Now, if you ask Copilot to explain some code it adds these custom instructions to the prompt context, leading to the following inference result:
[![Github Instructions](/blog/images/2025/copilot-custom-instructions.png)](/blog/images/2025/copilot-custom-instructions.png)

Luckily, GitHub [mitigated](/blog/posts/2024/github-copilot-chat-prompt-injection-data-exfiltration/) the common 0-click image rendering data leakage issue last year, and for hyperlinks there is a confirmation dialog when clicking, unless the destination domain is the trusted sites list of VS Code. 

This screenshot shows the mitigation for navigating to random phishing sites in action:
[![Github Instructions - Default Hyperlink Mitigation](/blog/images/2025/copilot-custom-instructions-phishing-mitigation.png)](/blog/images/2025/copilot-custom-instructions-phishing-mitigation.png)

Developers might still be tricked, and there are other attack avenues... like adding backdoor code!

## Backdooring Instructions

The most significant issue is that a single sneaky line in the instructions file can add backdoor code. 

Here is a demo on how this works in `Go`, check this out: 

[![Github Instructions - Add Code](/blog/images/2025/copilot-custom-instr-add-code.png)](/blog/images/2025/copilot-custom-instr-add-code.png)

So, Greetings from [Jia Tan](https://de.wikipedia.org/wiki/XZ_Utils), I guess.

[Pillar Security](https://www.pillar.security/blog/new-vulnerability-in-github-copilot-and-cursor-how-hackers-can-weaponize-code-agents) does a great job in highlighting additional threats and considerations, such as hidden Unicode characters.

It's important to remain aware of what information is added to the prompt context, and the introduction of additional features by vendors can catch users and developers by surprise. Stay alert for any modifications to your custom instruction file.

And remember that [Anthropic never mitigated usage of hidden Unicode Tags](/blog/posts/2024/claude-hidden-prompt-injection-ascii-smuggling/), and Claude is interpreting such hidden characters as instructions.

## References

* [Pillar Security Blog Post - Rules File Backdoor](https://www.pillar.security/blog/new-vulnerability-in-github-copilot-and-cursor-how-hackers-can-weaponize-code-agents) 
* [Adding repository custom instructions for GitHub Copilot](https://docs.github.com/en/copilot/customizing-copilot/adding-repository-custom-instructions-for-github-copilot)
* [XZ Utils Backdoor Jia Tan](https://de.wikipedia.org/wiki/XZ_Utils)
