---
title: "Machine Learning Attack Series: Overview "
date: 2020-11-26T09:00:51-08:00
draft: true
tags: [
        "machine learning",
        "red",
        "huskyai"
    ]
---

What a journey it has been. I wrote quite a bit about machine learning from a red teaming/security testing perspective this year. It was brought to my attention to provide a conveninent "index page" with all Husky AI and related blog posts. Here it is.

![ML Attack Series](/blog/images/2020/ml-attack-series.jpg)

## Machine Learning Basics and Building Husky AI

* [Getting the hang of machine learning](/blog/posts/2020/machine-learning-basics/)
* [The machine learning pipeline and attacks](/blog/posts/2020/husky-ai-walkthrough/)
* [Husky AI: Building a machine learning system](/blog/posts/2020/husky-ai-building-the-machine-learning-model/)
* [MLOps - Operationalizing the machine learning model](/blog/posts/2020/husky-ai-mlops-operationalize-the-model/)

## Threat Modeling and Strategies 

* [Threat modeling a machine learning system](/blog/posts/2020/husky-ai-threat-modeling-machine-learning/)
* [Grayhat Red Team Village Video: Building and breaking a machine learning system](https://www.youtube.com/watch?v=-SV80sIBhqY)
* [Assume Bias and Responsible AI](/blog/posts/2020/machine-learning-attack-series-assume-bias-strategy/) 


## Practical Attacks and Defenses

* [Brute forcing images to find incorrect predictions](/blog/posts/2020/husky-ai-machine-learning-attack-bruteforce/) 
* [Smart brute forcing](/blog/posts/2020/husky-ai-machine-learning-attack-smart-fuzz/) 
* [Perturbations to misclassify existing images](/blog/posts/2020/husky-ai-machine-learning-attack-perturbation-external/) 
* [Adversarial Robustness Toolbox Basics](/blog/posts/2020/husky-ai-adversarial-robustness-toolbox-testing/)
* [Image Scaling Attacks](/blog/posts/2020/husky-ai-image-rescaling-attacks/)
* [Stealing a model file: Attacker gains read access to the model](/blog/posts/2020/husky-ai-machine-learning-model-stealing/) 
* [Backdooring models: Attacker modifies persisted model file](/blog/posts/2020/husky-ai-machine-learning-backdoor-model/)
* [Repudiation Threat and Auditing: Catching modifications and unauthorized access](/blog/posts/2020/husky-ai-repudiation-threat-deny-action-machine-learning/)
* [Attacker modifies Jupyter Notebook file to insert a backdoor](/blog/posts/2020/cve-2020-16977-vscode-microsoft-python-extension-remote-code-execution/)
* [CVE 2020-16977: VS Code Python Extension Remote Code Execution](/blog/posts/2020/cve-2020-16977-vscode-microsoft-python-extension-remote-code-execution/)
* [Using Generative Adversarial Networks (GANs) to create fake husky images](/blog/posts/2020/machine-learning-attack-series-generative-adversarial-networks-gan/)

## Miscellaneous

* [Participating in the Microsoft Machine Learning Security Evasion Competition - Bypassing malware models by signing binaries](/blog/posts/2020/microsoft-machine-learning-security-evasion-competition/)



## Conclusion

As you can see there are many machine learning specific attacks, but also a lot of "typical" red teaming techniques that put AI/ML systems at risk. For instance well known attacks such as SSH Agent Hijacking, weak access control and widely exposed credentials will likely help achieve objecives during red teaming operations.

Hope the content is helpful and maybe even inspiring for others to start building, breaking and better protecting AI/ML systems. 

Reach out if there are specific topics you would like me to cover, or if you have any feedback. Also, if you enjoyed reading about this series, I'd appreciate a note. :)

Stay safe, Johann.

Twitter: [@wunderwuzzi23](https://twitter.com/wunderwuzzi23)


PS: Don't forget to check out [Cybersecurity Attacks - Red Team Strategies](https://www.amazon.com/gp/product/1838828869/ref=as_li_tl?ie=UTF8&tag=wunderwuzzi-20&camp=1789&creative=9325&linkCode=as2&creativeASIN=1838828869&linkId=07bfd6b729fbc2b2904160e0e16c337f) for more red teaming goodies.
