---
title: "Assuming Bias and Responsible AI"
date: 2020-11-24T14:00:50-08:00
draft: true
tags: [
        "machine learning",
        "red",
        "strategy"
    ]
---

There are plenty of examples of artificial intelligence and machine learning systems that made it into the news because of biased predictions and failures. 

Here are a few examples on AI/ML gone wrong:

* [Amazon had an AI recruiting tool](https://www.theguardian.com/technology/2018/oct/10/amazon-hiring-ai-gender-bias-recruiting-engine) which favored men over women for technical jobs
* [The Microsoft chat bot named "Tay"](https://www.reuters.com/article/us-microsoft-twitter-bot-idUSKCN0WQ2LA) which turned racist and sexist rather quickly
* [A doctor at the Jupiter Hospital in Florida](https://gizmodo.com/ibm-watson-reportedly-recommended-cancer-treatments-tha-1827868882) referred to IBM's AI system for helping recommend cancer treatments as "a piece of sh*t"
* [Facebook's AI got someone arrested for incorrectly translating text](https://mashable.com/2017/10/24/facebook-auto-translation-palestinian-man-arrest-israel/)


The list of AI failures goes on... 

If you built and tested AI yourself you know how powerful, yet brittle machine learning can be. 


## Assume Bias by Design

In the computer network security realm, the industry has adopted an "Assume Breach" mindset. Assume Breach highlights that we should assume (internal and trusted) systems are compromised. 

Getting the thinking of "secure zones" or impenetrable machines out of the way allows to focus on an important question, which is:

> Your company is compromised. What are you going to do about it?

This does not mean that one should not invest in protecting systems, but one must think beyond protection alone. Red teaming, threat hunting, detection and response are critical components of an Assume Breach security strategy.

We must apply a similar thinking when it comes to AI/ML systems:

> Your AI has biases, and it will make incorrect predications. What are you going to do about it?

Architects and designers of AI/ML systems need to embrace an "Assume Bias" mindset and implement mitigation, testing, detection, and response (including deprecation) strategies for AI systems.

![Assume Bias](/blog/images/2020/assumebias.jpg)

For instance, a system which makes critical decisions should not solely depend on predictions of a machine learning system. The risk tolerance might vary depending on use cases. 

With that mindset, we can start thinking of ways to mitigate all sorts of "fails" (*I am using bias in a wider context*) in order to build responsible AI. 

Bias might not just enter a machine learning model by accidents or negligence, it might also be introduced by an adversary via an attack ==> **Hello, AI Red Team**!

"Your model has biases. What are you going to do about it?" is a question that I will add to my threat modeling repertoire.


## Alignment and Control

The AI Control Problem "is the issue of how to build a super intelligent agent that will aid its creators and avoid inadvertently building a superintelligence that will harm its creators." -- [Wikipedia, Nov 2020](https://en.wikipedia.org/wiki/AI_control_problem)

There are "alignment approaches" which attempt to align AI behavior with human values, and then there are also "capability controls".

It seems that a combination of both will be needed to succeed: Teaching AI/ML proper ethics and values, which is already difficult as human values differ quite a bit, and biases one way or another seem inevitable. At the same time AI and its creators must be held accountable via laws, policies, and treaties. 

## Conclusion

There is a lot unpack with an "Assume Bias" mindset for AI and ML. Hopefully the basic idea resonates to explore this in more depth. Also in what it means for doing "AI Red Teaming". 

And I am curious around your thoughts around this as well.

Cheers.
Twitter: [@wunderwuzzi23](https://twitter.com/wunderwuzzi23)


## References

* [Amazon ditched AI recruiting tool that favored men for technical jobs](https://www.theguardian.com/technology/2018/oct/10/amazon-hiring-ai-gender-bias-recruiting-engine)
* [IBM Watson Reportedly Recommended Cancer Treatments That Were 'Unsafe and Incorrect'](https://gizmodo.com/ibm-watson-reportedly-recommended-cancer-treatments-tha-1827868882)
* [Learning from Tay’s introduction](https://blogs.microsoft.com/blog/2016/03/25/learning-tays-introduction/)
* [AI control problem](https://en.wikipedia.org/wiki/AI_control_problem)
* [Facebook's AI arrest](https://mashable.com/2017/10/24/facebook-auto-translation-palestinian-man-arrest-israel/)
