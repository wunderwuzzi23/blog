---
title: "Customized Hacker Shell Prompts"
date: 2022-05-28T14:00:54-07:00
draft: true
tags: [
        "pentest", "red", "documentation"
    ]

twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "Customized Hacker Shell Promptsn"
  description:  "Adding date and time to your shell prompts"
  image: "https://embracethered.com/blog/images/2022/ps1.png"
---

As the saying goes, a picture is worth a thousand words.

In order to improve your documentation and uplevel red team and pentest reporting, it's useful to add date and time information to screenshots and [`script` logs](https://www.man7.org/linux/man-pages/man1/script.1.html).

This helps the Blue Team (and yourself) reviewing past activity, reports and when deconflicting activity is required. Depending on the shell that is used there are different ways to go about it. Let's cover three common ones.


## Bash

In Bash adding time information is pretty straight forward, and below example also adds some more color:

```
PS1="[\d \t\[\033[0m\]]\[\033[1;32m\] \u@\h:\[\033[1;34m\]\w\[\033[0m\]\$ \[\e]0;Embrace the Red\a\]"
```
And this is how it looks like:
![Bash PS1](/blog/images/2022/ps1.png)

Implicitly documenting commands with timestamps is pretty neat. The the above example also updates the title of the shell, which is an additional neat thing that can be done.

You can also add this to the `~/.bashrc` file, so every new bash shell gets it. 

## macOS - zsh

On macOS it's possible to do it this way:

```
PS1="[20%D %T] %n@%m %1~ %# "
```

The result is as follows:

```
[2022-05-28 14:17] alice@alice-MAC ~ % 
```

The startup script for zsh is at `~/.zshrc` in case you are wondering where to set this.


## PowerShell

In PowerShell the prompt is controlled by implementing the `Prompt` function.

```
function Prompt { "[$(get-date )] $Env:USERNAME@$Env:COMPUTERNAME  [$PWD] PS> " }
```

The PowerShell documentation is [here](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_prompts?view=powershell-7.2) if you'd like to read up more on it.


## Conclusion

Leverage custom command prompts to improve reporting and logging. 
Share your favorite prompts!

Happy hacking~~~

[@wunderwuzzi23](https://twitter.com/wunderwuzzi23)


## References

* [script man page](https://www.man7.org/linux/man-pages/man1/script.1.html)
* [PowerShell Prompt](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_prompts?view=powershell-7.2)




