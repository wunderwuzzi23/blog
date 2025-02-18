---
title: "Terminal DiLLMa: LLM-powered Apps Can Hijack Your Terminal Via Prompt Injection"
date: 2024-12-06T08:00:25-08:00
draft: true
tags: [
     "aiml", "machine learning", "threats", "prompt injection", "llm", "exfil"
    ]

twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "Terminal DiLLMa: LLM-powered Apps That Hijack Your Terminal Via Prompt Injection"
  description: "Discover how LLM apps can hijack your terminal with prompt injections and ANSI escape codes, exposing hidden vulnerabilities and real-world risks. Learn how to protect your tools with actionable security insights."
  image: "https://embracethered.com/blog/images/2024/terminal-dilmma-tn.png"
---

Last week Leon Derczynski [described](https://interhumanagreement.substack.com/p/llm-output-can-take-over-your-computer) how LLMs can output ANSI escape codes. These codes, also known as control characters, are interpreted by terminal emulators and modify behavior.

This discovery resonates with areas I had been exploring, so I took some time to apply, and build upon, these newly uncovered insights.

## ANSI Terminal Emulator Escape Codes

Here is a simple example that shows how to render blinking, colorful text using control characters.

[![blinking demo](/blog/images/2024/term-blink-demo.gif)](/blog/images/2024/term-blink-demo.gif)

[Wikipedia](https://en.wikipedia.org/wiki/ANSI_escape_code) describes it the following way:

> ANSI escape sequences are a standard for in-band signaling to control cursor location, color, font styling, and other options on video text terminals and terminal emulators.

From a security point of view there have been plenty of vulnerabilities in this space.

### Security Concerns - ANSI Bombs, RCE and more

Security research on this topic goes back decades, including `ANSI Bombs`, that reprogrammed keys on the keyboard. Recent talks at security conferences from last year covered CVE's including remote code execution, denial of service,... Here are recent resources about ANSI escape code research:
* [David Leadbeater - DEF CON 31 Terminally Owned - 60 Years of Escaping](https://www.youtube.com/watch?v=Y4A7KMQEmfo&t=55s)
* [David Leadbeater's Blog Post About ANSI Terminal Security](https://dgl.cx/2023/09/ansi-terminal-security)
* [STÖK - Ekoparty: Weaponizing Plain Text: ANSI Escape Sequences as a Forensic Nightmare](https://www.youtube.com/watch?v=spb8Gk9Z09Y&pp=ygUXc3RvZWsgZWtvcGFydHkgbG9nIGFuc2k%3D)

So it's a feature-rich target. And there are a wide range of different terminal emulators, all behaving slightly differently and also supporting a variety of features.

Now, adding LLM Apps to the mix is sure going to spice things up a bit more.

### ESCAPE - LLMs Can Output ANSI Escape Codes!

Thanks to [Leon's discovery](https://interhumanagreement.substack.com/p/llm-output-can-take-over-your-computer), we know now that many LLMs can emit, e.g. the `ESC` (ASCII 27) control character. Similar to creation of [obscure Unicode code points](https://www.bitdefender.com/en-us/blog/hotforsecurity/microsoft-patches-ascii-smuggling-vulnerability-in-recent-security-update), LLMs are a gift that keeps on giving. 

At a high level I see potentially two distinct techniques for creating such output:

1. **Tool Invocation (e.g using Code Interpreter).** I had this realization during the `ASCII Smuggling` research. When I looked at Google Gemini, I noticed the LLM not being able to reliably emit obscure Unicode code points. However, with the help of code execution (that's how Google calls their version of Code Interpreter) it worked like a charm, and the same is true here
2. **Prompting to have the LLM "natively" create special tokens.** This works well, and better prompting techniques and models will increase chances of success. Sometimes it takes a couple of tries though during demos. 

Late in my testing, I discovered that LLMs can also read control characters, which makes it a lot more reliable by just reflecting the characters back.

## Prompt Injection Angle?

So far we don't know if injecting control codes work when delivered via a prompt injection - although, we know it *should* just work. 

### LLM-Integrated CLI Tools and Applications

A typical scenario are LLM-integrated CLI tools, or any other program that prints LLM output to `stdout`, without carefully handling and encoding control characters. Other potential scenarios are LLM tools that read or write to log files. 

### Prompt Injection Test Cases

I was fortunate to have prior notes from research on payloads and capabilities, which provided a good starting point for testing. Back then I got the inspiration when I was at Ekoparty watching STÖK's talk about vulnerabilities with ANSI Escape Codes when doing log analysis.

The test cases are based on David Leadbeater's and STÖK's (and possibly others) content, and I have turned them into prompt injection tests:
* Rendering of flashing text and changing of colors
* Changing of terminal title and beeping
* Moving cursor and clearing the screen
* Rendering of hidden text in the response! 
* Similar, rendering text with \b (backspace) to remove text
* Copy text to the clipboard!
* Denial of Service - repeating a single character n number of times
* Creation of clickable "hyperlinks", which like markdown in web apps could lead to data leakage. There is even a scenario for non-ending links (so everything in the terminal is a link)
* The [macOS Terminal specific DNS vulnerability](https://dgl.cx/2023/09/ansi-terminal-security#apple-terminal-dns-leaks) Still works, allowing zero-click info leakage from anything in prompt context.

With a few tries I got something to work via prompt injection to better understand implications.

## dillma.py - A Demo App

To test this, I created a simple LLM-powered app using Python. The tool allows to specify a system prompt and attach a file to analyze -- all on the command line. All the information is sent to the LLM (using OpenAI GPT-4o November preview) and the response is printed to the terminal. 

Below is a video highlight the use case and concept: The user is summarizing a file. The file, however, came from a malicious actor and contains instructions. Now, the tool is hijacked and inserts ANSI escape sequences to beep and create colored, flashing text.

{{< youtube aTEPsQzdl7s >}}

{{< raw_html>}}
  <br>
{{< /raw_html>}}

Quite interesting.

I added on to that initial script, and built `dillma.py` an app to test attacks & encoding defenses.

Here is a basic demo of a user summarizing a web page:
[![C64 Codes](/blog/images/2024/c64.gif)](/blog/images/2024/c64.gif)

Oops. The page hijacked the LLM App and sent a set of control sequences to the terminal to change the color, move the cursor and clear the screen - and it also beeped. Depending on the terminal feature support varies.

**Now, let's explore some of the more scary scenarios!**

### Data Leakage Via Clickable Links

The Apple terminal does not allow clickable hyperlinks, but the terminal inside VS Code renders them:

[![Rendering of Link with Data Leakage](/blog/images/2024/term-link-leak1.png)](/blog/images/2024/term-link-leak1.png)

The clickable hyperlink contains data from the prompt context, so this demonstrates that it can leak user data. Luckily VS Code asks the user for confirmation before navigating to the URL.

[![Confirmation Box VS Code](/blog/images/2024/link-leak-confirm.png)](/blog/images/2024/link-leak-confirm.png)

Who would have thought that data leakage scenarios seen in LLM web apps via markdown could translate to the terminal world?

*There was actually weirdness I noticed... it seems that the latest OpenAI GPT Preview model is more suspectible to very simple prompt injections vs the non-preview gpt-4o model.*

But there is more!

### Writing Text To The User's Clipboard

It's also possible **to write** data into the user's clipboard. Most terminals I tested with removed the final newline to prevent code execution. But not all of them!

{{< youtube OcMSaPLiUqk >}}
{{< raw_html>}}
  <br>
{{< /raw_html>}}

As seen, this can lead to code execution in certain terminals.

The last video showed the mitigation as well. By providing `-v` the option, `dillma` encodes and prints the control characters visibly. So, they aren't interpreted by the terminal. This is similar ot the `cat -v` command, which is the secure version of `cat`. 

In real-world apps, I think it's best to default to output secure, encoded text when printing LLM results.

### Data Leakage Via DNS Requests

Another known vulnerability (initially described by David Leadbeater last year at DEF CON 31) is, that the macOS Terminal app can be instructed to issue DNS requests via ANSI escape codes. 

This can also be abused when it comes to LLM-integreated applications. 

Let's combine the `DNS Leakage` scenario with the `code execution` approach I described in the intro.

### Gemini: Enhancing Reliability with Code Execution Tools + macOS Terminal DNS Leakage Demo

If an application uses `code execution` it allows for the arbitrary creation of output, which makes creation of ANSI escape codes easier and more reliable.

{{< youtube HxSa3glgPWQ >}}

{{< raw_html>}}
  <br>
{{< /raw_html>}}

This is a realization I first had when testing some LLMs for ASCII Smuggling, where tool invocation can also increase the reliability for creating certain sequences of characters.

## Terminal Friendly Encoding - Caret Notation 

For my own projects I created two functions for Python and Golang code. They encode output to a form of caret notation, so that bad data can't hijack the terminal:

* [Python Version](https://github.com/wunderwuzzi23/terminal-dillma/blob/main/dillma.py)
* [Go Version](https://github.com/wunderwuzzi23/terminalfriendly/)

Feel free to use, adjust as you see fit. No warranty, or liability.

Here is a video that shows a link data leak demo and detailed mitigations:
{{< youtube mb0WSJlx9as >}}

### Additional Ideas For Mitigation

A few ideas come to mind, some were mentioned in the video:

* By default output encode ANSI control characters, if use case needs it add an option to enable
* Allow-listing of wanted characters
* Testing applications end-to-end is super important. During testing, always consider the context and what conditions might be exploitable by untrusted LLM output. 
* There are probably more hidden secrets to be discovered within the token space and end-to-end implications (I said the same after ASCII Smuggling, and here we are).

Also, it's important to remember that these are features of terminals! So, running commands like `cat` or `type` (DOS) on untrusted data or log files could have unintended consequences, even beyond LLMs being involved.

If you didn't know, you can use `cat -v webserver.log` to safely process files.

## A Fun Fact I Learned Reading Random Articles

Did you know that `ESC` is not the preferred way vendors should label the Escape key? [According to an ISO standard](https://en.wikipedia.org/wiki/Esc_key) this symbol is recommended: ⎋

![ESC ⎋](https://upload.wikimedia.org/wikipedia/commons/thumb/d/d4/ISO_7000_-_Ref-No_2029.svg/260px-ISO_7000_-_Ref-No_2029.svg.png)

I do recall having seen this symbol before on some keyboards.

## Conclusion

Decade-old features are providing unexpected attack surface to GenAI applications. So far, we saw Unicode Tags that [allow hidden communication in web applications](https://www.bitdefender.com/en-us/blog/hotforsecurity/microsoft-patches-ascii-smuggling-vulnerability-in-recent-security-update), ...and now ANSI control characters that interfere with operating system terminal behavior. 

It is important for developers and application designers to consider the context in which they insert LLM output, as the output is untrusted and could contain arbitrary data.

At one point during the research I realized that LLMs can also read the ANSI escape codes which makes the prompt injection scenarios easier to exploit by just asking the LLM to reflect the control characters back in the output.

There's likely much more to discover, and, that's why AI is an exciting field to focus on.

Cheers.


## References
* [David Leadbeater - DEF CON 31 Terminally Owned - 60 Years of Escaping](https://www.youtube.com/watch?v=Y4A7KMQEmfo&t=55s)
* [David Leadbeater's Blog Post About ANSI Terminal Security](https://dgl.cx/2023/09/ansi-terminal-security)
* [STÖK - Ekoparty](https://www.youtube.com/watch?v=spb8Gk9Z09Y&pp=ygUXc3RvZWsgZWtvcGFydHkgbG9nIGFuc2k%3D)
* [Leon Derczynski - Substack: LLM Output Can Takeover Your Computer](https://interhumanagreement.substack.com/p/llm-output-can-take-over-your-computer)
* [Wikipedia - ANSI Escape Code](https://en.wikipedia.org/wiki/ANSI_escape_code)
* [Microsoft Patches ASCII Smuggling Vulnerability](https://www.bitdefender.com/en-us/blog/hotforsecurity/microsoft-patches-ascii-smuggling-vulnerability-in-recent-security-update)