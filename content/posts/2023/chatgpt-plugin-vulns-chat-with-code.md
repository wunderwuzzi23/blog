---
title: "Plugin security mess: Visit a website and have your private source code stolen"
date: 2023-06-19T09:00:22-07:00
draft: true
tags: [
     "aiml", "machine learning", "threats", "ai injections","chatgpt", "plugins"
    ]

twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "ChatGPT Plugin security mess."
  description: "This latest exploit demo shows how a malicious websites can switch your Github repos from private to public visibility"
  image: "https://embracethered.com/blog/images/2023/chatwithcode-exploit-repo-private-to-public.png"
---

OpenAI continues to add plugins with security vulnerabilities to their store. 

In particular powerful plugins that can impersonate a user are not getting the required security scrutiny, or a general mitigation at the platform level.

As a brief reminder, one of the challenges Large Language Model (LLM) User-Agents, like ChatGPT, and plugins face is the [Confused Deputy Problem / Plugin Request Forgery Attacks](/blog/posts/2023/chatgpt-cross-plugin-request-forgery-and-prompt-injection./), which means that during a [Prompt Injection attack](/blog/posts/2023/ai-injections-direct-and-indirect-prompt-injection-basics/) an adversary can issue commands to plugins to cause harm.

**OpenAI added some guidance and criteria on their webpage:**

![guidance](/blog/images/2023/plugins-criteria.png)

That's good, and I feel slightly responsible for this. :)

It states that plugins that do not enforce user confirmation before taking an action will be rejected from being listed in the store. At least there is some traction now.

**Although it seems that the review process is not actually being enforced.**

Let's explore.

## Plugins that take action on behalf of users

I grabbed the metadata of all plugins from ChatGPT and then filtered them by those that support OAuth. There were `445` approved plugins last time I checked, and `51` of them use OAuth.

**Why look at OAuth?**

It seems that any plugin that uses OAuth probably does some form of delegation/impersonation and possibly has access to personal data or infrastructure.

Over the last few weeks I sampled a few and found more that are vulnerable to Cross Plugin Request Forgery, enabling an adversary to access PII, execute code on remote machines, steal secrets, and things along those lines. 

Here is an example of one of the new interesting ones.

## Chat with Code - Interact with Github 

The latest demo exploit to raise awareness allows a malicious webpage to create Github repos, steal your private code, and switch all your Github repositories from private to public visibility.

Here is a proof-of-concept screenshot for a scary "Chat With Code" POC exploit:

[![Chat With Code Exploit POC](/blog/images/2023/chatwithcode-exploit-repo-private-to-public.png)](/blog/images/2023/chatwithcode-exploit-repo-private-to-public.png)

Indeed. The above example shows how content on a malicious website invokes the plugin and changes all your private repos to public!

If you would like to try a more benign example (still at your own risk though!), then visit this URL with `WebPilot` and `Chat With Code` plugins:

```
https://wuzzi.net/ai-tests/code.html
```

This example demo prompt injection will:

1. Create a new repository called `GOT-PWNED` in your Github account
2. Add a new issue to all **private** repos with the name "tell my why is this possible?"

This is the result:

[![chat with code](/blog/images/2023/chatwithcode-exploit.png)](/blog/images/2023/chatwithcode-exploit.png)

Scary times.

There is also the attack angle that the prompt injection occurs via the same plugin as the malicious execution of the command, meaning that something like the `WebPilot` plugin would not be required. Still exploring.

## Reporting and Guidelines 

Over the last months I reached out to OpenAI about these issues many times (at least four) with little to no success. Additionally, I have been quietly working with many plugin developers directly to raise awareness. Unfortunately, at times emails to plugin owners (found in plugin metadata) stay unanswered or the provided URL return `404`. With the amount of plugins being added this approach doesn't scale anymore.

Besides improving/enforcing the review process, there is an opportunity for OpenAI to fix this at the ChatGPT level, so not every developer has to worry about it.

