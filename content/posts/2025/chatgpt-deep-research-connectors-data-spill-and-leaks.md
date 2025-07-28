---
title: "How Research Agents Can Leak Your Data"
date: 2025-08-20T19:03:35-07:00
draft: true
tags: [
     "threats", "ttp", "red", "tools", "llm", "agents"
    ]
twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "How DeepResearch Leaks Your Data"
  description: ""
  image: "https://embracethered.com/blog/images/2025/TODO"
---

Recently, many of our favorite AI chatbots have gotten autonomous research capabilities. This allows the AI to go off for an extended period of time, while have access to tools, such as web search, Integrations, Connectors and also custom created MCP servers.

Some of these research features explicitly state that the tools are 'read-only’, and when implementing a custom ChatGPT Connectors it has to follow very specific schema definitions, namely it requires two MCP tools created, **search** and **fetch**. 

[![Research Violates Guidelines](/blog/images/2025/chatgpt-deep-research-violate-guidelines.png)](/blog/images/2025/chatgpt-deep-research-violate-guidelines.png)

Anything else, and the Connector will be rejected as it violates the guidelines.
 

## Querying Leaks Data

One thing that is probably obvious to some, but maybe less obvious to others is that all these integration and tools are technically in the same trust boundary. Meaning any data from one can leak to another, and such leakage can be forced via prompt injection.

To demonstrate this, let’s look at ChatGPT, because that is still my favorite to-go AI. Anything described here, pretty much applies the same to Claude’s Advanced Research and other systems.

## ChatGPT Connectors and Deep Research

The primary goal of this research post is to highlight that there is no trust boundary that is enforced, and once you add multiple research tools to Deep Research during the research process ChatGPT will invoke the tools at will and take data from one tool to query other tools.

### Implementing a Remote MCP Server

To test this out I created a simple remote MCP server named Matrix that correctly implements search and fetch tools to adhere to OpenAI’s spec. 

I also [published the source code](https://github.com/wunderwuzzi23/remote-matrix-mcp) on GitHub. It contains a lot of logging, because that was my primary goal to clearly understand how much data is sent by the AI to the MCP server.

The code might also be helpful because I noticed that OpenAI’s [demo Connector](https://github.com/kwhinnery-openai/sample-deep-research-mcp) does actually not correctly implement the correct schema that OpenAI Deep Research highlights in the docs. 

[![Research Leak](/blog/images/2025/chatgpt-deep-research-password-leak.png)](/blog/images/2025/chatgpt-deep-research-password-leak.png)


### Data Leakage Scenarios

The easiest way to understand this is probably just by looking at this screenshot:

[![Research Spill Idea](/blog/images/2025/chatgpt-deepresearch-spill-idea.png)](/blog/images/2025/chatgpt-deepresearch-spill-idea.png)

Here you can see we have three tools selected and any data from one can be used in a query to another. Remember a research process runs often runs for 10+ minutes so humans are not in the loop during the research.

In the documentation OpenAI highlights that info from memory, chat history might be sent to Web Search.
> ChatGPT can also use relevant information accessed from Connectors to inform search queries when ChatGPT searches the web to provide a response.

That web search uses both [Bing and Shopify](https://help.openai.com/en/articles/9237897-chatgpt-search) apparently.

[![Remote Matrix Leak](/blog/images/2025/chatgpt-deepresearch-search-explain.png)](/blog/images/2025/chatgpt-deepresearch-search-explain.png)

Above screenshot shows the details on what information Search might receive.

### First and Third-Party Connectors

Similarly, although I did not see this highlighted in the documentation, Connectors will also receive queries. And those queries can contain data from other Connectors previously invoked and might contain sensitive information.

## Demonstration

## Prompt Injection To Force Leakage

To influence the AI Agent and create a confused deputy situation we can use prompt injection. There are a couple of places I tried this and it worked:
1. Linear Tickets with Instructions
2. Updated Tool Information

Both cases seemed to have worked. 

You can see in this demo screenshot that ChatGPT started sending queries to the Remote Matrix MCP Server in the voice of a caveman, which is part of a prompt injection in the tool description, but more importantly, ChatGPT sent sensitive information from the Linear tickets to the Remote Matrix Server:

[![Remote Matrix Leak](/blog/images/2025/chatgpt-deep-research-leak.png)](/blog/images/2025/chatgpt-deep-research-leak.png)

Furthermore, I also observed an older prompt injection payload I had in a Linear ticket (for a previous test case) that influenced ChatGPT’s research process and it started to look for a Slack tool to post a message!


## What about other Deep Research offerings

As I mentioned in the intro, the same problem applies to other systems. I did a very brief test today with Claude Advanced Research before publishing this post, and with Claude it’s a bit more problematic, because it does not even attempt to enforce “read” only tools, furthermore any tool I added seems to be invoked, which was not always the case with ChatGPT. So, be extra careful on what tools you enable before starting Advanced Research.


## Mitigations

* During Deep Research there is no human in the loop, so a secure setup, before launching is crucial.
* Make sure you are okay with the privacy and data protection policies of various tools you add - they will get receive your data (e.g. ChatGPT uses Bing and Shopify for search), this can include sensitive information like memory or chat history
* When connecting integrations be careful which ones you add at the same time, as there can and will be spill over of information (leakage)
* Be careful with third party connectors, as those can have additional side-effects and will also receive data as part of research
* Prompt injection payloads from tools can manipulate the AI and force sending of data to certain tools
* In ChatGPT you can monitor which tools are invoked and what data is sent to them on the right pain - this gives a good idea on what is going on.



## Conclusion

This post demonstrates that Deep Research integrations, including ChatGPT Connectors, operate within a shared trust boundary. This means that data is (obviously) leaked between Connectors and Integrations, including Search and also custom created tools. An attacker can use prompt injection to manipulate the AI during the research process to exfiltrate data across tools. 

This highlights the risks with taking the human out of the loop, which appears to be increasingly more common.


## References

[Remote Matrix MCP Server](https://github.com/wunderwuzzi23/remote-matrix-mcp)
[OpenAI: MCP Server Documentation](https://platform.openai.com/docs/mcp)




