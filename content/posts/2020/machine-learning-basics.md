---
title: "My machine learning journey"
date: 2020-08-29T21:02:16-07:00
draft: true
tags: [
        "machine learning",
        "red",
        "huskyai"
    ]
---

This year I have been busy studying machine learning and artificial intelligence.

To come up with good attacks, I figured it is time to learn the fundamentals and start using software, tools and algorithms. My goal was to build a couple of end to end machine learning systems from scratch, and then attack them.

This post describes my studying approach, materials, courses, and learnings. I thought to share this, in case there are others who are interested to get started in this space but don't how and where.

Further posts will dive into AI/ML fundamentals and **adversarial techniques** with practical examples - for instance how to attack models, how to backdoor one, and so forth.

Along the way I even found a security bug, which I will share details about once fixed by Microsoft.

## My AI background

In college I took some **AI** and **Fuzzy Logic** classes, hence I was familiar with basic concepts around artificial intelligence. Although I had zero practical experience when it comes to machine learning, especially with current software such as **Tensorflow**, **Keras**, **PyTorch** or **fast.ai**. 

None of those even existed when I first learned about AI. 

Until a few months back, my last (and only) project in the AI space was building a "self-driving car parking simulator" using fuzzy logic - and that was in 2003 using **Matlab**. 

Interestingly, a peer student from college ended up building autonomous sailboats, and his [Roboat](http://www.roboat.at/en/home/) won the autonomous world sailing championships numerous times. 

At that time I got side-tracked with security engineering. :)

So, it seemed a good idea to step back and start from scratch when it comes to learning about AI and machine learning.

## The motivating factor

I'm a security engineer, mentor, builder and breaker. The motivating factor for me was to learn more about testing AI-powered systems, and more hopefully inspire others to do the same along the way. 

When I managed the **Azure Data Attack Team** at Microsoft's Cloud+AI division, we regularly were tasked to attack the Azure Machine Learning service offerings. Although the work focused on application and network side of things, rather than attacking ML models themselves. 