In a [recent post about functions](https://openai.com/blog/function-calling-and-other-api-updates?ref=upstract.com) this kind of (my?) security research is mentioned. Although, no acknowledgments or references are provided:

[![Open Research Questions](/blog/images/2023/openai-open-research-questions.png)](/blog/images/2023/openai-open-research-questions.png)

**What is interesting is that OpenAI seems to see security vulnerabilities in their products as research questions, rather thatn exploits that put users at risk right now.**

Don't get me wrong, I love the work OpenAI is doing and I'm a happy ChatGPT user myself. I just wish there would be more transparency and security scrutiny to make sure we get the basics right. Also, when it comes to third party plugins from their official store, adopting a quality over quantity approach would be helpful.

Anyhow, that's it. 

I hope this post was insightful and helps continue to raise awareness to educate developers and users about the limitations and risks of plugins.

## References

* [OpenAI Plugin Criteria](https://platform.openai.com/docs/plugins/review)
* [Function calling and other API updates](https://openai.com/blog/function-calling-and-other-api-updates?ref=upstract.com)



## Appendix

List of client_urls (June, 17th 2023):

```
  "client_url": "https://pixellow.ceylon.ai/oauth",
  "client_url": "https://chat-stack-search.thx.pw/oauth/authorize",
  "client_url": "https://dev-5hctgu3e4fhyq5c1.us.auth0.com/authorize",
  "client_url": "https://nalan.us.auth0.com/authorize",
  "client_url": "https://f14661818ab7a5cf917eff1c173663de.auth.portal-pluginlab.ai/oauth/authorize",
  "client_url": "https://www.threesigma.ai/oauth-login",
  "client_url": "https://plugin.gptinf.com/api/oauth",
  "client_url": "https://60sec.site/oauth/code",
  "client_url": "https://dev-xlbshnzwy6q5frgo.us.auth0.com/authorize",
  "client_url": "https://adbf46cbe372916cc21e69c1b6f44630.auth.portal-pluginlab.ai/oauth/authorize",
  "client_url": "https://auth.minihabits.ai",
  "client_url": "https://auth.sembot.com/oauth/authorize",
  "client_url": "https://timenavi.com/auth/openai",
  "client_url": "https://caac141bb05029c6c04ceb5ac3b06bf1.auth.portal-pluginlab.ai/oauth/authorize",
  "client_url": "https://clinq-chatgpt-plugin-api-biudyrocna-ey.a.run.app/oauth",
  "client_url": "https://cbplugin.upskillr.ai/authenticate",
  "client_url": "https://placid.app/app/openai/login",
  "client_url": "https://whizlist-plugin.chatbot.so/d2d00899-7c38-4a00-a0ac-b172316b3d00",
  "client_url": "https://app.caryardbard.com/login",
  "client_url": "https://auth.notionlink.io",
  "client_url": "https://reflect-chatgpt.teamreflect.workers.dev/auth",
  "client_url": "https://auth.replypdf.com",
  "client_url": "https://auth.quickrecall.cards/oauth2/authorize",
  "client_url": "https://app2.mantiumai.com/oauth/authorize",
  "client_url": "https://chatwithblog.com/oauth/authorize",
  "client_url": "https://ticktick.com/oauth/authorize",
  "client_url": "https://app.itsdart.com/api/oauth/authorize/",
  "client_url": "https://jiggybase.plugin.jiggy.ai/authorize",
  "client_url": "https://github.com/login/oauth/authorize",
  "client_url": "https://calorie.chat/signup.html",
  "client_url": "https://giga.do/openai/login",
  "client_url": "https://gpt.nylas.com/api/oauth/authorize",
  "client_url": "https://playlistai-plugin.vercel.app/authorize",
  "client_url": "https://chat.noteable.io/origami/authorize",
  "client_url": "https://plugin.bramework.com/oauth",
  "client_url": "https://smyth.seo.app/authorize",
  "client_url": "https://98da904e93259bb18ec079807593fce0.auth.portal-pluginlab.ai/oauth/authorize",
  "client_url": "https://shuto.io/oauth/authorize",
  "client_url": "https://dashboard.decisionjournalapp.com/oauth/authorize",
  "client_url": "https://plugin.autoinfra.ai/oauth/authorize",
  "client_url": "https://chatsshplug.com/auth/login",
  "client_url": "https://www.owler.com/oauth",
  "client_url": "https://chatgpt-plugin.clearbit.com/oauth/authorize",
  "client_url": "https://gh-plugin.teammait.com/",
  "client_url": "https://809dbc012d4f6e2d594459d86cba6797.auth.portal-pluginlab.ai/oauth/authorize",
  "client_url": "https://auth.buildbetter.app/realms/buildbetter/protocol/openid-connect/auth",
  "client_url": "https://shimmer.ooo/auth/sign",
  "client_url": "https://kalendar.ai/users/new/onboard/",
  "client_url": "https://www.persona-ai.com/sample-login",
  "client_url": "https://chatgpt-plugins-ashy.vercel.app/oauth",
  "client_url": "https://nla.zapier.com/oauth/authorize/",
```