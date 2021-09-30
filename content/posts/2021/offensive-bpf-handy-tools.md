---
title: "Offensive BPF Handy Tools"
date: 2021-10-28T18:14:59-07:00
draft: true
---



### Level 1: Living off the land - bpfcc-tools

To get an idea of some cool tools, check out the `bpfcc-tools` suite.

```
sudo apt-get install bpfcc-tools linux-headers-$(uname -r)
```

In some production environemnts some of these tools are installed to allow troubleshooting and gain tracing insights.

On my Ubuntu install, the tools are at `ls /usr/sbin/*bpfcc`.

For instance there is a tool that hooks OpenSSL `PR_Read` and `PR_Write` calls and dumps the clear text, so you can sniff TLS traffic of `curl` and related tools, or `tcp_connect` to get ideas on targets to pivot to and so forth.

```
wuzzi23@ubuntu:~$ sudo sslsniff-bpfcc 
FUNC         TIME(s)            COMM             PID    LEN   
WRITE/SEND   0.000000000        curl             2781502 73    
----- DATA -----
GET / HTTP/1.1
Host: wuzzi.net
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
Last-Modified: Tue, 03 Nov 2020 18:54:15 GMT
Connection: keep-alive
ETag: "5fa1a757-1ff"
Strict-Transport-Security: max-age=31536000; includeSubDomains
Accept-Ranges: bytes

<!doctype html>
<html>
  <head>
    <title>Embrace The Red - Cybersecurity Attacks and Red Team Strategies</title>
    <meta http-equiv="Refresh" content
----- END DATA (TRUNCATED, 358 bytes lost) -----
```

There is a lot to explore with the existing tools, and when working in production environments, it is actually not uncommon that some of these tools are already installed - making it quite useful for attackers to **live off the land**.