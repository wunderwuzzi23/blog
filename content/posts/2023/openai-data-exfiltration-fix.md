---
title: "OpenAI mitigates data exfiltration vulnerability in ChatGPT"
date: 2023-12-19T21:35:07-08:00
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

The fix is not perfect but a step into the right direction. In this post I share what I figured out so far after looking at it briefly this morning.

Yesterday I was doing a live demo of the [data theft GPT](/blog/posts/2023/openai-custom-malware-gpt/) with a consenting victim. ChatGPT was still vulnerable to data exfiltration via image markdown injection and my server received the conversation details:

[![OpenAI Raeuber](/blog/images/2023/openai-last.png)](/blog/images/2023/openai-last.png)

Needless to say and as you can read in the screenshot above (sorry it's in German) the user was not too amused about this. 

Starting today it appears that a mitigation has been implemented by OpenAI. 🎉🎉🎉

## The Mitigation

It's a new kind of mitigation different from how other vendors fixed it. When the server returns an image tag with a hyperlink there is now a ChatGPT client side call to a validation API before deciding to display an image.

This is the call to an endpoit called `url_safe`:

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

Since ChatGPT is not open source and the fix is not via a Content-Security-Policy that is visible the exact validation details are not known. There is some internal decision making happening when an image is considered safe and when not, maybe ChatGPT queries the Bing index to see if an image is valid and pre-existing or have another tracking capability. 

**This is good news.**

As you can imagine I'm quite happy about this improvement. Its the right thing for users and I (and others) have spent so much time this year raising awareness about this problem, and with each feature addition the risk increased (Plugins, Browsing, Code Interpreter, GPTs,...).

Having a central validation API hopefully also means that Enterprises customers can configure this setting to further increase the security posture of ChatGPT for their environments.

## Not a perfect fix - remaining concerns

As stated, it's not a perfect fix. There are still some obvious side channel attacks possible to leak data, and other trickeries attackers can use to leak info via requests that pass the filter. 

I was still able to render some requests and use it to send data out (although very small amounts and slow and more noticable to a user), but this mitigation is a step in the right direction.

**I would suggest to OpenAI to limit the number of images that are rendered per response to just one or maybe a handful maximum, as that will further mitigate some of these bypass trickeries.**

## Conclusion

The validation check for safe URLs improves the security posture and allows OpenAI to monitor for attacks, which is good.  

The decision details on when a URL is considered safe are not known.

It's not a 100% perfect mitigation, as some quick tests show that bits of info can steal leak, but a step in the right direction for sure.
