---
title: "gospray - Simple LDAP bind-based password spray tool"
date: 2022-09-18T08:00:01-07:00
draft: true
tags: [
        "pentesting",
        "red",
        "tool"
    ]
twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "gospray - Simple bind-based password spray tool"
  description:  "On a network and need credentials? Try password spraying the domain controller directly."
  image: "https://embracethered.com/blog/images/2020/hacker.png"
---

On a network and need credentials?  Try password spraying the domain controller directly. 

A few years ago, I wrote this password spray tool called `gospray` that was used succesfully in a couple of engagements since. It does an LDAP bind directly against the domain controller to validate credentials. This doesn't require an SMB server (or other servers) as target. So, it's pretty quiet and number of concurrent Go routines is configurable. :)

Check it out on Github: [GoSpray](https://github.com/wunderwuzzi23/GoSpray)

## High Level Features

At a high level the latest version supports two testing modes:

1. **Password Spray:** If both `-accounts` and `-passwords` command line arguments are specified, then a spray will be performed.
2. **Password Validation Mode:** providing the `-validatecreds` command line option is for validation. The file specified with `validatecreds` is parsed line by line, each line is split by colon (:) to retrieve `username:password`. Afterwards an authentication attempt will be performed against specified domain controller.

By default it waits 10 seconds after a round (per horizontal password round) - but you might want to adjust that depending on the number of accounts, users and lockout policies.

Happy hacking.

**Note:** Be careful about account lockout policies (know what you do!) and make sure you have authorization from appropriate stakeholders before engaging in this kind of testing.
