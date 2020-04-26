---
title: "Hunting for Credentials!"
date: 2020-04-24T23:31:29-07:00
draft: true
tags: [
        "red",
        "ttp",
        "book"
    ]
---

The idea of searching or tricking victims to surrender their credentials is nothing new. 

Adversaries are leveraging these techniques to gain access to sensitive information. At times the term "harvesting credentials" is used which is something that appears to be more opportunistic and I would **propose that security teams start to actively hunt for credential exposure that can put their organiztaion at risk** -- in case you are not yet doing that.

## Actively hunting for credential exposure

The idea of **credential hunting is targeted and focused**, leveraging intelligence about systems and combing it with powerful search techniques to identify exposure. 

Use the [homefield advantage](https://wunderwuzzi23.github.io/blog/posts/homefield-advantage/) and recon information to perform the best possible credential hunts. 


## Credential Hunting Cheat Sheet

First, I just want to share my little cheat sheet for credential hunting. I also have a *csv* version available on my github (see the references section on bottom): 
![Credentials - Cheat Sheet](/blog/images/2020/hunting-for-credentials-cheat-sheet.png)

It's nothing fancy just a reminder on what kind of information to look for and build into tooling. 

### VHD Files

**Bonus points:** Hunting for virtual machine disk image files can be very fruitful in environments. They often contain loads of password hashes and other sensitive data. 

## Building a public credential reference catalog
Ideally, we could build a comprehensive credential reference catalog together for the industry, so tooling for each type of credential can be built and is comparable.

Let me know if this is something you are interested in building a catalog for.


## References
* [Credentials Reference - Markdown](https://github.com/wunderwuzzi23/scratch/blob/master/creds.md)
* [Credentials Reference - CSV](https://github.com/wunderwuzzi23/scratch/blob/master/creds.csv)
