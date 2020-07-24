---
title: "Motivated Intruder - Red Teaming for Privacy!"
date: 2020-07-24T10:00:16-07:00
draft: true
tags: [
        "pentesting",
        "red",
        "privacy"
    ]
---

In this post I will discuss some testing techniques for internal red teams to identify privacy issues in services and infrastructure, most importantly a simple three step approach that might uncover interesting results. 

## Background story 

First, let me share a story from the past. When I did my master's I built an app that performs end to end encryption of Facebook posts. This means that only the intended audience for which your posts were encrypted for can decipher the posts. 

The project was called Project Cryptbook, and I found an old screnshot again:

[![Project Cryptbook](/blog/images/2020/cryptbook.png)](/blog/images/2020/cryptbook.png)

On the left you see the UI in the app which shows the clear text. And on the right, you see what everyone else, including Facebook sees. 

That was about 10+ years ago - interesting how time flies. Facebook replaced the "wall" with an ad-driven timeline and Windows Phone is dead.

Anyhow, I have always been a fan of privacy, data protection and giving user's control over their data. 

## Red Teaming for privacy!

When years later I oversaw planning and performing large red team operations I thought quite a bit on how to help organizations improve their understanding of privacy threats with practical excersices. It became even more interesting when new regulations started kicking in.

**Does your organization have procedures in place to regularly assess systems and infrastructure for privacy violations?**

