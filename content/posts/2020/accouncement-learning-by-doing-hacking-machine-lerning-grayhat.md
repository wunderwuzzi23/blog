---
title: "Coming up: Grayhat Red Team Village talk about hacking a machine learning system"
date: 2020-10-09T11:30:50-07:00
draft: true
tags: [
        "red",
        "blue",
        "ttp",
        "conference"
    ]
---

Excited to announce that I will be presenting at [Grayhat - Red Team Village](https://redteamvillage.io/) on October 31st 2020.

The presentation is about my machine learning journey and how to build and break a machine learning system. If you follow my blog, you can guess that there will be lots of discussion around "Husky AI". The bits and pieces that make up a machine learning pipeline, and how to threat model such a system.

[![Husky AI](/blog/images/2020/husky-ai.jpg)](/blog/images/2020/husky-ai.jpg)

I will talk about hands on attacks for identified threats and mitigations to improve the system. Attacks will include machine learning specific threats such as brute forcing predictions, perturbations, backdooring models, as well as more traditional red teaming style attacks to gain access to a model file for instance (e.g. via SSH agent hijacking). 

**This talk is about my learnings of practical attacks and defenses when building a machine learning system to gain a better intuition about the problem space. “Traditional” red teamers hopefully will have some good take-aways and better intuition about machine learning, and machine learning engineers about red teaming.**

There will also be discussion of GANs (Generative Adversarial Networks) which I used to create images of huskies from scratch, which end up tricking the model.

**Two more interesting things in the talk:**
1. Additionally, I will share details on the Microsoft Machine Learning security evasion competition and discuss my solution that ended up making 2nd place evading anti-virus models
2. Discussion of a CVE that I found a few months back in a popular Data Science/Developer tool - which was just fixed, so I can talk about it.

The Grayhat conference is October 29th-31st. Check out the [Grayhat website](https://grayhat.co/) for more talks, villages and details.

[![Grayhat](/blog/images/2020/grayhat.png)](https://grayhat.co)

Looking forward to seeing many of you.

Cheers.
