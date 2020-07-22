---
title: "Firefox - Debug Client for Cookie Access"
date: 2020-07-21T11:00:15-07:00
draft: true
tags: [
        "red","blue","cookies","ttp","post-exploitation"
    ]
---

Finally I got to writing some basic tooling for invoking the Firefox debugging API to send commands to the browser and read the responses. This can be useful for grabbing cookies in the post-exploitation phase.

It works for `Windows` and `macOS`, should also work on `Linux`.

![ffcm output filtered by google.com](/blog/images/2020/firefox/output.png)

This technique is probably most useful when we don't have root or the user's credentials to decrypt cookies or can't attach a regular debugger to the browser process.

## Source code

The source and additional information is available on [Github](https://github.com/wunderwuzzi23/firefox-cookiemonster).


## Technical things

The tool is written in `Golang` using concurrent sender/receiver routines. It uses basic `net.Dial` to get a TCP client (`Conn`) to Firefox. 

When the connection is established it sends debug messages as `serialized JSON objects` to setup things and use the `evaluateJSAsync` method to run JavaScript.

After my last blog post I noticed the `Services.cookies.cookies` array which holds all the cookies. So by default it will iterate over and return those.


### Requests 

These are the detailed messages the client sends to the server:

1. `36:{"type":"listProcesses","to":"root"}`
2. `61:{"type":"getTarget","to":"server2.conn0.processDescriptor31"}`
3. `60:{"type":"attach","to":"server2.conn0.parentProcessTarget35"}`
4. `139:{"type":"evaluateJSAsync","text":"Services.cookies.cookies[0].name","to":"server2.conn0.consoleActor36"}`
5. `103:{Type: "substring", Start: 1000, End: 186972, To: "server2.conn0.longstractor23"}`

The `number` before the JSON string is the length of the message. This is needed for proper deserialization by the other party. 

### Sending Javascript code to execute 

The most interesting call is `evaluateJSAsync` which sends the JavaScript command to execute. If the results are large, we have to call `substring` to get all of the results.

### Responses

Responses have the same structure. A typical response from the Firefox debug server looks like this:

`423:{"processes":[{"actor":"server2.conn190.processDescriptor1","id":0,"isParent":true,"traits":{"watcher":true}},{"actor":"server2.conn190.processDescriptor2","id":6816,"isParent":false,"traits":{"watcher":true}},{"actor":"server2.conn190.processDescriptor3","id":10700,"isParent":false,"traits":{"watcher":true}},{"actor":"server2.conn190.processDescriptor4","id":1952,"isParent":false,"traits":{"watcher":true}}],"from":"root"}`

What the tool does is to extract the necessary metadata needed to perform the subsequent requests (like extracting infromation, such as the proper `Actor`).

### Remarks about the implementation

There is likely a better way to implemented this, as Firefox recently (since 78) added a `Network.getAllCookies` Debug API.

The code leverages a single struct for 5 different "kind of" messages to the server. The structure is named `wireMessage` and represents all possible JSON requests/responses. Due to the use of a single message type for all requests/responses it can get a bit messy trying to understand the code. Yay. :)

## Detections and alerting

This is also were detections might come in handy:

* Looking for `-start-debugger-server` command line arguments to Firefox
* Modification of `user.js` and `prefs.js` files, especially modifications to the settings related to enabling remote debugging (see Appendix information for reference)

## Cookie Crimes 
The idea of using a browser's debugging API to access cookies goes to [Cookie Crimes for Chrome](https://github.com/defaultnamehere/cookie_crimes) by @mangopdf.


## Appendix: Remote debugging

Unlike Chrome, Firefox by default doesn't allow debugging. 

To enable remote debugging, the following Firefox settings have to be updated in the profile:

* *devtools.debugger.remote-enabled*
* *devtools.debugger.prompt-connection*
* *devtools.chrome.enabled*

Afterwards launching a new Firefox instance with `-start-debugger-server 9222` will have the debugger enabled. 

Yes, this means that you have to kill existing Firefox instances or wait until the restarts it. The `-new-instance` features is not working by Firefox it seems.

There is more info and how-to's about enabling it in the [README](https://github.com/wunderwuzzi23/firefox-cookiemonster)


## References

* [Firefox Documentation - Remote Debug Protocol](https://docs.firefox-dev.tools/backend/protocol.html)
* [Cookie Crimes](https://github.com/defaultnamehere/cookie_crimes)
* [Remote Debugging with Firefox](https://embracethered.com/blog/posts/2020/cookies-on-firefox/)
* [Post-Exploitation: Abusing Chrome's debugging feature to observe and control browsing sessions remotely](https://embracethered.com/blog/posts/2020/chrome-spy-remote-control/)
* [Cookie Crimes and the new Microsoft Edge Browser](https://embracethered.com/blog/posts/2020/cookie-crimes-on-mirosoft-edge/)


## Final remarks

As always the reminder that pen testing requires authorization from proper stakeholders.