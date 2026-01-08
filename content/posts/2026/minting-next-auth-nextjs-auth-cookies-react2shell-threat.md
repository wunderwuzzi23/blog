---
title: "Minting Next.js Authentication Cookies"
date: 2026-01-08T02:20:58-07:00
draft: true
tags: ["red", "cookies", "ttp"]
twitter:
card: "summary_large_image"
site: "@wunderwuzzi23"
creator: "@wunderwuzzi23"
title: "Minting Next.js Authentication Cookies"
description: "An adversary can mint authentication cookies for Next.js (next-auth) applications and maintain persistent access to the application as any user if they got a hold of the NEXTAUTH_SECRET used to encrypt and sign the auth tokens"
image: "https://embracethered.com/blog/images/2025/next-auth.png"
---

In this post, we'll look how an adversary can mint authentication cookies for Next.js (`next-auth/Auth.js`) applications to maintain persistent access to the application as any user.

The reason this is important is because of `React2Shell`, which is a deserialization vulnerability that allows an adversary to run arbitrary code. Much has been discussed about this vulnerability, and you can read up the original details from the finder [here](https://react2shell.com/).

  

### Exploitation of React2Shell

One of the challenges for defenders is that there is little evidence left behind when exploitation occurs.

Imagine an adversary exploited your application, and ran the `env` command. By doing so, the attacker likely retrieved OAuth (e.g. Entra ID,...) `client_id` and `client_secret` among other environment variables.

Defenders might rotate these OAuth secrets, and maybe that happens even automatically and regularly. However, for a typical Next.js app that uses the `next-auth` library (now also called `Auth.js)`, that is not a sufficient mitigation.

There is another critical environment variable, typically named `NEXTAUTH_SECRET` (or in the latest version just `AUTH_SECRET`), that needs rotation and serves as the standalone secret used to encrypt and authenticate `next-auth` session cookies.

## The NEXTAUTH_SECRET is all you need

The code that creates the authentication cookies is in this [GitHub repo](https://github.com/nextauthjs/next-auth).

The salt used for minting these cookies is the cookie name itself. So, it's pretty easy for an adversary to derive it as well.

## Creating a Next-Auth Cookie Minter

When tackling this, the easiest approach is to just call the exact functions from the `next-auth` library, as that is all that is needed.

Rather than researching this entirely manually, I used AI to go over the code and build a cookie encoder/decoder (those are the words used by `next-auth`) to allow minting and decoding valid cookies.


### A Few More Implementation Details

In particular `next-auth` uses **HKDF (HMAC-based Key Derivation Function)** to derive encryption keys from the secret when encoding/signing JWT cookies. Here's the complete process:

1. Salt Value: That's the session cookie name, e.g. `authjs.session-token`,  `__Secure-authjs.session-token`, `next-auth.session-token`,... (varies by version)
2. Key-Derivation: Using HKDF in `getDerivedEncryptionKey`
3. Encryption: The function `encode` encrypts the JWT using JWE (JSON Web Encryption)

The only input needed should be the `NEXTAUTH_SECRET`, and then cycle through a couple of default salt values that `next-auth` uses and differ between versions as far as I can tell.

**Note:** If you see multiple cookies with numbers at the end, concatenate their values in order (token0 + token1) and pass that single long string to this decoder utility (that should work).

## Source Code For A Token Minter

There isn't much special here, because it invokes the actual `next-auth` code to do all the heavy lifting. Nevertheless, I figured I'd raise awareness of this TTP. I created this a few weeks ago, and don't recall if I used GitHub Copilot or Claude Code right now. But anyhow you can find the basic code it [here](https://github.com/wunderwuzzi23/next-auth-cookie-tool).

## Demonstration Video and Walkthrough

A demonstration of a simple Next.js app to show how cookies can be decrypted and encrypted/signed.

{{< youtube abc >}}

Hope these insights help highlight the importance of regular Auth.js secret rotation.

## Persistent Access to the Application

If you had a public facing website vulnerable to `React2Shell` and you have not updated the `NEXTAUTH_SECRET` itmight mean that an adversary gained access to the `NEXTAUTH_SECRET` and can mint arbitrary authentication tokens.
  
This allows the adversary to impersonate any user, with any role, and maintain persistent access.
  
Ensure all secrets are rotated regularly, including the `NEXTAUTH_SECRET` or the newer `AUTH_SECRET`.
  
## Detection Opportunities
  
- Log the JWT ID on every session and alert on duplicates from different IP addresses
- Identify impossible travel by users (e.g. wildly different locations can be an indicator of abuse)
- Monitor for sessions without corresponding login events in auth logs
- Watch for off-hours access or unusual user-agent strings

## Conclusion

  
Regularly rotate secrets, and build automation that makes this seamless, so the process becomes hands-off and reliable. Pay special attention to the `NEXTAUTH_SECRET`, as we have shown this is the only secret needed for an adversary to maintain persistent access to your application.

Even if your OAuth credentials are safe and rotated, leaving the `NEXTAUTH_SECRET` (or `AUTH_SECRET` in newer versions) unchanged can leave a lasting backdoor.

Cheers.

## References

* [React2Shell (CVE-2025-55182)](https://react2shell.com/)
* [github.com/nextauthjs/next-auth](https://github.com/nextauthjs/next-auth)
* [Next.js](https://nextjs.org/)