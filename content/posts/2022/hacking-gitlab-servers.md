---
title: "Gitlab Reconnaissance Introduction"
date: 2022-02-28T04:22:22-22:22
draft: true
tags: [
        "pentest", "red","research", "cicd", "devops"
    ]

twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "Gitlab Reconnaissance Introduction"
  description:  "Hacking Gitlab Servers"
  image: "https://embracethered.com/blog/images/2020/hacker.png"
---

Although Gitlab is not as popular as Github, it’s common to run across it these days. Especially after Microsoft acquired Github it seemed more individuals and organizations flocked over to Gitlab. 

In this post I want to document a couple of recon commands that are useful post-exploitation, and for blue teamers to watch out for. 

Let's assume one has access to a Gitlab Token as a precursor. Let's walk through some interesting commands and script snippets to leverage to find out more.

## Found a Gitlab Token - what to do?

First, let's define a couple environment variables that will be used recurringly.

```
TOKEN=$(cat .token)
GITLAB_HOST=[yourserver]
GITLAB_API="https://$GITLAB_HOST/api/v4"
```

That's it, and these variables are pretty self-explanatory.


### Who I Am?

The first obvious command once a token has been found is to check what user it belongs to.

```
curl --silent --header "Authorization: Bearer $TOKEN" "$GITLAB_API/user" | jq .username
```

Note: One can also provide a dedicated HTTP header `PRIVATE-TOKEN` to the Gitlab instance. More useful information is documented by [Gitlab here](https://docs.gitlab.com/ee/security/token_overview.html).

## Retrieving Projects

With a valid token one can enumerate projects:

```
curl --silent --header "Authorization: Bearer $TOKEN" \
     "$GITLAB_API/projects/owned=true&simple=true&per_page=100" | jq 
```

An adversary might look for projects they "own", so they can do check-ins, like for instance check-in a `.gitlab-ci.yml` file to modify/create a CI/CD pipeline.

Assuming you have a list of tokens (we are red teamers after all), one can also loop over and call the API with something like this:

```
TOKENS=$(cat .tokens)
for T in $TOKENS
do
    curl --silent --header "Authorization: Bearer $T" \
         "$GITLAB_API/projects/owned=true&simple=true&per_page=100" | jq 
done
```

It might also be useful to pipe this information into a file for laters analysis.

## Cloning Projects

Source Code often contains clear text credentials. To search through the code for secrets and other information Gitlab provides the capability to leverage the API token using `git` to pull projects from the server.

Here is how this looks like in action:

```
PROJ_PATH=MyProject/KoiPhish/KoiPhish.git
git -C ./src/ clone "https://gitlab-ci-token:$TOKEN@$GITLAB_HOST/$PROJ_PATH
```

Notice the username `gitlab-ci-token` in the URL. There are also a couple of other ways to do this. Again, refer to the [Gitlab documenation](https://docs.gitlab.com/ee/security/token_overview.html) for further details.

## Enumerating Gitlab CI/CD Variables

Gitlab CI/CD variables are great! Why? Because the often contain clear text secrets as well! 

A command to enumerate them is:

```
PROJECT_ID=123
curl --silent \
     --header "Authorization: Bearer $TOKEN" \ 
     "$GITLAB_API/projects/$PROJECT_ID/variables" | jq 
```

An adversary might find interesting passwords, tokens, or access keys in CI/CD variables.

**Tip: Engineers can protect their variables and not expose them widely. One can use protected variables that are only accessible from certain branches for instance.**

## Group and Instance-level Variables

Wait, there is more:

* Gitlab also has Groups and [Groups can have variables](https://docs.gitlab.com/ee/api/groups.html) also!
* [Instance-level CI/CD variables](https://docs.gitlab.com/ee/api/instance_level_ci_variables.html) are only accessible to the administrator.

## Runners Token

Inside the projects details I also discovered a `runners_token`. 

```
PROJECT_ID=123
curl --silent --header "Authorization: Bearer $TOKEN" \
     "$GITLAB_API/projects/$PROJECT_ID" | jq .runners_token | jq 
```

From the Gitlab documentation: 

> After registration, the runner receives an authentication token, which it uses to authenticate with GitLab when picking up jobs from the job queue. The authentication token is stored locally in the runner’s config.toml file.

Initially I assumed that token is the authentication token, but it's actually the **Registration Token**. 

It took me a while to realize that. Initially I had tried to verify the token as an auth token using:

```
curl --request POST "$GITLAB_API/runners/verify" --form "token=$RUNNER_TOKEN"
```

But that didn't work. Then I realized that the `runners_token` is the shown in the Projects CI/CD pipelines page as the Registration Token. So, gaining access to this implies that you can register your own runners! 

More information on how Runners registration works can be found [here](https://docs.gitlab.com/runner/register/).

Interestingly, just two days ago there was a [critical CVE issued](https://about.gitlab.com/releases/2022/02/25/critical-security-release-gitlab-14-8-2-released/#runner-registration-token-disclosure-through-quick-actions) by Gitlab regarding Runner Registration Tokens.

## SSH Keys 

Admins can also get [SSH keys for users](https://docs.gitlab.com/ee/api/keys.html) - I have not tried this yet, but it seems like one of these features that shouldn't exist, and I thought to point it out.

## Detecting Access Anomalies and Mitigations

At a high level the biggest challenge for defenders is that it's not obvious when a token was stolen, if it's use is coming from a legitimate user or not.

* For Blue Teamers identifying access anomalies is a good approach to identify if a token has leaked and is actively being abused by an adversary.
* Using Protected Branches and Protected Variables can also help limit exposure - so educate your engineeres and leverage them.

There is a lot of good information availabe on the Gitlab documentation pages on how to [lock down](https://docs.gitlab.com/runner/security/) tokens and runners as well, so review the information there for more details.


## Conclusion

This is a short introduction around Gitlab based recon and CI/CD attacks. There are really a lot of fun avenues for adversaries to abuse DevOps infrastructure. There is a lot of information accessible once one gains access to DevOps infrastructure. Keep an eye out as I will be posting more interesting research down the road.


## References

* [Gitlab API Documentation](https://docs.gitlab.com/ee/api)
* [Gitlab Access Token Documenation](https://docs.gitlab.com/ee/security/token_overview.html)
* [SSH keys for users](https://docs.gitlab.com/ee/api/keys.html)
* [Critical Gitlab CVE](https://about.gitlab.com/releases/2022/02/25/critical-security-release-gitlab-14-8-2-released/#runner-registration-token-disclosure-through-quick-actions)
