---
title: "Offensive BPF: Tools"
date: 2021-10-08T12:14:59-07:00
draft: true
tags: [
        "pentest", "red","research","ebpf","blue"
    ]

twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "Offensive BPF: Tools"
  description:  "Using eBPF in offensive security settings and mitigations"
  image: "https://embracethered.com/blog/images/2021/obpf.png"

---

This post is part of a series about **Offensive BPF** that I'm working on to learn about BPF to understand attacks and defenses, click the ["ebpf"](/blog/tags/ebpf) tag to see all relevant posts.

I'm learning BPF to understand how its use will impact offensive security, malware and detection engineering. 

![Offensive BPF](/blog/images/2021/offensive-bpf.png)

This post is focused on "Level 1" on how to use existing BPF tools. Level 1 means using tools that are already written to get familiar with the regular use cases for BPF.

# Existing tools and Installation 

Performance and observability teams are pushing for BPF tooling to be present in production. Due to its usefulness, this is likely going to increase. 

Leaders in the industry are pushing for the installation of a common toolset. For instance, here is a [slide](https://www.slideshare.net/brendangregg/systemsscale-2021-bpf-performance-getting-started) from a presentation at Systems@Scale 2021 conference.

![BPF Tools Installation](/blog/images/2021/bpf-tools-install.png)

For offensive security this means that it will become more likely that adversaries will leverage already installed tooling to do their deeds.

In the offensive security space we call this **"Living off the land"**.

### Living off the land - bpfcc-tools

To get an idea of some cool tools, check out the `bpfcc-tools` suite.

```
sudo apt-get install bpfcc-tools
```

As mentioned before in some production environemnts these tools might already be installed.

The tools in `bpfcc-tools` generally all end with `-bpfcc` 
On my Ubuntu install, the tools are at `ls /usr/sbin/*bpfcc`.


The rest of this post goes over a few tools and one-liners that might can help with recon and mapping out the homefield.


## Sniffing TLS traffic

Sniffing TLS traffic is a common post exploitation attack angle to get access to sensitive information, such as Cookies or Bearer tokens.

Tracing traffic inside the kernel with existing techniques such as `tcpdump` does typically not lead to desired results (unless you have session keys for TLS - I wrote about this in my book by the way).

What BPF allows now is to create hook points for popular user mode functions that deal with reading/writing the TLS information before (or rigth after) it is encrypted.


A tool that hooks OpenSSL's `PR_Read` and `PR_Write` calls to dump clear text is part of the bpfcc-tools.

```
wuzzi23@ubuntu:~$ sudo sslsniff-bpfcc 
FUNC         TIME(s)            COMM             PID    LEN   
WRITE/SEND   0.000000000        curl             2781502 73    
----- DATA -----
GET / HTTP/1.1
Host: example.org
User-Agent: curl/7.68.0
Accept: */*

----- END DATA -----

READ/RECV    0.030690910        curl             2781502 822   
----- DATA -----
HTTP/1.1 200 OK
Server: nginx/1.14.0 (Ubuntu)
Date: Tue, 28 Sep 2021 16:55:13 GMT
Content-Type: text/html
Content-Length: 511
Set-Cookie: **ThisShouldBeASecret!**
Last-Modified: Tue, 03 Nov 2020 18:54:15 GMT
Connection: keep-alive
ETag: "5fa1a757-1ff"
Strict-Transport-Security: max-age=31536000; includeSubDomains
Accept-Ranges: bytes

<!doctype html>
<html>
...
```

My goal was to sniff traffic from Firefox or Chrome, but the `sslsniff-bpfcc` tools does not seem to hook those APIs. Firefox uses NSS.

## What other tools are in this toolset?

Brendan Gregg has a great diagram on his website that shows all the tools and where in the stack they are appliable to:

![BPF Tools](/blog/images/2021/bpf_performance_tools.png)

This is amazing.

## Reconaissance

## Trust

Given the amount of superpowers BPF programs have there seems to be general lack of allow only certain BPF programs to execute or at least have some kind of signature validation/sanity check to ensure the programs that are run are the known good ones.


## Detections and Threat Hunting

For living off the land style attacks, detection is often a bit tricky, since it can lead to creation or a lot of false-positives. Generally making sure to get logs off machines to analyze information is important.

In particular, it would be interesting to hunt for usage of tools such as `sslsniff-bpfcc`. 


# References

* https://www.slideshare.net/brendangregg/systemsscale-2021-bpf-performance-getting-started
* https://www.brendangregg.com/blog/images/2019/bpf_performance_tools.png
