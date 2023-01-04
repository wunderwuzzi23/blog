---
title: "Decrypting TLS browser traffic with Wireshark"
date: 2023-01-04T06:36:05-08:00
draft: true
tags: [
        "red team", "ttp"
    ]

twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "Decrypting TLS browser traffic with Wireshark and netsh"
  description:  "Decrypt https web browser traffic with Wireshark and netsh (doing it the Red Team way)"
  image: "https://embracethered.com/blog/images/2023/decrypt3-small.png"
---

There is a combination of lesser known tools and techniques to capture and later decrypt SSL/TLS network traffic on Windows. This technique is neat because it does not require the installation of additional driver/software when capturing the traffic.

## Technique, Tools and Steps 

It is quite straight forward and consists of:
1. Setting the `SSLKEYLOGFILE` environment variable to capture TLS session keys on target host
2. Use `netsh trace start` to capture traffic (no need to install additional driver/software!)
3. Convert the `.etl` file to a `pcap` using Microsoft's [etl2pcapng](https://github.com/microsoft/etl2pcapng)
4. Start `Wireshark`, open the pcap and set the sslkeys under: *Preferences->Protocols->TLS->Pre-Master secret*. 
This does not have to be on the same host as steps 1-2.
5. Enjoy the decrypted traffic!

If you can or want to capture traffic with Wireshark also, there is no need to use `netsh` of course.

## Video Tutorial

I put together a tutorial and you can watch it [here](https://www.youtube.com/watch?v=nulBm-VgT4s).

[![Decrypt https traffic with Wireshark](/blog/images/2023/decrypt-youtube-small.png)](https://www.youtube.com/watch?v=nulBm-VgT4s)

If you enjoy the content and/or video, please like it and Subscribe to the YouTube channel. I might post videos more regularly if these are useful.

Cheers and Happy Hacking!



## Appendix

List of commands:

```
[Environment]::SetEnvironmentVariable("SSLKEYLOGFILE", "c:\temp\sslkeys\keys", "MACHINE")

taskkill /im chrome.exe /f

netsh trace start capture=yes tracefile=c:\temp\sslkeys\trace.etl report=disabled
netsh trace stop

etl2pcapng trace.etl trace.pcap
```

## References

* [etl2pcapng](https://github.com/microsoft/etl2pcapng)