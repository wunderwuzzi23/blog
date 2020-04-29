---
title: "Cookie Crimes and the new Mirosoft Edge Browser"
date: 2020-04-28T18:10:46-07:00
draft: true
tags: [
        "cookies",
        "red",
        "passthecookie", "ttp","post-exploitation"
    ]
---

## Revisiting Cookie Crimes
In 2018 *mangopdf* described "Cookie Crimes", which is great research around Chrome's remote debugging feature that allows adversaries and malware to gain access to cookies quite convienently during post-exploitation.

Original research published <a href="https://mango.pdf.zone/stealing-chrome-cookies-without-a-password">here</a>, and it still works today.

## The new Microsoft Edge browser and Chromium

Microsoft's latest Edge browser is based on the same code, Chromium. I guess, you already know where this is going now...

Yes, this means that the same attacks described with "Cookie Crimes" work with the new Edge browser.

## Notable differences:

1. Cookie Crimes uses **chrome.exe**, but if one changes that to **msedge.exe** you can get it to work with Edge on Windows (haven't tried other operating systems)
2. The Edge user data folder is located at *%LOCALAPPDATA%\Microsoft\Edge\User Data* 

Additionally, the techniques around [**remote controlling the browser and oberserving browser behavior of users**](/blog//posts/2020/chrome-spy-remote-control) also works with the Chromium based Edge browser. 

## Mitigations and Detections
* Blue teams should look for **--remote-debugging-port** and custom **--user-data-dir**, and related command line arguments to potentially catch (mis)use for both Chrome and Edge.
* Firefox also has remote debugging, but it works differently (different command line option to look for) - I will post about this as well soon
* Out of due diligence I reported this to MSRC (but since it's post-exploitation nothing will be changed)
* I also suggested to add detections for the TTP to Windows Defender

There are some more mitigation ideas [in the previous blog post about Chrome](/blog/posts/2020/chrome-spy-remote-control) as well, please look at them for reference also.