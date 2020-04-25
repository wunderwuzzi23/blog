---
title: "Hunting for Credentials"
date: 2020-04-24T23:31:29-07:00
draft: true
tags: [
        "red",
        "ttp",
        "book"
    ]
---

## What does "Hunting for Credentials" mean?

The idea of searching or tricking victims to surrender their passwords is nothing new. Adversaries and pen testers are leveraging this tactics to gain access to sensitive information.

## Actively hunting for credential exposure

Below is my little cheat sheet for credential hunts as seen in my book. I also have a *csv* version available on my github (see the references section on bottom): 
![Credentials - Cheat Sheet](/blog/images/2020/hunting-for-credentials-cheat-sheet.png)



# VHD Files

I will post about this seperaatley, but hunting for virtual machine disk image files can be very fruitful in environments. They often contain amounts of password hashes and other sensitive data.

## Red Team Strategies
If you liked this post and found it informative or inspirational, you might be interested in the book ["Cybersecurity Attacks - Red Team Strategies"](https://www.amazon.com/Cybersecurity-Attacks-Strategies-practical-penetration-ebook/dp/B0822G9PTM). It is filled with further ideas, research and fundamental techniques, as well as red teaming program and people mangement aspects.

## References
* [Credentials Reference - Markdown](https://github.com/wunderwuzzi23/scratch/blob/master/creds.md)
* [Credentials Reference - CSV](https://github.com/wunderwuzzi23/scratch/blob/master/creds.csv)
* [Phabricator](https://phacility.com/phabricator)