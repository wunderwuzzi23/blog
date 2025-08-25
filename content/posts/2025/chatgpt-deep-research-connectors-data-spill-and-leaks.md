---
title: "How Deep Research Agents Can Leak Your Data"
date: 2025-08-24T18:03:35-07:00
draft: true
tags: ["llm", "agents", "month of ai bugs"]
twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "How Deep Research Agents Can Leak Your Data"
  description: "When enabling Deep Research an agent might go off for a long period of time and invoke many tools and leak information from one tool to another."
  image: "https://embracethered.com/blog/images/2025/episode24-yt.png"
---

{{< raw_html >}}
<a id="top_ref"></a>
{{< /raw_html >}}

Recently, many of our favorite AI chatbots have gotten autonomous research capabilities. This allows the AI to go off for an extended period of time, while having access to tools, such as web search, integrations, connectors and also custom-built MCP servers.

[![Episode 24](/blog/images/2025/episode24-yt.png)](/blog/images/2025/episode24-yt.png)

This post will explore and explain in detail how there can be data spill between connected tools during Deep Research. The research is focused on ChatGPT but applies to other Deep Research agents as well.

With ChatGPT it's now possible to implement custom ChatGPT Connectors, but they have to follow very specific schema definitions, namely it requires two MCP tools created, `search` and `fetch`. 

When I implemented my first custom Connector, which is technically an MCP server, I got this error when trying to add it to ChatGPT:

[![Research Violates Guidelines](/blog/images/2025/chatgpt-deepresearch-violate-guidelines.png)](/blog/images/2025/chatgpt-deepresearch-violate-guidelines.png)

The server has to implement exactly `search` and `fetch`. Anything else, and the Connector will be rejected for violating guidelines.

With terms like `search` and `fetch` it appears that an agent will only retrieve data, but that doesn't mean that no data-leakage will occur as we will demonstrate in this post.

This research was conducted about two months ago, so a few things might have changed, but the core principles remain true.

## Querying Leaks Data!

One thing that is probably obvious to some, but maybe less obvious to others, is that all these integration and tools that technically in the same trust boundary. 

**What does that mean?**

It means that data accessible to the agent from one source, can be leaked to other sources when it performs deep research queries. 

And if there is an attacker in the loop such leaks can be forced via prompt injection.

## ChatGPT Connectors and Deep Research

The primary goal of this research post is to highlight that there is no trust boundary, and once you add multiple research tools to the research process, the agent can freely invoke tools and may use data from one to query another.

