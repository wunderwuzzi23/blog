---
title: "Adversarial Prompting: Tutorial and Lab"
date: 2023-05-11T18:09:43-07:00
draft: true
tags: [
        "red", "ttp", "aiml", "video"
    ]

twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "Adversarial Prompting: Tutorial and Lab"
  description:  "If you want to get more hands-on experience learning about prompt engineering and security as it relates to prompt injections read on. JSON object injections, HTML Injection, XSS, data exfil and other scenarios are covered."
  image: "https://embracethered.com/blog/images/2023/adversarial_prompting.png"
---


If you want to get more hands-on experience learning about prompt engineering and security as it relates to prompt injections read on. 

To help myself learn more about Prompt Engineering and Prompt Injections I put together [this tutorial + lab](https://colab.research.google.com/drive/1qGznuvmUj7dSQwS9A9L-M91jXwws-p7k) as a Jupyter Notebook to experiement and play around with this novel attack technqieu.

The examples reach from simple prompt engineering scenarios, such as changing the output message to a specific text, to more complex adversarial prompt challenges such as JSON object injection, HTML injection/XSS, overwriting mail recipients or orders of an OrderBot and also data exfiltration.

The Colab Notebook is located [here](https://colab.research.google.com/drive/1qGznuvmUj7dSQwS9A9L-M91jXwws-p7k).

## Tutorial + Lab Walkthrough 

Additionally, I recorded this guided explanation of Prompt Engineering Techniques and Prompt Injection challenges to continue to raise awareness of this rising problem.

{{< youtube AQNV5U48Pho >}}


### Outline of the video

- Intro & Setup 
- Summarizations and Extractions
- GPT 3.5. Turbo vs GPT 4 5:55
- Inference and JSON Object Injection
- HTML/XSS + Data Exfiltration Scenarios 
- Expansion Prompts 


A quick reminder on why this attack is so powerful:

## Bypassing input validation

Attack payloads are natural language. This means there are lots of creative ways an adversary can inject malicious data that bypass input filters and web application firewalls.

## Leveraging the power of AI for exploitation 

Depending on scenario attacks can include JSON object injections, HTML injection, Cross Site Scripting, overwriting orders of an order chat bot and even data exfiltration (and many others) all with the power of AI and LLMs.

Thanks.