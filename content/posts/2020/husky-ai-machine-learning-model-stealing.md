---
title: "Machine Learning Attack Series: Gaining access to a model"
date: 2020-09-24T15:03:45-07:00
draft: true
tags: [
        "machine learning",
        "huskyai",
        "red"
    ]
---

This post is part of a series about machine learning and artificial intelligence. Click on the blog tag "huskyai" to see related posts. 

* [Overview](/blog/posts/2020/husky-ai-walkthrough/): How Husky AI was built, threat modeled and operationalized
* [Attacks](#appendix): The attacks I want to investigate, learn about, and try out

We talked about creating adversarial examples and "backdoor images" for Husky AI before. One thing that we noticed was that an adversary with model access can very efficiently come up with adversarial examples.

The goal of this post is to look for ways an adversary can gain access to a model. 

At a high level there are multiple ways, but I think they can be distinguised between "direct" and "indirect" approaches. 

1. **Direct approach: Gaining access to the actual model file:** Compromise systems and hunt for the model file.
1. **Indirect approach: Transfer Learning and Model Stealing:** Attacker builds a separat, yet similar model themselves and uses that to create adversarial examples that work against the live systems.

You might think that an indirect approach is far fetched, but to pull off certain attacks one does not need access to the real physical model file that is used by the systems.

Let's explore these two in a bit more detail.

## Direct approach: Gaining access to an actual model file

This is the most obvious way to steal a model. A perfect goal for a red team operation.

* Searching internal source code repositories for files with an `*.h5` extension. h5 is a commonly used model file format (take a look at [last weeks post about backdooring model files for reference as well](/blog/posts/husky-ai-machine-learning-backdoor-model/))
* Typical red team style attacks to gain access to engineering machines and production systems (phishing, weak passwords, exposed endpoints that allow remote management or code execution, SSH agent hijacking,...)

To keep a good balance in this blog between machine learning specific attacks and regular infrastructure attacks - let's talk about SSH agent hijacking. 

It's common to have jumpboxes or bastion hosts to access production systems. To make things convinient most setup up SSH Agent to forward SSH keys. 

ssh-add -l


 \.\pipe\openssh-ssh-agent
 https://github.com/PowerShell/Win32-OpenSSH/issues/1586

 [System.IO.Directory]::GetFiles("\\.\\pipe\\")
This is a common way I gained access to production systems with clients.

```
PS C:\WINDOWS\system32> Start-Service ssh-agent
PS C:\WINDOWS\system32>  [System.IO.Directory]::GetFiles("\\.\\pipe\\") | sls agent

\\.\\pipe\\openssh-ssh-agent
```

## Indirect approaches: Transfer Learning and Model Stealing

The second approach is less obvious for us wannabe ML red teamers. But an adversary can build a model offline to create adversarial examples and then try to use those adversarial examples against the production model to find bypasses.

Researches have shown that this is possible, so let's try it with Husky AI.

**Note about Model Stealing:** Along the same lines an adversary can steal model information by querying the external production API to get scores and labels to build a similar model themselves. In the case of Husky AI the system has a binary classifier. Stealing a model might require a lot of queries to the rate limited API endpoint. The idea would be to download thousands of husky and non-husky images and then send them to the prediction API to get the label (and confidence) values, and then build a similar model from scratch. Given the simple binary classifier for something like Husky AI this is not worth doing, its easier to build a model from scratch or use transfer learning - that also avoid running into the rate limiting issue as attacker.


Let's talk a little bit more about transfer learning and creation of adversarial images.



## Conclusion

That's it for gaining access to a model. This, in combination with backdooring attacks and creation of adversarial examples we can see how individual attacks can be fit nicely together aiding an adversary to trick a AI system.



### Appendix 

These are the core ML threats for Husky AI that were identified in the [threat modeling session](/blog/posts/2020/husky-ai-threat-modeling-machine-learning/) so far and that I want to research and build attacks for. 

Links will be added when posts are completed over the next serveral weeks/months.

1. [Attacker brute forces images to find incorrect predictions/labels](/blog/posts/2020/husky-ai-machine-learning-attack-bruteforce/) 
2. [Attacker applies smart ML fuzzing to find incorrect predictions](/blog/posts/2020/husky-ai-machine-learning-attack-smart-fuzz/) 
2. [Attacker performs perturbations to misclassify existing images](/blog/posts/2020/husky-ai-machine-learning-attack-perturbation-external/) 
3. **Attacker gains read access to the model - Exfiltration Attack (this post)**
4. [Attacker modifies persisted model file - Backdooring Attack](/blog/posts/2020/husky-ai-machine-learning-backdoor-model/)
5. Attacker denies modifying the model file - Repudiation Attack
6. Attacker poisons the supply chain of third-party libraries 
7. Attacker tampers with images on disk to impact training performance
8. Attacker modifies Jupyter Notebook file to insert a backdoor (key logger or data stealer)
