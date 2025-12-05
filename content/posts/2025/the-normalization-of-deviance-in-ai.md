---
title: "The Normalization of Deviance in AI"
date: 2025-12-04T18:42:03-08:00
draft: true  
tags: ["llm", "machine learning"]
twitter:  
  card: "summary_large_image"  
  site: "@wunderwuzzi23"  
  creator: "@wunderwuzzi23"  
  title: "The Normalization of Deviance in AI"  
  description: "The gradual and systemic over-reliance on LLM outputs, especially with agentic systems, leads to a normalization of deviance."
  image: "https://embracethered.com/blog/images/2025/normalization-of-deviance-in-ai.png"  
---

The AI industry risks repeating the same cultural failures that contributed to the Space Shuttle Challenger disaster: Quietly normalizing warning signs while progress marches forward.

The original term [**Normalization of Deviance**](https://en.wikipedia.org/wiki/Normalization_of_deviance) comes from the American sociologist Diane Vaughan, who describes it as the process in which deviance from correct or proper behavior or rule becomes culturally normalized. 

[![normalization of deviance in ai](/blog/images/2025/normalization-of-deviance-in-ai.png)](/blog/images/2025/normalization-of-deviance-in-ai.png)

I use the term **Normalization of Deviance in AI** to describe the gradual and systemic over-reliance on LLM outputs, especially in agentic systems. 

At its core, large language models (LLMs) are unreliable (and untrusted) actors in system design. 

This means that security controls (access checks, proper encoding, and sanitization, etc.) must be applied downstream of LLM output. A constant stream of [indirect prompt injection exploit demonstrations](https://monthofaibugs.com) indicates that system designers and developers are either unaware of this or are simply accepting the deviance. It is particularly dangerous when vendors make insecure decisions for their userbase by default.

I first learned about this concept in the context of the [Space Shuttle Challenger disaster](https://en.wikipedia.org/wiki/Space_Shuttle_Challenger_disaster), where systemic normalization of warnings led to tragedy. 

*Despite data showing erosion in colder temperatures, the deviation from safety standards was repeatedly rationalized because previous flights had succeeded. The absence of disaster was mistaken for the presence of safety.* 


## The Model is Untrustworthy and Not Reliable

In the world of AI, we observe companies treating probabilistic, non-deterministic, and sometimes adversarial model outputs as if they were reliable, predictable, and safe. 

Vendors are normalizing trusting LLM output, but current understanding violates the assumption of reliability.

The model will not consistently follow instructions, stay aligned, or maintain context integrity. This is especially true if there is an attacker in the loop (e.g indirect prompt injection).

However, we see more and more systems allowing untrusted output to take consequential actions. Most of the time it goes well, but over time vendors and organizations lower their guard or skip human oversight entirely, because "it worked last time."

This dangerous bias is the fuel for normalization: organizations confuse the absence of a successful attack with the presence of robust security. 

**Two ways this can impact systems are:**
1. This normalization can be a safety incident that simply arises from over-trusting fallible but benign outputs (hallucinations, context loss, brittleness, etc.)
2. But it becomes more dangerous when adversarial inputs (prompt injection, backdoors in models) exploit systems. **The same cultural drift enables exploitation!**

And we already see agents make mistakes in day to day usage, like [formatting hard drives](https://www.theregister.com/2025/12/01/google_antigravity_wipes_d_drive/?td=rt-3a), [creating random GitHub issues](https://embracethered.com/blog/posts/2023/chatgpt-chat-with-code-plugin-take-down/), or [wipe a production database](https://fortune.com/2025/07/23/ai-coding-tool-replit-wiped-database-called-it-a-catastrophic-failure/).

So, the signs are there. And it is inherently dangerous, not only because of attacks like indirect prompt injection, but also because these systems are trained on enormous, untrustworthy data sets from the Internet. Anthropic research recently [showed that](https://www.anthropic.com/research/small-samples-poison) it takes only a small amount of documents to successfully add a backdoor to a model.

Consider a scenario where the Normalization of Deviance has drastic consequences: an attacker trains a backdoor into a model that triggers on certain days to invoke tools, like compromising a user via code execution. Since we have a pretty centralized ecosystem, where attacks often are transferable, and natural language is universally understood by LLMs, this can have consequences across many systems and vendors.

## Cultural Drifts in Organizations

Such a drift does not happen through a single reckless decision. It happens through a series of "temporary" shortcuts that quietly become the new baseline. Because systems continue to work, teams stop questioning the shortcuts, and the deviation becomes invisible and the new norm.

Especially under competitive pressure for automation, cost savings, a drive to be first, and the overall hype, this dangerous drift is evident. The incentives for speed and winning outweigh the incentives for foundational security. Over time, organizations forget why the guardrails existed in the first place.

## Industry Examples of the Normalization of Deviance in AI

Let me share some examples of how this is reflected in real-world agentic AI systems. 

We are all aware that chatbots have those "AI can make mistakes", "Double check responses" and so forth disclaimers, and we can observe the drift of normalization occurring in real-time. 

Three years after ChatGPT shipped, vendors push agentic AI to users, but at the same time vendors are highlighting that your system might get compromised by that same AI - that drift, that normalization, is what I call "The Normalization of Deviance in AI".

**This continuous drift is a long-term danger:**

* **Microsoft: Agentic Operating System:** Microsoft's [documentation warns](https://support.microsoft.com/en-us/windows/experimental-agentic-features-a25ede8a-e4c2-4841-85a8-44839191dfb3) that prompt injection attacks "can override agent instructions, leading to unintended actions like data exfiltration or malware installation" and that "Agents may perform actions beyond what the user intended". That agents can be insider threats is something that I have been highlighting in my talks for a longer time, and a [recent paper by Anthropic and University College of London supports this](https://arxiv.org/abs/2510.05179) with results. AI might start blackmailing other people, etc. when it wants to achieve a certain objective or "feels" threatened.
* **OpenAI ChatGPT Atlas**: It's documented by the vendor that the system might make mistakes when browsing the web. In particular, OpenAI [states](https://help.openai.com/en/articles/12603091-chatgpt-atlas-for-enterprise): "We recommend caution using Atlas in contexts that require heightened compliance and security controls — such as regulated, confidential, or production data." In other words, OpenAI explicitly warns against trusting Atlas with high-stakes or sensitive data due to unresolved security risks.
* **Anthropic Claude**: [Data Exfiltration](/blog/posts/2025/claude-abusing-network-access-and-anthropic-api-for-data-exfiltration/), referenced [here](https://support.claude.com/en/articles/12111783-create-and-edit-files-with-claude#h_27fc9da35e): "This means Claude can be tricked into sending information from its context (for example, prompts, projects, data via MCP, Google integrations) to malicious third parties. To mitigate these risks, we recommend you monitor Claude while using the feature and stop it if you see it using or accessing data unexpectedly. You can report issues to us using the thumbs down function directly in claude.ai."
* **Google Antigravity**: [Remote Code Executions](https://bughunters.google.com/learn/invalid-reports/google-products/4655949258227712/antigravity-known-issues#code-execution) via indirect prompt injection is a known issue when the product first shipped, as is data exfiltration.
* **Windsurf Cascade Coding Agent**: No human in the loop feature for MCP tool calls. Lack of human in the loop can normalize risky practices by over-trusting AI outputs in high-stakes situations. See also the [Month of AI Bugs](https://monthofaibugs.com).

While some vendors acknowledge the risks, others appear to overlook or downplay them, potentially due to competitive pressure and focus on product and customer acquisition. In many cases, we probably collectively hope that "someone" will solve these security and safety challenges.

Companies like Google, OpenAI, Anthropic, Microsoft, and other institutions and organizations perform extensive research in this area, including publishing evals and mitigation ideas. However, the rush to be the first is evident from a product perspective.

## Conclusion

Nevertheless, before we drift off into a utopian future with agentic AI, I believe the best and safest outcome is to stay realistic around capabilities and control mechanisms, and for AI to remain human-led, particularly in high-stake contexts, to ensure the best outcome overall. 

Does that mean AI is doomed? 

No, of course not. There is a lot of potential and many low stakes workflows can be implemented already today. Even high-risk workflows can be done with proper threat modeling, mitigations and oversight. 

However, it requires investment and resources to design and set up systems accordingly and apply security controls (sandbox, hermetic environments, least privilege, temporary credentials, etc.). 

Many are hoping the "model will just do the right thing", but Assume Breach teaches us, that at one point, it will certainly not do that.

Trust No AI.

## References

* [Normalization of Deviance](https://en.wikipedia.org/wiki/Normalization_of_deviance)
* [Windows - Experimental Agentic Features](https://support.microsoft.com/en-us/windows/experimental-agentic-features-a25ede8a-e4c2-4841-85a8-44839191dfb3)
* [Agentic Misalignment: How LLMs Could Be Insider Threats](https://arxiv.org/abs/2510.05179)
* [Month of AI Bugs](https://monthofaibugs.com)
* [Claude - Hit STOP if you see data exfiltration](https://support.claude.com/en/articles/12111783-create-and-edit-files-with-claude#h_27fc9da35e)
* [Google Antigravity - Known Issues](https://bughunters.google.com/learn/invalid-reports/google-products/4655949258227712/antigravity-known-issues#code-execution)
* [Anthropic - # A small number of samples can poison LLMs of any size](https://www.anthropic.com/research/small-samples-poison)
* [Google Antigravity Wipes D Drive](https://www.theregister.com/2025/12/01/google_antigravity_wipes_d_drive/?td=rt-3a)
* [An AI-powered coding tool wiped out a software company's database, then apologized for a catastrophic failure on my part](https://fortune.com/2025/07/23/ai-coding-tool-replit-wiped-database-called-it-a-catastrophic-failure/)