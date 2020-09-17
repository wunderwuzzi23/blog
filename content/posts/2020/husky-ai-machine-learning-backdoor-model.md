---
title: "Machine Learning Attack Series: Backdooring"
date: 2020-09-16T18:59:47-07:00
draft: true
tags: [
        "machine learning",
        "huskyai",
        "red"
    ]
---

This post is part of a series about machine learning and artificial intelligence. Click on the blog tag "huskyai" to see related posts. 

* [Overview](/blog/posts/2020/husky-ai-walkthrough/): How Husky AI was built, threat modeled and operationalized
* [Attacks](#appendix): The attacks I want to investigate, learn about, and try out

The previous posts covered ...


In this post we will talk about two ways to backdoor a model, namely:

1. Backdooring a model file (`.h5`) by manually modifyting it
2. Backdooring via re-training the model to insert a backdoor pattern

For both attacks the adversary has access to the model file in production and be able overwrite it with a new model file.

Let's dive into it.

## Backdooring a model file 

A common format for models is the HDFS format, version 5. It uses the file extension `.h5`.


## Backdooring by re-training

The second approach
