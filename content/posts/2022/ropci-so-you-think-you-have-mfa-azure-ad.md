---
title: "ROPC - So, you think you have MFA?"
date: 2022-10-18T10:30:29-07:00
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

This post will highlight systemic pattern I have seen across multiple production Microsoft Azure Active Directory tenants which led to MFA bypasses using ROPC. 

**The key take-away upfront: Test your own Microsoft AAD tenant for ROPC based MFA bypass opportunities.**
 
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

If you are using Conditional Access Policy, be aware that the "Block legacy authentication" template does not block ROPC!

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


## Wait, I don't have ROPC OAuth apps in my AAD tenant!

AAD ships 50+ applications by default that support ROPC and are active in every tenant.

Here are a few examples:
[![How does an OAuth2 ROPC Request look like](/blog/images/2022/ropc-apps-msft-by-default.png)](/blog/images/2022/ropc-apps-msft-by-default.png)                    	

Combined the various applications have permissions such as reading AAD info (users, groups, permissions), reading/sending emails, accessing Azure, access and uploading information to SharePoint, etc.

So, what do I do with this info now?

**From an offensive security perspective, the following questions are interesting:**

* How to test for ROPC exposure?  
* What kind of tests to run? 
* What questions to ask or AD admins?