Microsoft is top-notch in the AI space when it comes to security by the way. There is a Trustworthy Machine Learning Group that does research and publishes information regularly. I recommend reading their [published material](https://docs.microsoft.com/en-us/security/engineering/failure-modes-in-machine-learning). Also, interesting to know that Facebook has a seperate dedicated [AI Red Team](https://www.zdnet.com/article/cybersecurity-how-facebooks-red-team-is-pushing-boundaries-to-keep-your-data-safe/). Machine Learning and AI at large are definitely an exciting places to mingle.

Naturally with the general AI/ML hype the last few years and news about adversaries misusing exposed systems, it seems good to expand skillset and knowledge regarding specific AI/ML testing skills (and integrate them in end-to-end red teaming scenarios). 

### Study material

Below is a list of core materials I have used for acquainting myself with the topic:

* [Machine Learning](https://www.coursera.org/learn/machine-learning), Andrew Ng
* [deeplearning.ai](https://deeplearning.ai), Andrew N
* [Tensorflow in practice](https://www.coursera.org/professional-certificates/tensorflow-in-practice), Laurence Moroney
* [Hacking Neural Networks - A short introduction](https://arxiv.org/pdf/1911.07658.pdf), Michael Kissner
* [fast.ai](https://www.fast.ai/), Jeremy Howard, Fast AI
* [Machine Learning and Security](https://www.amazon.com/Machine-Learning-Security-Protecting-Algorithms-ebook/dp/B079C7LKKY),  Clarence Chio, David Freeman
* [Failure modes in Machine Learning](https://docs.microsoft.com/en-us/security/engineering/failure-modes-in-machine-learning), Microsoft

In addition I watched countless YouTube videos, including Lex Fridman, Ariel Herbert-Voss, Fei Fei Li and many others.

When learning something new, I typically write the most important learnings and concepts down separately in a OneNote - *in my own words*. This helps me to internalize concepts. Additionally, I often create a `glossary` with key words and phrases to help me remember things:

![Machine Learning that I can understand](/blog/images/2020/ml-translation.jpg)

When I hear `derivative`, I just think `slope` for instance. Or, my favorite, `Dropout` is *like putting the neural network on drugs by dropping random cells*. Yes, that is actually a technique used to improve machine learning outcomes!

## Projects 

Taking courses or reading technical books **without actually building something** is not the best use of time in my opinion. When starting, I already had one project in mind and the second one came up soon after: 

1. **Mood prediction** Here is the hypothesis: Given a set of 20 features per day (such as distance run/biked, hours of sleep, playing violin, reading, number of social interactions, money spent, and so forth), can I predict my own mood? For that I have been collecting detailed data in an Excel sheet since the beginning of the year.

2. **Husky AI** The idea for the second project came as soon as I started taking Andrew Ng's course and worked on recognizing cats vs dogs. The main idea is to upload an image and the app will tell if the image a husky or not. 

![Husky AI](/blog/images/2020/huskyornot.jpg)

Going forward I will use this project to study and demo machine learning testing techniques.

All that said, let me share the ML specific material that taught me how to build something like "Husky AI" and do some kind of mini-review of the content.

## Machine Learning by Andrew Ng

I started by taking the [Machine Learning course by Andrew Ng](https://www.coursera.org/learn/machine-learning) on Coursera. 

The course was great I thought. It starts by teaching the basic math concepts with plenty of intuition - which was helpful for me. The material builds from the ground up and is very engaging and informative. You basically implement machine learning and gradient descent yourself with a scientific programming language called [Octave](https://www.gnu.org/software/octave/). The course is at least 7 years old now, which is probably the reason for that choice (compared to more popular Python these days). 

A lot of time is spent on understanding "gradient descent" and how logistic regression works. At that point I also started working on own AI experiments to play around with the algorithms and hyper-parameters to get better intuition around the algorithms. What I really liked was that we implemented the algorithm from scratch, rather then using frameworks.

Neural Networks are covered, but mostly in equal portion to other machine learning techniques. To me this was the most foundational course I took, really helping to get a good intuition what the math is doing.

There are also deep dives into **Anomaly Detection**, **Recommender Systems** and **Collaborative Filtering**.

Overall, I thought this course was put together very well, and I even had created my first couple of models and experiments myself at that point.

## deeplearning.ai

After finishing the Machine Learning course, I wanted to dive further into neural networks and focused on the content at [deeplearning.ai](https://deeplearning.ai).

It covers some of the same content as the previous class but is more modernized when it comes to tooling and nomenclature. The biggest difference is that it focuses solely on Neural Networks. The course covers machine learning aspects from image processing, natural language processing, as well as audio. 

You will learn a lot about the building blocks of neural networks, choosing testing parameters (hyper-parameters). It goes into details around activation functions such as ReLu, Softmax, tanh,...  so it's overall quite detailed and helps deepen the concepts. 

I found it interesting to look at the middle (also called hidden) layers of neural networks and inspect their results. This gives great insights into what happens (trying to make sense of it all) - or maybe it allows one to ask more questions. Certainly, something that you should do with your own models - it's fascinating to explore.

The deep dive into convolutional neural networks was very insightful.

Language wise the course focuses on Python (Jupyter Notebooks, numpy, some Tensorflow/Keras,...).

The course also covers RNNs (recurrent neural nets), and ResNets (residual networks) which I had heard of before but never knew what they meant. 

The most difficult part to understand for me were sequence models. For those I still need to build a separate project from scratch to play around more to understand it better

Overall, this is a polished and modernized version of the Machine Learning course with focus on neural nets and some overlapping content but also a lot of new material which makes it a good follow up course.

## TensorFlow in Practice, Laurence Moroney

This course was much more hands-on and with the knowledge I had previously acquired it went quite smoothly. It also became clear that ML courses seem to fall back to some basic datasets and examples (cats vs dogs, digits,...) when teaching the basics.

I liked the excercises around recognizing hand gestures, which I had not seen before.

Even though TensorFlow and Keras are the systems I know best at this point, I'm not quite sure I like TensorFlow that much. It doesn't seem very intuitive and there are odd breaking changes from TensorFlow 1.0 to 2.0 which break backwards compatibility for no good reason it seems.

A simple example is when it comes to *metrics measurement*. There is a dictionary with the key `acc` in TensorFlow 1 and that key was renamed to `accuracy` in version 2. Now all the scripts and examples, including those from Google themselves throw errors and are not working as expected.

The course itself is high quality in the beginning, but somewhere towards the middle it drops slightly with instructions for exercises - I believe they are still upgrading/improving it over time. 

Using `Keras` seems more straight forward compared to pure TensorFlow and [Google's Collab platform](https://colab.research.google.com) works great and it even includes free GPU based machines.

Along the same lines, I also ran across [Paperspace Gradient](https://gradient.paperspace.com/), who offer free GPU VMs!


## fast.ai by Jeremy Howard

After building my own models and experimenting for a while, I got curious about `fast.ai`. So far, I have not had the time to go through majority of the material. The library and API seem really intuitive and focused on productivity. I will spend more time the in the future learning how to use it. I might eventually port **Husky AI** to fast.ai or PyTorch though.

## Hacking Neural Networks by Michael Kissner 

Oh, wow - this post is getting very long. I still want to highlight the paper [Hacking Neural Networks - A short introduction](https://arxiv.org/pdf/1911.07658.pdf). It comes with a Github repo that helps you do exercises to learn more about adversarial techniques, and even **has a super cool example on how to exploit a buffer overflow in the GPU to overwrite model information**. 


## Conclusion

Without taking the Machine Learning or deeplearning.ai courses I would have not understood a lot of things in the paper of Michael Kissner for instance. There are other videos I am watching/re-watching on YouTube now, which make a whole lot more sense as well now. It was time well spent I think to go threw the material and work on the excercises, as well as building my own models to complement and put the learned information to good use.

## Going forward

My next post will describe "Husky AI" in more detail and how it was built and put into production. 

And then we are going to threat model and attack the system. The first attack will be bruteforcing from the outside to see how feasible it is to come with random images that trick the model, then backdooring and updating the model directly - which might be a useful technique to be aware of when performing internal red teaming operations. This should be fun!



Twitter: [@wunderwuzzi23](https://twitter.com/wunderwuzzi23)


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
* [Facebook AI Red Team](https://www.zdnet.com/article/cybersecurity-how-facebooks-red-team-is-pushing-boundaries-to-keep-your-data-safe/)
* [Simulation Neuronaler Netzwerke](https://www.amazon.com/Simulation-neuronaler-Netze-Andreas-Zell/dp/3486243500), Andreas Zell
* [Octave](https://www.gnu.org/software/octave/)