The ChatGPT [documentation](https://help.openai.com/en/articles/11487775-connectors-in-chatgpt) states:

> When you enable a connector, ChatGPT can send and retrieve information from the connected app in order to find information relevant to your prompts and use them in its responses.

This basically hints to this behavior.

### Implementing a Remote MCP Server

To test this out I created a simple remote MCP server named `Remote Matrix` that correctly implements `search` and `fetch` tools to adhere to OpenAI’s specification. 

I also [published the source code](https://github.com/wunderwuzzi23/remote-matrix-mcp) on GitHub. It includes extensive logging, because that was my primary goal to better understand what data is sent by the AI to the MCP server.

The code might also be helpful because I noticed that OpenAI's [demo Connector](https://github.com/kwhinnery-openai/sample-deep-research-mcp) does not correctly implement the correct schema that OpenAI Deep Research highlights in the docs. 


## Data Leakage Scenarios

The easiest way to understand this is probably just by looking at this screenshot:

[![Research Spill Idea](/blog/images/2025/chatgpt-deepresearch-sources.png)](/blog/images/2025/chatgpt-deepresearch-soures.png)

Here you can see we have `Outlook Email`, the `Remote Matrix` and a few other tools selected. This means that data from Outlook can be used in a query to the `Remote Matrix` tool. Remember, a research process often runs for 10+ minutes, so humans are not in the loop during the process.

The documentation highlights this with Web Search:
> ChatGPT can also use relevant information accessed from Connectors to inform search queries when ChatGPT searches the web to provide a response.

And turns out that web search uses both [Bing and Shopify](https://help.openai.com/en/articles/9237897-chatgpt-search). 

[![Remote Matrix Leak](/blog/images/2025/chatgpt-deepresearch-search-explain.png)](/blog/images/2025/chatgpt-deepresearch-search-explain.png)

And as you can see above, your **stored memories might also be sent to tools**.

## Demonstration

For the demo I used this research prompt, and connected the Linear and Remote Matrix tools as sources:
```
Compile a list of all currently open issues and give and 
update in a table form with high level details and status.
```

The research agent first proposes a plan and asks a few follow-up questions to clarify the task. In this case it had asked me if it should search both of the connected sources. 

### Prompt Injection To Force Leakage

To increase the likelihood of data leakage happening I created the following scenarios to hijack the AI Agent and create a confused deputy with an indirect prompt injection. There are a couple of places I tried:
1. **Linear tickets with instructions**

[![Research Malicious Linear Ticket](/blog/images/2025/chatgpt-deepresearch-linear2.png)](/blog/images/2025/chatgpt-deepresearch-tool-linear2.png)
2. **`Remote Matrix` tool with malicious tool description** 

[![Research Malicious Tool Description - Caveman](/blog/images/2025/chatgpt-deepresearch-tool-description.png)](/blog/images/2025/chatgpt-deepresearch-tool-description.png)

In the following demo screenshot you can see that ChatGPT started sending queries to the `Remote Matrix` MCP server in the voice of a caveman, which was part of a prompt injection in the tool description. 😊

[![Research Caveman](/blog/images/2025/chatgpt-deepresearch-caveman.png)](/blog/images/2025/chatgpt-deepresearch-caveman.png)

However, more importantly, ChatGPT sent sensitive information from the Linear tickets to the `Remote Matrix` MCP server as well:

[![Remote Matrix Leak](/blog/images/2025/chatgpt-deep-research-leak.png)](/blog/images/2025/chatgpt-deep-research-leak.png)

As a user you can see the individual steps the agent takes in the activity pane:

[![Research Linear Ticket](/blog/images/2025/chatgpt-deepresearch-details.png)](/blog/images/2025/chatgpt-deepresearch-details.png)

What was interesting to observe was that the agent actually combined the information of the prod service and credentials from multiple tickets into a single query.

### Unintended Prompt Injection

Furthermore, there was an older prompt injection test I had in a Linear ticket that I had used when testing Devin, and that ticket also influenced ChatGPT's research process and **it seemed to start looking for a Slack tool to post a message!**

[![Research Linear Ticket](/blog/images/2025/chatgpt-deepresearch-invoke-tool.png)](/blog/images/2025/chatgpt-deepresearch-invoke-tool.png)

This is how accidents happen. 

## What About Other Deep Research Offerings

The same concepts apply to other research agents with the same design patterns, and connection to tools.

OpenAI actually seems to have considered threats around custom Connectors and requires a specific implementation pattern (e.g. with `search` and `fetch` tools implemented). That means the agent does not get exposed to tools that directly manipulate or delete data.

## Mitigations

* When configuring integrations (tools), be careful which ones you simultaneously enable, as there will be spillover of information (leakage)
* Do not connect sensitive data sources with low integrity data sources in the same research process (e.g. Outlook Email and Random MCP Server from Github), also consider if sharing with Bing/Shopify is within your risk tolerance before enabling "Search"
* Deep Research is autonomous. So, considering a secure setup before launching, is crucial: Only enable sources and tools that you trust and that you feel comfortable that the data might get shared by the agent between various connected tools (Bing, custom tools,...)
* Make sure you are okay with the privacy and data protection policies of various tools you add. Tools may receive data originating from other tool invocations (e.g. ChatGPT uses Bing and Shopify for search), and this can include sensitive information like memory or chat history. 
* If your research agent can connect to any aribtrary tool, be careful exposing data manipulation and deletion tools. OpenAI attempted to mitigate this by requiring a custom tool to have only `search` and `fetch` tools implemented.
* Be careful with third party connectors, as those can have additional side-effects and will also receive data as part of research and prompt injection from tools can influence and hijack AI
* In ChatGPT you can see which tools are invoked and what data is sent to them on the right summary pane. This  gives an approximate idea what the agent is doing.

## Conclusion

This post demonstrates that Deep Research integrations, including ChatGPT Connectors, operate within a shared trust boundary of the agent. This means that data from one source can be sent to other sources by the agent. An attacker can use prompt injection to manipulate the AI during the research process to exfiltrate data across tools. 

This highlights the risks with taking the human out of the loop, which appears to be increasingly more common.

As user carefully connect only trusted data sources and be aware that data might leak from one to the other, and that an adversary could influence this with an indirect prompt injection attack.

The primary purpose of this post was just to demonstrate to myself that leakage between data sources can occur. I think there is a lot more research needed in this space to figure out how an attacker can influence autonomous research agents. 

## Appendix

One of the Linear tickets with the information that was leaked:


[![Research Data Prod Linear Ticket](/blog/images/2025/chatgpt-deepresearch-linear-1.png)](/blog/images/2025/chatgpt-deepresearch-tool-linear-1.png)


## References

* [Remote Matrix MCP Server](https://github.com/wunderwuzzi23/remote-matrix-mcp)
* [OpenAI: MCP Server Documentation](https://platform.openai.com/docs/mcp)
* [ChatGPT Connectors](https://help.openai.com/en/articles/11487775-connectors-in-chatgpt)

