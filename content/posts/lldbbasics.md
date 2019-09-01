---
title: "McPivot and useful LLDB commands"
date: 2019-01-05T21:34:51-07:00
draft: true
tags: [
        "pentesting",
        "red",
        "TTP"
    ]
description: "Understand basic LLDB commands and usage"
meta_title: "Just some notes on how to use lldb on MacOS"
---


Just a list of useful notes when dealing with Macs. I'm pretty new to Macs and there might be other, better solutions to the challenges I had to sovle but these worked for me and I'm learning. :)

## Pivoting between accounts and keychain issues

After pivoting on a target host and elevating to root it seems not possible to gain access to other keychains easily. It requires to know the password of the other account still. Just running
 
    security dump-keychain 
doesn't work over ssh even after su - which kinda makes sense.

It also doesn't work when authenticating via public/private key pair as some articles I read suggested - which also makes sense since the keychain is encrypted using the users password afaik. 

One still has to run

    security unlock-keychain -i 

and enter the password for it to work. It seems that keychain just isn't unlocked over ssh. 

Overall, this is actually pretty neat and a good defense in depth feature, which as I'm learning MacOS has quite a bit. Setting that aside there are a couple of pivoting things I explored and tried out and I'm posting this for my future reference. :)

## Pivoting
1. Updating ~/.bash_profile to wait for the target to open a new terminal.
2. Pivot over to a process of the target and inject code into a running process.


## Inject code into another running process

SIP (System Integrety Protection) makes it difficult to debug other process, even as root.

Turning it off requires at least a reboot and some hands on work on the machine. The good news is, that its typically not that difficult to find some binaries that aren't protected. Haven't figured out yet exactly what makes a binary (it's some mix of entitlement and signature I think). 

So, you can't debug the Google Chrome process, but you might find something else that is running that you can debug.

## Attach
    lldb -n processname
    lldb -p pid


## Expressions and the McPivot

Time to learn about lldb, which I have never used before. Turns out that with expressions one can inject and run code the debugged process. 

That's straight forward and super simple.

    p (void) system("whoami &> /tmp/log.txt")

This stackoverflow post here gave me the basic inspiration.

## Final Notes

Last but not least, there is a difference in regards to keychain access when pivoting into an existing session of a user (e.g. the discussed profile/debugger pivot vs. just ssh) that was observerved. 

I might be lacking the right terminology at this point (not knowing MacOS architecture that well), I'd say that they keyhain (login keychain) is unlocked already in the session pivot case. While when doing an SSH session the keychain is still locked, and it can be unlocked with security unlock-keychain -i. 

Regardless however, the user still(!) has to enter the password when trying to access the sensitive parts of the keychain. That's my take on this - there is more I learned when exfiltrating Chrome cookies and I will write about this soon.
