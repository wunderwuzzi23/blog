---
title: "Chatgpt Plugin Vulns Chat With Code"
date: 2023-06-19T23:44:22-07:00
draft: true
---

OpenAI continious adding plugins with security vulnerabilities to their store. In particular ones that are powerful and allow to impersonate the user, but it seems most do not have mitigation against [Request Forgery](/blog/posts/2023/chatgpt-cross-plugin-request-forgery-and-prompt-injection./) attacks.

Recently OpenAI added some guidance and criteria on their webpage:

![guidance](/blog/images/2023/plugins-criteria.png)

It clearly states that Cross Plugin Requests should require user confirmation, and that they reject plugins that do not do this.

Althought it seems that quality assurance is not yet gotten the memo.

## How many plugins can take action on behalf of users?

To answer above questions, I grabbed the metadata of all plugins from ChatGPT and then filtered them by those that support OAuth. It seemed that any plugin that uses OAuth does some form of impersonation and possibly has access to personal data or infrastructure.

Turns out that as of today, June 17th, there were `51` that supported OAuth. 

I sampled a few over the last couple of weeks and found more that are vulnerable to Cross Plugin Request Forgery. 

In addition, I attempted to reach out to OpenAI about this multiple times with little to no success.


## References

* [OpenAI Plugin Criteria](https://platform.openai.com/docs/plugins/review)


List of Apps with OAuth support and their client_url

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