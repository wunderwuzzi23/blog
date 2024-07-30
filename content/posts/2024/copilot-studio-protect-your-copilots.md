---
title: "Protect Your Copilots: Preventing Data Leaks in Copilot Studio"
date: 2024-07-30T10:00:36-07:00
draft: true
tags: [
    "threats", "ttp", "red","blue", "copilot", "data", "llm"
    ]

twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "Protect Your Copilots: Preventing Data Leaks in Copilot Studio"
  description: "Copilot Studio  (previously called Power Virtual Agents) is a platform to quickly create chatbots. But your company's internal data might leak. Apply mitigations, data loss prevention policies and security controls."
  image: "https://embracethered.com/blog/images/2024/copilot-data-leak-prevention-small.png"
---

**Microsoft's Copilot Studio** is a powerful, easy-to-use, low-code platform that enables employees in an organization to create chatbots. Previously known as **Power Virtual Agents**, it has been updated (including GenAI features) and rebranded to Copilot Studio, likely to align with current AI trends.

[![Thumbnail Copilot Studio](/blog/images/2024/copilot-data-leak-prevention-small.png)](/blog/images/2024/copilot-data-leak-prevention-small.png)

This post discusses security risks to be aware of when using Copilot Studio, focusing on data leaks, unauthorized access, and how external adversaries can find and interact with misconfigured Copilots. **Learn about security controls, like enabling Data Loss Prevention (DLP), which is currently off by default, to protect your organization's data.**

## Table of Contents

