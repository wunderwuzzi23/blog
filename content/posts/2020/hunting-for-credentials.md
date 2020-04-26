---
title: "Hunting for credentials and building a credential type reference catalog"
date: 2020-04-26T12:31:29-07:00
draft: true
tags: [
        "red",
        "ttp",
        "book",
        "credentials"
    ]
---

Adversaries are leveraging widely exposed clear text credentials to gain access to sensitive information. 

At times the term *"harvesting credentials"* is used when red teamers emulate these attacks - which is something that appears to be more opportunistic and I would **propose that security teams start to actively hunt for credential exposure that can put their organization at risk** -- in case you are not yet doing that.

## Actively hunting for credential exposure

The idea of **credential hunting is targeted and focused**, leveraging intelligence about systems and combing it with powerful search techniques to identify exposure. 

Use the [Homefield Advantage](https://wunderwuzzi23.github.io/blog/posts/homefield-advantage/) to perform the best possible credential hunts. 


## Credential Hunting Reference Sheet

Below is a reference sheet that I use for credential hunting during pentests. There is also a *csv* version available on my github (see the references section on bottom): 
[![Credentials - Cheat Sheet](/blog/images/2020/hunting-for-credentials-cheat-sheet.png)](/blog/images/2020/hunting-for-credentials-cheat-sheet.png)

It's nothing fancy just a reminder on what kind of information to look for and build into tooling to make sure they are not exposed widely, or accessible by adversaries. 

### VHD Files

**Bonus points:** Hunting for virtual machine disk image files can be very fruitful in environments. They often contain loads of password hashes and other sensitive data. 

# Building a public credential reference catalog

**A goal would be to build a comprehensive open credential type reference catalog together for the industry. So tooling (for discovery and validation) for each type of credential can be built and is comparable with each other across tools and vendors.**

Let me know if this is something you are interested in building or working together on.

## References
* [Credentials Reference - Markdown](https://github.com/wunderwuzzi23/scratch/blob/master/creds.md)
* [Credentials Reference - CSV](https://github.com/wunderwuzzi23/scratch/blob/master/creds.csv)