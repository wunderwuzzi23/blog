---
title: "Backdoor users on Linux with uid=0"
date: 2021-08-30T09:22:40-07:00
draft: true
tags: [ 
    "red", "ttp"
    ]
---

On Unix/Linux users with a `uid=0` are root. This means any security checks are bypassed for them.

An adversary might go ahead and create a new account, or set an existing account's user identifier (`uid`) or group identifier to zero. 



A simple way to do this is to update `/etc/passwd` of an account, or use `usermod -u 0 -o mallory`.

Let's create a new user named `mallory`:

```
wuzzi@saturn:/$ sudo adduser mallory   
[...]
wuzzi@saturn:/$ cat /etc/passwd | grep mallory
mallory:x:1001:1001::/home/mallory:/bin/sh
```

Observe that the user has the uid `1001`.

Next, set the uid to 0 using `usermod`:

```
wuzzi@saturn:/$ sudo usermod -u 0 -o mallory
wuzzi@saturn:/$ cat /etc/passwd | grep mallory
mallory:x:0:1001::/home/mallory:/bin/sh
```

Finally, use the account and observe what happens:

```
wuzzi@saturn:/$ su mallory 
Password: [....]
root@saturn:/# whoami
root
root@saturn:/# 

:)
```

## Detection and Threat Hunting

Depending on your central log collection tools, all you need to do is look for `:0:` in the `/etc/passwd` file for either user or group ids. On the command line on a host this can be done with something like:

```
$ cat /etc/passwd | grep ":0:"
```

The result of this command will look similar to:

![uid 0 backdoor](/blog/images/2021/user-uid0.png)

Neat.

Its unlikely, but if you encounter a BSD system you might actually see two user's with uid 0. One is name `root` and the other one `toor`. So on non-BSD systems a nifty attacker might attempt to trick Unix administrators and hide an additional backdoor user with the name `toor`.

In case you find backdoor users in your environment - I'd be curious to know. 

## Conclusion

Be aware that `uid=0` are root users. It's hardcoded.

One would expect the system to break, but things *seem* to continue working fine. I always assumed that the configuration is not really supported, but malware and malicious users rarely care about supported configurations.
