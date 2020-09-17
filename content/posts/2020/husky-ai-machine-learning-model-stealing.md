---
title: "Machine Learning Attack Series: Gaining access to a model"
date: 2020-09-20T15:03:45-07:00
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

The previous posts covered bruteforcing and perturbing pixels in order to come up with new images that trick the externally husky prediction API to misclassify images. Half-way through the experiments we realized that rate limiting of the prediction endpoint was indeed making life a little difficult for an adversary. 

**An adversary with model access can be most efficient in coming up with perturbations.**

The goal of this post is to look for ways an adversary can gain access to a model. At a high level I'd say there are two ways to explore this:

1. **Transfer Learning:** Attacker builds a seperate, but similar model offline. The attacker uses that model to build adversary examples. This was pointed out in papers, like the fast gradient sign method paper we talked about in the last post.
2. **Stealing**: Performing queries with many images to build our own offline dataset for training. In the case of Husky AI this is super simple, and not worth exploring further.
2. **Gaining access to the actual model file:** This is a typical goal for a red team operation. The idea is to look for models that are stored insecurely, or possibly actively compromising infrastructure to gain access to model files.

Let's explore these two, with a focus on the first one, as that is machine learning specific.

## Transfer Learning

For attacking Husky AI I thought it should easily be possible to take something like `ImageNet` which is a pre-trained model from millions of pictures and repurpose it so we can generate adversarial husky examples that will trick our real Husky AI model.


## Gaining access to an actual model file

* Searching internal source code repositories for files with an `*.h5` extension. h5 is a common model file format. We will learn more about this next week when we modify such a file to add a backdoor!
* Typical red team style attacks to gain access to engineering machines (phishing, weak passwords, exposed endpoints that allow remote management or code execution, SSH agent hijacking,...)



## Conclusion

That's it for the first round of attacks. I hope you enjoyed reading and learning about this as much as I do. I learned a lot already and am eager to dive learning smarter ways of coming up with malicious/adversarial examples.

Hopefully, you'll join more for the upcoming posts, where we will start backdooring model files!

Cheers,
Johann.


### Appendix 

These are the core ML threats for Husky AI that were identified in the [threat modeling session](/blog/posts/2020/husky-ai-threat-modeling-machine-learning/) so far and that I want to research and build attacks for. 

Links will be added when posts are completed over the next serveral weeks/months.

1. [Attacker brute forces images to find incorrect predictions/labels](/blog/posts/2020/husky-ai-machine-learning-attack-bruteforce/) 
2. [Attacker applies smart ML fuzzing to find incorrect predictions](/blog/posts/2020/husky-ai-machine-learning-attack-smart-fuzz/) 
2. [Attacker performs perturbations to misclassify existing images](/blog/posts/2020/husky-ai-machine-learning-attack-perturbation-external/) 
3. **Attacker gains read access to the model - Exfiltration Attack (this post)**
4. Attacker modifies persisted model file - Backdooring Attack
5. Attacker denies modifying the model file - Repudiation Attack
6. Attacker poisons the supply chain of third-party libraries 
7. Attacker tampers with images on disk to impact training performance
8. Attacker modifies Jupyter Notebook file to insert a backdoor (key logger or data stealer)
