---
title: "HITCON CMT 2023 - LLM Security Presentation and Trip Report"
date: 2023-09-25T17:24:51-07:00
tags: [
     "aiml", "machine learning","ai injections","conference"
    ]
twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "HITCON CMT 2023 - LLM Security Presentation and Trip Report"
  description: "Earlier this month I had the opportunity to attend HITCON CMT in Taiwan for the first time. It's an annual event hosted by the Hackers in Taiwan organization and CMT stands for the community version."
  image: "https://embracethered.com/blog/images/2023/aiinjection-image-joke-bard-response.png"
---

Earlier this month I had the opportunity to attend HITCON CMT in Taiwan for the first time. 

[![HITCON CMT Logo](/blog/images/2023/hitcon.logo.webp)](https://hitcon.org/2023/CMT/)
It's an annual event hosted by the Hackers in Taiwan organization and CMT stands for the community version. There is a second event for Enterprise later this year also - think of it like Blackhat vs Defcon in a way.

## Conference, Location and Registration

HITCON CMT 2023 was a two day event hosted in the east side of Taipei at Academia Sinica.

Overall there were three general tracks, villages, and many side events, like a CTF, scavenger hunt, workshops,... One thing that stood out to me from the get go was how well organized and high quality the event was. 

At speaker registration, I got lucky and got this cool Taiwan badge from a couple of years ago:

![HITCON Badge](/blog/images/2023/hitcon-badge.jpg)

Registration went very smoothly and I got a goodie bag, t-shirt and a bubble tea kit.

## Talks

Majority of talks are given in Mandarin, but there is one track that always had an English speaker and the HITCON does community provided translation services for all talks (both English to Mandarin and vice versa).

![HITCON - Stage](/blog/images/2023/hitcon-stage.jpg)

The keynote was given by Zion Basque who talked about reversing with the help of Generative AI, and showed a very cool LLM integration with IDA to aid reversing.

I got to meet Orange Tsai, who delivers impressive security research and discovered some [cool VPN security issues in the past](https://devco.re/blog/2019/07/17/attacking-ssl-vpn-part-1-PreAuth-RCE-on-Palo-Alto-GlobalProtect-with-Uber-as-case-study/). And I got to watch his talk "A 3-Year Tale of Hacking a Pwn2Own Target: The Attacks, Vendor Evolution, and Lesson Learned". It was the talk that he would have given at DEF CON but he wasn't granted the visa to enter the US and had to withdraw as speaker.

"Ghosts of the Past" from Jia Hao Poh, talking about old school PHP vulnerabilities in Trend Micro Enterprise products. Carl Smith from Google talked about fuzzing the V8 JavaScript engine.

Wojciech Reguła had a really insighful talk about his latest tool [Electroniz3r](https://github.com/r3ggi/electroniz3r), that allows code injection in Electron apps on macOS. And to stay with Electron exploits, Li Jiantao talked about "What You See IS NOT What You Get: Pwning Electron-based Markdown Note-taking Apps". 

This talk made me think quite a bit about the applicability of some of these TTPs with  Large Language Models integrated applications and how Indirect Prompt Injection could be a future attack angle for exploitation.

Yours truly was also on stage talking about **Indirect Prompt Injections in the Wild**.

![HITCON - Speaker](/blog/images/2023/hitcon-johann.jpg)

There were lots of follow up questions and interest in the topic of LLMs and safe and secure useage.

A couple of other great and highly technical talks where about Windows Kernel Exploitation by Sheng Hao Ma (@aaaddress1) and another one by Zeze. The later one I didn't catch personally though. 

There was also a merry dinner after the first day to mingle and exchange hacking and war stories.

![HITCON - Hackers](/blog/images/2023/hitcon-hackers.png)

Overall, HITCON is a great conference and I will definelty try to make it again next time.

A big Thanks and Kudos to the organizers.

下次見


## References

* [HITCON CMT 2023](https://hitcon.org/2023/CMT/)
* [Ghosts of the Past: Classic PHP RCE Bugs in Trend Micro Enterprise Offerings]()
* [A 3-Year Tale of Hacking a Pwn2Own Target: The Attacks, Vendor Evolution, and Lesson Learned]()
* [ELECTRONizing macOS privacy - a new weapon in your red teaming armory](https://github.com/r3ggi/electroniz3r)
* [Attacking SSL VPN - DevCore](https://devco.re/blog/2019/07/17/attacking-ssl-vpn-part-1-PreAuth-RCE-on-Palo-Alto-GlobalProtect-with-Uber-as-case-study/)


