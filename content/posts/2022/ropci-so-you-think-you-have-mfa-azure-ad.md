---
title: "ROPC - So, you think you have MFA?"
date: 2022-10-20T08:00:00-07:00
draft: true
tags: [
        "pentest", "red","research","cloud","tool"
    ]   

twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "ropci - So, you think you have MFA?"
  description:  "Abusing ROPC to identify MFA bypass opportunities in Microsoft AAD. Test your own tenant!"
  image: "https://embracethered.com/blog/images/2022/ropci.png"
---

This post will highlight a pattern I have seen across multiple production Microsoft Azure Active Directory tenants which led to MFA bypasses using ROPC. 

**The key take-away: Always enforce MFA! Sounds easy, but there are often misconfigurations and unexpected exceptions. So, test your own AAD tenant for ROPC based MFA bypass opportunities.**

**Github**: https://github.com/wunderwuzzi23/ropci

**Update**: The latest free issue of Pentest Magazine has a [ropci article](https://pentestmag.com/product/pentest-open-source-pentesting-toolkit/). Check it out.
 
![How does an OAuth2 ROPC Request look like](/blog/images/2022/ropci.png)

# What is ROPC?

`Resource Owner Password Credentials` (ROPC) is an OAuth2 authorization grant type (“flow”) defined in [RFC 6749](https://www.rfc-editor.org/rfc/rfc6749.html).

It uses `username` and `password` directly to obtain an access token.

**This is problematic because:**
* The client process sees the user's password.
* It maintains the password "anti-pattern" that OAuth2 otherwise solves. 

The reason it exists is to support a smoother migration from legacy applications to OAuth2.

Microsoft highlights this in their documentation:

![Microsoft ROPC Legacy](/blog/images/2022/ropci-legacy.png)

And [OAuth 2.0 Security Best Current Practice](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics#page-9) states it even more clearly:

![RFC ROPC Legacy](/blog/images/2022/rfc-legacy.png)

So, the take-away is: **ROPC MUST NOT be used**

If you are using Conditional Access Policy, be aware that the "Block legacy authentication" template policy does not block ROPC by default. You have to specify to block "Mobile apps and desktop clients" which seems to block ROPC.

Wait, but what is this ROPC and am I using it?

# How does the ROPC flow look like?

At a high level the `grant_type` in the OAuth2 POST request will contain this parameter:

```
grant_type=password
```

This highlights the `resource owner password credentials` flow.

A complete `ROPC` request looks as follows:

![How does an OAuth2 ROPC Request look like](/blog/images/2022/ropc-post.png)

As can be seen, the request contains `username/password` and the `client_id`. 

Microsoft refers to `client_id` often as `appID` in their internal documentation and UI. There is no `client_secret` with these appIDs - they are also called public clients.

If authentication succeeds, the response will contain an `access_token` amongst other information: 

![How does an OAuth2 ROPC Response look like](/blog/images/2022/ropc-response.png)


## Wait, I don't have ROPC OAuth apps in my tenant!

AAD ships 50+ apps by default that support ROPC and are active (as service principals) in every tenant.

Here are a few examples:
[![How does an OAuth2 ROPC Request look like](/blog/images/2022/ropc-apps-msft-by-default.png)](/blog/images/2022/ropc-apps-msft-by-default.png)                    	

Combined the various applications have permissions such as reading AAD info (users, groups, permissions), reading/sending emails, accessing Azure, access and uploading information to SharePoint, etc.

So, what do I do with this info now?

**From an offensive security perspective, the following questions are interesting:**

* How to test for ROPC exposure?  
* What kind of tests to run? 
* What questions to ask or AD admins?

# Introducing ropci

To aid with testing I wrote `./ropci` which is a Microsoft AAD ROPC assessment and attack tool.

You can download the code and pre-compiled binaries for Windows, Linux and macOS from Github: [https://github.com/wunderwuzzi23/ropci](https://github.com/wunderwuzzi23/ropci)

![ropci tool - Microsoft Azure AAD OAuth2 Assessment Tool](/blog/images/2022/ropci-options.png)

I will do another post to highlight its usage more clearly, but if you go over to the [Github](https://github.com/wunderwuzzi23/ropci) it should have everything needed to get you started!

## Basic Usage

To test a single account of your AAD tenant run:

```
./ropci auth logon -t $YOUR_TENANT -u $YOUR_USER -P --discard-token
```

Where you set or replace `$YOUR_TENANT` and `$YOUR_USER` with the account and tenant to test.

* `-P`: means to prompt for password, rather than reading from config file or command line.
* `--discard-token`: if authentication succeeds, the returned access token will not be stored.

It will look something like this:

```
./ropci auth logon -t contoso.onmicrosoft.com -u test@example.org -P --discard-token
```

* If authentication succeeds, you will see a success message and the retrieved scopes.
* If authentication fails, you will see the AAD error message that was returned.

## Configuration 

To use all features run `./ropci configure`. This will ask you for tenant, username, and password.

Afterwards you can run `./ropci logon` to use the configuration, get a token and run commands:

![ROPC apps](/blog/images/2022/ropci.logon.png)

That's it.

Note: Even if ROPC logon doesn't work and you'd like to use `ropci`, you can attempt devicecode phishing, or just use devicecode auth to get a token for yourself (see `./ropci auth devicecode`).


## Getting offensive

If you get a token, you can use all the other features of ropci to access the AAD tenant, including:

* Accessing Graph APIs and information such as users and groups in the tenant
* Search the user's mailbox, and send mail
* Download or upload files to SharePoint, 
* Call Azure Resource Manager and run commands on VMs
* Enumerate applications and scopes 
* ...

I will write about how to use these features more in upcoming posts. For this post, let's just discuss the basic test to see if there is an ROPC based attack avenue.

## Go and Test ROPC scenarios!

Test your own tenant for these attacks to make sure an adversary can't exploit them:

###  Attempt ROPC logons for your AAD tenant using

These are some of the basic accounts you should try this out for:

1. **Your own user account**
2. **Service accounts**
3. **AAD only accounts** (consider hybrid and federation scenarios)

Its also good to discuss this topic with your IT admins.

### Identify applications that support ROPC (the offsec way)

Check all apps in your tenant (in case you also have custom ones) and attempt an ROPC logon through them and what scopes are retrieved.

1. `./ropci apps list`: Enumerate all OAuth apps in your tenant
2. `./ropci auth bulk`: Test all apps, and review results and granted scopes 

![ROPC apps](/blog/images/2022/ropci.apps.png)


### ROPC Password Sprays

1. An adversary can use ROPC to perform a password spray (`./ropci auth spray`)

fyi: `ropci` doesn't change IPs during sprays. Checkout [TeamFiltration](https://github.com/Flangvik/TeamFiltration) if you need that capability.

# Take-aways for ROPC 

* Always explicitly enforce MFA! 
* Security defaults might not adequately protect your user accounts
* If you block legacy auth via policy, make sure to include "mobile apps and desktop clients"
* Hybrid and federated MFA enforcement can leave "native" AAD accounts vulnerable
* **Test and validate your configurations from an offsec point of view!**
* Some scenarios might remain vulnerable to SFA  => **Should a conscious decisions**
* Know your weaknesses, monitor exposure, and continue mitigating and locking down exposures

Hope this post and tool will help in improving the security posture of AAD tenants.

Greetings.

知己知彼,百戰百勝.


## Related Research and Tools

A lot of relevant and great research and tooling out there, e.g.:

* [AADInternals by @DrAzureAD](https://o365blog.com/aadinternals/) 
* ["Abusing Family Refresh Tokens" by SecureWorks](https://github.com/secureworks/family-of-client-ids-research)
* ROADTools and more recently TeamFiltration,...


## References

* [OAuth RFC](https://www.rfc-editor.org/rfc/rfc6749.html)
* [AAD Internals](https://o365blog.com/aadinternals/)
* [OAuth 2.0 Security Best Current Practice](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics#page-9)
* [Hackers are using this sneaky exploit to bypass Microsoft's MFA](https://www.zdnet.com/article/hackers-are-using-this-sneaky-trick-to-exploit-dormant-microsoft-cloud-accounts-and-bypass-multi-factor-authentication/)
* [ropci on Github](https://github.com/wunderwuzzi23/ropci)
* [Pentest Magazine with ropci aritcle](https://pentestmag.com/product/pentest-open-source-pentesting-toolkit/)