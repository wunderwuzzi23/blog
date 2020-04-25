---
title: "Mozilla Bug Bounty Credential Hunt Phabricator Token"
date: 2020-04-25T14:20:25-07:00
draft: true
tags: [
        "red",
        "ttp",
        "book",
        "bugbounty"
    ]
---

### Mozilla Phabricator Token Exposure

A few days ago I did some security research on Firefox, learning more about code and testing features. One thing I noticed during this research was that Mozilla uses *Phabricator*.

### What is Phabricator?

*Phabricator* is a collaborative web-based toolset for code reviews, checkins, bugs, work items, wiki, pastes, credentials and many other useful things. 

### API Tokens

From past work experience I know that its common for developers to create API tokens for command line usage to interact with Phabricator. There is a tool called **arcanist** that is typically used to for instance perform code checkins via ```arc land``` and so forth.

These tokens usually start with ```api-``` or ```cli-``` followed by 28 random characters.

Mozilla actively encourages security research and bounty hunting to help improve the security posture of their software and services. [Here](https://www.mozilla.org/en-US/security/bug-bounty/) an excerpt from Mozilla's bug bounty page:

> Mozilla strongly supports security research into our products and wants to encourage that research.

Equipped with this intel and knowing that Mozilla encourages security research, I performed a **targeted Phabricator Credential Hunt** for Mozilla.


### Targeted Credential Hunt

As described in an earlier post, **[Credential Hunting](/blog/posts/2020/hunting-for-credentials)** is a dedicated and **targeted activity** that I recommend performing during red teaming and penetreation testing. After gaining some basic intel into systems used, targeted credential hunting can yield great results.

Using the info and techniques from the previous post and my book, **I had found a valid Mozilla Phabricator API token within a few minutes**.

### Token Validation

I used the Phabricator **whoami** API to validate that the token is valid and which account it was impersonating.

```
curl https://*[phabricatorinstancehere]*/api/user.whoami -d api.token=*api-xxxx*
```

The response showed that the token impersonated a Mozilla employee. Being an *ethical hacker* and security professional I obviously did not attempt to perform malicious activity or exfiltrate data.

### Reporting
**At that point I promptly reported this issue to Mozilla and they fixed it basically immediatley**. They also performed forensic investigation to ensure no other misuse had occured. 

**It was very impressive to watch Mozilla's professionalism during the response.**


### The power of Credential Hunting

This is an example to show how powerful it can be to perform **active credential hunts** for your clients or corporation. Tokens often are out there but automated scanning tools might not be able to identify or validate them yet. 


## Red Team Strategies
If you liked this post and found it informative or inspirational, you might be interested in the book ["Cybersecurity Attacks - Red Team Strategies"](https://www.amazon.com/Cybersecurity-Attacks-Strategies-practical-penetration-ebook/dp/B0822G9PTM). It is filled with further ideas, research and fundamental techniques, as well as red teaming program and people mangement aspects.


## References
* [Mozilla Bug Bounty](https://www.mozilla.org/en-US/security/bug-bounty/)
* [Credentials Reference - Markdown](https://github.com/wunderwuzzi23/scratch/blob/master/creds.md)
* [Credentials Reference - CSV](https://github.com/wunderwuzzi23/scratch/blob/master/creds.csv)
* [Phabricator](https://phacility.com/phabricator)