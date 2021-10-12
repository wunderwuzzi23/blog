---
title: "Sniffing Firefox traffic with bpftrace"
date: 2021-10-12T00:10:16-07:00
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

One of the issues I ran into when trying out `sslsniff-bpfcc` was that it did not work with Firefox or Chrome traffic.

This post is about me learning how to hook user space APIs with `bpftrace` using uprobes.

## Network Security Services Library

The first question I had was what library does Firefox use for TLS? 

I used the `ldd` tool to look at linked libraries.

```
$ ldd  /usr/lib/firefox/firefox
	linux-vdso.so.1 (0x00007ffcf8d50000)
	libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007f2f49f80000)
	libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007f2f49f79000)
	libstdc++.so.6 => /lib/x86_64-linux-gnu/libstdc++.so.6 (0x00007f2f49d60000)
	libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007f2f49c12000)
	libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007f2f49bf7000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f2f49a0b000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f2f4a06e000)
```

Mmmmh. This didn't provide any clue to which SSL library is being used.

A few Google searches revealed that Mozilla has a library called [NSS (Network Security Services)](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/NSS).

The Firefox executable was found at `/usr/lib/firefox/firefox`. In that same directory are NSS and related libraries.

```
$ ls -lha /usr/lib/firefox

[....]
-rw-r--r--   1 root root 234K Sep 30 14:39 libnspr4.so
-rw-r--r--   1 root root 667K Sep 30 14:39 libnss3.so
-rw-r--r--   1 root root 514K Sep 30 14:39 libnssckbi.so
-rw-r--r--   1 root root 199K Sep 30 14:39 libnssutil3.so
-rw-r--r--   1 root root  23K Sep 30 14:39 libplc4.so
-rw-r--r--   1 root root  19K Sep 30 14:39 libplds4.so
-rw-r--r--   1 root root 156K Sep 30 14:39 libsmime3.so
-rw-rw-r--   1 root root  899 Sep 30 14:39 libsoftokn3.chk
-rw-r--r--   1 root root 323K Sep 30 14:39 libsoftokn3.so
-rw-r--r--   1 root root 390K Sep 30 14:39 libssl3.so
[...]
```

My assumption was that Firefox is loading these, rather than generic system libraries. I'm pretty sure now that this is the reason that `sslsniff-bpfcc` is not working with Firefox.

Since I didn't find the library dependencies via `ldd` or `objdump` I used `pldd` to attach to an already running Firefox instance to see what libraries it had loaded:

```
$ sudo pldd 88953 | grep nss
/usr/lib/firefox/libnssutil3.so
/usr/lib/firefox/libnss3.so
[...]
```

Bingo. Now I was sure the local NSS libraries are being used.

### Creating the bpftrace script with uprobes 

Finding the correct function took a while. Using the following line, I searched for clues by dumping symbols:

```
objdump -tT *.so | grep -i write
``` 

I also searched Mozilla's NSS documentation, and found this article about [dumping Zoom traffic](https://confused.ai/posts/intercepting-zoom-tls-encryption-bpf-uprobes) quite useful and interesting.

After identifying `PR_Read` and `PR_Write` as the functions of interest, I wrote up a little `bpftrace` script to hook `PR_Write` in `libnspr4.so`. 

```
uprobe:/usr/lib/firefox/libnspr4.so:PR_Write
{ 
    printf("%s[%d](len=%d): %s (%r)\n", 
        comm, pid, arg2, 
        str(arg1, arg2), 
        buf(arg1, arg2));
}
```

The idea is to just print the buffers that Firefox sends in to these functions. 

