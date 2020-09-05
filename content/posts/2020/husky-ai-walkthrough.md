---
title: "The machine learning pipeline and attacks"
date: 2020-09-02T12:04:29-07:00
draft: true
tags: [
        "machine learning",
        "huskyai"
    ]
---

This post is part of a series "Machine learning for pen testers" to help pen testers and security engineers learn more about AI/ML. 

In the [previous post](/blog/posts/2020/machine-learning-basics/) I talked about good resources for learning more about artificial intelligence and machine learning in general, and how I started my journey in this space. 

## What is Husky AI?

Husky AI allows a user to upload an image, and get an answer back if the image contains a husky or not. Below is a screenshot of the application:

[![Husky AI](/blog/images/2020/husky-ai.jpg)](/blog/images/2020/husky-ai.jpg)

If you watched the show "Silicon Valley", you might find this quite funny.

## What will be covered?

In the next few posts I will describe how I built and operationalized "Husky AI", which is a playground application I use to learn about machine learning and perform attacks.

1. [**Machine Learning Pipeline**](/blog/posts/2020/husky-ai-walkthrough/) - High Level overview of the bits and pieces (this post)
2. [**Building a machine learning model**](/blog/posts/2020/husky-ai-building-a-machine-learning-model/) - The steps involved in building Husky AI model and training it
3. [**Operationalizing the model**](/blog/posts/2020/husky-ai-mlops-operationalize-the-model/) - how did I make the model invokable from a web site
4. [**Threat Modeling the system**](/blog/posts/2020/husky-ai-walkthrough/) - Identifying threats and attacks, with a focus on the AI/ML threats


Let's get started with an overview of the machine learning pipeline. This will already highlight many of the moving pieces that can be attacked!

# Part 1 - Machine Learning Pipeline{#part1}

I assume that the reader is familiar with security and pen testing, but less so with machine learning.

The following images shows the big components and stages of the pipeline:

[![Husky Machine Learning Pipeline](/blog/images/2020/machine-learning-pipeline.jpg)](/blog/images/2020/machine-learning-pipeline.jpg)

This overview of the ML pipeline will lead eventually help us build a data flow diagram (DFD) that we can use to identify threats and attacks later on:

These are the individual parts involved:

1. **Collecting training and validation images:** The process starts with gathering images. These are pictures of huskies and random other pictures.
2. **Pre-Processing:** The images go through some pre-processing steps (this includes data cleansing, rescaling, labeling, storing,..)
3. **Defining the initial model:** We have to start somewhere, my very first test was a network with a single neuron (basically just logistic regression). At this point it is also **important to set a measurable metric** that can be used to compare training runs with each other. This is to be able to meaningfully improve the model
4. **Training the model**
5. **Evaluation:** After a training (even during) measure performance of the model (e.g. accuracy)
6. **Improve the model:** If performance is not met, update the model (e.g. trying different algorithms, learning rates, neural network layouts,...) and start training again
7. **Deployment:** If the performance of the model looks good, deploy to production
8. **Operationalization:** Make the model available for usage via REST API. In my production deployment there is an API gateway which routes traffic to the Husky AI web server. The REST endpoint then calls into the model give predictions for uploaded images
9. **End user runs predictions:** A user can invoke the REST API to upload an image and get a prediction
10. **(Online Feedback Loop)**: Not visible in the image, because Husky AI does not do that. Production systems typically collect user data and feed it back into the pipeline (Step 1) - this turns the pipeline into an "AI lifecycle". 
 
That's it. Those are the big conceptual pieces of a machine learning pipeline. 

The next post covers technical details on how the Husky AI machine learning model was built and trained.


