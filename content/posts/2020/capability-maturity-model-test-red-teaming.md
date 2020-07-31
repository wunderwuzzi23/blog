---
title: "Illusion of Control: Capability Maturity Models and Red Teaming"
date: 2020-07-31T12:08:00-07:00
draft: true
tags: [
        "red","book","kpi","philosophy"
    ]
---

Throughout my career I have been fascinated with quality assurance and testing, especially security testing and red teaming. One discussion that comes up frequently is how to measure the maturity of such programs and processes. 

My answer is straight forward as there are already existing frameworks that can be leveraged, adjusted and borrowed from to fit the needs of offensive security programs.

You are likely familiar or have at least heard of the **Capability Maturity Model Integration** from Carnegie Mellon University. In particular [CMMI](https://en.wikipedia.org/wiki/Capability_Maturity_Model_Integration) defines five levels to measure software engineering processes as follows:

1. Initial
2. Mangaged 
3. Defined
4. Measured
5. Optimized


**If you run an internal pen test program, the program is somewhere along those 5 stages.** 

There is one caveat which we will go into more detail in this post when introducing **Level 6 - Illusion of Control**, which I argue is missing from the existing frameworks.

The question arises how does the Capability Maturity Model apply to testing systems in particular? 

Luckily, there is already a Test Maturity Model which puts the perspective of software testing in the CMM model. It is called **Test Maturity Model integration**, or short [TMMi](https://www.tmmi.org/tmmi-model/).

## Test Maturity Model integration (TMMi®)

The Test Maturity Model Integration developed by the TMMi Foundation explores and defines a framework for measuring test maturity and process improvements with detailed examples.

**Let's explore how this can be applied to offensive security testing and red teaming.**

The following image shows the five stages as defined by TMMi®, and we will put those in context for offensive security in this post.

![TMMi Model Picture](/blog/images/2020/TMMi-model-picture.png)

*Image from taken from https://www.tmmi.org/tmmi-model*

Now, let us put some more content behind what these stages mean for offensive security. The following is an attempt to describe and bucketize the stages for offensive security needs:


## Level 1: Initial

There are no defined processes in place at this stage. Finding defects or vulnerability is by pure chance and often considered as part of debugging. Or security issues are reported mainly by customers.

## Level 2: Managed

The philosophy that vulnerabilities can be prevented is the majority opinion within the organization, hence at this stage the focus of investments is mainly in the preventive bucket, applied only at the end of the development cycles or before deployment. 

Maybe once a year a penetration test is performed to find vulnerabilities. 

The efforts might be driven and revolving around compliance requirements, and not yet focusing on pushing the broader security maturity forward. Some form of Blue Team is established, and response processes are being built. 

Offensive security and penetration testing are performed at times, but no dedicated organization wide efforts are implemented.

If already present at this stage, red and blue team have a pure adversarial stance and are not collaborating besides handing pen test reports over to the other side.


## Level 3: Defined

This is the stage where an organization **introduces an internal offensive security team** with a broad agenda, including cyber operational red teaming. The **offensive security testing process is established**, and the organization realizes not all vulnerabilities can be found and addressed and investments are done regarding detection of breaches and incident response. 

This includes creation of Rules of Engagement, Operating Procedures, Issue Tracking, Engagement Management, Cleanup and Debriefs - there are lot of [useful details and guidelines in my book by the way in regards to the creation of an internal offensive security program](https://www.amazon.com/Cybersecurity-Attacks-Strategies-practical-penetration-ebook/dp/B0822G9PTM).

Centralized security event aggregation is implemented to enable better response and hunting of adversaries. This is the typical stage an internal offensive security simulates malicious insiders and adversaries. Or the organization might hire external vendors regularly to perform red team operations in addition to regular security assessments.

Organizationally a formalized Security Operation Center to drive detection and incident response is established.


## Level 4: Measured

With the establishment of a Security Operation Center and formalized processes for incident response, the organization starts to attempt to measure its progress and identify areas for improvement.

In case the organization has multiple penetration test teams (yes, large organization have multiple) this is the stage they will collaborate and share information to create centralized view and help peer review each other’s offensive findings and work to improve and find issues across organization boundaries. 

At this stage **KPIs are introduced** to help find gaps and weaknesses in the program and services to help improve the environment. Leadership will be briefed regularly on the magical identified KPIs that are seemed the most important ones to help make progress in those areas.

This could be as simple as highlighting the critical issues identified by offensive security teams and their status and proposed resolutions.

The organization invests in getting the “big picture” and a ton of security intelligence about state of systems is being aggregated, and that security monitors being present on all knows systems is being tracked.

## Level 5: Optimized

In the optimized stage the organization continuous to improve and fine-tune the testing process, a test process improvement group is established officially in the organization. Another important factor is that processes are automated as much as possible and helpful. 

Putting this in the realm of offensive security, this would be the stage where an organization would likely **create a catalog of TTPs and attacks and associated mitigations specific to the organization**, consuming TTPs from others in the industry. As well as implementing automated testing using a framework like the MITRE ATT&CK matrix which can greatly help understand the realm of attacks the organization might face.

Reaching this stage means that things and are supposed to be predictable and that’s where the big risk lies when it comes to security.

Those are the five stages; however I suggest introducing a Level 6 - Illusion of Control as follows.


## Level 6: Illusion of control – the red team strikes back

Personally, I suggest adding another level of maturity, I have been calling it **Illusion of Control**. 

If an organization reached the stage of “Optimized” the key stakeholders in the organization, including the test process improvement group could be self-validating their own views. This could lead to being blindsided to other trends in the industry or novel attack areas that are being missed due to group think and normalization of deviance.

The idea of Level 6 is to introduce chaos and **stand up an entirely new red team** – because likely a critical piece of the puzzle is incorrect and the way to improve if to start from scratch without any assumptions, strict processes and insider knowledge. 

**In the end, the Illusion of Control is what red teaming is about.**


## Next steps

Given the info above it should be pretty obvious where your company's offensive security program is at currently. 

An interesting next step will be to define more precise descriptions for each level to furthermore objectively measure which level an organization fits in. Although, this might be a rabbit hole and more paperwork then the use we get out of it - but it might be helpful in the long run.

## Conclusion

In this post we looked at the CMM model for testing and how it can be used to more objectively define the maturity of offensive security programs. We also introduced an additional level called "Illusion of Control" to reflect the true nature of what red teaming is about.

Hope this was interesting and useful.

There are many more examples and practical ideas on how to measure red teaming in my book about [Red Team Strategies](https://www.amazon.com/Cybersecurity-Attacks-Strategies-practical-penetration-ebook/dp/B0822G9PTM).

Feel free to follow or message me on Twitter: [@wunderwuzzi23](https://twitter.com/wunderwuzzi23)


## References

* [Capability Maturity Model Integration, CMU](https://en.wikipedia.org/wiki/Capability_Maturity_Model_Integration)
* [Test Maturity Model integration (TMMi®)](https://www.tmmi.org/tmmi-model/)
