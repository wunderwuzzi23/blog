---
title: "MCP Server for Hosting COM Servers"
date: 2025-05-03T09:36:40-07:00
draft: true
tags: [
     "threats", "ttp", "red", "tools", "llm"
    ]
twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "MCP Server that hosts COM Servers"
  description: "Model Context Protocol -- MCP Server for Hosting COM Servers"
  image: "https://embracethered.com/blog/images/2025/heyyou-pi.jpg"
---

When the [Model Context Protocol](https://modelcontextprotocol.io/introduction) (MCP), a standard for how applications provide context to LLMs, came out it reminded me of the [Common Object Model](https://learn.microsoft.com/en-us/windows/win32/com/the-component-object-model) (COM) from Microsoft. 

`COM` has been around for decades and it's used for programming, sharing of functionality at a binary/object level across languages and hosts, the usage from scripting languages, via DCOM all of this can be done remotely even, and well, it's also useful for red teaming. A lot of software on Windows was implemented as `COM objects`, including Microsoft Office.

Here are some of the similarities of the two standards:
* Interface-based Communication and Abstraction 
* Scripting and Automation
* Integration with External Tools and Services
* Local and Remote Server Options

Very similar things could be said about most RPC based communications system (CORBA, SOAP,...), so MCP isn't really that new in principle at all. And the security issues that come with such RCP communication endpoints are also not new.

Anyhow, we are getting side tracked, but because of these similarities I thought why not build an AI Agent that can host any `COM` object, and then I realized... **why not build an MCP server that abstracts COM - essentially, an MCP server for COM servers.**

Due to the dynamic discovery nature of COM, all that should be needed is the `CLSID` or the `ProgID` to launch, interact and script any COM object.

## Welcome the MCP COM Server

To get the prototype off the ground I did some vibe coding using ChatGPT, Claude, Cursor and VS Code and a few hours of debugging and fixing some basic bugs before I had it working.

The `mcp-com-server` offers these tools

* CreateObject (basically CoCreateInstance, or how it was called in VB was actually CreateObject)
* Get/Set Property 
* InvokeMethod
* QueryInterface
* ListAllHostedServers: This allows to list all the currently active servers that are hosted.

## Challenges

The main challenge I had to create a work-around for was that sometimes calling Invoke or Get/Set Property on a COM object actually returns a new `COM object`. That was a corner case I hadn’t anticipated and it took some time to figure out how to abstract that via MCP, and it's probably a bit hacky at the moment, but it works.


## Security

A core challenge of course is security, as the LLM can basically instantiate any COM server, this means it can open a `Shell.Application` or `FileSystemObject` and perform dangerous operations. As a basic mitigation I added an Allow List for `CLSID`s and `ProgIDs`, where the MCP server will refuse to instantiate and host only allow listed COM objects. This can be expanded to include specific interfaces/methods as well. Overall, like with a lot of MCP servers, it's risky business.

## The Result

The result is pretty impressive, I installed the MCP server with Claude and it can now create, open and edit Excel files, save them, then open Outlook and send them via email.

[![basic interaction com-mcp-server](/blog/images/2025/claude-mcp-com-excel.png)](/blog/images/2025/claude-mcp-com-excel.png)

There is also SAPI COM object that I used to have Claude speak and as you might well be aware of there many other possibilities available with COM.

## Advanced Agent-Driven Windows and Office Automation 

This opens up the door for sophisticated automation, software and agents that can perform precise actions at the API/COM level, rather then depending on clicking on UI elements. 

## Offensive Security!

As said, this is useful for general Windows and Office automation tasks, however my goal for this is of course more in the [red teaming and offensive security space](https://embracethered.com/blog/posts/2021/automating-office-to-achieve-redteaming-objectives/), as it can be used for phishing, lateral movement, and maybe even fuzzing and finding 0-day or UAC bypasses using AI.

## Conclusion

Hope this was an interesting read to demonstrate the power of MCP and how one can build a quite sophisticated Windows automation MCP server that allows to control many aspects of Windows and Office (and many other vendors).

It's always a good idea to implement and use new tech to learn about the risks from first principle, so I always encourage people asking for advice to not just try to break things, but also actually go and build things from scratch. As that helps with brainstorming and gathering ideas for how someone might abuse certain parts, and generally just to see what pitfalls and security bugs might easily get introduced.


Happy hacking.

## References

* [Common Object Model](https://learn.microsoft.com/en-us/windows/win32/com/the-component-object-model)
* [Automating Office To Achieve Red Teaming Objectives](https://embracethered.com/blog/posts/2021/automating-office-to-achieve-redteaming-objectives/)
* [MCP](https://modelcontextprotocol.io/introduction)