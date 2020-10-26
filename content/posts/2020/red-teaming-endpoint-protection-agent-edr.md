---
title: "Leveraging the Blue Team's Endpoint Agent as C2"
date: 2020-10-26T06:00:00-07:00
draft: true
tags: [
        "red","blue",
        "ttp",
        "c2"
    ]
---

A few years back the Blue Team of a company asked to be targeted in a Red Team Operation. 

That was a really fun, because Rules of Engagement commonly prevent targeting Blue Teams. Blue's infrastructure, systems and team members are often out of scope, unfortunately.

> Blue team infrastructure is a gold mine for credentials, recon but also for remote code execution!

Often companies do not have adequate protection, procedures (MFA, multi-person attestation), monitoring and auditing in place when it comes to accessing data from endpoint agents. There is also frequently a lack of knowledge on what Endpoint Agents are capable of doing. 

**So, what better way to compromise the blue team via their own tooling I thought? :)**

![Attack](/blog/images/2020/binary.jpg)

This post is a reiteration of a section in Red Team Strategies about focusing on EDR and endpoint agents, as I still think this threat needs a lot more attention.

## Endpoint Agents as C2

Endpoint Protection and Response (EDR) agents and solutions like Carbon Black, Crowdstrike's Falcon, Windows Defender ATP and so forth commonly provide built-in Command & Control infrastructure. 

The agents are typically managed via a web interface, so tactics such as ["Pass the Cookie"](https://attack.mitre.org/techniques/T1550/004/) might be performed by adversaries after compromising blue team members.

Obviously this can be misused by an adversary. 

Once an adversary has access to the Web UI, there are features such as "Go Live", or "Live Response" that allow access to any host in the organization where the EDR agent is running.

**No kidding!**

Using this technique is quite elegant, as it does not require the red team to maintain or run their own infrastructure. 


## Products and Examples

There is a wide range of products, but they mostly converge around a common feature set, including establishing interactive shells and command execution on hosts. Here are some examples and details from vendors:

* [Windows Defender ATP - Initiate Live Response Session](https://docs.microsoft.com/en-us/windows/security/threat-protection/microsoft-defender-atp/respond-machine-alerts#initiate-live-response-session)

    > "Live response is a capability that gives you instantaneous access to a device by using a remote shell connection." 

* [Crowdstrike - How to Remotely Remediate an Incident](https://www.crowdstrike.com/blog/tech-center/remote-remediation-real-time-response/)

    > "With the ability to run commands, executables and scripts, the possibilities are endless."
    > [![FalconD](/blog/images/2020/RTR-gif.gif)](/blog/images/2020/RTR-gif.gif)

* [Carbon Black - "Go Live"](https://www.carbonblack.com/blog/screenshot-demo-carbon-black-live-response-in-action/) 
    > With Live Response, Carbon Black gives you a terminal right in our Web console and via the already-existing sensor on the endpoint. You just click “Go Live” and you’re in.

During red team operations I have used some of these to get quick root access on arbitrary hosts.

**Fun times!**

## Are these features really needed?

To be fair, for some investigations a blue team member might indeed find a manual shell useful to grab certain log files or other evidence. Although, that should really be an exception - as tasks should be automated as much as possible. 

But, it's also worth noting that these are cloud systems. This means that your blue team is not the only one with access. The vendor might also directly or indirectly have access to this capabilitiy. The attack surface is quite large.

If these risks are acceptable to your business depends on your organization.

## So, why should the Red Team test for this?

A couple of things can be highlighted with such an operation:

1. **Detection and Monitoring**: Does the Blue Team and SOC have proper detections and monitoring in place to catch compromise or misuse? 
2. **Security Controls**: Are proper security controls in place for the EDR web portal (MFA, multi-person attestation before "going live"), etc.
3. **Auditing**: Are the commands that the blue team member (or attacker) is running logged?
4. **Raising awareness**: Interesting questions might come up as well, for instance does the IT team know about this "backdoor" access to the fleet?
5. **Who watches the watchers?** Requiring rigorous monitoring of these features and performing drills to validate they are not misused is part of the red teams goals


Hope this provides some ideas on how to validate the infrastructure and procedures of the blue team is in good shape.

Cheers.
[@wunderwuzzi23](https://twitter.com/wunderwuzzi23)


## References

* [Crowdstrike - How to Remotely Remediate an Incident](https://www.crowdstrike.com/blog/tech-center/remote-remediation-real-time-response/)
* [Defender ATP - Initiate Live Response Session](https://docs.microsoft.com/en-us/windows/security/threat-protection/microsoft-defender-atp/respond-machine-alerts#initiate-live-response-session)
* [Carbon Black Go Live](https://www.carbonblack.com/blog/screenshot-demo-carbon-black-live-response-in-action/)