1. [Creating Custom Copilots](#creating-custom-copilots)
2. [Danger Zone: Authentication](#danger-zone-authentication)
3. [Hacker's Gonna Hack!](#hackers-gonna-hack)
4. [Enumerating Copilots](#enumerating-copilots)
    - [Tenant ID - What is this?](#tenant-id---what-is-this)
    - [Environment ID - Derived from Tenant ID](#environment-id---derived-from-tenant-id)
    - [Schema Name - The Name of the Copilot](#schema-name---the-name-of-the-copilot) 
    - [Another Enumeration Option: Token Endpoints](#another-enumeration-option-token-endpoints)
6. [Secured Access is off by default](#secured-access-is-off-by-default)
7. [Let's chat with a custom Copilot!](#lets-chat-with-a-custom-copilot)
8. [Real-world Enumeration - Is it possible?](#real-world-enumeration---is-it-possible)
9. [Mitigations and Security Controls](#mitigations-and-security-controls)
10. [Auditing](#auditing)
11. [Conclusion](#conclusion)

Let's get started.

## Creating Custom Copilots

Creating a new bot with [Copilot Studio](https://copilotstudio.microsoft.com) is simple and takes only a few clicks:

[![copilot studio - create](/blog/images/2024/copilotstudio-create3.png)](/blog/images/2024/copilotstudio-create3.png)

And users can upload custom knowledge and even connect the Copilot to internal data sources via a wide range of connectors.

[![add knowledge](/blog/images/2024/copilotstudio-addknowledge2.png)](/blog/images/2024/copilotstudio-addknowledge2.png)

This allows for the quick creation of custom chatbots that might leak internal data. Why?

## Danger Zone: Authentication

When publishing a Copilot, employees have the option to set **no authentication**. 

[![no auth copilot](/blog/images/2024/copilotstudio-authentication.png)](/blog/images/2024/copilotstudio-authentication.png)

In my tenant, the default was Microsoft Authentication, which is good. And the documentation also discusses this:

> Selecting the No authentication option will allow anyone who has the link to chat and interact with your Copilot. ([Publication Fundamentals](https://learn.microsoft.com/en-us/microsoft-copilot-studio/publication-fundamentals-publish-channels?tabs=web))

However, in reality, it's still up to the individual. For one case I'm aware of, there was an error message when selecting "Microsoft Authentication", so the next best option to make it "work" was selecting "No Authentication", allowing anyone to chat with a bot.

**Admins can set Data Loss Prevention (DLP) policies and features in Copilot Studio, but they are not enabled by default.**

Additionally, internal company chatbots (even those requiring authentication) might still leak sensitive data - so be aware of these risks.

## Hacker's Gonna Hack!

Is it easy to find or guess a Copilot name or URL to talk with widely exposed ones? At first glance, it's not obvious. Your company might expose some Copilots (accidentally or intentionally) in search or the Wayback Machine though, so be aware of that also.

But, what about guessing the URL? It seems when the system was called **Power Virtual Agents**, bots were solely based on UUIDs as identifiers. The UUID is in the URL but generally those are impossible to guess.

However, there is also a schema name and a `/botsbyschema/` route visible in the web traffic. Can someone guess that?

For a few Copilots that I had created months ago, the schema name starts with `Default_` plus the name of the Copilot without spaces. Not very difficult to guess...

A more recently created one looks like this: `cr5f7_chatWithEmployeeInfo`

And this was another one created in the same tenant:

[![second copilot](/blog/images/2024/copilotstudio-copirate.png)](/blog/images/2024/copilotstudio-copirate.png)

Do you notice anything?

The schema name is `cr5f7_copilot`! So, they start with the same `cr5f7_` prefix in this tenant.

## Enumerating Copilots

While capturing requests, I observed one that indicates if a Copilot with a certain name exists or not. It was this one: `https://default8b63cc9ba2f948ea9cb6f82c96c9ac.08.environment.api.powerplatform.com/powervirtualagents/botsbyschema/Default_copilot1/canvassettings?api-version=2022-03-01-preview`.

If the Copilot exists, the response is:
```
{
    "botCanvasSettings": {
        "botId": "966a369a-babb-ebd5-5d64-0f4155544d97",
        "botName": "Copilot 1",
        "tenantId": "8b63cc9b-a2f9-48ea-9cb6-f82c96c9ac08"
    }
}
```

In case a Copilot does not exist, the response is:

```
{
  "demoWebsiteErrorCode": "404"
}
```

So, one needs these pieces for the API call:

* Tenant ID
* Environment ID (which is most often `Default-` plus the Tenant ID)
* Schema Name of Copilot (or the Bot ID)

Let's explore.

### Tenant ID - What is this?

The mapping from Tenant Name to Tenant ID is straightforward, it's not a secret. You can use the `https://login.microsoftonline.com/<TENANT_NAME>.onmicrosoft.com/.well-known/openid-configuration` to retrieve the details. For instance:

```
curl --silent https://login.microsoftonline.com/microsoft.onmicrosoft.com/.well-known/openid-configuration | jq .authorization_endpoint
```

Returns `"https://login.microsoftonline.com/72f988bf-86f1-41af-91ab-2d7cd011db47/oauth2/authorize"`

So, `microsoft.onmicrosoft.com` has Tenant ID `72f988bf-86f1-41af-91ab-2d7cd011db47`. 

And for this demo the tenant, named `tx23xg.onmicrosoft.com` maps to `8b63cc9b-a2f9-48ea-9cb6-f82c96c9ac08`.

Great, so now let's dive into the next layer!

### Environment ID - Derived from Tenant ID

The obscure part of the DNS name, `default8b63cc9ba2f948ea9cb6f82c96c9ac.08`, is based on the Tenant ID. It's derived by removing the dashes and separating the last two characters into a separate subdomain.

Later, we'll see the Environment ID used in the URL path, starting with `Default-` and Tenant ID is appended.

### Schema Name - The Name of the Copilot

The last piece of the request puzzle is the Copilot name. After trying in a few tenants and speaking with a few other organizations it appears this pattern is not uncommon, the name:
* starts with `cr`, then 
* 3 random characters and an underscore and 
* the name of the Copilot without spaces.

This is entirely based on empirical evidence, and might be configured/derived from some specific place. Sometimes it's also just `Default_` plus the name of the Copilot.

#### Another Enumeration Option: Token Endpoints

While researching this, another option for enumeration was discovered, it's by using the token endpoint (and that leaks a bit more information in error cases):

`https://copilotstudio.microsoft.com/environments/Default-8b63cc9b-a2f9-48ea-9cb6-f82c96c9ac08/bots/Default_copilot1/canvas?__version__=2`

Here are two ways to call it:

* By Schema Name
`https://default8b63cc9ba2f948ea9cb6f82c96c9ac.08.environment.api.powerplatform.com/powervirtualagents/botsbyschema/Default_copilot1/directline/token?api-version=2022-03-01-preview`

* By Bot ID
`https://default8b63cc9ba2f948ea9cb6f82c96c9ac.08.environment.api.powerplatform.com/powervirtualagents/bots/966a369a-babb-ebd5-5d64-0f4155544d97/directline/token?api-version=2022-03-01-preview`


The returned (base64 decoded) JWT token looks like this:
```
{
    "bot": "966a369a-babb-ebd5-5d64-0f4155544d97",
    "site": "0Tm3SQ43pLs",
    "conv": "QPbc3pjmlTERRLOgJTJrn-us",
    "user": "a1e0bef7-3a11-44f9-9d4a-cf818ffe7bbe",
    "nbf": 1717371284,
    "exp": 1717374884,
    "iss": "https://directline.botframework.com/",
    "aud": "https://directline.botframework.com/"
}
```

**Note:** If you have a Copilot's Bot ID you don't need the Environment ID at all. One can use any DNS name of a valid environment. The Environment ID is only required when querying via the schema name.

#### Enumeration Errors from Token Endpoint

There are different errors returned, in particular these two are worth highlighting:

* `{"ErrorCode":4103,"ErrorMessage":"Bot with ID cr5f7_copirate can't be found."}`
* `{"ErrorCode":4011,"ErrorMessage":"The bot isn't available at this time. Please try again later. If you have questions, contact your admin."}` > **This likely means that Secure Channel was enabled!**

What is that Secure Channel?

## Secured Access is off by default

The reason the demo website and directly retrieving an access token works, is because "Secured Access" is not enabled by default. In the following screenshot, you can see where to find this setting:

[![secured channel](/blog/images/2024/copilotstudio-channel-security-off-by-default.png)](/blog/images/2024/copilotstudio-channel-security-off-by-default.png)

If this is enabled, then *directline* and the *demo website* won't work!

According to documentation, it can take up to two hours for changes to take effect.

Okay, so how to chat with a newly discovered bot?

## Let's Chat with a Custom Copilot!

If you know the schema name of a Copilot (or the Bot ID) you start a conversation with the demo UI: `https://copilotstudio.microsoft.com/environments/Default-TENANT-NAME/bots/<COPILOT_SCHEMA_NAME>/webchat`

This is a demo Copilot I created a while ago for research purposes:
[![chat with address book](/blog/images/2024/copilotstudio-hr-asked-me-todo-this.png)](/blog/images/2024/copilotstudio-hr-asked-me-todo-this.png)

There is a second, slightly different, UI experience as well: `https://copilotstudio.microsoft.com/environments/Default-TENANT-NAME/bots/<COPILOT_SCHEMA_NAME>/canvas?__version__=2`

**Error Messages**

There are errors one might see when interacting with the Copilot this way.

* Error Case: Copilot is not yet published
`Sorry, something unexpected happened. We’re looking into it. Error code: LatestPublishedVersionNotFound.`

* Error Case: Microsoft Authentication is enabled
`Sorry, something unexpected happened. We’re looking into it. Error code: IntegratedAuthenticationNotSupportedInChannel.`

One can also chat with a bot using the API endpoints (see [DirectLine](https://learn.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-direct-line-3-0-api-reference?view=azure-bot-service-4.0)).

## Real-world Enumeration - Is it possible?

**Yes.** 

I wrote some code, and it works! 

The script successfully retrieved a set of Copilot's from **Microsoft's Power Platform Azure Tenant**. 😈

None of those Copilots were published though. So, I wasn't able to chat with them. And I didn't poke any further. Although, there isn't a high security vulnerability per se (yet), I informed Microsoft about this issue to make sure they are aware of it and possibly consider enabling data loss prevention features by default. 

Let's talk about what you need to do to stay ahead of the game and what mitigations can be applied at the enterprise level.

## Mitigations and Security Controls

Currently Data Loss Prevention is not enabled by default. After asking for feedback on this post a few months back, Microsoft said that they will enable DLP by default, although no specific timeline was provided. 

**The best advice for organizations at the moment is to enable Data Loss Prevention!**

The [Copilot security and governance section](https://learn.microsoft.com/en-us/microsoft-copilot-studio/security-and-governance) highlights mitigations, including options such as:

* Disabling Copilot publishing
* Disabling publishing of bots that have no authentication
* An admin can disable Copilot for the entire organization (requires a support ticket currently)
* Restrict data movement between regions
* Web Channel Security - Enable *secure access* to disable the demo webchat site and directline

**Configure DLP according to your threat model and risk tolerance to protect your organization from accidental (or intentional) data leaks.**

## Auditing

For further insights around monitoring, check out the [auditing capabilities](https://learn.microsoft.com/en-us/microsoft-copilot-studio/admin-logging-copilot-studio) of Copilots for admins.

## Conclusion

The creation of powerful chatbots is made easy with **Copilot Studio**. It builds upon **Power Virtual Agents**, which have been around for a while and provide a feature rich framework for creating chatbots, including ones that leverage GenAI.

There are security risks, including data leakage with the proliferation of custom Copilots in organizations. In particular, widely exposed or misconfigured Copilots (that might have access to internal data) can be discovered and accessed by adversaries.

Long story short: Enterprises should evaluate their risk tolerance and exposure to prevent data leaks from Copilots (formerly Power Virtual Agents), and enable Data Loss Prevention and other security controls accordingly to control creation and publication of Copilots.

Cheers.

## References

* [Copilot Studio Documentation](https://learn.microsoft.com/en-us/microsoft-copilot-studio/)
* [Copilot Studio - Security and Governance](https://learn.microsoft.com/en-us/microsoft-copilot-studio/security-and-governance)
* [Copilot Studio - Configure Data Loss Prevention](https://learn.microsoft.com/en-us/microsoft-copilot-studio/admin-data-loss-prevention)
* [Direct Line](https://learn.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-direct-line-3-0-api-reference?view=azure-bot-service-4.0)