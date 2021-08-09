---
title: "Using procdump on Linux to dump credentials"
date: 2021-08-09T10:00:20-07:00
draft: true
tags: [
        "tools" ,"red"
    ]
---


I like using `procdump` on Windows. 

It’s quite handy for software development when systems have memory leaks or performance issues, `procdump` allows to set thresholds to trigger creation of a core dump. 

**BUT, it’s also super useful to search processes for secrets and other information.** 

For instance, this one liner will dump the memory of all processes to hard disk and then you can search them as you see fit.

```
Get-Process | % { procdump.exe -ma $_.Id }
```

This is not the most elegant way of course, but I have found this extremely useful at times. 

This post is solely focused on `procdump`, so I wanted to highlight that there is another useful tool on Windows called [`Mimikittenz`](https://github.com/putterpanda/mimikittenz), which utilizes `ReadProcesMemory` to search process memory without creating a file on disk. **But, let's focuson on procdump and Linux!**

## Procdump on Linux!

`procdump` is has also available on Linux for some time now. Although it has not gotten as much attention from the security community and pen testers.

![procdump in action](https://raw.githubusercontent.com/Sysinternals/ProcDump-for-Linux/master/procdump.gif)

In this post I want to highlight some examples on how to use it on Linux to improve your pen testing skills.


## Installation

On Ubuntu the Microsoft keys are not trusted by default, so you can either manually build the project or download and trust Microsoft's keys, and then use `sudo apt` to install. Detailed instructions for various platforms are located [here](https://github.com/Sysinternals/ProcDump-for-Linux/blob/master/INSTALL.md).

### Installing Microsoft's key on Ubuntu 

```
wget -q https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
```

### Installing `procdump`

```
sudo apt-get update
sudo apt-get install procdump
```

### Building `procdump` from scratch

For pen testing the preferred option (if the tool is not yet available) to build it from scratch. Instructions can be found on [`procdump`'s Github repo](https://github.com/Sysinternals/ProcDump-for-Linux/blob/master/INSTALL.md).


## Basic usage and features

When invoking `procdump` specify the `PID` and the tool will do its magic to create a core memory dump.

```
sudo procdump -o /tmp -p 27390
```

Above I also specified the `-o` to write the dump to a specific folder. 

Most features are useful for debugging performance and memory leak kind of issues. For instance, it’s possible to dump the memory when the CPU is under a certain load and so on. 

These features are less interesting for red teamer's nefarious purposes.

It's also possible to use the `imagename` instead of the `PID` with the `-w` options, for instance:

```
sudo procdump -w firefox
```

This will write a core memory dump of the browser to the hard drive, and afterwards one go [credential hunting and look for passwords and cookies](/blog/posts/2020/hunting-for-credentials) inside it.

## Demo Scenario

Let's say the malicious engineer wants to see what another user is up to on a Jumpbox. 

The attacker already pivoted onto that host and now is inspecting the processes and notices that user `Bob` is working in nano. 

### Attacker uses `procdump` on Bob's nano process

```
sudo procdump -w nano
```

This is how the output looks like:
![Attacker dumping nano](/blog/images/2021/procdump.png)

### Attacker searches the core memory dump for interesting information

The next step is to analyze the memory dump, let's to a simple string search and grep:

```
~$ strings nano_time_2021-08-09_10\:23\:42.3197  | grep password
password="
password=helloWorld!
```

Screenshot of the procedure:

![Attacker searching for passwords](/blog/images/2021/procdump_grep.png)

After getting a hold of the process memory, you can use `grep`, `strings` or the previously mentioned [Silver Searcher](/blog/2021/silversearcher-ag) to search through the core dump files.

### What did Bob see work on exactly at that moment?

Finally, as a reference, this is how Bob's session looked like at that point. He was working in `nano`, using it as a scratchpad for passwords, which he does never store onto disk.

![Bob uses nano](/blog/images/2021/procdump_nano.png)

The interesting thing is that many developers and engineers do exactly as Bob did. They use ephemeral, non-persistent notepad files, etc. to temporarily hold credentials.

## Conclusion

`procdump` is quite neat and useful. Hope this mini-intro was helpful to learn more about `procdump` and its existance on Linux.


Cheers,
[@wunderwuzzi23](https://twitter.com/wunderwuzzi23)


## References

* [Microsoft Github Repo - ProcDump for Linux](https://github.com/microsoft/procdump-for-linux)
* [Mimikittenz](https://github.com/putterpanda/mimikittenz)
