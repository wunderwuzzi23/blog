---
title: "Google Leaks Your Alternate Email Addresses to Unauthenticated Users"
date: 2019-06-04T21:51:52-07:00
draft: true
tags: [
        "research",
        "bugbounty"
    ]
---

The Google Login Flow leaks additional email account information to unauthenticated users. I discovered this in the Google Account Login flow while building KoiPhish.

## Responsible Disclosure

I reported this issue to Google and they looked into it and after a about 5 weeks of back and forth they decided that this is not an issue worth fixing. After asking if I can post about it publicly I got Google's okay. 

So here we go! :)


## Issue 
All that is needed to trigger the leakage is to call the unauthenticated signin endpoint at

    https://accounts.google.com/_signin/sl/lookup 
    
and in the f.req parameter provide an email address.

This is how it looks via Fiddler:

![Google Email Leak](/blog/images/google.alternateaccount.leak.png)

An adversary could do this at scale to retrieve additional accounts for phishing or possibly learn about recovery account - which in my case was the alternate account. lol. So, I'd assume others might have done the same.

## Mitigations 

Remove all alternate account associations. Since the altnernate account might be your recovery address - it was setup for a couple of friends I shared this with, so it's likely a normal case.

Make sure that any alternate account is not your password recovery or 2FA to minimize attack surface. 

*Originally written up February, 8th 2019. Posted June 2019*
