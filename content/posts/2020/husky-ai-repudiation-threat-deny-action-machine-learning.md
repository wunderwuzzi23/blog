---
title: "Machine Learning Attack Series: Repudiation Threat Deny Action Machine Learning"
date: 2020-10-08T05:50:21-07:00
draft: true
tags: [
        "machine learning",
        "huskyai",
        "red",
        "ttp"
    ]
---

This post is part of a series about machine learning and artificial intelligence. Click on the blog tag "huskyai" to see related posts. 

* [Overview](/blog/posts/2020/husky-ai-walkthrough/): How Husky AI was built, threat modeled and operationalized
* [Attacks](#appendix): The attacks I want to investigate, learn about, and try out

In this post we are going to look at the "repudiation threat", which is one of the threats often overlooked when performing threat modeling.

Repudiation means that someone denies having performed an action. Its a "human threat", machines do not repudiate - unless AI takes over one day. :) 

In the case of Husky AI a good example is the attacker Mallory replaced the original machine learning model file with a backdoored one, as we had done in the previous post of this series.

How can we add proper logging and auditing to better understand who performed an action, and how can we have proof who indeed updated the file, or at least support an investigation to help put the pieces together to uncover the truth --> and beachhead/initial compromise of the breach.

A pracital way is to leverage `auditd` in Linux, and push log files up into a central monitoring system. Companies typically have a product such as Splunk, the Elastic Stack, or Azure Sentinel to name a few systems that can help analyze and visualize the logging data.

To keep things scope, I will discuss the basic setup to make sure change to the model file create a proper audit event.


## auditd

```
sudo apt install auditd
```

```
ausearch -r
```



## Conclusion

That's it for discussing the repudation threat around modifying the model file (but also other chnges to the system).

Hope it was interesting.

Cheers.

### Appendix 

These are the core ML threats for Husky AI that were identified in the [threat modeling session](/blog/posts/2020/husky-ai-threat-modeling-machine-learning/) so far and that I want to research and build attacks for. 

Links will be added when posts are completed over the next serveral weeks/months.

1. [Attacker brute forces images to find incorrect predictions/labels](/blog/posts/2020/husky-ai-machine-learning-attack-bruteforce/) 
2. [Attacker applies smart ML fuzzing to find incorrect predictions](/blog/posts/2020/husky-ai-machine-learning-attack-smart-fuzz/) 
2. [Attacker performs perturbations to misclassify existing images](/blog/posts/2020/husky-ai-machine-learning-attack-perturbation-external/) 
3. [Attacker gains read access to the model](/blog/posts/2020/husky-ai-machine-learning-model-stealing.md/) 
4. [Attacker modifies persisted model file - Backdooring Attack](/blog/posts/2020/husky-ai-machine-learning-backdoor-model/)
5. **Attacker denies modifying the model file - Repudiation Attack (this post)**
6. Attacker poisons the supply chain of third-party libraries 
7. Attacker tampers with images on disk to impact training performance
8. Attacker modifies Jupyter Notebook file to insert a backdoor (key logger or data stealer)


## References

* Image "Access Denied" by [Elchinator from Pixabay](https://pixabay.com/photos/no-access-access-denied-monitor-5043758/)
