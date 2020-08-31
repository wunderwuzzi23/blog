---
title: "A machine learning journey"
date: 2020-08-29T21:02:16-07:00
draft: true
tags: [
        "machine learning",
        "red"
    ]
---

Over the last several months I have been busy studying machine learning and artificial intelligence.

To come up with good attacks, I figured it is time to learn the basics and start using software, tools and algorithms. My goal was to build a couple of end to end machine learning systems from scratch.

This post describes my studying approach, materials and learnings. I thought to share this information, in case there are others who are intersted to get started in this space and need some pointers.

Further posts will dive into AI/ML fundamentals and **adversarial techniques** with practical examples (e.g. attacking and backdooring models and so forth).

Along the way I did even find a security bug already - which I will share details about once fixed by Microsoft.

## My artificial intelligence background

In college I took some `AI` and `Fuzzy Logic` classes, hence I am familiar with some concepts around artificial intelligence. Although I basically had zero practical experience when it comes to machine learning, especially with current software such as `Tensorflow`, `Keras`, `PyTorch` or `fast.ai`. None of those even existed when I first learned about AI. 

Until a few months back, my last (and only) project in the AI space was building a "self-driving car parking simulator" using fuzzy logic - and that was in 2003 using `Matlab`. Interestingly, a peer student from back then ended up building autonomous sailboats, and his [Roboat](http://www.roboat.at/en/home/) won the autonomous world sailing championships numerous times. 

I got sidetracked with security engineering at that time. 

So, it seemed a good idea to basically start from scratch when it comes to learning AI and machine learning.

## The motivating factor

I'm a professional security engineer, breaker and tester. The main motivating factor is to learn about testing AI-powered systems, and be able to inspire others to do the same.

When I managed the **Azure Data Attack Team** at Microsoft's Cloud+AI division, we regularly were tasked to attack Azure Machine Learning service offering. Although the work focused on application and network side of things, rather than attacking models themselves. Microsoft is top-notch in the AI space when it comes to security. There is even a **Trustworthy Machine Learning Group** that does research and publishes information regularly. I recommend reading up about their [work](https://docs.microsoft.com/en-us/security/engineering/failure-modes-in-machine-learning). Also, interesting to know that Facebook has a seperate dedicated [AI Red Team](https://www.zdnet.com/article/cybersecurity-how-facebooks-red-team-is-pushing-boundaries-to-keep-your-data-safe/).

Naturally with the general AI/ML hype the last few years and news about adversaries misusing them, it seems good to expand skillset and knowledge in regard to specific AI/ML testing skills (in addition to typically end-to-end red teaming scenarios). 

### Study material so far

Below is a list of the core materials I have used for studying so far:

* [Machine Learning](https://www.coursera.org/learn/machine-learning), Andrew Ng
* [deeplearning.ai](https://deeplearning.ai), Andrew Ng
* [Hacking Neural Networks - A short introduction](https://arxiv.org/pdf/1911.07658.pdf), Michael Kissner
* [Tensorflow in practice](https://www.coursera.org/professional-certificates/tensorflow-in-practice), Laurence Moroney
* [fast.ai](https://www.fast.ai/), Jeremy Howard, Fast AI
* [Machine Learning and Security](https://www.amazon.com/Machine-Learning-Security-Protecting-Algorithms-ebook/dp/B079C7LKKY)
* [Failure modes in Machine Learning](https://docs.microsoft.com/en-us/security/engineering/failure-modes-in-machine-learning), Microsoft

When studying I typically write the most important learnings and concepts down separately in a OneNote - *in my own words*. This helps me to internalize concepts. It also led to this `glossary` that I use now regularly, here is a screenshot:

![Machine Learning that I can understand](/blog/images/2020/ml-translation.jpg)

When I hear `derivative`, I just think `slope` for instance. Or, my favorite, `Dropout` is like putting the neural network on drugs by dropping random cells. Yes, that is actually a technique used to improve machine learning outcomes!

## Projects 

It turns out that doing and building something is the best learning approach. When taking the class, I immediately had two projects that I wanted to work on and apply machine learning to. 

1. **Mood prediction** Here is the hypothesis: Given a set of 20 features per day (such as distance run/biked, hours of sleep, playing violin, reading, number of social interactions, money spent, and so forth), can I predict my own mood? For that I have been collecting detailed data in an Excel sheet since the beginning of the year.

2. **Husky AI** The idea for the second project came as soon as I started taking Andrew Ng's course and worked on recognizing cats vs dogs. I will focus on this project to learn more about adversarial machine learning techniques.

![Machine Learning that I can understand](/blog/images/2020/huskyornot.jpg)

Now, let me discuss the  material that taught me how to build something like that.

## Machine Learning by Andrew Ng

I started by taking the [Machine Learning course by Andrew Ng](https://www.coursera.org/learn/machine-learning) on Coursera. 

The course was great I thought. It starts by teaching the basic math concepts with plenty of intuition - which was helpful for me. The material builds from the ground up and is very engaging and informative. You basically implement machine learning and gradient descent yourself with a scientific programming language called [Octave](https://www.gnu.org/software/octave/). The course is about 7-8 years old now, which is probably the reason for that choice (compared to more popular Python these days). 

A lot of time is spent on understanding "gradient descent" and how logistic regression work. At that point I also started working on own AI experiments to play around with the algorithms and hyper-parameters.

Neural Networks are covered, but mostly in equal portion to other machine learning techniques. To me this was the most foundational course I took, really helping to get a good intuition of some of the concepts behind the math.

There is also an entire sections and deep dives into `Anomaly Detection`, `Recommender Systems` and `Collaborative Filtering`.

Overall, I really thought this course was put together very well.

## deeplearning.ai

After finishing the Machine Learning course, I focused on taking the [deeplearning.ai](https://deeplearning.ai) classes. 

It covers some of the same content as the previous class but is more modernized when it comes to tooling and nomenclature. The biggest difference is that it focuses solely on Neural Networks. The course covers machine learning aspects from image processing, natural language processing, as well as audio. 

You will learn a lot about the building blocks of neural networks, choosing testing parameters (hyper-parameters). It goes into details around activation functions such as ReLu, Softmax, tanh,...  so it's overall quite detailed and helps build good intuition. 

I found it super interesting to look at the middle (also called hidden layers) of neural networks and inspect their results. This gives great insights into what happens - or maybe it allows one to ask more questions. 

Language wise the course focuses on Python (Jupyter Notebooks, numpy, some Tensorflow/Keras,...).

The course also dives into RNNs (recurrent neural nets), and ResNets (residual networks) which I had heard of before but never knew what they meant. 

The most difficult part to understand - which I still have not grasped are sequence models. For that I might have to build a separate project to play around more to understand it.

Overall, this is a polished and modernized version of the Machine Learning course - with some overlapping content but also a lot of new material which makes it a good follow up course.

## TensorFlow in Practice, Laurence Moroney

This course was much more hands-on and with the knowledge I had previously acquired it went quite smoothly. It also becomes clear that most courses seem to fall back to some basic datasets and examples when teaching machine learning and neural networks.

Even though TensorFlow and Keras are the systems I know best at this point, I'm not quite sure I like TensorFlow that much. It doesn't seem very intuitive and there are odd breaking changes from TensorFlow 1.0 to 2.0 which reflects Google's lack of being able to maintain backwards compatibility (they are kind of known for terminating products and services out of the blue). 

A simple example is when it comes to metrics measurement when training. There is a dictionary with the key `acc` in TensorFlow 1 and that key was renamed to `accuracy` in version 2. Now all the scripts and examples, including those from Google themselves through errors and are not working as expected.

The course is high quality in the beginning, but I think towards the middle it drops slightly with instructions for exercises - I believe they are still upgrading/improving it over time. 

As long is one can use `Keras` things seem to be a bit more straight forward, and [Google's Collab platform](https://colab.research.google.com) works great and it even includes free GPU based machines.

## fast.ai by Jeremy Howard

After building my own models and experimenting for a while, I got curious about `fast.ai`. So far, I have not had the time to go through majority of the material. The library and API seem really intuitive and focused on productivity. I will spend more time the next few weeks learning how to use it. I'm not yet sure if I will port `Husky AI` to fast.ai or PyTorch.

## Hacking Neural Networks by Michael Kissner 

Oh, wow - this post is getting very long. But, I still want to highlight the paper [Hacking Neural Networks - A short introduction](https://arxiv.org/pdf/1911.07658.pdf). It also comes with a Github repo that helps you do exercises to learn more about adversarial techniques towards ML models, and even has one super cool example on how to exploit a buffer overflow in the GPU to overwrite model information. 

Without having taken the Machine Learning or deeplearning.ai courses I would have not understood a lot of things in the paper of Michael Kissner. But since I had done some of the groundwork, it turned out to be an excellent exercise to go through the hacking exercises. It was fantastic.


## Going forward

My next post will describe "Husky AI" in more detail and how it was built, and then we go into the details of performing attacks on the model. The first attack will be backdooring and updating the model itself - which might be a useful technique to be aware of when performing internal red teaming operations.


If you found this interesting or have questions, feel free to send a DM to [@wunderwuzzi23](https://twitter.com/wunderwuzzi23)


## References

* [Machine Learning](https://www.coursera.org/learn/machine-learning), Andrew Ng
* [deeplearning.ai](https://deeplearning.ai), Andrew Ng
* [Hacking Neural Networks - A short introduction](https://arxiv.org/pdf/1911.07658.pdf), Michael Kissner
* [TensorFlow in practice](https://www.coursera.org/professional-certificates/tensorflow-in-practice), Laurence Moroney
* [fast.ai](https://www.fast.ai/), Jeremy Howard, Fast AI
* [Machine Learning and Security](https://www.amazon.com/Machine-Learning-Security-Protecting-Algorithms-ebook/dp/B079C7LKKY)
* [Failure modes in Machine Learning](https://docs.microsoft.com/en-us/security/engineering/failure-modes-in-machine-learning), Microsoft
* [Husky AI](https://wuzzi.net/huskyai)
* [Autonomous Sailboat - Roboat](http://www.roboat.at/en/home/)
* [Can machine learning be secure?](http://people.ischool.berkeley.edu/~tygar/papers/Machine_Learning_Security/asiaccs06.pdf) 
* [Facebook AI Red Team](https://www.zdnet.com/article/cybersecurity-how-facebooks-red-team-is-pushing-boundaries-to-keep-your-data-safe/).
