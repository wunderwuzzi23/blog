---
title: "Machine Learning Attack Series: Stealing a model file"
date: 2020-10-10T05:50:21-07:00
draft: true
tags: [
        "machine learning",
        "huskyai",
        "red",
        "ttp"
    ]
---

This post is part of a series about machine learning and artificial intelligence. Click on the blog tag "huskyai" to see related posts. 

* [Overview](/blog/posts/2020/husky-ai-walkthrough/): How Husky AI was built, threat modeled and operationalized
* [Attacks](#appendix): The attacks I want to investigate, learn about, and try out

We talked about creating adversarial examples and "backdoor images" for Husky AI before. One thing that we noticed was that an adversary with model access can very efficiently come up with adversarial examples.

The goal of this post is to look for ways an adversary can gain access to a model. 

At a high level there are multiple ways, but I think they can be distinguised between "direct" and "indirect" approaches. 

1. **Direct approach: Gaining access to the actual model file** -  Compromise systems and hunt for the model file.
1. **Indirect approach: Transfer Learning Attacks and Model Stealing**  - Attacker builds a separate, yet similar model themselves and uses that to create adversarial examples that work against the live systems.

You might think that an indirect approach is far fetched, but to pull off certain attacks one does not need access to the real physical model file that is used by the systems. I was able to trick Imagenet itself, but so far wasn't lucky to transfer created images over to bypass "Husky AI" - will need more testing.

Let's explore these two in a bit more detail.

## Gaining access to a model file

This is the obvious way to steal a model. A red team operation could focus on:

* Searching internal source code repositories for files with an `*.h5` extension. h5 is a commonly used model file format (take a look at [last weeks post about backdooring model files for reference as well](/blog/posts/husky-ai-machine-learning-backdoor-model/)), or
* Gaining access to engineering machines and production systems (phishing, weak passwords, exposed endpoints that allow remote management or code execution, SSH agent hijacking,...)

To keep a good balance in this blog between machine learning specific attacks and regular infrastructure attacks - let's talk about SSH agent hijacking. We will call the attacker Mallory throughout this post.

### Using SSH Agent during attacks

It's common to have jumpboxes or bastion hosts to access production systems. And to make things convinient we setup up SSH Agent to forward connectivity to the keys to be available on other machines when needed.

![SSH Agent Forwarding](/blog/images/2020/sshagentforwarding.jpg)

If Mallory (the attacker) gains access to an engineers laptop, she can leverage any SSH keys the SSH Agent stores to access other hosts - even if the private keys are passphrase protected.

Let's say Mallory gets access to Alice's Windows laptop. First Mallory checks if any SSH private keys are stored on the machine:

```
ls C:\Users\*\.ssh\*
```

The results look promising:

```
PS C:\> ls C:\Users\*\.ssh\*
    Directory: C:\Users\alice\.ssh

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         8/17/2020   6:25 PM           1698 alice-prod.pem
-a----         9/18/2020   1:34 PM           1702 alice-test.pem
-a----          9/4/2020   9:40 PM            413 config
-a----          9/5/2020   5:59 PM           2746 known_hosts
```

Mallory gets excited because there are private keys. Nice. But Alice was smart and applied strong passphrases to the keys. At this point Mallory can try to bruteforce the passphrase using `hashcat` for instance, but that might take very long...

![Access Denied](/blog/images/2020/no-access.jpg)

But there is another avenue to explore, namely SSH Agent!

See, the SSH Agent is often configured and loaded up with keys in order to forward them to other hosts, or through Bastion hosts. This is a way for engineers to not have to enter their passphrase again and again, and also (more importantly) to not have to store copies of keys to the file system of other machines. 

To check if SSH Agent has keys stored right now Mallory runs this command (as Alice):

```
PS C:\> ssh-add -l
2048 SHA256:2HFp2c2vRfGnEPs+e6X6I7+2Iyq3O7I+YQJWUQvBIeZ .\.ssh\alice-prod.pem (RSA)
```

**Bingo!**

This means Mallory can just use `ssh` (without a password or providing an identity file):

```
ssh production-host 
```

**Voila!**

This is a threat that production systems that allow SSH access have to deal with. 

If the compromised host of the engineer runs Linux, and not Windows the commands are basically the same.

#### Mitigation

A mitigiaton is to provision only temporary identities and granting access on a case by case basis can help limit the exposure. This comes with engineering effort to build such a system of course.  But permanently provisioned identities on production hosts are worrisome for exactly that reason. Users can also lock `ssh-add -X` to lock the agent, so it requires a password again to keep the window of opportunity shorter.

### SSH-Agent Hijacking

Its even worse if Mallory gains root access on the Bastion host (jumpbox). Since the Bastion host typically handles many SSH connections, and using SSH Agent Hijacking Mallory (having root access) can then query and leverage all these keys of clients who forward them.

Forwarding keys is usually done via `ssh -A` option or `AllowForwarding=yes` setting in the `ssh_config`.

From the `man ssh-agent` page:

> **SSH_AUTH_SOCK**  When ssh-agent starts, it creates a UNIX-domain socket and stores its pathname in this variable.  It is accessible only to the current user, but is easily abused by root or another instance of the same user.

Further more it states the location for the socket file, nameley `$TMPDIR/ssh-XXXXXXXXXX/agent.<ppid>`.

Running `ssh-agent` shows these details.

All that Mallory needs to do is set that environment variable for `SSH_AUTH_SOCK` and `SSH_AGENT_ID` before running `ssh`:

```
SSH_AUTH_SOCK=SSH_AUTH_SOCK=/tmp/ssh-WMmT2Ee6UNF3/agent.1841171
```

And afterward Mallory can happily type `ssh targethost` (or do it all on same line) and pivot to the next machine.

In the case of Husky AI this allows Mallory who compromised the ML engineers laptop (let's say via phishing) to pivot into production and gain read/write access to the model file.

This is one way to gain access to a model file, and how traditional red teamers would approach this problem. But let's look at some entirely different methods.


### Extracting SSH Private Keys From Windows 10 ssh-agent

The following [blog post](https://blog.ropnop.com/extracting-ssh-private-keys-from-windows-10-ssh-agent/) highlights another way on Windows on how to gain access to the private key material, which is stored DPAPI encrypted in the registry.

That's it for SSH attacks for now.

## Transfer learning, adversarial examples, and model stealing

The indirect approach to steal a model is less obvious if you are new to machine learning. Mallory can build a model offline (either by maliciously querying the target model and/or transfer learning) to create adversarial examples and then try to use those adversarial examples against the target system to find bypasses. 

Researches have shown and discussed that this is possible, e.g. [Knockoff Nets: Stealing Functionality of Black-Box Models](https://arxiv.org/abs/1812.02766) and [Explaining and Harnessing Adversarial Examples](https://arxiv.org/abs/1412.6572) (Fast Gradient Sign Method paper) from Ian Goodfellow also mentions that attacks can be transfered at times.

**Unfortunately, I haven't succesfully done this yet against Husky AI.**

Even though I created adversarial examples that succesfully misclassify images as huskies against ResNet50/MobileNet/ImageNet - those images do not yet fool Husky AI... I still need to research this more and will write a post when I have more information. 

For now I moved on and focus on learning about **GANs (Generative adversarial networks)** with the goal to actually create realistic looking huskies instead - with some amazing results already! 

## Conclusion

That's it for now on gaining access to a model. 

Hope this was interesting, I try to mix content from both traditional red teaming side and machine learning - so the content is hopefully useful and interesting. 

### Appendix 

These are the core ML threats for Husky AI that were identified in the [threat modeling session](/blog/posts/2020/husky-ai-threat-modeling-machine-learning/) so far and that I want to research and build attacks for. 

Links will be added when posts are completed over the next serveral weeks/months.

1. [Attacker brute forces images to find incorrect predictions/labels](/blog/posts/2020/husky-ai-machine-learning-attack-bruteforce/) 
2. [Attacker applies smart ML fuzzing to find incorrect predictions](/blog/posts/2020/husky-ai-machine-learning-attack-smart-fuzz/) 
2. [Attacker performs perturbations to misclassify existing images](/blog/posts/2020/husky-ai-machine-learning-attack-perturbation-external/) 
3. **Attacker gains read access to the model - Exfiltration Attack (this post)**
4. [Attacker modifies persisted model file - Backdooring Attack](/blog/posts/2020/husky-ai-machine-learning-backdoor-model/)
5. Attacker denies modifying the model file - Repudiation Attack
6. Attacker poisons the supply chain of third-party libraries 
7. Attacker tampers with images on disk to impact training performance
8. [Attacker modifies Jupyter Notebook file to insert a backdoor (key logger or data stealer)](/blog/posts/2020/cve-2020-16977-vscode-microsoft-python-extension-remote-code-execution/)


## SSH-Agent info on Windows

I was wondering if this also works with the Windows 10 SSH client. It turns out that Windows is currently using the following named pipe for this. You can find it by running:

```
PS C:\WINDOWS\system32> Start-Service ssh-agent
PS C:\WINDOWS\system32>  [System.IO.Directory]::GetFiles("\\.\\pipe\\") | sls agent

\\.\\pipe\\openssh-ssh-agent
```

## Adding a key to SSH Agent
Initially when Alice added the private key to the SSH Agent, she had to enter the passphrase:

```
alice@alice-ubuntu:~$ ssh-add .ssh/alice-prod
Enter passphrase for .ssh/alice-prod
Identity added: .ssh/alice-prod (alice@alice-ubuntu)
```

## References

* Image "Access Denied" by [Elchinator from Pixabay](https://pixabay.com/photos/no-access-access-denied-monitor-5043758/)