I found the arguments and their order in an [older Mozilla archive website](https://www-archive.mozilla.org/projects/nspr/reference/html/priofnc.html#19250). Interestingly the latest NSS documentation's hyperlinks are all broken...

* `comm`: is the name of the process/thread 
* `pid`: is the process identifier
* `arg1`: is the user space buffer
* `arg2`: is the length of the buffer

If you read [my previous post about BPF function hooking](/blog/posts/2021/offensive-bpf-bpftrace-message-based/), you know that for some APIs we have to trace both `uprobe` and `uretprobe` calls in order to access buffers. 

This pattern would have to be followed here as well for the `PR_Read` call. Because only in the `uretprobe` of `PR_Read` the buffer is filled. Since we deal with `PR_Write` this isn't needed here and keeps it quite simple.

## Initial results - seeing traffic!

This `bpftrace` script already gave great results and allowed to understand the details of what’s going on a bit better, here is some output of the BPF program running:

```
$ sudo bpftrace scratch.bt
Attaching 1 probe...
Socket Thread[88953](len=17):  (\x00\x00\x08\x07\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00)
Timer[88953](len=1): M (M)
GeckoMain[88953](len=1): M (M)
Socket Thread[88953](len=148): PRI * HTTP/2.0

SM

 (PRI * HTTP/2.0\x0d\x0a\x0d\x0aSM\x0d\x0a\x0d\x0a\x00\x00\x12\x04\x00\x00\x00\x00\x00\x00\x01\x00\x01\x00\x00\x00\x04\x00\x02\x00\x00\x00\x05\x00\x00@\x00\x00\x00\x04\x08\x00\x00\x00\x00\x00\x00\xbf\x00\x01)
Socket Thread[88953](len=488): GET / HTTP/1.1
Host: example.org
User-Agent: Mozilla/5.0 (X1 (GET / HTTP/1.1\x0d\x0aHost: example.org\x0d\x0aUser-Agent: User-Agent: Mozilla/5.0 (X1)
Web Content[89017](len=1): M (M)
Web Content[89017](len=1): M (M)
[...]
```

Great, there is data! But what are we looking at?

### Initial analysis

There was a lot of information dumped that was not of interest. Here are my initial observations:

* Looking the output of `comm`, I learned that "Socket Thread" is really what I want to filter on
* Many websites support HTTP/2 these days and Firefox uses that. It's a binary protocol - so dumping traffic at this level will not be useful for HTTP/2 web servers. We would need to hook higher level APIs or parse HTTP/2 (which might be challenging within `bpftrace`)
* Most importantly, the `str()` function cuts off our buffer! **Look at GET request to example.org and the `User-Agent`. Notice how it abruptly ends, even though the buffer length (`arg2`) is way over 400 characters long.**

The problem with this approach is that `bpftrace` `str` function only allows to read buffers of up to 60 bytes. There is an environment variable to increase this called `BPFTRACE_STRLEN=n`, but there is an upper boundary of a few hundred.

Dealing with storage limitations on the stack is quite a common problem with BPF programs I have learned already. This goes along the same lines.

The solution I came up is the following: 

```
  $i = (int64)0;
 
  while ($i <= 4096)  //ideally this would be arg2, but Verifier complains
  {       
     printf("%s", str(arg1+$i, 16));  //doesn't read beyond it seems (see appendix for better version though)
     $i = $i + 16;
     if ($i > arg2)
     {
       printf("\n");
       break;
     }
   }
```

This basically loops over the buffer sequentially while printing it out along the way. 

And it does work:

```
$ sudo bpftrace foxsniff.bt 
Attaching 3 probes...
Welcome to Offensive BPF... Use Ctrl-C to exit.

GET / HTTP/1.1
Host: example.org
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:93.0) Gecko/20100101 Firefox/93.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: keep-alive
Upgrade-Insecure-Requests: 1
DNT: 1
Sec-GPC: 1
Pragma: no-cache
Cache-Control: no-cache
```

Pretty cool. This dumps HTTP headers of requests - which is what I wanted.

If you want to dump the `PR_Read` requests you can follow the same approach, but I noticed that most HTTP responses have a Content-Type of `gzip`, so they are also binary and hence not as useful, unless you are drooling for `Set-Cookie` headers in a red team op. :)

Hope this was interesting and useful.

Cheers!

[@wunderwuzzi23](https://twitter.com/wunderwuzzi23)



## References

* [Mozilla Network Security Services](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/NSS)
* [Mozilla NSPR Documentaiton](https://www-archive.mozilla.org/projects/nspr/reference/html/priofnc.html#19250)
* [Sniffing Zoom Traffic Post](https://confused.ai/posts/intercepting-zoom-tls-encryption-bpf-uprobes)


## Appendix

The following is the complete program.

```
#include <net/sock.h>

// Basic demo on how to hook user space APIs. This bpftrace script that 
// traces uprobes for Firefox (NSS) write API and prints out the buffer as string. 
// There is a filter for "Socket Thread".

BEGIN
{
  printf("Welcome to Offensive BPF... Use Ctrl-C to exit.\n");
}

uprobe:/usr/lib/firefox/libnspr4.so:PR_Write
/ comm == "Socket Thread" /
{ 
    $i   = (uint64) 0; 
    $adj = (uint64) 0;

    if ((str(arg1, 14) == "PRI * HTTP/2.0"))
    {
          //HTTP/2 Connection
          //return;
    }

    while ($i <= 4096)  //ideally this would be arg2, but Verifier complains
    {        
      if ((4096 - $i) < 0)
      {
         $adj = 4096 - $i;
      }

      printf("%s", str(arg1+$i, 16-$adj));  
      $i = $i + 16;
      if ($i > arg2)
      {
        printf("\n");
        break;
      }
    }
}

END
{ 
  printf("Exiting. Bye.\n");
}
```
