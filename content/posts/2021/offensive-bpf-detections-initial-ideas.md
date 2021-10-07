---
title: "Offensive BPF: Detection Ideas"
date: 2021-10-07T02:00:00-07:00
draft: true
tags: [
        "research","ebpf","blue"
    ]

twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "Offensive BPF: Detection Ideas"
  description:  "Using eBPF in offensive security settings and mitigations"
  image: "https://embracethered.com/blog/images/2021/obpf-detections.png"

---

This post is part of a series about **Offensive BPF** that I'm working on to learn how BPFs use will impact offensive security, malware and detection engineering. 

Click the ["ebpf"](/blog/tags/ebpf) tag to see all relevant posts.

![Offensive BPF](/blog/images/2021/offensive-bpf.png)

In the last few posts, we talked about a `bpftrace` and how attackers can use it to their advantage. This post is about my initial ideas and strategies to detecting malicious usage. 

## Detecting BPF misuse

There are a set of detection ideas for Blue Teams. Since we primarily talked about `bpftrace` so far, let's explore that angle. 

There will be more recommendations down the road as I learn and understand BPF better.

### Telemetry

Collecting telemetry around [BPF syscalls](https://www.kernel.org/doc/html/latest/userspace-api/ebpf/syscall.html) is a crucial to getting insights into its usage across the fleet. Make sure that whatever tool or product you are using does collect the proper information and that it is centrally available in your threat detection pipeline.

### Inspecting loaded BPF programs

Loaded BPF programs can be inspected via the `bpftool`.

For instance, `bpftool prog` will show you the details and you can see the loaded programs:

![BPF prog output](/blog/images/2021/bpfprog.png)

Collecting this information will allow to build an inventory on what BPF running, and if there are any unexpected outliers.

### Unsafe bpftrace usage and system() calls

The usage of a `system()` call seems very unusual. So, looking for command line logs that contain `bpftrace --unsafe` seems a good way to catch **dangerous** bpf programs also.

### Persistence

BPF programs don't survive a reboot, so an adversary will try to restart them (cron jobs, etc..). Be on the lookout.

### Supply chain attack or tampering with valid BPF programs

There is also the attack avenue to backdoor existing programs that performance teams use or that are executed regularly on hosts.  

I haven't seen any signature validation approaches yet, which could help detect such changes. Which brings us to the next point of the idea of having an inventory of known good BPF programs.

### Creating an inventory of known BPF programs

Creating and maintaining and inventory of valid BPF programs of your environments seems a good strategy. This allows catching unknown ones to investigate.

### Perform a Red Team Exercise using the BPF TTP

Assume Breach operations can help identify detection and prevention opportunities in your environments. Hence, I'd recommend performing a red or purple team operation that emulates usage of malicious BPF programs.

![OBPF Detections](/blog/images/2021/obpf-detections.png)

### Hooking the BPF system call itself!

A nifty attacker will likely hook the `bpf()` system call itself to change blue teams reality. 

Performing static analysis on BPF programs can help identify such programs, as can monitoring all calls to the `bpf` system call dynamically.

### Investigate and leverage existing BPF malware detection kits

There has not been a lot of work in this regard, but there are two projects I want to look at more closely when it comes to detections:

* https://github.com/Gui774ume/ebpfkit-monitor
* https://github.com/pathtofile/bpf-hookdetect (might not be just about eBPF, but general kernel rootkits)


## Conclusion

Hope this post is helpful to provide more practical and technical perspective on what defenders need to start looking for. 

I'm still pretty new to BPF, but the possibilities for attack seem to be limited by creativity, hence **I think BPF malware will be quite common in the not-so-distant future, so let’s get a step ahead with testing and building detections.**

Cheers!

[@wunderwuzzi23](https://twitter.com/wunderwuzzi23)

