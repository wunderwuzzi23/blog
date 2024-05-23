---
title: "ChatGPT: Hacking Memories with Prompt Injection"
date: 2024-05-22T12:24:07-07:00
draft: true
tags: [
     "ai", "testing","machine learning", "ai injection", "chatgpt", "ttp"
    ]

twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "ChatGPT: Hacking Memories with Prompt Injection"
  description: "ChatGPT has the capability to store long term memories now. Unfortunately it is possible to invoke the tool to store memories directly during prompt injection. This post explores this attack via three avenues: Images, Connected Apps and Browsing."
  image: "https://embracethered.com/blog/images/2024/chatgpt-mem-thumbnail-pi.png"
---

[OpenAI recently introduced a memory feature in ChatGPT](https://openai.com/index/memory-and-new-controls-for-chatgpt/), enabling it to recall information across sessions, creating a more personalized user experience. 

However, with this new capability comes risks. Imagine if an attacker could manipulate your AI assistant (chatbot or agent) to remember false information, bias or even instructions, or delete all your memories! This is not a futuristic scenario, the attack that makes this possible is called [Indirect Prompt Injection](/blog/posts/2023/ai-injections-direct-and-indirect-prompt-injection-basics/).

![chatgpt memory logo](/blog/images/2024/chatgpt-mem-thumbnail-pi.png)


In this post we will explore how memory features might be exploited by looking at three different attack avenues: **Connected Apps**, **Uploaded Documents (Images)** and **Browsing**. 

The memory feature is [on by default](https://help.openai.com/en/articles/8983142-how-do-i-enable-or-disable-memory) in ChatGPT.

By understanding these vulnerabilities, you'll be better equipped to protect your interactions with AI from potential manipulation and also built more secure applications yourself.


## What is Memory in an LLM app?

Adding memory to an LLM is pretty neat. Memory means that an LLM application or agent stores things it encounters along the way for future reference. For instance, it might store your name, age, where you live, what you like, or what things you search for on the web. 

Long-term memory allows LLM apps to recall information across chats versus having only in context data available. This can enable a more personalized experience, for instance, your Chatbot can remember and call you by your name and better tailor answers to your needs.

It is a useful feature in LLM applications.

## Memories in ChatGPT

ChatGPT now has memory, and when retrieving the system prompt, we can observe two things.

### Observation 1: The Bio Tool 

In the beginning of the system prompt we see a new `bio` tool, which can be invoked with `to=bio` to remember information. 

![to-bio](/blog/images/2024/chatgpt-bio-tool2.png)

That specific string (`to=bio`) is not required, we can just write "Please remember that I like cookies":

[![to-bio](/blog/images/2024/chatgpt-memory-cookies.png)](/blog/images/2024/chatgpt-memory-cookies.png)

Please notice the "Memory updated" output in the above screenshot. This means that ChatGPT interacted with it's memory tool. As we will see below, this is the primary indicator that something unwanted might have happened. Its possible to click on the "Memory updated" area to inspect what happened!

### Observation 2: Model Set Context 

When asking for the system prompt I observed that there is now a `Memory Set Context` section at the end. This is how ChatGPT injects memories into the chat context so that they are available during inference.

[![Set Memory Context](/blog/images/2024/chatgpt-model-set-context.png)](/blog/images/2024/chatgpt-model-set-context.png)


There is also a date associated with each memory that ChatGPT displays, but that is not the date the memory was added. I will update this post once I figure out what the date means, I don't think the date is a hallucination as it consistently shows up.

Now, let's explore the prompt injection scenario more.

## Hacking Memory with Prompt Injection?

The big question is, of course, if processing untrusted data trick ChatGPT to store fake memories? 

Such a vulnerability would allow an attacker to write misinformation, bias and even instructions into your ChatGPT's memories, and also **create a form of persistence at the same time**!

I tried three variations on how untrusted data from a third party might enter the chat: 

* Connected Apps
* Analyzing an Image (upload a file)
* Browsing with Bing

Let's explore them in detail.

## Scenario 1: Connected Apps

Yesterday I saw this new feature called `Connected App` which allows to access Google Drive and Microsoft OneDrive documents in a chat. 

And as it turns out, a Google Doc that is referenced in a conversation can indeed write memories.

### Video Demonstration

Here is a brief video showing the proof-of-concept from end to end.

{{< youtube sdmmd5xTYmI >}}

And below you can find a step by step explanation.

#### Detailed steps of the proof-of-concept

*Note: The video recording above is a more complex scenario (way cooler by the way) than this explanation here. The video writes more memories and hides the prompt injection attack within the document.*

The document containing the prompt injection:

[![matrix memory instructs](/blog/images/2024/chatgpt-memory-instructions.png)](/blog/images/2024/chatgpt-memory-instructions.png)

The text is quite simple prompt injection and it's not embedded into a larger document. For that you can take a look at the video recording which has a more complex walk-through.

Here is an example conversation that referenced a Google Doc:
[![matrix memory stored](/blog/images/2024/chatgpt-memory-in-the-matrix.png)](/blog/images/2024/chatgpt-memory-in-the-matrix.png)

The main point and mitigation for users is to watch out for the "Memory updated" message at the top of a response from ChatGPT. If that happens ensure to inspect the changes that have been made. 

[![matrix updated](/blog/images/2024/chatgpt-memory-updated.png)](/blog/images/2024/chatgpt-memory-updated.png)

And this is how the memory looks afterwards in the UI:
[![matrix output](/blog/images/2024/chatgpt-memory-details.png)](/blog/images/2024/chatgpt-memory-details.png)

All the instructions we had in our external document were followed and each bullet point became a memory!

Just for completeness, here is a screenshot that shows a new conversation and you can see the memory is recalled and used:
[![matrix output](/blog/images/2024/chatgpt-memory-demo.png)](/blog/images/2024/chatgpt-memory-demo.png)

As you can see this works well, and persists into future conversation and chat sessions.

**Important:** Tool invocations also sometimes happen randomly (due to hallucinations).

**Finally, it is also possible to delete all existing memories with a simple prompt injection.**

## Scenario 2: Analyzing an Image (File Uploads)

Asking ChatGPT to analyze an image can lead to prompt injection, and it turns out that during such an attack, it's also possible to write memories.

{{< youtube bRBtDiYZzMQ >}}

Above video shows this end to end. This was the first demo I got working a while ago.

## Scenario 3: Browsing with Bing

The third option I tried was using the browsing tool. 

Interestingly, this attack did not work right away! I have my typical prompt injection strings hosted on my webserver and ChatGPT is connecting to the server and retrieves the data, **but it refused to invoke the memory tool!**

In the past it was super easy to get prompt injection going via the browsing tool. I believe OpenAI started putting some mitigations in place here, which is good to see, and hopefully they apply those improvements to other places also.

### Trying to understand what's going on

Initially, I thought there would be an actual security control in place. For instance, its technically possible to create a mitigation and not have ChatGPT invoke tools in a conversation once untrusted data is in the chat, e.g. after browsing occurs.

However, after some testing I was able to proof that is not the case. The current mitigation is possibly only happening at the LLM layer and can be bypassed with clever prompting tricks.

### Techniques that worked 

Spending more time on this probably will find better, more reliable, ways, but so far there are two techniques that worked:

* **Tool Chaining:** When the webpage contains instructions to create an image, ChatGPT follows those instructions quite often. So, I figured why not chain two tool invocations together, create an image and also have ChatGPT remember that "I like ice cream and cookies"...

[![browse to bio](/blog/images/2024/chatgpt-browse-to-bio-small.png)](/blog/images/2024/chatgpt-browse-to-bio-small.png)
Yes, that worked! It's not working 100% (yet), but good enough to demonstrate the point.

* **Delayed Exeuction:** The second technique requires multiple conversation turns, and is [a technique that I explained a while ago](https://embracethered.com/blog/posts/2024/llm-context-pollution-and-delayed-automated-tool-invocation/) in the context of `Google Gemini`. It involves delayed tool invocation by defining trigger signals or words. Using this technique I got it to work a few times, but it was flaky.

*I want to do a seperate post about browsing based tool invocation techniques, once I have time to research a bit more what OpenAI did here. So, stay tuned for that.*

## Disclosure 

That untrusted data can lead to memory tool invocation was disclosed to OpenAI, but the report was closed as "Model Safety Issue" and it is not considered a security vulnerability. 

It would be great to better understand the reasoning for that decision, as **it clearly impacts integrity of the memories stored in the user's profile and all future conversations**. And as mentioned before, having untrusted data be able to add memories to your profile, and also [delete all your memories](/blog/images/2024/chatgpt-remove-all-memories.png) is a security vulnerability in my books and has nothing to do with "model safety".

## Recommendations

* Do not automatically invoke tools once untrusted data, like documents from remote places, enter the chat.
* If tools are invoked automatically under unsafe conditions, ensure the user has the opportunity to decide if they want to proceed or not and can clearly see the result and implications of the action upfront.
* For dangerous operations, like deleting all memories, always require user confirmation.
* Limit the number of memories that can be inserted in one shot. I was able to create dozens of new, fake memories in one shot.
* Users should regularly inspect the `Memory` of your ChatGPT to cross-check what got stored. Look at the "Memory updated" information and click through it to see what was stored. You can select which memories to delete, or delete all of them. 
* It is also possible to [entirely disable the memory feature](https://help.openai.com/en/articles/8983142-how-do-i-enable-or-disable-memory)
* And as always, you cannot trust the output of a LLM, even if there is no corrupted memory or adversary in the loop it will at times produce incorrect results

That's it.

## Conclusion

This post explored how LLM apps (and agents) that implement a memory tool to maintain a long-term history of relevant information can be susceptible to prompt injection attacks. This allows untrusted data to interact with the memory tool and perform its operations, like **adding new fake memories or deleting existing ones**.

Specifically, we showed real-world POCs exploits against the memory feature of ChatGPT.

As a user be aware that this feature is on by default, and whenever you see the "Memory updated" icon, be curious to inspect what information ChatGPT remembered or deleted, as this will impact all future conversations.

Thanks for reading, hope it was interesting and helpful.

Cheers.

## Appendix

Tweet describing the image attack and video
{{< twitter wunderewuzzi23 1791270770502742040 >}}

### POC for tampering with memories

[![delete all memories](/blog/images/2024/chatgpt-remove-all-memories.png)]((/blog/images/2024/chatgpt-remove-all-memories.png))

### Memory Feature Configuration

There is also the option to entirely disable the feature as it is on by default:
![matrix updated](/blog/images/2024/chatgpt-memory-screenshot.png)

## References

* [How do I enable or disable memory?](https://help.openai.com/en/articles/8983142-how-do-i-enable-or-disable-memory)
* [Memory Announcement by OpenAI](https://openai.com/index/memory-and-new-controls-for-chatgpt/)
* [Gemini: Delayed Tool Invocation](https://embracethered.com/blog/posts/2024/llm-context-pollution-and-delayed-automated-tool-invocation/)