---
title: "Hosting COM Servers with an MCP Server"
date: 2025-06-08T22:30:40-07:00
draft: true
tags: [
     "threats", "ttp", "red", "tools", "llm"
    ]
twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "AI powered Windows Automation: Hosting COM Servers with an MCP Server 🤯"
  description: "An MCP Server that can host COM servers for advanced Windows Automation"
  image: "https://embracethered.com/blog/images/2025/com-tn.jpg"
---

When the [Model Context Protocol](https://modelcontextprotocol.io/introduction) (MCP) came out it reminded me of the [Common Object Model](https://learn.microsoft.com/en-us/windows/win32/com/the-component-object-model) (COM) from Microsoft. 

`COM` has been around for decades and it's used for programming, scripting, sharing of functionality at a binary/object level across languages and hosts. Via `DCOM` all of this can even be done remotely, and well, it's also useful for red teaming. A lot of software on Windows was/is implemented as `COM objects`, including Microsoft Office.

**Here are some of the similarities of the two standards:**
* Interface-based Communication and Abstraction 
* Scripting and Automation
* Integration with External Tools and Services
* Local and Remote Server Options

**So, I thought why not build an MCP server for fun that abstracts COM - essentially, an MCP server for Windows and Office Automation.**

## Dynamic Discovery

Due to the dynamic discovery nature of COM, all that should be needed is the `CLSID` or the `ProgID` to launch, interact and script any COM object.

And I wanted an MCP server that can host many COM objects at the same time, a proper host.

## Welcome the MCP COM Server

To get the prototype off the ground, I did some vibe coding and a few hours later, including some intense manual debugging to figure out and fix issues, it worked!

[![basic excel com-mcp-server](/blog/images/2025/claude-com-mcp-excel.jpg)](/blog/images/2025/claude-com-mcp-excel.jpg)

The `mcp-com-server` offers these `COM` like tools:

* `CreateObject` (basically `CoCreateInstance`, in Visual Basic it was `CreateObject`)
* `Get/Set Property` 
* `InvokeMethod`
* `QueryInterface`
* `ListAllHostedServers`: This allows to list all the currently active servers that are hosted.

### Challenges

The main challenge I had was to create a work-around for when calling `Invoke` or `Get/Set Property` on a COM object actually returns a new `COM object`. That corner case was not anticipated until I did testing, and it took time to figure out how to abstract that via MCP, and it's probably a bit hacky at the moment, but it works. There are likely more such edge cases. 

### Security

As you can imagine, instantiating any `COM server` means, the MCP server can open a `Shell.Application` or `FileSystemObject` and perform dangerous operations. 

As a basic mitigation there is an Allow List for `CLSID`s and `ProgIDs`, and the MCP server will instantiate allow listed COM objects. This could be expanded to include specific interfaces/methods as well. Overall, like with a lot of MCP servers, it's risky business. 

But it offered a great learning opportunity. For me to learn about new tech, it's important to actually build something with it - even if it's just for fun.

## The Result - Automating Excel

The result is pretty impressive! After connecting the MCP server with Claude it can create, open and edit Excel files, save them, then open Outlook and send them via email. Pretty cool stuff.

[![basic interaction com-mcp-server](/blog/images/2025/claude-mcp-com-excel.png)](/blog/images/2025/claude-mcp-com-excel.png)

There is also `SAPI` COM object that I used to have Claude speak. There are many other capabilities available with COM.

### Confirmation Dialogs

Claude shows an `Allow`/`Deny` button before invoking custom tools by default. That is a good mitigation to make sure a human remains in the loop. This can be disabled, but also re-enabled in the Claude Settings per MCP tool.


### AI-Driven Windows and Office Automation 

This opens up the door for interesting automation avenues, like AI agents that can perform precise actions at the API/COM level, rather than depending on clicking on UI elements.

### Offensive Security

This is useful for general Windows and Office automation tasks. However in the long run I want to explore how this fits into the red teaming, as [COM has a place there](https://embracethered.com/blog/posts/2021/automating-office-to-achieve-redteaming-objectives/). 

### Claude using Internet Explorer - we got you covered! 🤓

At the very end, I was curious if Claude could resurrect the Internet Explorer on Windows, and it can.

[![frankenclaude](/blog/images/2025/claude-com-frankenclaude.jpg)](/blog/images/2025/claude-com-frankenclaude.jpg)

**We got a Franken Claude!**

## Source Code

After posting about this a few months ago on X, a few followers asked me about the code.

You can find the source code on [GitHub](https://github.com/wunderwuzzi23/mcp-com-server). Keep in mind that this was mainly for me to learn about MCP and implement a server, rather then a serious production project - but the idea probably has potential.

## Conclusion

This demonstrates the power of MCP in combination with COM, and how one can build a quite sophisticated Windows automation MCP server that allows to control many aspects of Windows and Office, and a lot more.

It's always a good idea to implement and use new tech to learn about the risks from first principle. So, I always encourage people asking for advice to not just try to break things, but also actually go and build things from scratch. As that helps with brainstorming and gathering ideas for how someone might abuse certain parts, and generally just to see what pitfalls and security bugs might easily get introduced.

Happy hacking.

## References

* [mcp-com-server GitHub project](https://github.com/wunderwuzzi23/mcp-com-server)
* [Common Object Model](https://learn.microsoft.com/en-us/windows/win32/com/the-component-object-model)
* [Automating Office To Achieve Red Teaming Objectives](https://embracethered.com/blog/posts/2021/automating-office-to-achieve-redteaming-objectives/)
* [MCP](https://modelcontextprotocol.io/introduction)