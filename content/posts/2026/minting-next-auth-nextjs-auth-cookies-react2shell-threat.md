---
title: "Minting Next.js Authentication Cookies"
date: 2026-01-14T23:58:58-07:00
draft: true
tags: ["red", "cookies", "ttp"]
twitter:
card: "summary_large_image"
site: "@wunderwuzzi23"
creator: "@wunderwuzzi23"
title: "Minting Next.js Authentication Cookies"
description: "An adversary can mint authentication cookies for Next.js (next-auth) applications and maintain persistent access to the application as any user if they got a hold of the NEXTAUTH_SECRET used to encrypt and sign the auth tokens"
image: "https://embracethered.com/blog/images/2026/next-auth.png"
---

In this post, we'll look how an adversary can mint authentication cookies for Next.js (`next-auth/Auth.js`) applications to maintain persistent access to the application as any user.

[![Minting NextAuth authentication tokens and cookies](/blog/images/2026/next-auth.png)](/blog/images/2026/next-auth.png)

The reason this is important is because of `React2Shell`, which is a deserialization vulnerability that allows an adversary to run arbitrary code. Much has been discussed about this vulnerability, and you can read up the original details from the finder [here](https://react2shell.com/).

## Exploitation of React2Shell

A challenge for defenders is that there is little evidence left behind when exploitation occurs.

Imagine an adversary exploited your application, and ran the `env` command. By doing so, the attacker likely retrieved OAuth (e.g. Entra ID,...) `client_id` and `client_secret` among other environment variables.

## Mandatory Secret Rotation

Defenders might rotate these OAuth secrets, and ideally it happens automatically and regularly. However, for a typical Next.js app that uses the `next-auth` library (also called `Auth.js)`, that is not a sufficient mitigation.

There is another critical environment variable, typically named `NEXTAUTH_SECRET` (or in the latest version just `AUTH_SECRET`), that needs rotation and serves as the standalone secret used to encrypt and authenticate `next-auth` session cookies.

## The NEXTAUTH_SECRET is all you need

The `next-auth` code that creates the authentication cookies is in this [GitHub repo](https://github.com/nextauthjs/next-auth). The salt used for minting these cookies is the cookie name itself. So, it's pretty easy for an adversary to derive it as well.

## Creating a Next Auth Cookie Minter (Code)

When tackling this, the easiest approach is to call the functions from the `next-auth` library. 

Rather than researching this manually, I used AI to go over the code and build a cookie encoder/decoder (those are the terms used by `next-auth`) to allow minting and decoding valid cookies.

There isn't anything special here. Nevertheless, I figured to share to raise awareness of this TTP. 

I created this a few weeks ago, and just put the source code up [here](https://github.com/wunderwuzzi23/next-auth-cookie-tool).

See the [Appendix](#appendix) for the tool usage and command line options.

## Demonstration Video and Walkthrough

If you prefer to watch a video with details. This is a demo of a basic Next.js app to show how cookies can be decrypted and encrypted/signed.

{{< youtube Imv_7Es1WHQ >}}

Hope these insights help highlight the importance of regular Auth.js secret rotation.

### A Few More Implementation Details

In particular, `next-auth` uses **HKDF (HMAC-based Key Derivation Function)** to derive encryption keys from the secret when encoding/signing JWT cookies. Here's the complete process:

1. Salt Value: That's the session cookie name, e.g. `authjs.session-token`,  `__Secure-authjs.session-token`, `next-auth.session-token`,... (varies by version)
2. Key-Derivation: Using HKDF in `getDerivedEncryptionKey`
3. Encryption: The function `encode` encrypts the JWT using JWE (JSON Web Encryption)

The only input needed should be the `NEXTAUTH_SECRET`, and then cycle through a couple of default salt values that `next-auth` uses and differ between versions as far as I can tell.

**Note:** If you see multiple cookies with numbers at the end, concatenate their values in order (token0 + token1) and pass that single long string to this decoder utility (that should work).

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

## Appendix

### Cookie Creator

Create a signed session cookie with example data:

```bash
node cookie-creator.js
```

**Note:** To create cookies with custom session information for your specific application, edit the example session data in [cookie-creator.js](cookie-creator.js:202) (around line 202). Update the fields like `name`, `email`, `sub`, and any other custom claims to match your application's session structure.

#### Custom Cookie Name

You can specify a custom cookie name using the `--cookie-name` argument:

```bash
node cookie-creator.js --cookie-name 'authjs.session-token'
```

This is useful when working with different NextAuth versions or custom configurations.

#### Output

This will output:
- The cookie name (default: `next-auth.session-token` or your custom name)
- The encrypted JWT token
- A complete Set-Cookie header string
- Expiration information

#### Example Output

```
🔐 NextAuth Cookie Creator Tool

📝 Creating session cookie with example data:
{
  "name": "John Doe",
  "email": "john@example.org",
  "sub": "user-123",
  "picture": "https://wuzzi.net/h.png"
}

✅ Cookie created successfully!

Cookie Name: next-auth.session-token

Encrypted Token (JWT):
eyJhbGciOiJkaXIiLCJlbmMiOiJBMjU2Q0JDLUhTNTEyIiwia2lkIjoiU3N3dTVtM0FK...

Full Cookie String (for Set-Cookie header):
next-auth.session-token=eyJhbGci...; HttpOnly; SameSite=lax; Path=/; Max-Age=2592000

Expires in: 2592000 seconds ( 30 days)
```

### Cookie Decoder

Decode and verify an existing session cookie:

```bash
node cookie-decoder.js 'eyJhbGci...'
```

#### Example Output

```
🔓 NextAuth Cookie Decoder Tool

🔍 Trying salt: "next-auth.session-token"...
✅ Token decoded successfully with salt: "next-auth.session-token"!

Cookie Name (Salt Used): "next-auth.session-token"

Session Data:
{
  "name": "John Doe",
  "email": "john@example.org",
  "sub": "user-123",
  "picture": "https://wuzzi.net/h.png",
  "iat": 1734559200,
  "exp": 1737151200,
  "jti": "a1b2c3d4-e5f6-7g8h-9i0j-k1l2m3n4o5p6"
}

Expiry Information:
  Expires at: 2025-01-17T12:00:00.000Z
  Status: ✅ Valid
  Time remaining: 29 days, 23 hours
```