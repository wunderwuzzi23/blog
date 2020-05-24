---
title: "Phishing metrics - what to track?"
date: 2020-05-24T00:26:01-07:00
draft: true
tags: [
        "red",
        "metrics"
    ]
---

The results of phishing campaigns are often not comparable with each other over time. Various security vendors and red teams use different tooling and techniques - which is totally fine. 

I recommend **tracking a minimum set of metrics to be able to compare results over time**.

*Funny side facts: At times employees are messing with the red team, entering invalid creds for CISO or CEO and things along those lines. Some employees (often engineers) are curious and open the mail in isolated VMs to debug and explore the phishing site.* 

**These side effects make phishing stats that don't track various stages precisley less meaningful.**

[![KoiPhish](/blog/images/2020/koiphish-logo.png)](/blog/images/2020/koiphish-logo.png)

**A phishing campaign should track the following data points at a minimum to provide insights:**

1. **Were security controls artificially weakened?** Yes/No - some vendors might require to be whitelisted, a red team (hopefully) tries to circumvent controls.
1. **Number of users the phishing email was sent to**
1. **The bounce rate** (were any of the mails outright rejected)
1. **Number users who opened/viewed the email** (rough approximation via a tracking pixel)
2. **Number of users who clicked the link**
3. **Number of users who entered *valid* username** (must match some identifier in email to filter out, and identify, trolling)
4. **Number of users who who entered a password**
5. **Number of users who entered their valid(!) password**
6. **Number of users who entered MFA/code or submitted a push notification** - more variants possible, can be split out by MFA method
7. **Number of users who's auth cookies got stolen (full compromise)**
8. **Number of users who reported the phish**

Its best to also provide the ratio for each of the metrics. Without sharing the details its at times difficult to compare phishing results. If you do it in purple teaming fashion you can get more precise information around delivery and who viewed/reported/deleted it.

**Interesting fact: What I have seen is that the number of users entering their valid password and those that complete the MFA flow on the phishing site is often nearly the same - more then 90% of users who entered their password, will also complete the MFA login flow on the phishing site.**

Clients are at times still surprised that the a full acconunt compromise is possible, even though MFA is in place. Deployment and adoption of security keys (usage of WebAuthN and FIDO2) is less well understood. We need to continue advocating for broader adoption.

---

[![KoiPhish](/blog/images/2020/koiphish.png)](/blog/images/2020/koiphish.png)

For phishing campaigns [WUNDERWUZZI, LLC](https://www.wunderwuzz.net) uses a variant of [KoiPhish](https://github.com/wunderwuzzi23/KoiPhish).