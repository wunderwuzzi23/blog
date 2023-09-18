---
title: "HITCON CMT 2023 - LLM Security Presentation and Trip Report"
date: 2023-09-18T03:24:51-07:00
tags: [
     "aiml", "machine learning","ai injections","conference"
    ]
twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "HITCON CMT 2023 - LLM Security Presentation and Trip Report"
  description: "Last month I had the opportunity to attend HITCON CMT in Taiwan for the first time. It's an annual event hosted by the Hackers in Taiwan organization and CMT stands for the community version."
  image: "https://embracethered.com/blog/images/2023/hitcon1.PNG"
---

Last month I had the opportunity to attend HITCON in Taiwan for the first time. It's an annual event hosted by the Hackers in Taiwan organization and CMT stands for the community version. 

[![HITCON CMT Logo](/blog/images/2023/hitcon.logo.webp)](https://hitcon.org/2023/CMT/)

There is a second event for enterprises later this year also - think of it like Blackhat vs Defcon in a way.

## Conference, Location and Registration

HITCON CMT 2023 was a two day event hosted in the east side of Taipei at Academia Sinica.

![HITCON - Main Stage](/blog/images/2023/hitcon2.jpg)

Overall there were three general tracks, villages, and many side events, like a CTF, scavenger hunt, workshops and a lot more. The Evening before the conference there was a nice event to get to know the organizers, volunteers and speakers.

[![HITCON - Evening Event Group Picture](/blog/images/2023/hitcon8.jpg)](/blog/images/2023/hitcon8.jpg)

One thing that stood out to me from the get-go was how well organized and high quality the event was. At speaker registration, I was lucky and got this cool Taiwan badge from a couple of years ago:

![HITCON Badge](/blog/images/2023/hitcon-badge.jpg)

Registration went very smoothly and I got a goodie bag, t-shirt and a bubble tea kit.

## Talks and Translations

Majority of talks are given in Mandarin, but there is one track that always had an English speaker and the HITCON does community provided translation services for all talks (both English to Mandarin and vice versa).

![HITCON - Translation Services](/blog/images/2023/hitcon7.jpg)

The keynote was by Zion Basque who talked about reversing with the help of Generative AI, and showed a very cool LLM integration with IDA to aid reversing.

I met Orange Tsai, who discovered some [cool VPN security issues in the past](https://devco.re/blog/2019/07/17/attacking-ssl-vpn-part-1-PreAuth-RCE-on-Palo-Alto-GlobalProtect-with-Uber-as-case-study/), and was able watch his talk "A 3-Year Tale of Hacking a Pwn2Own Target: The Attacks, Vendor Evolution, and Lesson Learned". It was the talk that he would have given at DEF CON but he wasn't granted the visa to enter the US and had to withdraw as speaker.

![HITCON - Speaker Orange Tsai](/blog/images/2023/hitcon6.jpg)

"Ghosts of the Past" from Jia Hao Poh, talking about old school PHP vulnerabilities in Trend Micro Enterprise products. Carl Smith from Google talked about fuzzing the V8 JavaScript engine. Lot's of great stuff.

Yours truly was also on stage talking about **Indirect Prompt Injections in the Wild**.

![HITCON - Speaker Johann Rehberger](/blog/images/2023/hitcon1.PNG)

There were lots of follow up questions and interest in the topic of LLMs and safe and secure useage.

[![HITCON - Speaker Johann Rehberger](/blog/images/2023/hitcon3.jpg)](/blog/images/2023/hitcon3.jpg)

Wojciech Reguła had a really insighful talk about his latest tool [Electroniz3r](https://github.com/r3ggi/electroniz3r), that allows code injection in Electron apps on macOS. 

And to stay with Electron exploits, Li Jiantao talked about "What You See IS NOT What You Get: Pwning Electron-based Markdown Note-taking Apps". 

This talk made me think quite a bit about the applicability of some of these TTPs with  Large Language Models integrated applications and how Indirect Prompt Injection could be a future attack angle for exploitation. Chatbots are often Electron apps.

A couple of other great and highly technical talks were about Windows Kernel Exploitation by Sheng Hao Ma (@aaaddress1) and another one by Zeze. I didn't catch the latter personally, though.

There was also a merry dinner after the first day to mingle and exchange hacking stories. Made many new connections and friends.

![HITCON - Orange, Jie Hao, Johann, Ravi](/blog/images/2023/hitcon-hackers.png)

Overall, HITCON is a great conference and I will definitely try to make it again next time.

A big Thanks and Kudos to the organizers and all the volunteers!

[![HITCON - Hackers](/blog/images/2023/hitcon4.jpg)](/blog/images/2023/hitcon4.jpg)

See you next time -- 下次見

Johann (@wunderwuzzi23)


## References

* [HITCON CMT 2023](https://hitcon.org/2023/CMT/)
* [Ghosts of the Past: Classic PHP RCE Bugs in Trend Micro Enterprise Offerings](https://hitcon.org/2023/CMT/slide/Ghosts%20of%20the%20Past_Classic%20PHP%20RCE%20Bugs%20in%20Trend%20Micro%20Enterprise%20Offerings.pdf)
* [A 3-Year Tale of Hacking a Pwn2Own Target: The Attacks, Vendor Evolution, and Lesson Learned](https://hitcon.org/2023/CMT/agenda/3404e5c1-67a4-4085-847a-511a91a6b6a0/)
* [ELECTRONizing macOS privacy - a new weapon in your red teaming armory](https://github.com/r3ggi/electroniz3r)
* [Attacking SSL VPN - DevCore](https://devco.re/blog/2019/07/17/attacking-ssl-vpn-part-1-PreAuth-RCE-on-Palo-Alto-GlobalProtect-with-Uber-as-case-study/)
* [Indirect Prompt Injections in the Wild](/blog/static/downloads/HITCON_CMT_Indirect_Prompt_Injections_2023_v1.0.pdf)
* [Most Photos from HITCON Flickr Page](https://www.flickr.com/photos/hitcon)
