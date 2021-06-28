---
title: "Airtag hacks - scanning via browser, removing speaker and data exfiltration"
date: 2021-06-28T08:00:52-07:00
draft: true
tags: [
        "red", "hardware"
    ]
---

Until the Apple Airtag came out a few months ago I hadn't really looked into the tag tracking market. Turns out there were already quite a lot of offerings available before Apple joined the market, most notably *Tile*.

However, I wanted to try out the Airtag and ended up ordering a few. 

This post will explore three things:
1. [Removing the speaker of my Airtag](#removing-the-speaker)
2. [Using Browser APIs to scan for Airtags](#browsers-to-the-rescue) (if you don't have an iPhone but someone tries to stalk you this might be handy)
3. [Explore data exfiltration via Airtags and Apple's "Find My" network](#novel-data-exfiltration)

By the way, when you order your Airtags online you can customize them. So, I have some cool icons on mine, like this one:

![Airtag](/blog/images/2021/airtag.png)

The Airtags have a speaker built in which goes off when they are away from the owner, but following another device for an extended period of time. The time period was actually changed recently from 3 days to I think 8 hours, after privacy concerned articles and blog posts appeared.

Most importantly, a sound will be emitted once the Airtag appears to be following someone.

## Removing the speaker

A very obvious hack of course is to remove the speaker, which I was able to do in about 30 seconds using a sharp knife. Please note that this might destroy your Airtag - so know what you do. 

This is how the microphone looks like (on the right, next to the main Airtag):

![Airtag Mic Remove](/blog/images/2021/airtag-remove-mic.png)

Afterwards no sounds are emitted from the device anymore. 

Technically, iPhone users should still be in a good shape as the "Find My Device" app should signal a warning via the app once a stalker Airtag is detected - which might take a few hours.

![Airtag Mic Removed](/blog/images/2021/airtag-with-removed-mic.png)

For Android users the story is different though.

## Waiting for an Apple Android App...

Apple has not yet released an Android app - so stalkers can happily go after Android users with little risk of being detected. 

The good news is that there are a set of Android apps that show you Bluetooth devices - which is one way to identify Airtags as well that might be in your proximity. 

[Recent research by the TU Darmstadt](https://arxiv.org/pdf/2103.02282.pdf) contains a lot of useful information regarding the protocol and usage.

![Bluetooth Whitepaper Research](/blog/images/2021/ble-ad.PNG)

Using information from the paper I checked to see if it's possible to implement a simple Airtag scanner for the browser.

## Browsers to the rescue

Turns out that we can also use the browser to scan for Bluetooth device advertisements. In particular Chrome 92+ ([currently in beta for Android users](https://chromereleases.googleblog.com/2021/06/chrome-beta-for-android-update_0893144828.html)) supports `requestDevice` and filtering via a `dataPrefix`.

The code I stitched together looks like this:

```
 navigator.bluetooth.requestDevice({
      filters: [{ manufacturerData: [ 
                  {  
                    companyIdentifier: 0x004c,            // Apple 
                    dataPrefix: new Uint8Array([0x12])    // Airtag
               }],
      optionalServices: ['battery_service']  
     }]   
  })
```

[A simple webpage to check for Airtags is hosted here](/blog/bluetooth/scan.html).

*Note: It requires Chrome 92+ ([currently in beta for Android users](https://chromereleases.googleblog.com/2021/06/chrome-beta-for-android-update_0893144828.html)) to work. Sometimes it seems flacky. I'm not sure exactly how frequently an Airtag indeed emits an Bluetooth advertisement.*

![Airtag Browser Scan](/blog/images/2021/airtag-browser-scan.png)

Above you can see a scan run on my Android device which shows that two Airtags are close by.

# Novel Data Exfiltration

One thing I was wondering if Apple's Airtag/device tracking network can be used to beaconing out arbitrary data. 

The idea would be to piggy-back on Apple's "Find My" network (*offline finding*) to do so. Turns out that this is totally possible via fake Airtag Bluetooth advertisements. 

The TU Darmstadt published a toolset called [OpenHayStack](https://github.com/seemoo-lab/openhaystack) and there was another article I found about a tool call [send-my](https://positive.security/blog/send-my).

All that is needed seems to have a simple Windows app that performs **Bluetooth advertisements** according to the two articles above to send data out via nearby Airtags. 

This will be one of my upcoming weekend projects. :)

## Conclusion

Airtags and other tracking devices enable stalkers, and its quite scary how easy a stalking device can be created. Let's hope that we soon see more common frequency tracking apps that can raise alerts, ideally built into the Operating Systems. 

It's also worth keeping an eye on potential data exfiltration attacks that seem possible via Airtags, although throughput appears quite limited.


Cheers,
[@wunderwuzzi23](https://twitter.com/wunderwuzzi23)

## References

1. [Best Airtag alternatives](https://www.igeeksblog.com/best-airtag-alternatives/)
2. [Recent research by the TU Darmstadt](https://arxiv.org/pdf/2103.02282.pdf)
3. [Chrome 92 Beta Release Announcement](https://chromereleases.googleblog.com/2021/06/chrome-beta-for-android-update_0893144828.html)


