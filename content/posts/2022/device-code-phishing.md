---
title: "Device Code Phishing Attacks"
date: 2022-11-21T00:00:33-08:00
draft: true
tags: [
        "phishing","red", "ttp", "ropci"
    ]

twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "Device Code Phishing for Access Tokens"
  description:  "Hardware tokens and Yubikeys will not protect you from device code phishing."
  image: "https://embracethered.com/blog/images/2022/device-code-phish.png"
---


As more organizations move to hardware tokens and password-less auth (e.g. Yubikeys, Windows Hello,...) attackers will look for other ways to to trick users to gain access to their data. 

One such technique is by using the [OAuth2 Device Authorization Grant](https://www.rfc-editor.org/rfc/rfc8628).

# Attacker initiates the phishing flow

The attacker starts a Device Code flow by issuing a request to the device code token endpoint (e.g. `https://login.microsoftonline.com/{tenant}.onmicrosoft.com/oauth2/v2.0/devicecode`). 

[ropci](https://github.com/wunderwuzzi23/ropci) has this feature built in, but it can be issued with `curl` or `Burp`.

`./ropci auth devicecode`

[![ropci](/blog/images/2022/device-code-phishing-attacker.png)](/blog/images/2022/device-code-phishing-attacker.png)

The command returns the `URL` and a `Code` to complete the authorization. 

## Attacker sends the phishing email to the victim

The attacker grabs the `Code` puts it into a phishing email:

[![Device Code Phishing Email](/blog/images/2022/device-code-phishing.png)](/blog/images/2022/device-code-phishing.png)

To make the phish more realistic, one can add additional query parameters, like a `docid` above.

## Victim falls for the phish

There are a couple of steps the victim has to complete in order to grant authorization:

[![Device Code Phishing Steps](/blog/images/2022/device-code-phish.png)](/blog/images/2022/device-code-phish.png)

The above images shows the individual steps:

1. Click the link in the email and enter the device code
2. Login to Microsoft Online (in many cases the user might already be logged in)
3. Authorize the OAuth app (in this example Office, which is the default client_id ropci uses)
4. Success

Completing these steps is necessary before an access token will be issued.

## Attack receives the access token

Once the victim completes the OAuth flow and grants access, the attacker receives the access token! 

[![ropci success - stealing/obtaining access token via device code phishing](/blog/images/2022/device-code-phishing-attacker-success.png)](/blog/images/2022/device-code-phishing-attacker-success.png)


The attacker can now use `ropci` to read the users email, search mail/SharePoint, send mail, read info from all users/groups/apps/devices in the AAD tenant, basically anything that [Microsoft Graph API](https://learn.microsoft.com/en-us/graph/use-the-api) allows.


# Attacker limitations

There is typically a short time window while the device code is valid, which the attacker has to leverage. This means attack campaigns will be automated at scale to increase chances of success (requesting of token and automatic sending of mails).

# Mitigations

1. The easiest is likely to entirely disable the Device Code flow in your organization, unless there is a true business need to support it.
1. Keeping the time window short to ensure an attack succeeds are unlikely.
2. Reviewing and monitoring the device code flow initiator and the host that grants access. Both of these hosts/devices should likely be coming from a similiar IP address. 




# References

* RFC -  OAuth 2.0 Device Authorization Grant: https://www.rfc-editor.org/rfc/rfc8628
* ropci - ROPC Attack tool (https://github.com/wunderwuzzi23/ropci)
* [Graph API](https://learn.microsoft.com/en-us/graph/use-the-api)