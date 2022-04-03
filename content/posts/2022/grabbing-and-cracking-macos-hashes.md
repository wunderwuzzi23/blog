---
title: "Grabbing and cracking macOS hashes"
date: 2022-04-03T10:46:07-07:00
draft: true
tags: [
        "pentest", "macos", "research", "TTP"
    ]

twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "Grabbing and cracking macOS hashes"
  description:  "Let's dive into the details on grabbing hashes from Mac's!"
  image: "https://embracethered.com/blog/images/2022/macos-dscl-hashes-2.png"
---

Information for red teaming macOS and info on real world TTPs are still a bit sparse. That makes it difficult for defenders to know what attackers do on macOS compared to Windows. Some organizations might have a bigger blind spot when it comes to macOS. 


This post describes how an adversary can grab hashes from a macOS machine, how to convert it to a hashcat friendly format and use hashcat to crack it. 

More than once have I seen clients that have the same password for an admin user account provisioned on every Mac. Although, that's not the only scenario grabbing a hash and cracking it can be useful.

Let's dive into the details on grabbing hashes from Mac's!


# Basics - The Local Directory Service

The precious hashes for accounts on macOS are stored in the local directory service. They can be found in `.plist` files at `/var/db/dslocal/nodes/Default/users/*`.

On newer versions of macOS, System Integrity Protection (SIP) prevents root to access the files directly. To check if SIP is enabled use the `csrutil` utility:

```
$ csrutil status
System Integrity Protection status: enabled.
```

Luckily, we don't have to read the files directly though, we can use `dscl` instead. 

As root, you can read the information of other accounts on the system using `dscl`:

```
$ dscl . -read /Users/Bob
```

The output looks like this:

[![reading the user info](/blog/images/2022/macos-dscl-hashes.png)](/blog/images/2022/macos-dscl-hashes.png)

Besides a lot of interesting information about the user, the picture they use, etc. the password hash is what attackers are after. 


# Precious Hashes

You might have noticed in the screenshot that it also includes the password hash. To drill down and just grab the `ShadowHashData` specifically using `dscl` as well:

```
$ dscl . -read /Users/Bob/ dsAttrTypeNative:ShadowHashData
```

This will print out the hash and salt of the account's password:

[![reading the hash](/blog/images/2022/macos-dscl-hashes-2.png)](/blog/images/2022/macos-dscl-hashes-2.png)

Now the goal is to crack them with hashcat. For that the base64 values have to be converted into their hex representation. This is a bit tricky and I recall the first time doing this took me a while, but the the [following post](https://apple.stackexchange.com/questions/220729/what-type-of-hash-are-a-macs-password-stored-in/220863) was quite useful.

This is how the conversion of one of the values of the `SALTED-SHA512-PBKDF2` hash looks like:

```
$ echo ADXA/4Prd48YIy7BhVVhA5gpBD54zK2UkiZdzEKXRTs= | base64 -D  | xxd -p
0035c0ff83eb778f18232ec1855561039828043e78ccad9492265dcc4297453b
```

# Using Hashcat to crack macOS hashes

Now it's time to spin up hashcat and start trying to crack the hash.

```
$ hashcat -m 7100 hash.txt -a 0 wordlist.txt
```

The following screenshot shows how this looks in action:

[![DSCL Hashcat](/blog/images/2022/macos-dscl-hashes-3.png)](/blog/images/2022/macos-dscl-hashes-3.png)


Voila! The password was cracked successfully!


## Authentication Hints

As mentioned earlier there is a lot of other useful info that can be read with `dscl`. 

Apple stores the authentication hint the user created in clear text. So that can be useful as well to build a good wordlist for cracking a specific password:

```
$ dscl . -read /Users/Bob/ AuthenticationHint
Authentication Hint:
The password hint is not encrypted
```

Pretty interesting, although not entirely unexpected that a password hint could be visible to anyone.


# Directory Utility

Finally, if you prefer using the UI rather than the command line tool `dscl` you can hit `Command` + `SPACE` and type `Directory Utility` to bring up Apple's directory utility program. 

This is a super handy tool to explore accounts and Active Directory information if the machine is joined to a Windows domain. You can get the same information around accounts as with `dscl`, including password hints and hashes from the Directory Utility as you can see below:

[![Directory Utility](/blog/images/2022/macos-directory-utility.png)](/blog/images/2022/macos-directory-utility.png)

Hope this was interesting to highlight some of the TTPs that adversaries perform on macOS to gain access to passwords for lateral movement.


Cheers!

[@wunderwuzzi23](https://twitter.com/wunderwuzzi23)


## References

* [Hashcat](https://hashcat.net/hashcat/)
* [What type of hash are Macs password stored in](https://apple.stackexchange.com/questions/220729/what-type-of-hash-are-a-macs-password-stored-in/220863)
