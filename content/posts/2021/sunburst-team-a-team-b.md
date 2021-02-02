---
title: "Team A and Team B: Sunburst, Teardrop and Raindrop"
date: 2021-02-02T10:35:43-08:00
draft: true
---

The other day [Microsoft published a great deep dive](https://www.microsoft.com/security/blog/2021/01/20/deep-dive-into-the-solorigate-second-stage-activation-from-sunburst-to-teardrop-and-raindrop/) around the second stage payloads and hands-on hacking activities of the Solarwinds/Sunburst incidents that where uncovered late 2020.

One thing that is so interesting is the use of off the shelve Cobalt Strike tooling and templates for command & control. After doing all the hard work and customization to seamlessly backdoor binaries the adversaries sem to use Cobalt Strike. 


### Intel for Red Teamers

Although, they did add  customizations with some interesting features, which are interesting for red teamers to be aware of. For instance each instance of the zombie would be unique in name, folder locations, etc. to make it more difficult to identify in environments. 

As red teamer this is something that is not that uncommon when creating your custom C2 systems and when your internal blue team is quite good. For instance, advanced red teams have tooling where the zombie can recompile itself during each pivot to look unique again and make it harder to track.

### Team A and Team B 

The distinct work flows and tool usage between foothold creation and lateral movement indicates to me that the adversaries had mulitiple seperate teams working on this operation. 

Team A  was tasked to get the foothold via quite sophisticated methods and custom tooling, and other teams then were tasked to operate within the compromised environments. This might be because of the extreme success of their operation or according to plan, that is difficult to tell. 

It's all quite interesting, and I'm sure we will continue to learn more about this in future.
