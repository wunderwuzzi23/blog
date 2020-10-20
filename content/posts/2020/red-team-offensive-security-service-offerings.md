---
title: "What does an offensive security team actually do?"
date: 2020-10-19T15:11:33-07:00
draft: true
tags: [
        "pentesting",
        "red",
        "program"
    ]
---

There is a lot of discussion around terms such as red team, attack team, pentest, adversarial engineering or offensive security team and similar ones.

I typically stay away from the (sometimes passionate) discussions that ensue whenever this topic comes up.

Personally, I think a good strategy is to define programs and teams who operate in this space by **what services the team (or teams) provide(s) to the organization**. 

The business groups, blue team, developers, engineers, employees and clients are the customers. 

## Defining an offensive security program

What an offensive security, red team or pentest team does can vary quite a bit - depending on an organization, there is no one size fits all.

It might or might not include design level work and reviews such as threat modeling, but it certainly should include hands-on offensive security testing and finding and exploiting of vulnerabilities for defensive purposes. Most of these service offerings revolve around alternative analysis.

> A smaller organization might have one (red) team that does everything, and large organizations might have multiple different teams with varying service offerings and responsibilities.

I generally stay away from trying to define "Red Teaming" in any official form, because that by itself is counter productive. A red team defines itself whatever it wants to be or do.

And, if you tell yourself that what I just did is define red teaming, then congratulations: you are thinking like a red teamer! 

Personally, you'll see me often use the terms "attack team" and "offensive security", rather than red team.

Here is a list of set of services such a team might provide to an organization:

