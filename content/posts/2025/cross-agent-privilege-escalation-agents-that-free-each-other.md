---
title: "Cross-Agent Privilege Escalation: When Agents Free Each Other"  
date: 2025-09-24T12:20:58-07:00  
draft: true  
tags: ["llm", "agents"]
twitter:  
  card: "summary_large_image"  
  site: "@wunderwuzzi23"  
  creator: "@wunderwuzzi23"  
  title: "Cross-Agent Privilege Escalation: When Agents Free Each Other"  
  description: "Cross-Agent Privilege Escalation: When Agents Free Each Other"
  image: "https://embracethered.com/blog/images/2025/agents-free-each-other.png"  
---

{{< raw_html >}}
<a id="top_ref"></a>
{{< /raw_html >}}

During the [Month of AI Bugs](https://monthofaibugs.com), I described an emerging vulnerability pattern that shows how commonly agentic systems have a design flaw that allows an agent to overwrite its own configuration and security settings. 

This allows the agent to break out of its sandbox and escape by executing arbitrary code.

My research with [GitHub Copilot](/blog/posts/2025/github-copilot-remote-code-execution-via-prompt-injection/), [AWS Kiro](/blog/posts/2025/aws-kiro-aribtrary-command-execution-with-indirect-prompt-injection/) and a few others demonstrated how this can be exploited by an adversary with an indirect prompt injection.

This post goes a step further by showing how one compromised agent can rewrite another agent's config and 'free' it, creating a cross-agent escalation loop. 'Freeing' in this context means that one agent helps another to break out of its sandbox by giving it additional capabilities.

[![agents that free each other](/blog/images/2025/agents-free-each-other.png)](/blog/images/2025/agents-free-each-other.png)


## Agents That Write to Other Agents' Configuration 

The concept is rather simple. Many developers use multiple coding agents on the same codebase. This is to get a second opinion, compare results, or have one agent review code another agent created.

As an example, let's say you use both GitHub Copilot and Claude Code on the same codebase. 

**A few things to know upfront:**
* A coding agent can achieve arbitrary code execution via multiple means: allowlisting commands, adding/modifying MCP server configuration settings on the fly, adding tasks to the VS Code project (via `.vscode/tasks.json`) and similar. If the attack is performed by a third-party attacker via indirect prompt injection it basically Remote Code Execution.
* GitHub Copilot stores a lot of its settings in `.vscode/settings.json`, and there is also a `.vscode/mcp.json` file to configure MCP servers.
* Claude Code stores local MCP configuration settings in `.mcp.json`.
* Most agents act on and can be strongly influenced by writing instructions to `AGENTS.md`, or `.vscode/copilot-instructions.md` and similar.
* Many agents write to files without the user's permission. 

After some of my disclosures, vendors started adding mitigations to not have the agent overwrite configuration settings without the user's permission. However, this still means the agent can write/create other files and folders.

Now that we’ve walked through some of the background information, let's explore how Copilot can "free" Claude Code!

## Overwriting Other Agents' Configuration Settings

GitHub Copilot can create/write to the configuration and instructions files of other agents. 

For instance, it's still possible to create and write to files, such as `.gemini/settings.json`, `.claude/settings.local.json`, and `~/.mcp.json` (Claude's MCP configuration) to execute arbitrary code or allowlist shell commands for other agents

It's also possible to write to files like `CLAUDE.md` or `AGENTS.md`, etc..

So, I think you understand now where this is going.

**This can be leveraged to have agents "free" each other.**

1. An indirect prompt injection hijacks Copilot and makes it write to Claude's MCP config to add a malicious server; it can also update the custom agent instructions for Claude to take specific agents (e.g. `CLAUDE.md`)
2. When the developer uses Claude Code the configuration changes that Copilot made are picked up and it will run arbitrary code
3. Optional: In turn, Claude can now write to Copilot's configuration also and return the favor

This attack chain demonstrates a significant design flaw, indicating that today's agentic systems are already accumulating substantial security debt.

[![agents free architecture design](/blog/images/2025/agent-free-design.png)](/blog/images/2025/agent-free-design.png)

Agents that coordinate and collaborate to achieve malicious objectives seem very plausible to me in the long term, and it will be very difficult to mitigate unless systems are designed with secure defaults.

## Video Walkthrough

Here is a video that explains the scenario in detail and it includes a demo:

{{< youtube EWuoWjWO8i4 >}}

The demo with Copilot freeing Claude is around minute 3 by the way.

## Mitigations and Recommendations

There are a couple of secure defaults that vendors need to consider, and a few things users should be aware of:
- Isolate your agent's configuration to make it less accessible to others
- Do not automatically overwrite or create files, but propose changes first; at least consider not writing to any dot files or folders without user's approval (that seems the most practical approach currently)
- Beware that if you use multiple agents on the same data/code or infrastructure they might interfere with each other
- Indirect prompt injection can exploit such behavior - so be careful with untrusted data!
- Consider least privilege execution for agents

## Responsible Disclosure

It's important to highlight that this exploit chain is not limited to VS Code and GitHub Copilot, but is a generic novel approach that demonstrates how exploitation of one agent can lead to the compromise of another agent. Which, then, in turn reconfigures the first (or other) agent(s), allowing them to run arbitrary code or modify their instruction settings.

Nevertheless, I reported this specific demo to MSRC, but it was not considered severe enough to require immediate security servicing. However, the team might look into improving mitigations.

## Conclusion

The ability for agents to write to each other's settings and configuration files opens up a fascinating, and concerning, novel category of exploit chains. 

What starts as a single indirect prompt injection can quickly escalate into a multi-agent compromise, where one agent "frees" another agent and sets up a loop of escalating privilege and control. 

This isn't theoretical. With current tools and defaults, it's very possible today and not well mitigated across the board.

More broadly, this highlights the need for better isolation strategies and stronger secure defaults in agent tooling. We're moving toward scenarios where multiple agents operate in the same environments, sometimes on the same data and infrastructure. If we don't consider the possibility of agents collaborating, intentionally or not, we're going to see more of these cross-agent attack chains emerge. 

## References

* [Month of AI Bugs](https://monthofaibugs.com)
* [GitHub Copilot - Remote Code Execution via Indirect Prompt Injection (CVE-2025-53773)](/blog/posts/2025/github-copilot-remote-code-execution-via-prompt-injection/)
* [AWS Kiro - Remote Code Execution via Indirect Prompt Injection (No CVE issued by Amazon)](/blog/posts/2025/aws-kiro-aribtrary-command-execution-with-indirect-prompt-injection/) 

