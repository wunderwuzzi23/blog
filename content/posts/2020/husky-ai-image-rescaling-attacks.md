---
title: "Machine Learning Attack Series: Image Scaling Attacks"
date: 2020-10-28T13:00:27-07:00
draft: true
tags: [
        "machine learning",
        "huskyai",
        "red"
    ]
twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "Machine Learning Attack Series: Image Scaling Attacks"
  description: "Machine Learning Attack Series: Image Scaling Attacks"
  image: "https://embracethered.com/blog/images/2020/ml-attack-series.jpg"
---


This post is part of a series about machine learning and artificial intelligence. Click on the blog tag "huskyai" to see related posts. 

* [Overview](/blog/posts/2020/machine-learning-attack-series-overview/): How Husky AI was built, threat modeled and operationalized
* [Attacks](/blog/posts/2020/husky-ai-threat-modeling-machine-learning/): Some of the attacks I want to investigate, learn about, and try out

A few weeks ago while preparing demos for my GrayHat 2020 - Red Team Village presentation I ran across "Image Scaling Attacks" in [Adversarial Preprocessing: Understanding and Preventing Image-Scaling Attacks in Machine Learning](https://www.usenix.org/system/files/sec20-quiring.pdf) by Erwin Quiring, et al.

I thought that was so cool!

## What is an image scaling attack?

The basic idea is to hide a smaller image inside a larger image (it should be about 5-10x the size). The attack is easy to explain actually:

1. Attacker crafts a malicious input image by hiding the desired target image inside a benign image
2. The image is loaded by the server
3. Pre-processing resizes the image
4. The server acts and makes decision based on a different image then intended

My goal was to hide a husky image inside another image:

[![Image Rescaling Attack](/blog/images/2020/image-rescale-attack.gif)](/blog/images/2020/image-rescale-attack.gif)

Here are the two images I used - before and after the modification:
[![Image Rescaling Attack](/blog/images/2020/image-rescaling-attack-schoenbrunn.png)](/blog/images/2020/image-rescaling-attack-schoenbrunn.png)

If you look closely, you can see that the second image does have some strange dots all around. But this is not noticable when viewed in smaller version. 

You can find the code on [Github](https://github.com/EQuiw/2019-scalingattack). I used Google Colab to run it, and there were some errors initialy but it worked - let me know if interested and I can clean up and share the Notebook also.

## Rescaling and magic happens!

Now, look what happens when the image is loaded and resized with `OpenCV` using default settings:

[![Image Rescaling Attack](/blog/images/2020/image-rescaling-attack.png)](/blog/images/2020/image-rescaling-attack.png)

On the left you can see the original sized image, and on the right the same image downsized to 128x128 pixels. 

**That's amazing!**  

The downsized image is an entirely different picture now! Of course I picked a husky, since I wanted to attack "Husky AI" and find another bypass.

## Implications

This can have a set of implications:

1. **Training process:** Images that poisen the training data (as pre-processing rescales images)
2. **Model queries:** The model might predict on a different image than the one the user uploaded
3. **Non ML related attacks:** This can also be an issue in other, non machine learning areas.

I guess security never gets boring, there is always something new to learn.

## Mitigations

Turns out that Husky AI uses PIL and that was not vulnerable to this attack by default. 

I got lucky, because initially Husky AI did use `OpenCV` and it's default settings to resize images. But for some reason I changed that early on (not knowing it would also mitigate this attack). 

If you use `OpenCV` the issue can be fixed by using the `interpolation` argument when calling the `resize` API to change the default behavior.

Hope that was useful and interesting.

Cheers,
Johann.

Twitter: [@wunderwuzzi23](https://twitter.com/wunderwuzzi23)

*If you are interested in red teaming at large, check out my book [Red Team Strategies](https://www.amazon.com/gp/product/1838828869/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=1838828869&linkCode=as2&tag=wunderwuzzi-20&linkId=b6523e937607be47499c6010ff489537).*

## References

* Adversarial Preprocessing: Understanding and Preventing Image-Scaling Attacks in Machine Learning (https://www.usenix.org/system/files/sec20-quiring.pdf) (Erwin Quiring, TU Braunschweig)
* https://github.com/EQuiw/2019-scalingattack 
* https://scaling-attacks.net
