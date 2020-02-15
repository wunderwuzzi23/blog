---
title: "Security Principles Revisited"
date: 2020-02-12T17:03:32-08:00
draft: false
tags: [
        "strategy",
        "principles",
        "web"
    ]
description: "Security Principles Revisited. Web Application Security Prinicples, Bachelor of Science"
meta_title: "Security Principles Revisited"

---

## Final Year Project - Web Application Security Principles
About 18 years ago I worked on the final year project for my Bachelor's degree in Computer Science. I had just gotten interested in security and was learning about security principles. 

The title of the project was **["Web Application Security Principles - Designing Secure Web Based Enterprise Solutions"](/blog/papers/Web_Application_Security_Principles.pdf).**

Looking back, a really cool thing was that I had just started working at Microsoft as an Associate Development Consultant and was bold enough to send the paper last minute over to **Michael Howard** - who responded and indeed reviewed it! That was so cool! :) 

And later when I started working at Microsoft in Redmond, I was so thrilled to meet him in person. And he used a few bugs I had found during some of his famous SDL presentations.


## Two big things that we did not know

It was quite difficult to find the paper again, but I was able to dig it up... It is amazing looking back and reading the work again now. Even though threats like Cross Site Scripting and SQL Injection attacks and of course "basic" buffer overflows were already very well known and researched. What caught my eye right away is that 18 years back for web apps, we did not know about:

* Cross Site Request Forgery

* Server Side Request Forgery

Indeed, there are of course  other vulnerabilities and vulnerabiliy classes that weren't as well known, but those two stood out to me immediately.

## The Value of Security Principles

Skimming over the the document, one reference in the paper that stood out to me again is:

> "A comprehensive security strategy first requires a high level recognition of overall Security Principles"
(Romanosky, 2002)

Anyhow, the reason for me to go back and read up on is to get back to some of the basics. In the modern world, with fast paced changes CI/CD pipelines, DevOps and "code velocity" it's worth taking a step back and look at the bigger picture from a security point of view. 

I'm a strong believer in following core security principles to guide throughout the daily decision process as engineer. Below is a comparison chart from the paper (remember this is 2002), and it overall highlights 17 principles:

![Comparison of Security Principles (2002](/blog/images/security_principles_comparison.png)

Going forward I will make serious of posts about some of the core principles to try to distill them down to a handful. At the same time I'll expore examples throughout my career that show why following such simple guidance pays off in the long run with typically little investment in the short term - e.g. by doing some basic threat modeling of the system.

## Zero Trust and Assume Breach
Interestingly, **Zero Trust** was already present via attack surface reduction, compartmentalization, assume external systems are insecure, and be reluctant to trust. It was just that few people (even today) follow these principles, especially when it comes to network security. 

**Assume Breach** (one of my favorite ones these days as red teamer) was less clearly defined back then. I'd imagine to find a **assume failure** principle, but there is only fail securely.

## Conclusion

Overall however, we just have to stick to the basics and apply/adjust the thinking to new scenarios.

If you are interested to go back in time and  look at my final year project from 2003(!), here it is:

[Final Year Project - Web Application Security Principles](/blog/papers/Web_Application_Security_Principles.pdf)


Cheers,
Johann.