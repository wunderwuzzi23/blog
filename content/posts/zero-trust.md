---
title: "Zero Trust and Disabling Remote Management Endpoints"
date: 2020-02-06T14:08:55-08:00
draft: true
tags: [
        "strategy",
        "principles"
    ]
description: "Disabling remote management endpoints (SSH, WinRM, SMB, RDP, VNC,..) by default across your corp infrastructure."
meta_title: "Limiting lateral movement. Zero Trust. "

---

This post highlights a simple mitigation to improve the security posture of your organization. The idea is to, by practical means, limit attack surface and prevent spreading of automated malware, as well as limiting lateral movement by adversaries.

## Network security over the last 15 years

Malware can spread fast and damage businesses at scale. 

SQL Slammer [1] and WannaCry [2] are two well-known cases that showed how quickly and damaging this can be. Interstingly, both of these disasters were nearly 15 years apart.

>Did we not learn how to defend against automated attacks in the last 15 years?

**The network is hostile** and that is something security experts agree upon and keep highlighting. 

## Corporate IT Security 

**Zero Trust** is typically used to refer to the fact that the network infrastructure cannot be trusted. However, remote management endpoints (and others also) are constantly **permanently** exposed to untrusted networks. Especially corporate devices (such as laptops and phones) are often exposed to quite hostile networks.

>To this day, malware, adversaries and red teams move through environments via remotely exposed endpoints, most of them time using dedicated admin management endpoints. 

In your company the IT department might be permanently enabling SSH (or WinRM on Windows) on your laptop. This allows to connect and assert administrative privileges on your (and other employee's) machines whenever the need arises. 

## Why is this bad?

Here are a few reasons why permanently exposing remote management endpoints is bad:

1. **Manual Access:** There is little control on **who**, from **where**, or **when**. Basically anyone can easily pivot to the endpoint, and when possessing credentials (which adversaries always do) the machine can be taken over (including all passwords and secrets stored on it)

3. **Bruteforce and Password Spray:** The machine now is open to bruteforce attacks (remember, the network is hostile).

4. **Bruteforce attacks in the coffee shop:** Just to reiterate and highlight the exposure more. An employee might be in a coffee shop or airport (again, the network is still hostile) and if remote management endpoints are exposed, the employee's laptop will likely be automatically attacked

5. **Spreading of Automated Malware:** Malware spreads throughout the entire environment from one machine to the next within seconds. 

6. **Enormous Blast Radius:** Imagine malware that steals credentials, then connects to the next machine and steals credentials again, then the next one, and so forth. Such automated attacks are rather scary and easy to implement for an adversary. A red team can perform such an blast radius analysis - starting from a single leaked credential. Ask your red team to perform such a test.

## Are remote management endpoints really needed?

Probably not - at least not permanently and widely. To manage machines in a corporate environment there are technologies such as group policies and Chef that help manage things at scale.

## Mitigations

It is a frustrating thing for a red teamer to have a valid password (certificate,...) but not being able use it because there is no endpoint exposed. 

>**So, we should assume it is similarly frustrating for a real world adversary!**

One of the simplest mitigations that prevents malware from spreading is to **not expose remote endpoints** (especially remote management endpoints) by default, hence, ensure that endpoints are not exposed widely, and look into:

* Disabling SSH
* Disabling RDP
* Disabling WinRM/PowerShell, WMI, DCOM, SMB (yes, disable SMB, the red team and adversaries will not like you for this)
* Disabling VNC (and Apple Remote Management)
* Disabling Telnet (yeah, that still shows up often)

Interestingly, by default these ports are typically locked down by the Operating System vendors.

If you want to be more drastic (full zero trust so to speak), consider to block all inbound traffic in general by default. 

Anything that allows to run admin commands should be turned off for sure by default. This includes a lot of other products and services that can be found out there (like mssql, some web apps,...).

And here are a few other things to remember:

* Only temporarily expose remote management endpoints
* When exposing them, expose them to a subset of machines (and identities) that has a business need to connect
* Strong passwords, MFA, Certificates, Smart Cards
* Do not reuse local admin passwords (this is common in Corp IT land, especially for a local admin and root accounts)

### Testing for Exposure

The intersting thing is that we often do not realize that remote endpoints are exposed on our machines. Hence, I suggest to regularly perform port scans on your own devices to understand there exposure.

## Exceptions
There are reasons that a subset of employees might want to have these features enabled (and they can turn them on and lock them down if needed). For instance developers might want RDP or SSH to remotely connect to a second machine for debugging and so forth. But this is not the most common scenario in an organization! Engineers can enable these scenarios easily, including locking machines down to subset of source addresses.

There might also be other technical or business reasons at times, but the stance should be to default to a locked down state unless absolute necessary.

## References

[1] https://en.wikipedia.org/wiki/SQL_Slammer

[2] https://en.wikipedia.org/wiki/WannaCry_ransomware_attack