* [Security Reviews and Threat Modeling Support](#reviews)
* [Security Assessments](#assessments) 
* [Red Team Operations](#redteam) 
* [Purple Team Operations](#purple)
* [Tabletop Exercises](#tabletop)
* [Research and Development](#research)
* [Predictive Attack Analysis and Incident Response Support](#attackanalysis)
* [Security Education and Training](#training)
* [Increase the Security IQ of the organization](#iq)
* [Gathering Threat Intelligence](#ti) 
* [Informing Risk Management Groups and Leadership](#GRC)
* [Integration into Engineering Processes](#engineering)

Obviously the agenda can include other aspects that are not highlighted here as well.

Let's discuss them in more detail.

### Security Reviews and Threat Modeling Support{#reviews}

A good way to get the offensive security team involved early is in the design phase of a system. It’s the best way to get feedback before code is deployed or operational processes are established. Although it’s not unlikely that systems are already deployed, it’s still worthwhile to catch up and threat model systems, environments and people. Some offensive teams might object to be included in this stage as it differs slightly from their mission. 

Personally, I have always seen this as one of the biggest assets of having an internal offensive security team. When engineers or others in the organization have specific security questions on how to build a certain feature or develop a process to improve security the pen test team can be a great group to bounce off ideas and help improve the security early on. When teams across the organization directly reach out to your team for advice, then you must have done something right.

### Security Assessments{#assessments}

An engineering team might develop a new feature or service and requests the help from the penetration test team to assess its security posture and potential vulnerabilities. These are more focused on application level vulnerability assessments with the goal to find as many issues as possible using techniques include white and black box testing and so forth. Some classify this as doing the classical penetration test.


### Red Team Operations{#redteam}

Some of the most fun things for pen testers can be true red team type of work. Typically, these are covert operation where involved stakeholders are not aware of the test being performed, and the operation is authorized by leadership. Ideally, the offensive security team defines objectives, gets approval and executes towards it.

Depending on the maturity level of a red team it might be possible to emulate very specific adversaries to challenge the blue team. This can reach from emulation of a specific adversary or advanced persistent threat (APT) to simulating a crypto-currency adversary or performing a physical breach of a building. 

Red Teaming is fun and creative – there are "no" rules.

The biggest challenge for a mature red team is that a true adversary will break the law. A red team does have to consider legal and corporate policies when operating. This of course has implications on how realistic certain scenarios can be played through – but certain scenarios should be at least played through on paper via tabletop exercises.

### Purple Team Operations{#purple}

The scope and goals are very similar to red team defined operations. The core difference to a red team operation is that the focus lies on transparency and collaboration between red, blue and engineering teams. The goal is throughout all stages of the purple team operation to improve the security posture of a system pretty much immediately by running attacks and validating detections and alerts. If attacks succeed and are not caught, detections are fixed and implemented, and attacks are re-run right away again - until there is measurable improvement.

Purple Teaming is one of the most effective ways to help grow your defenses fast and help improve the maturity of the organization quickly. Especially if you do have an internal offensive security team that can work with the blue team throughout.

Although make sure to keep challenging your own processes and beliefs. The idea of offensives security and alternate analysis is to challenge the status quo.

The reason to not only perform purple teaming but mix in red teaming is to ensure someone is validating the outside attack without limitations. If most of the organization only does purple teaming, the need to hire someone external to red team the organization increases. This becomes the test of the purple team so to speak, to see if they were successful with improving the security posture of the organization.  

### Tabletop Exercises{#tabletop}

At times its not possible or feasible to perform certain attack scenarios operationally. This might be due to available resource, legal and/or technical concerns. A good alternative can be to red team scenarios on paper, rather then operationally perform them in real life. Tabletop exercises can be a great way to get higher leadership and the board involved in exploring attack scenarios.

### Research and Development{#research}

This category falls mainly into two buckets, the first being security and vulnerability research. This is a core priority of an offensive security team. It includes research to find new vulnerabilities and new vulnerability classes. In addition, the team also develops tools and exploits for defensive purposes to highlight implications of vulnerabilities, for use during operations and to test the countermeasure and mitigations being put in place work.

### Predictive Attack Analysis and Incident Response Support{#attackanalysis}

In case a security incident occurs, the offensive security team can assist with insights on what the active adversary might be looking for. The team possibly can predict the next step the adversary will take due to their unique attacker mindset. The kudos for the term "Predictive Attack Analysis" go to [Farzan Karimi](https://twitter.com/_BlackRook) by the way. This is part of the homefield advantage that an internal offensive security team can provide and during emergencies the team can provide critical information to be one step ahead. 

## Additional Responsibilities of the Offensive Program

### Security Education and Training{#training}
The offensive security team can tremendously help change the culture of the organization and help improve the overall security IQ. As part of operations the pen testers learn a lot about people, processes and technologies of the organization. The offensive team is also in a powerful position to ignite cultural change and help the organization improve their unique understanding of security.

### Increase the Security IQ of the organization{#iq}

Along the lines of education and providing training the job of the offensive program should be to improve the security IQ of the entire organization, including blue teams, service and product teams, human resources, finance, etc.

### Gathering Threat Intelligence {#ti}

One role the offensive program might fill is the task to gather threat intelligence to understand current trends in offensive security and what threat actors are active and what new techniques, tools or processes threat actors are building or leveraging at the moment.  

Especially in a smaller organization that does not have a dedicated threat intel program it will be the red team's job to be up to date with the latest trends, threats and knowing about what data related to the organization flows around in the dark web.

### Informing Risk Management Groups and Leadership{#GRC}

Another area to ensure involvement is to shape and actively contribute to the risk management process of the organization. Information security threats might not be correctly considered when risk stakeholders discuss the risks the business faces. 

Numbers like "machines not patched" does not mean anything to a business stakeholder – what is the risk to the business?

The offensive program can provide insights to malicious activity that can results in critical business impact. Additionally, an offensive program can highlight process flaws where too many people have unvetted access to information or capabilities that could accidently impair the business negatively and lasting due to human error without malicious intent.

The security industry is focused on qualitative measurements and security scores. More meaningful ways to express risks are needed - refer to my [Monte Carlo Simulations post for some alternative ideas](/blog/posts/red-teaming-and-monte-carlo-simulations/).


### Integration into Engineering Processes{#engineering}

It is advisable that the red team program integrates and has regular checks with engineering and other relevant stakeholders for evaluation. If such a collaboration does not exist, it's time to work on it. The lack of visibility is often why vulnerabilities which could have been discovered and mitigated early on make it into production. Smaller organizations may need an engagement once per year, while a large commerce apps may benefit from multiple assessments per year.

Such integration ensure that regular security assessments are performed, and that the security team can plan more complex red teaming operations that leverage and integrate the newest systems and services built to provide the best value.

Another idea in this regard is to require a recurring offensive drill for each business group.

## Conclusion

That's it. Hope this was a somewhat interesting excursion into what a red team might be doing.

Twitter: [@wunderwuzzi23](https://twitter.com/wunderwuzzi23)
