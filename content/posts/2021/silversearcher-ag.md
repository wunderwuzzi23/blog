---
title: "Using The Silver Searcher to search through code and files quickly"
date: 2021-07-28T11:44:20-07:00
draft: true
tags: [
        "tools"
    ]
---


In this very short post I wanna talk mention [The Silver Searcher](https://github.com/ggreer/the_silver_searcher), which I just learned about a few weeks ago.

In the past I have written quite a bit about the importance of [credential hunting for your organization](/blog/posts/2020/hunting-for-credentials) and some cool [built-in operating system indexing features](/blog/posts/2021/invoke-windowssearch-credential-hunt) that can be used as well. 

Of course `grep` and `findstr` are also in every red teamers toolbox. 

As part of a coding project I recently learned about "The Silver Searcher", which is very fast and has some neat features built it. It's focus is source code searching.

## Installation

On Ubuntu just run the following command:

`sudo  apt-get install silversearcher-ag`

or if you are on macOS you can grab it via `brew`:

`brew install the_silver_searcher`

Its also available for Windows - check out the [Silver Searcher Github repo](https://github.com/ggreer/the_silver_searcher) for details.


## Usage and useful features

You can search for specific file types only.

`ag --html shadowbunny`

Here are some useful options:

```
-A  Print lines after match (Default: 2)
-B  Print lines after match (Default: 2)
-u --unrestricted 
-z --search-zip
-v --invert-match
--ignore PATTERN  
-p --path-to-ignore STRING
-C --context [LINES]  
```

## Credential hunting!

Credential hunting is important and its good to know about more useful tools that can help in this space. Check it out. 


## References

* [The Silver Searcher Github Repo](https://github.com/ggreer/the_silver_searcher)