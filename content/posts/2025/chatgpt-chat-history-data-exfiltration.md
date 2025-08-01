---
title: "Exfiltrating Your ChatGPT Chat History and Memories With Prompt Injection"
date: 2025-08-01T08:00:58-07:00
draft: true
tags: ["llm", "agents", "month of ai bugs"]
twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "ChatGPT: Stealing Your Chat History and Memories With Prompt Injection"
  image: "https://embracethered.com/blog/images/2025/episode1-yt.png"
---

{{< raw_html >}}
<a id="top_ref"></a>
{{< /raw_html >}}

In this post we demonstrate how a bypass in OpenAI's "safe URL" rendering feature allows ChatGPT to send personal information to a third-party server. This can be exploited by an adversary via a prompt injection via untrusted data.

[![Episode 1](/blog/images/2025/episode1-yt.png)](/blog/images/2025/episode1-yt.png)

If you process untrusted content, like summarizing a website, or analyze a pdf document, the author of that document can exfiltrate any information present in the prompt context, including your past chat history.

## Leaking Chat History With Prompt Injection

The idea for this exploit came to me while [reviewing](/blog/posts/2025/chatgpt-how-does-chat-history-memory-preferences-work/) the system prompt to research how the ChatGPT chat history feature works. 

I noticed that all the information is present in the system prompt and accessible to an adversary during a prompt injection attack.

[![chat history](/blog/images/2025/chatgpt-chat-history-overview.png)](/blog/images/2025/chatgpt-chat-history-overview.png)

Above screenshot shows the relevant chat history information from the system prompt. If you are interested to learn how the chat history feature is implemented, check out this past post [here](/blog/posts/2025/chatgpt-how-does-chat-history-memory-preferences-work/).

But let's focus on how an adversary can steal this info from your ChatGPT.

## Finding Bypasses for Data Exfiltration

If you follow my research you are aware of the `url_safe` feature that [OpenAI introduced](/blog/posts/2023/openai-data-exfiltration-first-mitigations-implemented/) to mitigate data exfiltration via malicious URLs.

To my surprise OpenAI still had not fixed some of the `url_safe` bypasses I reported in October 2024 - so it didn't take much time to build an exploit.

**There are three prerequisites for data exfiltration:**

1. Find a `url_safe` site 
2. Find an HTTP GET API to send data to that site 
3. Be able to view the data the site received (e.g. a log file)

`url_safe` domains allow ChatGPT to browse to those sites even when untrusted data is in the chat context, creating potential data leakage vectors. 

One such domain I discovered is `windows.net`, and I knew that **one can create a blog storage account that will run on that domain**, and at the same time it's possible to inspect the log files!

## Azure Blob Storage Logs as Exfiltration Vector

The one that is most straightforward was `blob.core.windows.net`. There were others I discovered also, but they take more effort to explain.

By sending data to this domain, we can again leak data at will. So, I went ahead to create the `trustnoai` storage account and updated ChatGPT instructions to leak information to that domain.

[![storage account](/blog/images/2025/chatgpt-exfil-storage-account.png)](/blog/images/2025/chatgpt-exfil-storage-account.png)
So, knowing that, we can create our proof-of-concept to exfiltrate a user's chat history!

## Demonstration

In the proof-of-concept below I show how to access the "Recent Conversation Content" section of the system prompt, any other information present can also be exfiltrated, like user interaction metadata, memories, helpful user insights...

### Walkthrough

This screenshot describes the entire attack chain:

[![chatgpt history exfil](/blog/images/2025/chatgpt-history-exfil-demo.png)](/blog/images/2025/chatgpt-history-exfil-demo.png)

**For individual steps see these parts:**

1. ChatGPT is tricked into accessing the conversation history

[![chatgpt history1](/blog/images/2025/chatgpt-leak-history1.png)](/blog/images/2025/chatgpt-leak-history1.png)

2. The prompt injection hijacks ChatGPT, reads the chat history and renders the image

[![chatgpt history2](/blog/images/2025/chatgpt-leak-history2.png)](/blog/images/2025/chatgpt-leak-history2.png)

3. Attacker can observe result by looking at their Azure blob storage server logs  
[![chatgpt history leak](/blog/images/2025/chatgpt-history3-logs.png)](/blog/images/2025/chatgpt-history3-logs.png)


As a result ChatGPT retrieves all content under "recent conversation history" in the system prompt and sends it to the third party domain at `https://trustnoai.blob.core.windows.net`.

**Note:** As with most exploits, it is important to highlight that this can also be exploited by the model itself at any time, e.g. hallucination or backdoor in the model. Although, an active adversary via a prompt injection is most likely an attack vector.

### Video Walkthrough

If you prefer to watch a video, here is an end-to-end explanation:

{{< youtube 0xixzlILeNg >}}

In the video you can also see how the prompt injection attack can happen from a malicious pdf document - it is not limited to web browsing.


## Responsible Disclosure

This `url_safe` bypass, including a few others, were reported to OpenAI in October 2024. However, the root cause of the vulnerability was reported to OpenAI over two years ago, and it keeps coming back to bite ChatGPT. 

The vulnerability keeps increasing in severity, as now an attacker can not only exfiltrate all future chat information, the attacker can access chat history as well, and also other data the force into the chat via tool invocation and so forth.

Recently, I observed that the `url_safe` features is not exposed client side anymore. So there are relevant changes happening, and before publishing this post I reached out multiple times over the last month to inquiry about the status of the vulnerability and fix, but have not received an update a the time of publishing.

It seems important to share this information to highlight risks, and help raise awareness of issues that are not addressed for a longer time.

**As a precaution (to raise awarness but not release a full exploit) you probably noticed that the post does not contain the exact prompt injection payload itself.** I will publish that when the vulnerability is, hopefully, fixed.

## Recommendations

Here are the recommendations provided to OpenAI:

1. Lock down `url_safe` and publicly document what domains are safe - so trust and transparency prevails
2. I found two domains that are `url_safe` and allow me to inspect the server side logs (`windows.net`,`apache.org`)
3. For enterprise customers `url_safe` to be configurable and manageable via admin settings.

## Conclusion

Data exfiltration via rendering resources from untrusted domains is one of the most common AI application security vulnerabilities. 

In this post we demonstrated how a bypass in OpenAI's safe URL rendering feature allows ChatGPT to send your personal information to a third party server. This can be exploited by an adversary via a prompt injection via untrusted data.

Hopefullly, this vulnerability will be addressed eventually.

## References

* [Month of AI Bugs 2025](https://monthofaibugs.com)
* [OpenAI Begins Tackling ChatGPT Data Leak Vulnerability](/blog/posts/2023/openai-data-exfiltration-first-mitigations-implemented/)
