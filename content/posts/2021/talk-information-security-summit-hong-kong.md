---
title: "Hong Kong InfoSec Summit 2021 Talk - The adversary will come to your house!"
date: 2021-03-03T11:37:20-08:00
draft: true
tags: [
        "red", "conference", "兵法"
    ]
---

Next week (on March, 9th 2021) I will be speaking at the Hong Kong Information Security Summit 2021. 

歡迎, 你好! 

[![HK Summit 2021](/blog/images/2021/hk-summit.png)](https://www.issummit.org/)

I was invited to share my thoughts around protecting the modern (and remote) workplace. Of course, my talk is addressing this topic from a red teaming point of view. Conference details are [here](https://www.issummit.org/).

## The adversary will come to your house  

The name of the talk is "Red Team Strategies for Helping Protect the Modern Workplace" which might seem less creative, but there is some (hopefullly) good and interesting information in my talk.

![Adversary Haus](/blog/images/2021/haus.png)

In retrospect I would call the talk something more catchy, like "The adversary will come to your house!", since one of the key messages is that in the modern workplace adversaries will commonly end up compromising personal assets and home infrastructure.

### LinkedIn Compromise

Writing this blog post reminded me of the LinkedIn compromise of 2012, where a Russian hacker compromised a LinkedIn employee's home network and then brute forced his way to personal machines which had LinkedIn company SSH keys on them. 

This allowed the adversary to gain access to LinkedIn's internal network and subsequently gain access to millions of customer data records. 

If you are interested in more details about the LinkedIn compromise, then check out [this DarkNet Diaries session](https://darknetdiaries.com/episode/86/).

## The Modern Workplace

With the modern workplace personal infrastructure compromises will become more and more common. Your home network is "just" collateral damage when it comes to gaining access to company resources. This means that establishing network isolation between your personal and company assets is a good idea when working from home.

**I think that companies should educate their employees on how to protect and isolate their personal assets from company assets when working from home.**


## Key Strategies

Besides the fact that adversaries will more often have access to people's personal stuff when targeting companies. The core of my presentation will focus on 3 topics an organization should look into:

* Zero Trust
* Assume Breach
* Homefield Advantage

Some cool buzz words. Yay! I am sure you heard of  **Zero Trust** and **Assume Breach**, if not my talk will give you a great 5-10 minute intro. 

![Survivor](/blog/images/homefield-advantage.png)

Unless you read my book (or follow this blog), you might not yet have heard of **Homefield Advantage**, which is a strategic way to improve the effectiveness of a internal red teaming program. 

Finally, I will dive more into **Survivorship Bias**, the MITRE ATT&CK Matrix for Mobile.

![Survivor](/blog/images/2021/mitre-attack-survivorship-bias.png)

Overall some great content packed into this presentation I think. Hope many of you check out the talk.


知己知彼, 百戰不殆.

Cheers,
[@wunderwuzzi23](https://twitter.com/wunderwuzzi23)


## References

* [Hong Kong InfoSec Summit 2021](https://www.issummit.org/)
* [DarkNet Diaries - LinkedIn Compromise](https://darknetdiaries.com/episode/86/)
* [Survivorship Bias and Red Teaming](https://embracethered.com/blog/posts/2021/survivorship-bias-and-red-teaming/)
