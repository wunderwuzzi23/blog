---
title: "Somewhere today a company is breached"
date: 2021-06-09T08:30:00-07:00
draft: true
tags: [
        "red",
        "program"
    ]
---

This rather lengthy post goes into reasons for having an offensive security program and in particular, on how a red team can help improve the immune system of your organization. This is the high-level outline of the post:

- [Security breaches cannot be entirely prevented](#security-breaches-cannot-be-entirely-prevented)
- [Implications of a breach](#implications-of-a-breach)
- [Automated malware can hit your organization at any time](#automated-malware-can-hit-your-organization-at-any-time)
- [Security investments - run as fast as you can, just to stay in place](#security-investments)
- [The immune system of your organization](#the-immune-system-of-your-organization)
- [Embracing the red](#embracing-the-red)


With regular cadence companies are compromised and suffer breaches. Organizations do not realize a compromise until many days, months, sometimes even years later.

# Security breaches cannot be entirely prevented

One of the challenges building out an internal offensive security program is to get buy-in and support from leadership. 
Often there is a lack of awareness of threat landscape, including assumption that breaches can be prevented, and that investing in protective measurements alone is enough to secure the organization. 

![/blog/images/2021/hacked.jpg](/blog/images/2021/hacked.jpg)


This philosophy and thinking is misguided. It reflects a defensive and immature way to look at security and how to manage risks. Given enough resources (or by chance) an adversary will succeed in compromising targets.

An organization that focuses solely on protective measurements will find themselves in a situation of not detecting a breach themselves. Or if detected, there is no clear understanding on how to investigate or recover from such an incident. 

> Offensive security reduces uncertainty around the state of the security of an organization. Therefore, an offensive security program is perfectly positioned to inform the risk management process to help make informed business decisions.

If a security program is built on preventive measures alone it, the inability to effectively respond to a breach falls flat and the adversary potentially even stays entirely undetected.

## Identifying security vulnerabilities

Most organization realize the need for penetration testing. Typically, it is forced upon them via compliance requirements. But there is a entire offensive security world beyond that has to be explored.

Less mature organizations do not have a holistic red teaming strategy across the organization, including internal and external attack surface, people, processes, and technologies. 

> Consider that your organization is breached right now – chances are fairly high that it is while you are reading this. 

Recently I talked with a security engineer of a large telecommunication provider who must creatively work around internal policies, because it is only allowed to perform penetration testing during maintenance windows. 

This limits the effectiveness and realism a real adversary has. It reminded me of the earlier days in my career facing similar issues when proposing offensive ideas and testing techniques to leadership.

The longer your organization does not have a security event, the more you should entertain the idea that there might be a larger blind spot. An adversary might already have established a foothold, but is undetected.

# Implications of a breach

Consider that your organization is breached right now – chances are fairly high that it is while you are reading this. 

Many breaches stay entirely unnoticed. An employee might click a phishing email and their computer is compromised or their password stolen. 

There is a wide range of attacks an adversary might perform, including data theft, leverage compromised resources for their own gain, sit and wait, disrupt, or destroy the data or service offerings. 

## What is the core business of your organization?

A good way to analyze business risks and put them in a security context is to ask the business owners what the most existential risks to the core of their business are. 

The business owners see risks from different perspectives and good security engineers highlight and put security threats into an understandable business context. 

A business owner does not understand the meaning or risk of a legacy server not being patched. In discussions with stakeholders, one might hear that a system is **legacy** to downplay a risk. There are no legacy systems, if a system is in use and processes data or is connected to other infrastructure it is not legacy.

![/blog/images/2021/ransomware.jpg](/blog/images/2021/ransomware.jpg)

Ransomware has become quite common over the last several years. Colonial Pipelines for instance, or take the outbreak [WannaCry Ransomware](https://en.wikipedia.org/wiki/WannaCry_ransomware_attack) was a great reminder how vulnerable the ecosystem and organizations are. 

WannaCry leveraged an already patched vulnerability to compromise hosts and encrypt data to hold for ransomware, it spread to infect other reachable hosts which caused a networking effect. Thousands of organizations were impacted, and for many it had drastic impact on their day-to-day operations. 

The National Health Service Hospitals in the England and Scotland had computers infected, other impact organization included Renault, Nissan and countless others. 

Overall, it has been reported that at least 200.000 computer systems were compromised across pretty much every country on earth. 

The WannaCry ransomware (or the recent [Exchange Server vulnerabilities](https://www.fireeye.com/blog/threat-research/2021/03/detection-response-to-exploitation-of-microsoft-exchange-zero-day-vulnerabilities.html)) show that organizations do not patch their (internal) infrastructure fast enough, and as a result many were breached and could not operate for days. And they ended up in news reports damaging the corporate image and trust. 

More even more ransomware attacks have made the news, such as [Colonial Pipeline Attack](https://en.wikipedia.org/wiki/Colonial_Pipeline_cyber_attack) and other similar attacks. Information about attacks can be read daily in the news for quite a while.

A preventable vulnerability, like an unpatched machine or a weak password can lead to drastic implications for the overall business.


# Automated malware can hit your organization at any time

The idea of malware spreading automatically is well-known and the industry has seen successful large-scale malware like this before.

For instance, the [SQL Slammer Worm](https://en.wikipedia.org/wiki/SQL_Slammer) comes to mind which caused major service disruption across the entire Internet when it started spreading end of January 2003. It infected tens of thousands of machines within minutes.

**It is worth highlighting that the worst case was by far not reached or attempted so far.**

Technically an adversary can create malware is fully automated and autonomous. Compromising, credential discovery and pivot from one target destination to another entirely automatically.

> Imagine malware that infects your computer. Afterwards it automatically reads all passwords and cookies stored on your machine and uses that information to navigate to other systems, including your WiFi router, phone, social media accounts, online backups, your bank account and so forth. 

Certain threat actors are also becoming more courageous. Rather than performing targeted attacks they compromise as many targets as possible via automation or supply chain attacks. Becoming collateral damage is quite likely these days for an organization.

The implications and damage to individuals and the industry is already quite disastrous and further malware automation will make this worse. Luckily WannaCry and more recent malware have not attempted fully automated and autonomous attacks and let’s hope something like this won’t happen anytime soon.

Hoping that this won’t happen is probably not the best strategy though.

# Continous security investments

The industry has made great progress and leveraging security measures like encryption, multi-factor authentication, encryption, FIDO, etc. and all are there to help improve the security posture and protect our information. 

Despite this, many organizations are not correctly prioritizing investments to help protect themselves from real world threats. Lewis Carroll already knew about these security challenges:

> "My dear, here we must run as fast as we can, just to stay in place. And if you wish to go anywhere you must run twice as fast as that." (Alice in Wonderland)

This phrase frequently pops into my head when discussing security threats, triaging vulnerabilities, and considering long term investments and strategic design decisions. 

![/blog/images/2021/coins.jpg](/blog/images/2021/coins.jpg)

To effectively secure systems today and in the future, active measures must be taken. 

Your organization must stay ahead of the curve and ensure that systems are being updated and patched, and that breaches can be detected and mitigated. 

Just staying in place works against you. A great example is cryptography: 

- If your organization is encrypting data right now, what are you doing to ensure that five years from now you will be able to upgrade the old data to newer algorithms? 
- Is your organization considering the cost and making the right design decision today to ensure a seamless upgrade will be possible? 
- Will you decrypt and re-encrypt, or will you just add a newer layer of encryption on top of it? 
- How will you store the metadata about what algorithm has been used? 
- How will the certificates and keys be stored? If you are using TLS are you supporting perfect forward secrecy?

Your adversaries have a seemingly easy task. All a current (or a future adversary) must do is find an unpatched system or one exposed vulnerability, while the defenders need to protect from all and every possible threat, even those that are not yet known! 

> Staying in place in the security race will render your organization vulnerable. If you are in an acceptable shape today, a year from now you are not.

What can an organization do to tackle these challenges, to identify weaknesses and highlight areas for investment to actively protect the business? How can it be measured?

# The immune system of your organization

Imagine it is that time of the year, when the weather is getting dark and cold and to prevent getting sick you start wearing a coat, scarf, some nice warm pants, and comfy shoes. A nice woolen hat and some gloves will help to keep your warm in those cold and dark winter days. 

In case it gets extremely cold we put on additional measures like a face mask and wear additional base layers to help our body store heat. 

![/blog/images/2021/tea.jpg](/blog/images/2021/tea.jpg)

These are all great preventive measures to help protect our body from external threats. Besides clothes you likely also enjoy some hot chocolate, or some refreshing and activating tea to warm up. Having a healthy diverse diet with lots of vitamins helps also. 

People get vaccinated to ensure active protection against possibly falling sick during times of elevated threat conditions. Additionally, people cover their coughs (and wear masks - as we all did the last 12 months). 

These measures are taken to support our immune system to keep us and our communities healthy. In the end though, we still might get sick and, in that case, our immune system will take additional measures and concerted and concentrated efforts to ensure a speedy recover. 

The immune system has amazing capabilities for preventing, responding, and recovering from attacks. Our white blood cells are actively scanning cells for malicious or cancerous infections. In the case they encounter a threat they can kill off these cells as necessary to prevent disease or infections from spreading. 

They also help with active defense to ensure our immune system better adapts to real world threats and viruses. To boost the immune system, it helps if we expose ourselves to real world threats. 

![/blog/images/2021/immune_system.jpg](/blog/images/2021/immune_system.jpg)

Kids that play in the dirt improve their immune system to build up the strength to fight of the real adversary. Many organizations invest in firewalls, intrusion prevention systems, anti-virus detection, spam filters, with the assumption that a breach is preventable. 

All those actions are useful to build a strong security posture. Despite preventive measures (like hats, gloves, vitamins, masks, etc..) some threats will still get your organization sick.

This analogy can be applied to computer systems and networks. Exposure to threats helps to actively defend systems and to improve our defensive capabilities. We can build up the strength to fight off real adversaries and their attacks should the situation arise. 

# Does your organizations possess an effective immune system? 

Do you regularly challenge your defense and detection capabilities to ensure it can withstand real world threats before they impact your business?

The takeaway of this analogy is that as much as we would like to be able to prevent a breach, it is not possible. 

Organizations must change the threat model they operate with to embrace the mentality that preventive measures will fail and that we have to strengthen our ways and remedies on what happens while and after a breach occurs. How will you manage risks knowing that you are breached?

The defenders in most organization are referred to as the blue team. They take on the task to protect the organization from adversaries and breaches. 

They are there to protect, monitor and respond to security events. Typically, there is a core team of responders that work shifts to provide 24/7 coverage. 

> Effective defense is difficult to achieve without having resources dedicated to testing, measuring and improve the organization’s immune system, and this is where an Offensive Security Program comes into the picture.

In extension the blue team gets help and support from everyone else in the organization. So, in many ways in the blue team is the immune system of a healthy organization. 

## Changing the threat model

A modern threat model must assume that the organization or certain systems have already been breached. 

![/blog/images/2020/binary.jpg](/blog/images/2020/binary.jpg)

The idea is that a breach cannot be prevented and rather than focusing all your resources on the prevention side, one should invest similarly with the mindset that a breach has already occurred and that there are compromised assets in the organization. 


## Zero Trust, Assume Breach and Homefield Advantage

This is even more important if you are not just founding a company but work for an organization that has been operating for many years with focus on preventive or perimeter defenses only.

-  **Zero Trust** highlights that perimeter defense by itself is not an effective security measure. Google introduced the BeyondCorp security approach after their perimeter was breached. BeyondCorp is based on Zero Trust.

- Microsoft has embraced an **Assume Breach** mindset as core security strategy.  

- **Homefield Advantage** is what makes it possible to stay ahead of adversaries. Its best described with this picture:
![/blog/images/homefield-advantage.png](/blog/images/homefield-advantage.png)

Chances are that if you work in a mid-size to large company, your organization most likely has already been breached. Maybe your organization was not victim of a targeted attack of a competitor or nation state but just collateral damage of wide-ranging attacks that occur across the landscape, like the WannaCry worm or mass phishing attacks. 

And if you are a small sized business, you aren’t safe either, 74% of small and mid-sized companies have experienced a breach according to UK Government survey, compared to 90% of large organizations. To change the discussion, we must shift from purely preventive measures to include detection, remediation, eviction, and full recovery from adversarial events. 

> Do not wait for an external occurrence of a breach. Go, and actively prepare for such events and conditions. Consider your organization breached right now - you just have not found the adversary yet. 

Effective defense is difficult to achieve without having resources dedicated to testing, measuring and improve the organization’s immune system, and this is where an Offensive Security Program comes into the picture.

# Embracing the Red  

Invest in offensive security to breach yourself. Establishing an internal offensive security program will help build up and improve the immune system of an organization. It gives the organization the opportunity to develop a better understanding of and resilience to attacks.

![/blog/images/2021/matrix2.jpg](/blog/images/2021/matrix2.jpg)

If real breaches occur, the organization can effectively act to detect and remediate the threat. The overarching goal is to help the organization improve protection, detection, and response capabilities. 

Offensive security reduces uncertainty around the state of the security of an organization. Therefore, an offensive security program is perfectly positioned to inform the risk management process to help make informed business decisions.

Cheers.
[@wunderwuzzi23](https://twitter.com/wunderwuzzi23)





## References

1. [WannaCry Ransomware](https://en.wikipedia.org/wiki/WannaCry_ransomware_attack)
2. [Exchange Server Exploits](https://www.fireeye.com/blog/threat-research/2021/03/detection-response-to-exploitation-of-microsoft-exchange-zero-day-vulnerabilities.html)
3. [Colonial Pipeline Attacks](https://en.wikipedia.org/wiki/Colonial_Pipeline_cyber_attack)
4. [SQL Slammer Worm](https://en.wikipedia.org/wiki/SQL_Slammer)
5. [Google BeyondCorp](https://cloud.google.com/beyondcorp/)
6. [Microsoft Enterprise Cloud Red Teaming Paper](https://download.microsoft.com/download/C/1/9/C1990DBA-502F-4C2A-848D-392B93D9B9C3/Microsoft_Enterprise_Cloud_Red_Teaming.pdf)
7. [Huge rise in hack attacks as cyber criminals target small business](https://www.theguardian.com/small-business-network/2016/feb/08/huge-rise-hack-attacks-cyber-criminals-target-small-businesses)


### Free Images from Pixabay 
1. [Immune system](https://pixabay.com/illustrations/defense-protection-threat-1403072/)
2. [Teapot](https://pixabay.com/get/g5d83c35d98eca01317ef609d7f680c8eb428f691ae25e895100677d323339fa563e906817e3e453cf234aacebd4ed04c_1280.jpg)
3. [Cyber data digits](https://pixabay.com/illustrations/binary-black-cyber-data-digits-2170630/)
4. [Matrix Computer](https://pixabay.com/illustrations/matrix-computer-hacker-code-2354492/)
5. [Ransomware](https://pixabay.com/get/g6dfbd8c96574b4057c20eae1499f1ea812b96561023c41ed1df3b48e386c3a4401835c7cfd71e1fa0bca0dcb1265fbd4_1280.jpg)
6. [Coins](https://pixabay.com/get/g2dd8e6c6e25a8f878d4baa408612279b98c284076c54e540a7289d42562928f7bfd907198aeb127b2d104c48124bdaf4_1280.jpg)
7. [Hacker Cybercrime](https://pixabay.com/illustrations/hacker-cyber-crime-internet-2300772/)

*Images have been slightly modified (mostly cropped)*