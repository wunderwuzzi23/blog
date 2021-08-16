---
title: "Machine Learning Attack Series: Repudiation Threat and Auditing"
date: 2020-11-10T16:00:21-07:00
draft: true
tags: [
        "machine learning",
        "huskyai",
        "red",
        "ttp",
        "blue"
    ]
---

This post is part of a series about machine learning and artificial intelligence. Click on the blog tag "huskyai" to see related posts. 

* [Overview](/blog/posts/2020/machine-learning-attack-series-overview/): How Husky AI was built, threat modeled and operationalized
* [Attacks](#appendix): The attacks I want to investigate, learn about, and try out

In this post we are going to look at the "Repudiation Threat", which is one of the threats often overlooked when performing threat modeling, and maybe something you would not even expect in a series about machine learning.


## What is Repudiation?

Repudiation is the threat that someone denies having performed an action. 

For example, in the case of Husky AI the attacker Mallory replaces the original machine learning model file with a backdoored one, but Mallory just ends up denying having done such a thing!

[![Audit](/blog/images/2020/audit.png)](/blog/images/2020/audit.png)

How can we add proper auditing to better understand who and when such an action was performed? And how can we have proof which account indeed updated the file, or at least support an investigation to help put the pieces together to uncover the truth.

## Auditing, Centralized Monitoring and Notifications

A practical way is to leverage `auditd` in Linux, and push log files up into a centralized monitoring system. 

Companies typically use products such as Splunk, the Elastic Stack, or Azure Sentinel (to name a few systems) that can help logging, auditing, analyze and visualize the audit data. Research which product your organization is using and then integrate accordingly.

I will discuss the Linux Audit Daemon `auditd` in this post, which is what I am using with Husky AI to monitor file access.

## Audit Daemon - auditd

To see if `auditd` is installed already on your Linux (Ubuntu) machine you can run:

```
sudo apt list --installed | grep auditd
```

And it should show something like this if it's installed:

```
auditd/bionic,now 1:2.8.2-1ubuntu1 amd64 [installed]
```

If not, it’s easy to install using:

```
sudo apt install auditd
```

Now `auditd` should be started. You can double check by running `sudo service auditd status`. If for some reason is not running, it can be started using `sudo service auditd start`.

## Monitoring a specific file 

Auditd is configured using rules to determine what files and what kind of access will be audited and logged. 

With `auditctl` you can look at the currently configured rules:

```
sudo auditctl -l

No rules
```

Now, to add files for auditing you can either modify the configuration file of auditd (e.g `sudo cat /etc/audit/rules.d/audit.rules`), or use the `auditctl` command.

Let's monitor the model file:

```
sudo auditctl -w /var/www/huksyai/models/huskymodel.h5 -p rwa -k huskyai
```

* -w specifies the path of the file to audit
* -p specifies what operations should be audited, in this case read, write and append
* -k tells auditd to tag each audit entry with the provided string. This is useful for searching

Now we are ready to simulate someone accessing the file:

```
cp /var/www/huskyai/models/huskymodel.h5 /tmp/
```

And using utility `ausearch` we can take a look at audit events:

```
sudo ausearch -i -k huskyai 
```

The `-i` argument interprets the user ids with the actual account name to make it human readable, and `-k` filters by the keyword used to tag events.

The results look like the following:

```
type=PROCTITLE msg=audit(11/10/20 20:09:44.150:414) : proctitle=cp /var/www/huskyai/models/huskymodel.h5 /tmp/exfil 
type=PATH msg=audit(11/10/20 20:09:44.150:414) : item=0 name=/var/www/huskyai/models/huskymodel.h5 inode=256232 dev=ca:01 mode=file,664 ouid=ubuntu ogid=ubuntu rdev=00:00 nametype=NORMAL cap_fp=none cap_fi=none cap_fe=0 cap_fver=0 cap_frootid=0 
type=CWD msg=audit(11/10/20 20:09:44.150:414) : cwd=/var/www/huskyai/models 
type=SYSCALL msg=audit(11/10/20 20:09:44.150:414) : arch=x86_64 syscall=openat success=yes exit=3 a0=0xffffff9c a1=0x7ffd37ab0466 a2=O_RDONLY a3=0x0 items=1 ppid=6007 pid=6914 auid=ubuntu uid=ubuntu gid=ubuntu euid=ubuntu suid=ubuntu fsuid=ubuntu egid=ubuntu sgid=ubuntu fsgid=ubuntu tty=pts0 ses=108314 comm=cp exe=/bin/cp key=huskyai 
```

As can be seen the audit entry contains time, username and the details of the operation, in this case `/bin/cp`. This information now can be correlated with logon times, and other data in case a real breach of the system occurred.

Pretty cool. 

### Monitoring directories

You can also monitor an entire directory not just a single file, e.g. to monitor all files in a directory add the following line to the `audit.rules` file:

`-w /var/www/huskyai/`

Quite simple.

## Offloading events to a different machine

In a production environment you want to offload audit events quickly to a remote machine for security reasons. This is because an adversary might try to hide their tracks by deleting or tampering audit information. 

If you are interested to learn more look at Filebeat and [Auditbeat](https://www.elastic.co/beats/auditbeat) from Elastic for instance. Also, my book about [Red Team Strategies](https://www.amazon.com/gp/product/1838828869/ref=as_li_tl?ie=UTF8&tag=wunderwuzzi-20&camp=1789&creative=9325&linkCode=as2&creativeASIN=1838828869&linkId=07bfd6b729fbc2b2904160e0e16c337f) contains more details on how to setup auditing - and also how to setup Honeypot files to trick adversaries!

## Audit Dispatcher - Notifications

The audit framework can be extended with custom dispatchers!

If you are interested to learn more or build one that sends email notifications, check out my Github repo [audisp-sentinel](https://github.com/wunderwuzzi23/audisp-sentinel/).

Notifications on critical assets are important to provide insights and highlight misuse. A practical way is also to have the blue team create summary reports with activity and share that with engineers, so they can spot potential misuse.


## Conclusion

This was a brief introduction to auditing to highlight its importance to mitigate repudiation threats. We walked through the setup of `auditd` and monitor read and write access to the model file. 

Hope it was useful. Cheers.


Twitter: [@wunderwuzzi23](https://twitter.com/wunderwuzzi23)


### Appendix 

These are the core ML threats for Husky AI that were identified in the [threat modeling session](/blog/posts/2020/husky-ai-threat-modeling-machine-learning/) so far and that I want to research and build attacks for. 

Links will be added when posts are completed over the next several weeks/months.

1. [Attacker brute forces images to find incorrect predictions/labels](/blog/posts/2020/husky-ai-machine-learning-attack-bruteforce/) 
2. [Attacker applies smart ML fuzzing to find incorrect predictions](/blog/posts/2020/husky-ai-machine-learning-attack-smart-fuzz/) 
2. [Attacker performs perturbations to misclassify existing images](/blog/posts/2020/husky-ai-machine-learning-attack-perturbation-external/) 
3. [Attacker gains read access to the model](/blog/posts/2020/husky-ai-machine-learning-model-stealing.md/) 
4. [Attacker modifies persisted model file - Backdooring Attack](/blog/posts/2020/husky-ai-machine-learning-backdoor-model/)
5. **Attacker denies modifying the model file - Repudiation Attack (this post)**
6. Attacker poisons the supply chain of third-party libraries 
7. Attacker tampers with images on disk to impact training performance
8. [Attacker modifies Jupyter Notebook file to insert a backdoor (key logger or data stealer)](/blog/posts/2020/cve-2020-16977-vscode-microsoft-python-extension-remote-code-execution/)
9. [Attacker uses Generative Adversarial Networks to create fake husky images](/blog/posts/2020/machine-learning-attack-series-generative-adversarial-networks-gan/)

## References

* [Elastic Auditbeat](https://www.elastic.co/beats/auditbeat)
* [Auditd man page](https://linux.die.net/man/8/auditd)
* [Free image from Pixabay](https://pixabay.com/illustrations/audit-report-verification-magnifier-4576720/) - no attribution required, still giving a shout out. I modified the original image.
