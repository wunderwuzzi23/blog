---
title: "KoiPhish - The Beautiful Phishing Proxy"
date: 2019-01-10T21:40:59-07:00
draft: true
tags: [
        "pentesting",
        "red",
        "code",
        "phishing"
    ]

---

**KoiPhish is a simple yet beautiful relay proxy idea.**

The idea for this little project goes back many years. Since I started learning Golang I figured it would be good exercise to finally go ahead an implement it. So, last December during the 35C3 (which is always inspiring congress) I wrote it up.

It relays requests a client makes to the KoiPish to the actual target and responses are sent back to the client. On the way in and out common links are overwritten in order to not break the user experience and functionality. 

The benefit of this approach compared to cloning a website is that it will have the same look and feel as the target, and automatically adjust to changes down the road.

The code is basic at this point, and it's intentionally not "point and click".


## Illustration


                                            Keep Relaying 
    End User  +------------->   KoiPhish   +------------->   Actual Login Page
                                           <-------------+            
               Keep Relaying
              +------------->              +------------->   and MFA Provider
              <-------------+              <-------------+           


An adversary can continue this until the passwords and/or session tokens (after 2FA) are grabbed.

The KoiPhish Golang code can be found on Github: [KoiPhish Code](https://github.com/wunderwuzzi23/KoiPhish)

## Mitigation

Leverage security keys and U2FA to help mitigate phishing attacks. Learn more here, this is important if you want to tackle phishing:

* FIDO Alliance
* WebAuthN


## Disclaimer

Pentesting requires authorization and consent by appropriate stakeholders. Don't do illegal things.
