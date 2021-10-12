---
title: "What's in the bpfcc-tools box?"
date: 2021-10-09T14:00:59-07:00
draft: true
tags: [
        "pentest", "red","research","ebpf","blue"
    ]

twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "What is in the bpfcc-tools box"
  description:  "Using eBPF in offensive security settings and mitigations"
  image: "https://embracethered.com/blog/images/2021/obpf.png"

---

This post is part of a series about **Offensive BPF** that I'm working on to learn about BPF to understand attacks and defenses. Click the ["ebpf"](/blog/tags/ebpf) tag to see all relevant posts.

![Offensive BPF](/blog/images/2021/offensive-bpf.png)

In the previous posts I spend time learning about `bpftrace` which is quite powerful. This post is focused on basics and using existing BPF tools, rather then building new BPF programs from scratch. 

# Living off the land: bpfcc-tools

Performance and observability teams are pushing for BPF tooling to be present in production. Due to its usefulness, this is likely going to increase. 

Influencers in the industry are pushing for the installation of a common toolset. 

For instance, here is a [slide](https://www.slideshare.net/brendangregg/systemsscale-2021-bpf-performance-getting-started) from a Systems@Scale 2021 conference presentation.

![BPF Tools Installation](/blog/images/2021/bpf-tools-install.png)

For offensive security this means that it will become more likely that adversaries will leverage already installed tooling to do their deeds. 

This certainly also applies to the even more flexible `bpftrace` tool [discussed previously](/blog/posts/2021/obpf-bpftrace/).

In the offensive security space we call this **"Living off the land"**.

### Living off the land - bpfcc-tools

To get an idea of some cool tools, check out the `bpfcc-tools` suite.

```
sudo apt-get install bpfcc-tools
```

As mentioned even in production environments these tools might already be installed.

The tools in `bpfcc-tools` generally all end with `-bpfcc`. On my Ubuntu install the tools are at `ls /usr/sbin/*bpfcc`, but they might lack this suffix, and older versions of the tools were written in Python.

Let's look at some tools and what they can do for recon and mapping out the homefield.

## Sniffing TLS traffic

Sniffing TLS traffic is a common post exploitation attack angle to get access to sensitive information, such as Cookies or Bearer tokens.

Tracing traffic inside the kernel with existing techniques such as `tcpdump` does typically not lead to desired results (unless you have session keys for TLS - I wrote about this in my book by the way).

What BPF allows now is to create hook points for popular user mode functions that deal with reading/writing the TLS information before (or rigth after) it is encrypted.

A tool that hooks OpenSSL's `PR_Read` and `PR_Write` calls to dump clear text is part of the bpfcc-tools.

```
hacker@server:~$ sudo sslsniff-bpfcc 
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

My goal was to sniff traffic from Firefox or Chrome, but the `sslsniff-bpfcc` tool did not work there. 

Check out my other post about [sniffing some Firefox traffic](/blog/posts/2021/offensive-bpf-sniffing-traffic-bpftrace/) with `bpftrace`.

## What other tools are in this toolset?

Brendan Gregg has a great diagram on his website that shows all the tools and where in the stack they are appliable to:

![BPF Tools](/blog/images/2021/bpf_performance_tools.png)

This is quite the list of observability and performance tools. 

## Reconaissance, Credential Hunting, etc...

The tools in the BFP toolset are powerful and might be used by an adversary (incl. a malicious insider):

* `execsnoop-bpfcc`: shows the path and arguments of all exec calls 
* `opensnoop-bpfcc`: shows all the `open()` syscalls, can filter by UID, PID, etc...
* `tcptracer-bpfcc`, `tcpconnect` and `tcplife`: Recon on TCP connections (source, destination, port)
* `tcpaccept-bpfcc`: who connects to this machine?
* `solisten-bpfcc`: shows any new socket that starts listening
* `ttysnoop-bpfcc`: snoop on other tty sessions (could be quite useful when hunting for credentials)
* `sslsniff-bpfcc`: sniffing TLS traffic (e.g. openssl)
* `inject-bpfcc`: allows to fail kernel calls when certain conditions are met 


*Note:* The binaries/tools might not end with `-bpfcc` on your machine depending on version and how they got installed. Older versions of these tools were Python based for instance.


## Detections and Threat Hunting

For living off the land style attacks, detection is tricky, since it can lead to creation or a lot of false-positives. Generally making sure to get logs off machines to analyze information is important.

Check out the previous post about [detecting offensive BPF usage](/blog/posts/2012/obpf/detections/) as these are applicable.

In particular, it would be interesting to hunt or investigate usage of tools such as `sslsniff` or `ttysnoop`.

Cheers.


# References

* https://www.slideshare.net/brendangregg/systemsscale-2021-bpf-performance-getting-started
* https://www.brendangregg.com/blog/images/2019/bpf_performance_tools.png
