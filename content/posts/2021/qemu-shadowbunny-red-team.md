---
title: "xxxShadowbunny goes QEMU"
date: 2022-10-22T11:17:58-07:00
draft: true
tags: [
        "pentest", "appsec","webapp"
    ]

twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "Shadowbunny goes QEMU"
  description:  "Using virtual machines to persist and evade detections"
  image: "https://embracethered.com/blog/images/2020/shadowbunny-transparent.png"

---

For the last serveral years ransomware has become more and more prevelant, and modern day malware leverages virutalization technologies more commonly to do the dirty deats.

I wrote about how it is important that red teams perform operations to emulate such threats, or at least do some table-top excercises, and we also talked about detections strategies for virtualization based malware.

A good summary is my [Shadowbunny post](/blog/posts/2020/shadowbunny-virtual-machine-red-teaming-technique) and [BSides Singapore talk](https://www.youtube.com/watch?v=deGrbmTkRjQ).

## What is QEMU?

According to the QEMU wiki pages, it is "a generic and open source machine emulator and virtualizer".


"When used as a machine emulator, QEMU can run OSes and programs made for one machine (e.g. an ARM board) on a different machine (e.g. your own PC). By using dynamic translation, it achieves very good performance."


"When used as a virtualizer, QEMU achieves near native performance by executing the guest code directly on the host CPU. QEMU supports virtualization when executing under the Xen hypervisor or using the KVM kernel module in Linux. When using KVM, QEMU can virtualize x86, server and embedded PowerPC, 64-bit POWER, S390, 32-bit and 64-bit ARM, and MIPS guests."

So this means that under Windows we won't be having the same isolation and performance as under Xen or KVM.

In this post I will focus on Windows

## Installing

Head over to https://qemu.weilnetz.de/ to download the Windows port.

`--accel whpx`


"If you want to have a fast x86 emulation running on Windows
without such restrictions, I suggest using VirtualBox instead
of QEMU."


## Sharing folders and making host accessible to guest

https://wiki.qemu.org/Documentation/9psetup


## Detections



## References

* [QEMU Wiki](https://wiki.qemu.org/Main_Page)
* [QEMU Windows Download](https://qemu.weilnetz.de/)
* [Shadowbunny post](/blog/posts/2020/shadowbunny-virtual-machine-red-teaming-technique)
* [BSides Singapore Shadowbunny talk](https://www.youtube.com/watch?v=deGrbmTkRjQ)
* 
