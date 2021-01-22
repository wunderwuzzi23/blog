---
title: "Survivorship Bias and Red Teaming"
date: 2021-01-22T12:00:34-08:00
draft: true
tags: [
        "red", "mitre"
    ]
---

Survivorship bias is an interesting thing that we can observe nearly daily. It is the success stories of exponentially growing startups, the motivational speaker who shares their insights on how to be successful and so forth. 

What is it exactly?

## What is survivorship bias?

Wikipedia defines it as "...the logical error of concentrating on the people or things that made it past some selection process and overlooking those that did not, typically because of their lack of visibility." 

It occurs when focusing on a subset of information, while happily ignoring other areas. 

> "Every inspirational speech by someone successful should have a start with a disclaimer about survivorship bias." [xkcd](https://xkcd.com/1827/)


One of the most interesting examples of survivorship bias is the story of Abraham Wald. During World War II Wald was part of a group of researchers at Columbia University who investigated fighter planes and where they were being hit. This was investigated in order find out what areas of the plane should be improved. The US military argued to focus on the spots that showed hits, but Wald proposed that the spots that are not showing damages are the more important ones - because that is where the planes that do not return were hit!

[![Survivorship Bias](/blog/images/2021/survivor.png)](/blog/images/2021/survivor.png)
[Source Wikipedia](https://en.wikipedia.org/wiki/Survivorship_bias)

Another interesting story I read recently is that during World War 1 the British Army issued helmets to soldiers and afterwards noticed an increase in head injuries. Why would that be?

These are quite interesting stories, but how does this relate to red teaming?

## Where do adversaries hit?

The MITRE ATT&CK framework allows us to highlight what techniques adversaries leverage to hit an organization. MITRE has a tool called ATT&CK Navigator that can be used to highlight those areas. 

Here is an example from my book that was built with MITRE's ATT&CK Navigator to show detections of a red team operations:

[![MITRE ATTACK Navigator](/blog/images/2021/attack-navigator.png)](/blog/images/2021/attack-navigator.png)

The similarities are quite interesting when putting the matrix next to the Wikipedia image of the plane:

[![MITRE ATT&CK Survivor Bias](/blog/images/2021/mitre-attack-survivorship-bias.png)](/blog/images/2021/mitre-attack-survivorship-bias.png)

A picture similar to the one above correlating the MITRE ATT&CK matrix with the image of the plane from the Wikipedia article was not my idea - I saw it first by Matt Snyder from VMWare during one of the monthly [MITRE ATT&CK Power Hour](https://www.mitre.org/attackcon) talks. I highly recommend tuning in to the Power Hour by the way.

But how does this now relate to red teaming and survivorship bias?

## Red teaming and survivorship bias

Remember that the ATT&CK matrix covers only open-source threat intelligence by design. It does not cover behaviors, tactics or techniques that are not publicly disclosed or known. It also does not cover insider threats explicitly. This is something to be aware of when using the matrix, as it might not always be the appropriate tool by itself.

MITRE also offers [CAPEC](https://capec.mitre.org/) which is something to look into if you are a red teamer, as you might find additional ideas or operations hidden in the CAPEC enumerations.

In my opinion a mature red team in a mature organization should focus on finding new exploits, and novel ways to bypass security controls, and leave the test validation to other teams - e.g., forming a purple team or security test validation v-team to tackle that is typically most efficient.

A red team should entertain focusing on areas that are not highlighted by the blue team in the matrix AND even more important the red team should focus on expanding the matrix with relevant threats to the business that are not covered in the matrix at all.

It is quite common that resources get allocated on researching the most recent incidents and security vulnerabilities. That is good, especially the investigation of variants for vulnerabilities is critical. But keep in mind that advanced adversaries have probably already moved on working on the next "big thing" after things like the SolarWinds attack unwinds.

## Conclusion

A red team should spend a significant amount of time investigating and exploring uncovered areas and attempting to find unknowns. If this is possible depends often on the maturity of the organization and team. To cover and validate existing detections an organization can stand up a purple team or build blue team automation. In my opinion that is not what a red team should spend most of their time doing, although red team's certainly can (and should) help if needed. And in smaller organization these functions typically overlap.

Remember, the matrix only covers things that have already been seen and publicly documented in the wild. It does not cover unknown threats or techniques that lack open-source threat intel or have not been used by real world malware.

I will speak more about this at my talk at the [Hong Kong - Information Security Summit](https://www.issummit.org/Mr_Johann_Rehberger.html#summit-speaker-dtl) this year. Check that out as well.


Cheers, 

Johann.

[@wunderwuzzi23](https://twitter.com/wunderwuzzi23)

## References

1. [xkcd - Survivorship Bias](https://xkcd.com/1827/)
1. [Survivorship Bias Wikipedia](https://en.wikipedia.org/wiki/Survivorship_bias)
1. [Matt Snyder ATT&CK Power Hour Slide](https://www2.slideshare.net/attackcon2018/mitre-attckcon-power-hour-November/44)
1. [Common Attack Pattern Enumeration](https://capec.mitre.org/)
1. [Information Security Summit - Challenge of Securing the New Norm - the Remote, Mobile, Decentralised and Virtual Business](https://www.issummit.org/Mr_Johann_Rehberger.html#summit-speaker-dtl)
