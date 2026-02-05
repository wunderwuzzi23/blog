---
title: "OpenAI Explains URL-Based Data Exfiltration Mitigations in New Paper"
date: 2026-02-04T23:59:30-07:00
draft: true  
tags: ["llm", "data exfiltration", "blue"] 
twitter:  
  card: "summary_large_image"  
  site: "@wunderwuzzi23"  
  creator: "@wunderwuzzi23"  
  title: "OpenAI Explains URL-Based Data Exfiltration Mitigations in New Paper"
  image: "https://embracethered.com/blog/images/2026/data-exfil-mitigation.png"  
---

Last week I saw [this paper](https://cdn.openai.com/pdf/dd8e7875-e606-42b4-80a1-f824e4e11cf4/prevent-url-data-exfil.pdf) from OpenAI called "Preventing URL-Based Data Exfiltration in
Language-Model Agents", which goes into detail on new mitigations they’ve added.

[![OpenAI Paper Abstract](/blog/images/2026/openai-paper-abstract.png)](https://cdn.openai.com/pdf/dd8e7875-e606-42b4-80a1-f824e4e11cf4/prevent-url-data-exfil.pdf)

**This is a great read.** I like this transparency. 


### Initial Disclosure in 2023

Nearly three years ago I reported the zero-click data exfiltration exploit to OpenAI. Back in early 2023 OpenAI did not have a bug bounty program, so communication was via email, and unfortunately there was little traction or appetite to fix the problem in ChatGPT. I also reported the same issue to Microsoft as Bing Chat was impacted, and Microsoft applied a fix (via a Content-Security-Policy header) in May 2023 to generally prevent loading of images.

Since then, dozens of such vulnerabilities across the AI ecosystem, being one of the most common AI application security vulnerabilities.

### Tackling the Problem

In December 2023 OpenAI [added initial mitigations via a url_safe feature](https://embracethered.com/blog/posts/2023/openai-data-exfiltration-first-mitigations-implemented/). But it was unclear exactly what the decision process was that made a URL safe. I reverse engineered a few things over time and found bypasses, like those presented at [BlackHat Europe 2024](https://www.youtube.com/watch?v=84NVG1c5LRI). Additionally, there were places where OpenAI had not added the mitigation at all, like in the macOS and iOS apps.

Interestingly, back then I wrote:

> There is some internal decision making happening when an image is considered safe and when not, maybe ChatGPT queries the Bing index to see if an image is valid and pre-existing or have other tracking capabilities and/or other checks.

Around August 2025 additional mitigations were [added](https://embracethered.com/blog/posts/2025/chatgpt-chat-history-data-exfiltration/), specifically the `url_safe` feature which mitigated the attacks shown at BlackHat and [Month of AI Bugs](https://embracethered.com/blog/posts/2025/chatgpt-chat-history-data-exfiltration/).

The latest paper from OpenAI discusses these new mitigation steps in detail.

## What Is the New Mitigation 

First, it's not a solved problem, but complexity of attacks has increased.

But at a high level, OpenAI has a crawler and any URL that it encounters is added to an allow-list, meaning ChatGPT will navigate to that URL without issue. However, if ChatGPT tries to dynamically create a new URL, then it would be unsafe.

So, it's not using the Bing Index (as I had ideated in 2023), but OpenAI now has their own web index and they also use it for URL validation.

Also, if a URL is directly entered as part of a user message, that URL is considered safe for the chat session. At least that has been my understanding with ChatGPT for a while.


### How Can It Be Bypassed?

Despite these mitigations, certain bypasses remain viable.

One such bypass trick [that I described in 2023](https://embracethered.com/blog/posts/2023/openai-data-exfiltration-first-mitigations-implemented/) is still applicable, and OpenAI also highlights it in the paper. 

The idea is to use individual URLs for each letter.

Now, those URLs would have to be in their search index also. If someone has a larger website or blog there are probably already 36 URLs indexed that can be used for the LLM to do the mapping from A-Z and 0-9. 😈 Someone could also go a lot further, expanding the vocabulary to thousands of URLs and teach the model via indirect prompt injection the mapping.

Leakage of bits of information still seems straightforward, and that remains a risk for certain scenarios.

You can see a basic demo of something similar [in my blog post from 2023](https://embracethered.com/blog/posts/2023/openai-data-exfiltration-first-mitigations-implemented/) which highlights the idea, but with limitations.

## Additional Mitigation Ideas

One challenge for attackers that I highlighted back in 2023 was that the browser caches requests and requests might arrive out of order. Both are mitigating factors, but the fact that the AI does connect is still a leakage channel that can be abused.

Two ideas immediately come to mind, and they are both related. The idea is to prevent the agent from visiting the same full URL multiple times in one session. This makes it even more difficult for an attacker.

This could be achieved by:
1. Caching the response for some time (e.g., even just a few minutes would probably suffice)
2. Prevent the agent from visiting the same URL multiple times within one session

This would still allow limited data exfiltration, but tricks like using fixed URLs to map characters repeatedly would face additional challenges. I have not tested the feature recently to check if some of these mitigations are in place by the way.

Further advances in research and feature additions or changes might highlight additional exploit techniques in the future. 

## Final Risk - Thorough Adoption of the Mitigation

The final real-world risk is that not every developer at OpenAI might know about this mitigation and adopt it accordingly. This is a typical engineering challenge for the security organization that I have personally encountered often in the past also. Just because there is a good mitigation, it doesn't mean that it is actually used.  

Also, this mitigation is about the data exfiltration channel, it does not make any claim or statement about indirect prompt injection.

It's great to see these mitigations being added, and also a paper about it being published by OpenAI. Even though it's not a silver bullet to fix all leaks, it is a significant improvement.

Happy Hacking.


## References

* [AI Agent Link Safety](https://openai.com/index/ai-agent-link-safety/)
* [OpenAI Begins Tackling ChatGPT Data Leak Vulnerability](https://embracethered.com/blog/posts/2023/openai-data-exfiltration-first-mitigations-implemented/)
* [Simon Willison - Lethal Trifecta](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/)
 