Article 32 of the [General Data Protection Regulation (GDPR)](https://eur-lex.europa.eu/legalcontent/EN/TXT/?qid=1528874672298&uri=CELEX%3A32016R0679) highlights the requirement and necessity to test regularly when processing:

> "a process for regularly testing, assessing, and evaluating the effectiveness of technical and organizational measures for ensuring the security of processing."

Even for privacy minded folks and organizations, there is little information available on how to perform privacy testing nor is there much literature out there (to my knowledge). Chances are you might have not even thought about privacy focused testing before.

[![GDPR Logo](/blog/images/2020/gdpr.png)](https://gdpr.eu/)

Let's dive into the threats of data and motivated intruder.

## Data Intruder Threat

Privacy violations are often insider threats. Employees who have access to production and PII data can easily exfiltrate large amounts of data or be coerced into doing so by an adversary.

In more mature organizations such access is only granted on a case-by-case basis and temporarily for support, debugging or when security incidents arise. And access to the systems and data records is *(should be)* monitored thoroughly. 

### Easy access to customer data?

How difficult is it for a developer, a marketing person, support personnel, a data scientist, or someone from HR to access customer data?

**Are access requests possibly auto-approved!?!? **

The data intruder threat is important as more and more of the world's data is being stored in the cloud. Insiders indeed exploit their privileges at times, for instance: [Twitter (2020)](https://www.reuters.com/article/us-twitter-cyber-access-exclusive-idUSKCN24O34E), [Facebook (2018)](https://www.nbcnews.com/tech/social-media/facebook-investigating-claim-engineer-used-access-stalk-women-n870526) and [Google (2010)](https://techcrunch.com/2010/09/14/google-engineer-fired-security/).

How is the situation in your organization?

Ideally, customers should be notified whenever employees or contractors access their information through support, maintenance, or bug fixes. That would set a high standard for ethics in the industry.

What safeguards are in place to prevent anyone from downloading the customer database, copying it onto a USB device to exfiltrate, sell the information, or take it home when departing the company? 


## Motivated Intruder

Furthermore, NIST highlights the **Motivated Intruder** as being an internal adversary who has access to **anonymized datasets and uses data science techniques to [reidentify and deanonymize the data](https://csrc.nist.gov/csrc/media/publications/sp/800-188/archive/2016-08-25/documents/sp800_188_draft.pdf)**.


![Hacker](/blog/images/2020/hacker.png)

I have been using the term **Motivated Intruder** in a broader sense for any privacy focused red teaming operations throughout my career.

## Getting started with privacy focused testing 

There are some basic ways to test and identify privacy violations in your organization.  

Here are a couple ideas for operations, such as:
 
1. **Identifying customer data in widely accessible locations**. For various reasons PII might end up accidentally in widely accessible telemetry, source code, logs, event streams, shared email inboxes, data science notebooks,...
2. **Simulate a Data Intruder who can legitimately gain access to customer data for business purposes** and exfiltrate sentinel records to validate monitoring solutions
3. **Exploit a system vulnerability** or weakness to gain access to customer data
4. Perform **de-anonymization attacks** - teaming up with data scientists during red team operations

In this post I will be focusing on the first two scenarios.

Discuss with compliance and legal teams before engaging with them to ensure you have a well-rounded and authorized test plan in place.

## Scenario 1: Identifying customer data in widely accessible locations 

I am a fan of creating sentinel datasets and have the red team find and exfiltrate those as objective. 

![background](/blog/images/2020/background.filler.png)

### What is a sentinel record?

A sentinel data record is basically a test record, often with an easily identifiable pattern.

Sentinel data records are useful for privacy focused testing. One way to get basic end-to-end test coverage is to use your own systems as an external user:

1. **Data creation with unique patterns** - Sign up with a unique and easily identifiable pattern as PII
2. **Exercising features** of the system under test - Then exercise and use all the features of the system to trigger various workflows internally
2. **Hunting for patterns in internal systems** - Finally, switch hats and search internally in widely accessible places for that **unique pattern**

### How does this work practically? 

Let's say your organization runs a service that allows users to register with their first name, last name, address, and credit card, and users can upload text and comments. Let's walk through a test scenario step by step.

### Step 1: Data creation with unique, easily identifiable patterns

In a practical fashion, this could be as simple as the following: 

* As username, pick **John55444562200** **Tester55444562200** 
* The address could be **Test Road55444562200**
* Credit card contains **55444562200** as well
* Phone number contains  **55444562200**
* The email address contains it, for instance **johntester+55444562200**@example.org

**You get the idea!** 

### Step 2: Exercising features 

After creating test data, use the system and its features. Consider triggering side conditions and error cases to get a wide range of test cases. Upload/post/edit/delete information in the same using the pattern **55444562200**. 

There is also the opportunity to automate and leverage already existing test frameworks to become more repeatable and have better test coverage. 

### Step 3: Hunting phase

Afterward, it is time to hunt for the *specific* string pattern - in this case for **55444562200**. 

This means searching various data stores and widely accessible information repositories for the pattern that used during signup. 

![Searching](/blog/images/2020/lookfor.png)

At this stage there is **no** need to actively compromise systems - ideally you just define a scope of systems to analyze upfront. Searching widely accessible **telemetry stores** and **supposedly anonymized datasets in data lakes** is a good place to start. 

Some surprise hits might also be visible in places like **internal source code repositories** - maybe a data scientists checks in customer data when building or analyzing models, and of course our very own **security monitoring solutions** might contain the pattern as well. Other places are **Intranet portals**, **file servers** and **O365/SharePoint/GSuite**.

Using this simple technique, it's possible to raise awareness and identify potential privacy violations and improve the overall security posture. It also helps protect the organization from certain data intruder threats.

**Note:** You can also do a privacy hunt without creating sentinel records. For instance searching for regular expression of phone numbers, email addresses or credit card numbers can help identify sensitive datasets in widely exposed locations pretty quickly.

### Scenario 2: Malicious insider and privacy implications 

When troubleshooting issues and debugging purposes, as well as security investigations, it is likely that engineers in your organization can request access to production machines where they might be exposed to customer data. In less mature organizations the process might be even simpler - and the question is if auditing is taking place. This is an important angle to validate as a red team.

![Numbers](/blog/images/2020/number.png)

What if someone legitimately gains that level of access to debug a performance problem but at the same time exfiltrates customer data on a production system? 

Would the blue team detect that? Is there enough logging in place to figure out what happened? 

The actual data to exfiltrate can again be sentinel production records. This is to avoid having the red team touch real customer data – which is always preferable. Although, as a safety pre-caution red teamer must have proper authorization if they get exposed to customer data during testing (and yes, red teams have to be audited also). Discuss your testing objectives and approach with your legal counsel to ensure its legitimate and authorized.

**In the end, I think that an organization has to engage in privacy testing to help protect customer data and to demonstrate adhering to privacy regulation.**

## Conclusion

In this post we discussed how Red Teams can help identify privacy violations by performing motivated intruder testing. We also learned about the Data Intruder and walked through some basic test techniques to help identify and minimize risk associated with accidently leaking customer data.

Hope this was interesting and useful. If you kick off such tests in your organization, I'd be curious to learn about findings or other approaches and ideas you might have.

Feel free to follow or message me on Twitter: [@wunderwuzzi23](https://twitter.com/wunderwuzzi23)


## References

* [De-identifying Government Datasets - Motivated Intruder](https://csrc.nist.gov/csrc/media/publications/sp/800-188/archive/2016-08-25/documents/sp800_188_draft.pdf)
* [More than 1,000 people at Twitter had ability to aid hack of accounts, 2020](https://www.reuters.com/article/us-twitter-cyber-access-exclusive-idUSKCN24O34E)
* [Facebook investigating claim engineer used access to stalk woman, 2018](https://www.nbcnews.com/tech/social-media/facebook-investigating-claim-engineer-used-access-stalk-women-n870526)
* [This is the second time Google fired an engineer for accessing user data, 2010](https://techcrunch.com/2010/09/14/google-engineer-fired-security/)
