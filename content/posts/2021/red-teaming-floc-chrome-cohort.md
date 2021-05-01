---
title: "Google's FLoC - Privacy Red Teaming Opportunities"
date: 2021-05-01T10:10:08-07:00
draft: true
tags: [
        "pentesting",
        "red",
        "privacy"
    ]
---

Recently Google's FLoC proposal has been making the rounds in the news. FLoC stands for "federated learning of cohorts" and is Google's vision how to perform user profiling in Chrome going forward. 

Currently user tracking and profiling happens (mostly) via cookies, but many browser vendors have been supportive of protection of their users and started blocking third party and tracking cookies - or at least offer features in their browser to enable blocking.

This threatens the business model of the ad industry and hence Google is looking for an alternative. Google's intention might be good with the introduction of FLoC, but FLoC by itself is certainly making tracking and profiling a lot easier now.  

This means your privacy is suffering if you are one of the guinea pigs selected for testing.

## FLoC enables easier fingerprinting of users

The reason is that FLoC creates additional entropy that can easily be combined with just IP address (which is already a neat fingerprint). These two together can probably 99+% uniquely identify an individual, even if the user is behind a giant NAT. So it does indeed make tracking and profiling easier. Although, Google said FLoCs get recalculated regularly.

It is also worth highlighting that Google does not enable FLoC for users in the European Union at the moment. This highlights that even Google is not so sure if FLoC is a privacy friendly (GDPR friendly) solution at the moment.

But, what I wanted to focus on in this post is one thing that I have not seen being discussed anywhere. 

**What do advertisers and web servers do with the FLoC ID?**

## How to access the FLoC ID via JavaScript?

Let's start with the web page first. 

FLoC assigns you to a cohort, which can be queried by any website you visit using the DOM API:

`document.interestCohort()`

This returns the cohort you are currently in. To test this, I launched the latest Chrome version using the following command line arguments to enable FLoC voluntarily:

```
& "C:\Program Files (x86)\Google\Chrome\Application\chrome.exe" --enable-blink-features=InterestCohortAPI --enable-features="FederatedLearningOfCohorts:update_interval/10s/minimum_history_domain_size_required/1,FlocIdSortingLshBasedComputation,InterestCohortFeaturePolicy"
```

And then I hosted a website with the following HTML:

```
<html>

Hello, Cohort!
<br><br>
Cohort: <span id="id"></span><br>
Version: <span id="version"></span>

<script>
async function getFLoC() 
{
    const { id, version } = await document.interestCohort();
    document.getElementById("id").innerText = id
    document.getElementById("version").innerText = version
}
getFLoC(); 
</script>

</html>
```

That's it. Now when I visit that website, it shows my Cohort/FLoC ID. The identifier is calculated by hashing the domain names of your browsing history using something called `simhash`.

In my experiments it was a 4-5 digit number, such as: `32536`, `22521` or `6419`.

After deleting my browsing history in Chrome, the Cohort API returns nothing. 

Then I visited two websites, including `https://grc.com/` and I was put in the following cohort: `19802`.

*Note: Via command line I had set `setting minimum_history_domain_size_required/1` which seems to cause FLoC calculation happening as soon as one site is visited. I'm assuming by default Google requires more then just one website to calculate the FLoC ID.*

This is how a webpage can get access to the user's cohort. But what to do with that cohort ID?

## What about matching cohorts to interests?

The piece I have not yet seen described by Google is how do advertisers come into the picture now - How can they show ads based on "interests" the cohort with the ID `19802` reflects. So, they can start a bid or show an ad? This is needed for targeted advertising.

I read about something called [TURTLEDOVE](https://github.com/WICG/turtledove) when trying to understand this better. But I have not figured out how this relates to FLoC. 

The TURTLEDOVE paper has some interesting points, as it states everything related to ads and auctions will be done client side - not server side. 

That said, the information and resources for these topics are quite dispersed, and I haven't found a single resource from Google that would explain how this all is actually going to work. 

Curious if anyone knows more?


## Privacy Red Teaming Opportunities!

There are certainly a set of privacy focused red teaming opportunities here:

* Is is possible to re-identify webpages a user visits based on Cohort ID?
* Furthermore, it would be neat to have a website that shows what "profile" and "interests" you have, rather than just the FLoC ID. Wonder if that is possible?
* Can a "rainbow table of FLoCs" be precalcuated? This would allow to re-identify certain browsing habits of users
* **In fact what if someone creates a Chrome extension that publishes visited domain names and their resulting FLoC ID - Imagine many people download and use it for fun! This would sort of decentralize the previous mentioned de-identifcation attacks and render FLoC useless for all other privacy concerned users.**
* How much easier is it now to track a user with just `IP address + FLoC ID` now

These are some of the privacy focused threats that I can think of.

## Conclusion

Overall, I think the intention are not bad with this. Google is looking for a legitimate and privacy friendly way to keep their business model alive in a world very privacy becomes more and more important. I'm wondering though, if detailed profiling is really needed to serve good ads and make a living.

One criticism at the moment is the lack of transparency and detailed explaination on how this is all going to work is probably the reason Google faces great pushback because the system is not well defined or communicated yet.

It will be good to keep an eye on the progress and changes.

## References

* [EFF - Google's FLOC is a terrible idea](https://www.eff.org/deeplinks/2021/03/googles-floc-terrible-idea)
* [Google - Measuring Sensitivity of Cohorts Generated by the FLoC API](https://docs.google.com/a/google.com/viewer?a=v&pid=sites&srcid=Y2hyb21pdW0ub3JnfGRldnxneDo1Mzg4MjYzOWI2MzU2NDgw)
* [TURTLEDOVE](https://github.com/WICG/turtledove)
