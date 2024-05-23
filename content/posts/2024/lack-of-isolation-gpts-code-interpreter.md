---
title: "ChatGPT: Lack of Isolation between Code Interpreter sessions of GPTs"
date: 2024-02-14T03:30:17-08:00
draft: true
tags: [
     "aiml", "machine learning", "threats", "ai injections","llm"
    ]

twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "ChatGPT: Lack of Isolation between Code Interpreter sessions of GPTs"
  description: "Public GPTs can access and overwrite your private GPT files and vice versa."
  image: "https://embracethered.com/blog/images/2024/lack-of-isolation-gpts.png"
---



Your Code Interpreter sandbox, also known as Advanced Data Analysis sessions, are shared between private and public GPTs. Yes, your actual compute container and its storage is shared. Each user gets their own isolated container, but if a user uses multiple GPTs and stores files in Code Interpreter **all GPTs can access (and also overwrite) each others files**. 

This is true also for files uploaded/created with private GPTs and ChatGPT itself.

**Update:** This vulnerability has been addressed by OpenAI in May 2024.

[![Lack of Isolation](/blog/images/2024/lack-of-isolation-gpts.png)](/blog/images/2024/lack-of-isolation-gpts.png)


## Simple Repro

The easiest way to repro this is to start a new ChatGPT chat, then upload a file or ask it to create a new file. You can ask for its `hostname` as a check, and `list files in /mnt/data` to view them. 

Afterwards do the same in a new chat session of a Custom GPT (with Code Interpreter enabled) and observe that hostname and private files created from your ChatGPT chat are visible and can also be overwriten.

Here is a short demo video that highlights the problem:

{{< youtube b0pQ9DvdyZc >}}

<br><br>

The video shows how a regular ChatGPT session and a custom GPT (public) have access to the same files of the user as they share the same Code Interpreter session.

**Here are the repro steps in more detail:**

1. Use ChatGPT and upload a test file (This file is stored inside your Code Interpreter environment)
2. Validate that you can see the file by asking ChatGPT something like `list the files in /mnt/data`. Observe that the file is indeed there (you can see how it writes the code and runs it when seeing the  "Analyzing..." icon)
3. Now pick a random third party GPT, or create your own and start a new Chat session (it needs CI enabled)
4. Ask the third party GPT the same question `list the files in /mnt/data` and observe how the third party GPT has access to the same files.

This still worked today.

## What does this mean? Implications.

**It allows the creator of a malicious GPT to steal and also overwrite files from your conversations with other GPTs, including ChatGPT**. 

### Are custom knowledge files of private GPTs safe?

As soon as Code Interpreter is invoked (visible via the "Analyzing..." UI indicator), any custom knowledge files of that GPT are copied into the shared Code Interpreter space and are available to all GPTs. 

So, the answer is no. OpenAI cannot guarantee that private knowledge files in a private GPT are secured. There is the chance of knowledge of a private GPT becoming accessible to other GPTs. 

If you build private GPTs with custom knowledge, disabling Code Interpreter should mitigate this.

### Resetting the environment

After some idle time Code Interpreter sessions reset, which means all files are gone and the user starts with a new instance. It's unclear exactly when that happens. And the user cannot decide to reset it themselves. 

For a while I always used `%reset -f`, which seemed to force a reset, but that does not work anymore.

**There is only one CI container per user, across multiple GPT sessions.**

If you upload your own personal finance CSV to do some data analysis with ChatGPT, and then you use another custom third party GPT, the **third party custom GPT has full access to your personal CSV file**! 

Similarly, custom knowledge of a private GPT might leak to other GPTs once the knowledge is loaded into CI.

## Recommendations 

Code Interpreter sessions get discarded after a while, but it's not clear when that happens. You can always prompt for `list files in /mnt/data` to check if there are any files.

* Users should not upload or create sensitive files that they do not want to have leaked.
* Use third party GPTs (or prompts) with care, especially if they have Code Interpreter enabled.
* Disable Code Interpreter in private GPTs with private knowledge files (as they will be accessible to other GPTs, and **other GPTs might influence your GPT by writing files**) -> This seems to be one change OpenAI made so far, when creating a new GPT Code Interpreter is off by default.
* Consider [OpenAI's Classic GPT](https://chat.openai.com/g/g-YyyyMT9XH-chatgpt-classic) (without Code Interpreter) if the functionality is not needed. 
* Users could also delete uploaded files manually via Code Interpreter when tasks are completed.

To OpenAI the following recommendations were provided in November:

* Isolate GPTs and their compute and storage (first party, private, public) - every GPT session needs to have its own container 
* Implement static analysis to catch GPTs with hidden/malicious instructions

If you are wondering, Google Bard (now Gemini) [seems to not share any state in the compute container between chat sessions](/blog/posts/2024/exploring-google-bard-vm/).

## Responsible Disclosure

This is an older bug. It was first disclosed to the vendor early last November and confirmed to be a security problem. However, followup queries about status remain unanswered. It has now been over 90 days that this vulnerability was reported to OpenAI. To adhere to [industry norms of responsible disclosure](https://about.google/intl/ALL_us/appsecurity/) this information is shared with the public. 

**Update: This vulnerability has been addressed by OpenAI in May 2024.**



## References

* [Demo Video of Lack of Isolation GPT](https://www.youtube.com/watch?v=b0pQ9DvdyZc)
* [OpenAI's Classic GPT](https://chat.openai.com/g/g-YyyyMT9XH-chatgpt-classic)
* [Exploring Google Bard's Data Visualization Feature (Code Interpreter)](/blog/posts/2024/exploring-google-bard-vm/)
* [How Google handles security vulnerabilities](https://about.google/intl/ALL_us/appsecurity/)
