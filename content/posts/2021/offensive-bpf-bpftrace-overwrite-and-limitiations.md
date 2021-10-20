---
title: "Offensive BPF: Overwriting syscall return values and bpftrace limitations"
date: 2021-10-22T01:02:26-07:00
draft: true
tags: [
        "pentest", "red","research","ebpf","blue"
    ]

twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "Sniffing Firefox traffic with bpftrace"
  description:  "Using eBPF in offensive security settings and mitigations"
  image: "https://embracethered.com/blog/images/2021/obpf.png"

---

This post is part of a series about **Offensive BPF** that I'm working on to learn how BPFs use will impact offensive security, malware, and detection engineering. 

Click the ["ebpf"](/blog/tags/ebpf) tag to see all relevant posts.

![Offensive BPF](/blog/images/2021/offensive-bpf.png)


In this series so far we used eBPF in passive "tracing and sniffing" techniques. The next thing I wanted to investigate is what options are available to actually modify data structures during BPF execution.

What I wanted to do is make it impossible for someone to read contents of files under `/tmp/test`. This by preventing calls to the folder to fail, or by modifying the file system contents itself on the fly. 

The later is what I really wanted to achieve long term as it is something I need for Red Teaming Ransomware simulation in the near future.


At a high level, I identified two scenarios that should be possible:

* Overwriting syscall return values
* Overwriting user space buffers to modify file contents on the fly

Since, so far I exclusively used `bpftrace` to write BPF programs, that what I wanted to do in this scenario as well.


## Modifying return values of syscalls

`bpftrace` does allow modificiation of `retval` during execution, but only for `kprobes`.

I found this feature while reading through the [Refernence Guide](https://github.com/iovisor/bpftrace/blob/master/docs/reference_guide.md#20-override-override-return-value).

There are two pre-conditions for using this feature:
1. The kernel has to be compiled using `CONFIG_BPF_KPROBE_OVERRIDE`
2. Only functions tagged with `ALLOW_ERROR_INJECTION()` macro can be traced using an overwrite.


Quickly checking my kernel and build settings:

![OBPF override settings](/blog/images/2021/obpf-config-override.png)

All good to go!

```
$ sudo bpftrace -lv kprobe:*openat*
BTF: using data from /sys/kernel/btf/vmlinux
kprobe:do_sys_openat2
kprobe:__ia32_sys_openat2
kprobe:__x64_sys_openat2
kprobe:__x32_compat_sys_openat
kprobe:__x64_sys_openat
kprobe:__ia32_compat_sys_openat
kprobe:__ia32_sys_openat
kprobe:path_openat
kprobe:__io_openat_prep
kprobe:io_openat2
```

I received the following error when override is not possible for the given kprobe:

```
ioctl(PERF_EVENT_IOC_SET_BPF): Invalid argument
Error attaching probe: 'kprobe:do_sys_openat2'
```

> "One of the major advantages of having an in-kernel BPF sandbox is to never crash the kernel - and allowing BPF programs to just randomly modify the return value of kernel functions sounds immensely broken to me."

This is became apparent when I did my first experiments and crashed the OS:

![OBPF override crash](/blog/images/2021/obpf-config-crash.png)

:)




## References

* https://github.com/iovisor/bpftrace/blob/master/docs/reference_guide.md#20-override-override-return-value
* BPF based Error Injection: https://lwn.net/Articles/740146/






