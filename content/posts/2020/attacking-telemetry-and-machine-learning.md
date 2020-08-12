---
title: "Red Teaming Telemetry Systems"
date: 2020-08-12T13:28:00-07:00
draft: true
tags: [
        "red",
        "ttp",
        "machine learning",
    ]
---

These days business decisions and feature development often is influenced heavily by telemetry information. Telemetry is baked into the programs, services and applications we use.

**Companies are hungry for telemetry because with machine learning and Deep Neural Networks "data is the new oil"**.

Telemetry provides insights into how users use a particular system, what features they exercise, how they configure the system, what errors they trigger and what buttons they like clicking on. 

> Organization make business decisions and future feature development based on telemetry

The goal is to leverage this information to re-iterate on software and improve it or add missing features, or remove unpopular features, or show you relevant ads.

But, how secure are telemetry pipelines?

## Commodore 64 - Red Teaming Example

A while back during a red teaming engagement I spoofed the operating system from which telemetry was sent. Instead of sending the correct Windows or Linux version, the red team sent **millions of spoofed requests** coming from a “Commodore 64” up the telemetry endpoint. 

**The result was that this information became visible in dashboards inside the company, and the C64 was the most popular operating system the software ran on! Yay!** 

![Commodore 64](/blog/images/2020/commodore64.jpg)

This raised awareness of tightening the security and data sanitization of telemetry pipelines.

This somewhat benign (yet widely visible) example can help to understand how such attacks could be misused to manipulate decision making.

## Red Teaming Telemetry Pipelines

As the example showed an adversary can send targeted messages to highlight features or flows that in reality are not frequently used, which can lead to de-investments in areas that are more important.

There are multiple ways to target telemetry, especially when it comes to **AI Red Teaming** at large, as the data is often fed into machine learning training. In this post I am talking about the outside approach of spoofing telemetry requests (which often has no or very weak authentication by design).

## Conclusion

Internal red teams can run operations that target telemetry. This ensure to pipelines are assessed and proper sanitization, sanity checks, input validation for telemetry data is in place.

As always, feel free to follow or DM me on Twitter: [@wunderwuzzi23](https://twitter.com/wunderwuzzi23)

## References

https://en.wikipedia.org/wiki/Commodore_64
