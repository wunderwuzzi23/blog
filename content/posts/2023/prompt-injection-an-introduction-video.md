---
title: "Video: Prompt Injections - An Introduction"
date: 2023-05-10T07:00:40-07:00
draft: true
tags: [
        "red", "ttp", "aiml", "video", "ai injections", "chatgpt"
    ]

twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "Video: Prompt Injections - An Introduction"
  description:  "There are many prompt engineering classes and currently pretty much all examples are vulnerable to Prompt Injections. Especially Indirect Prompt Injections are dangerous. They allow untrusted data to take control of the LLM (large language model) and give an AI a new instructions, mission and objective."
  image: "https://embracethered.com/blog/images/2023/ai-injections-thumbnail.png"
---

There are many prompt engineering classes and currently pretty much all examples are vulnerable to Prompt Injections. Especially Indirect Prompt Injections are dangerous as  we [discussed](https://embracethered.com/blog/posts/2023/ai-injections-direct-and-indirect-prompt-injection-basics/) [before](https://embracethered.com/blog/posts/2023/ai-injections-threats-context-matters/).
 
Indirect Prompt Injections allow untrusted data to take control of the LLM (large language model) and **give an AI a new instructions, mission and objective**. 

### Bypassing Input Validation

Attack payloads are natural language. This means there are lots of creative ways an adversary can inject malicious data that bypass input filters and web application firewalls.

### Leveraging the Power of AI for Exploitation 

Depending on scenario attacks can include JSON object injections, HTML injection, Cross Site Scripting, overwriting orders of an order chat bot and even data exfiltration (and many others) all with the power of AI and LLMs.

This video aims to continue to raise awareness of this rising problem.

{{< youtube Fz4un08Ehe8 >}}


Hope you enjoy this video about the basics of prompt engineering and injections.


### Outline of the presentation

- What is Prompt Engineering?
- Prompt Injections Explained 
- Indirect Prompt Injection and Examples 
- GPT 3.5 Turbot vs GPT-4 
- Examples of payloads
- Indirect Injections, Plugins and Tools 
- Algorithmic Adversarial Prompt Creation 
- AI Injections Tutorials + Lab
- Defenses 
- Wrap Up & Thanks 


## Injections: Tutorial + Lab

The Colab Notebook referenced in the video is located [here](https://colab.research.google.com/drive/1qGznuvmUj7dSQwS9A9L-M91jXwws-p7k).


## References

* [AI Injections](https://embracethered.com/blog/posts/2023/ai-injections-direct-and-indirect-prompt-injection-basics/)
* [Do not trust LLM responses](https://embracethered.com/blog/posts/2023/ai-injections-threats-context-matters/)