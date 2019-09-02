---
title: "Coinbase under attack and cookie theft"
date: 2019-09-01T13:25:07-07:00
draft: true
tags: [
        "pentesting",
        "breaches"
    ]
---

Recently Coinbase published a well written blog post on how they were under attack. The adversaries exploite Firefox 0-days. [Details can be found here.](https://blog.coinbase.com/responding-to-firefox-0-days-in-the-wild-d9c85a57f15b)  One intersting aspect is the following:

> "We have also observed the attackers specifically target cloud services, e.g. gmail and others, via browser session token theft via direct access to browser datastores. This activity also offers the opportunity for behavior-based detection, as relatively few processes should be directly accessing those files."

With organizations moving away from having internal corporate networks and adopting "zero trust" strategies these cloud focused attacks will become more common, and eventually they will be the new norm.


References:

1. Coinbase blog post: https://blog.coinbase.com/responding-to-firefox-0-days-in-the-wild-d9c85a57f15b
2. Pass the Cookie: [Pass the Cookie at the Chaos Communication Congress (35C3)](https://c3lt.de/35c3/talk/CK3DWH/)
