---
title: "Leveraging the Blue Team's Endpoint Agent as C2"
date: 2020-10-26T08:00:00-07:00
draft: true
tags: [
        "red",
        "ttp",
        "c2"
    ]
---

A few years back the Blue Team of a company asked to be targeted in a Red Team Operation. 

That was a really fun, because Rules of Engagement commonly prevent targeting Blue Teams. Their infrastructure and systems, which are a big Achilles' heel in many organizations, are often out of scope.

Unfortunately many companies do not have adequate protection, procedures (MFA, multi-person attestation), monitoring and auditing in place in these areas. There is often also a lack of knowledge on what Endpoint Agents are capable of doing. 

> Blue Team infrastructure is a gold mine for credentials, recon but also for remote code execution!

**So, what better way to compromise the blue team via their own tooling I thought? :)**

![Attack](/blog/images/2020/binary.jpg)

This post is a reiteration of a section in Red Team Strategies about focusing on EDR and endpoint agents, as I still think this threat needs a lot more attention.

## Endpoint Agents as C2

Endpoint Protection and Response (EDR) agents and solutions like Carbon Black, Crowdstrike's Falcon, Windows Defender ATP and so forth commonly provide built-in Command & Control infrastructure. 

Obviously this can be misused by an adversary. 

The "beauty" of this is that the red team does not even have to maintain or run their own infrastructure.

## Perform an EDR focused Red Team Operation

Your red team should run an operation to gain administrative access to the portal of these systems.

The portals are pretty much always web based, so tactics such as ["Pass the Cookie"](https://attack.mitre.org/techniques/T1550/004/) might be performed by adversaries after compromising blue team members. 

Once an adversary has access to the Web UI, there are features such as "Go Live", or "Live Response" that allow access to any host in the organization where the EDR agent is running.

**No kidding!**


## Examples for Endpoint Agents

Here are some examples and information from vendors of EDR:

* [Windows Defender ATP - Initiate Live Response Session](https://docs.microsoft.com/en-us/windows/security/threat-protection/microsoft-defender-atp/respond-machine-alerts#initiate-live-response-session)

    > "Live response is a capability that gives you instantaneous access to a device by using a remote shell connection." 

* [Crowdstrike - How to Remotely Remediate an Incident](https://www.crowdstrike.com/blog/tech-center/remote-remediation-real-time-response/)

    > "With the ability to run commands, executables and scripts, the possibilities are endless."
    > ![Attack](/blog/images/2020/RTR-gif.gif)

* Carbon Black - During Red Team operations I have also used Carbon Black's "Go Live" feature to get quick root access on arbitrary hosts with this technique. 

Fun times!

## Is this really needed?

To be fair, for some investigations a blue team member might indeed find a manual shell useful to grab certain log files or other evidence. 

Although, that should really be an exception - as tasks should be automated as much as possible. The goal for the red team is to ensure procedures are in place to catch misuse or identify lack of security controls.

Additionally, organization need to worry about malicious insiders (blue team members) who might be taking advantage of these features.

## So, why should the Red Team test for this?

A couple of things can be highlighted with such an operation:

1. **Detection and Monitoring**: Does the Blue Team and SOC have proper detections and monitoring in place to catch compromise or misuse? 
2. **Security Controls**: Are proper security controls in place for the EDR web portal (MFA, multi-person attestation before "going live"),etc.
3. **Auditing**: Are the commands that the blue team member (or attacker) is running logged?
4. **Who watches the watchers?** Requiring rigorous monitoring of these features and performing regular drills to validate they are not misused is something the red can help with.

Hope this provides some ideas on how to validate the infrastructure and procedures of the blue team is in good shape.

Cheers.
[@wunderwuzzi23](https://twitter.com/wunderwuzzi23)


## References

* [Crowdstrike - How to Remotely Remediate an Incident](https://www.crowdstrike.com/blog/tech-center/remote-remediation-real-time-response/)
* [Defender ATP - Initiate Live Response Session](https://docs.microsoft.com/en-us/windows/security/threat-protection/microsoft-defender-atp/respond-machine-alerts#initiate-live-response-session)