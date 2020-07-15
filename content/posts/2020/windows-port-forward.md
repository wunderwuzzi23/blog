---
title: "portproxy - Performing port-proxing on Windows"
date: 2020-07-11T18:18:51-07:00
draft: true
tags: [
        "red", "ttp", "windows"
    ]
---

A technique on Windows that is less known is how to do basic port-proxing and port-forwarding. 

When is this useful?

> Proxing traffic is useful when a process binds on one (maybe the local) interface and you want to expose the traffic on another network interface.

Let's say you have a process that  binds on the loopback interface, but you want to expose it remotely. Or another example is to that there are two network interfaces and you want to expose an endpoint on one interface on the other - think of basic pivoting techniques between networks.

I remember it took me quite a while to figure how to do this on Windows the first time I needed this. If you know Linux, you probably are familiar with the power of `ssh` and it's range of command line options. The good news is that Windows 10 ships with `ssh` - but this post is not about ssh.

In this post we will look at built-in Windows tools such as `netsh` and `portproxy` that can be used.

In Windows this can be done by an Administrator using: 

`netsh interface portproxy` 

The following command shows how this is performed:

``` netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=48333 connectaddress=127.0.0.1 connectport=3389 ``` 

This is the basic setup to configure proxing traffic. Although, remote connections to port 48333 will not be allowed because the firewall blocks them. 


We have to add a new firewall rule to allow `port 48333`. 

How does that work on Windows you might ask?

## Allowing traffic through the firewall

There are multiple ways to update firewall rules in Windows. Besides the UI the following commands might come in handy: 

1. Use netsh to add a new firewall rule:
    ` netsh advfirewall firewall add rule name="Open Port 48333" dir=in action=allow protocol=TCP localport=48333 `
2. On modern Windows machines, this can also be done via PowerShell commands:

    ` New-NetFirewallRule -Name FirefoxRemote -DisplayName "Open Port 48333" -Direction Inbound -Protocol tcp -LocalPort 48333 -Action Allow -Enabled True ` 

Now, it is all setup to connect to from another machine to port 48333. 

It also works between IPv4 and IPv6. Very cool. :)

## Cleaning up and reverting changes 

In order to remove port forwarding and revert to the defaults, we can run the following commands: 

`netsh interface portproxy reset`

Alternatively, there is a delete argument. 

The same applies for opening port 48333 on the machine. The firewall rule can be removed again using the following command:

` netsh advfirewall firewall del rule name="Open Port 48333" `

or if you like PowerShell:

` Remove-NetFirewallRule -Name FirefoxRemote `

That's it.

## Conclusion

In this post we explored some simple commands to expose ports (proxy/forward them) on different network interfaces. Hope the post was informative and useful. 

Feel free to follow or message me on Twitter [@wunderwuzzi23](https://twitter.com/wunderwuzzi23).


## References

The detailed documentation for `netsh interface portproxy` can be found here:
https://docs.microsoft.com/en-us/windows-server/networking/technologies/netsh/netsh-interface-portproxy