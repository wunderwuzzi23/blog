---
title: "BashSpray - Simple Password Spray Bash Script"
date: 2019-07-03T21:58:01-07:00
draft: true
tags: [
        "pentesting",
        "red",
        "code",
        "tool"
    ]
description: "Series of posts for Password Spraying Technques"
meta_title: "Bash and password spraying"
---

One thing every red team should attempt early on and regularly is to perform some password spray testing across their organization to identify and help remediate usage of weak passwords.

In the past I have done this on Windows a lot, but now I built a simple version for it for Bash to run it also from a Mac.

Check it out: [Bash Spray](https://github.com/wunderwuzzi23/BashSpray)

Ideally, a script like bashspray.sh is integrated into your response pipelines, and SOC, Blue Team as well as account owner get notified - so they change their password right away, and any SOC investigation can be performed if necessary. 

Be careful about account lockout policies and make sure you have authorization from appropriate stakeholders before engaging in this kind of testing.
