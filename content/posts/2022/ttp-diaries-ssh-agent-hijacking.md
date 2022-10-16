---
title: "TTP Diaries: SSH Agent Hijacking"
date: 2022-10-16T11:30:29-07:00
draft: true
tags: [
        "pentest", "red","ttp"
    ]   

twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "TTP Diaries: SSH Agent Hijacking"
  description:  "Abusing SSH Agent for lateral movement!"
  image: "https://embracethered.com/blog/images/2022/ssh-hijacking.png"
---

There are some neat TTPs that I don't use frequently, and if the time arises, I need to dig up details again. So, I figured to write some of them down, starting with **SSH Agent Hijacking**. 

### What is SSH Agent Hijacking?

Short story, if you have keys added to an SSH Agent an adversary with root permissions can use them. If you forward the SSH Agent to another host, an adversary with root permission on that other host can exploit and leverage your keys as well.

### Jumpboxes 

The typical scenario where this is quite powerful are `jumpboxes`. One can query the SSH Agent's socket of others and then connect the `ssh` client to that socket, and leverage the victim's SSH private keys that way.

Forwarding is typically done by updating the `ssh_config` file to include:

```
    ForwardAgent yes
```

Manually this can be done by using `ssh -A`.


### SSH_AUTH_SOCK

What makes this all possible is the environment variable `SSH_AUTH_SOCK`.

On that `jumpbox` with multiple users logged in using SSH Agent forwarding an adversary can:

1. Query for other user's processes, e.g. via `ps aux ww` or `pstree -p victim`.
2. Take note of a process id (PID) of a shell started by the `sshd`, like for instance `bash`
3. `grep` through the env variables (as root) of the process to find the `SSH_AUTH_SOCK`
4. Use that socket and pivot to the same or another machine as the victim user

That's it.

### Commands 

Command line wise, it looks like this:

![SSH Agent](/blog/images/2022/ssh-hijacking.png)


1. Find the sshd that launched the users shell (e.g. bash).

```
$ pstree -p sarah | grep sshd
```

2. Take note of the PID of the bash process and fill in the PID accordingly when running:

```
$ sudo cat /proc/{PID}/environ | tr '\0' '\n' | grep SSH_AUTH_SOCK 
```

3. If all works you can ssh to other machines where sarah has access to. 

```
$ sudo SSH_AUTH_SOCK={value} ssh sarah@localhost
```



This is a neat TTP that is especially impactful when an adversary gains access to a shared host.

Greetings.


### References

* [Endpoint Protection - SSH and ssh-agent](https://community.broadcom.com/symantecenterprise/communities/community-home/librarydocuments/viewdocument?DocumentKey=dfe66853-a519-4b96-81b6-e7cbbdfc8c53&CommunityKey=1ecf5f55-9545-44d6-b0f4-4e4a7f5f5e68&tab=librarydocuments)
* [MITRE Attack TTP T1563 - SSH Hijacking](https://attack.mitre.org/techniques/T1563/001/)


### Appendix

Just some random notes around launching ssh-agent

When launching `ssh-agent` it will return the commands to the set the environment variable accordingly. This is also the reason why SSH Agent must be launched using an `eval` statement to set the `SSH_AUTH_SOCK` variable.

```
eval $(ssh-agent -s)
```

If you don't do this, you get `Could not open a connection to your authentication agent.` as an error message when trying to list or add keys via `ssh-add`.

```
ssh-add -l
The agent has no identities.
```
