---
title: "Machine Learning Attack Series: Generative Adversarial Networks (GANs)"
date: 2020-11-25T17:55:15-08:00
draft: true
tags: [
        "machine learning",
        "red",
        "huskyai"
    ]
---

In this post we will explore Generative Adversarial Networks (GANs) to create fake husky images. The goal is, of course, to have "Husky AI" misclassify them as real huskies.

If you want learn more about Husky AI visit the [Overview](/blog/posts/2020/husky-ai-walkthrough/) post.


## Generative Adversarial Networks

One of the attacks I wanted to investigate for a while was the creation of fake images to trick Husky AI. The best approach seemed by using Generative Adversarial Networks (GANs). [It happened that right then deeplearning.ai started offering a GAN course by Sharon Zhou](https://www.deeplearning.ai/generative-adversarial-networks-specialization/). 

That was excellent timing, and I signed up right away.

The course teaches the basics of GANs with plenty of examples - from building basic `DCGAN` to `WGAN-GP` and even `StyleGAN`. The cool thing is that the course teaches everything from scratch. I had only used `TensorFlow` before and this course is entirely in `PyTorch`, so I finally got to use `PyTorch` a bit as well.

## What is a GAN?

If you are not at all familiar with GANs, let me give you a brief overview. 

GANs were [invented by Ian Goodfellow](https://www.iangoodfellow.com/slides/2016-12-9-gans.pdf). The basic idea is that there are two neural networks, namely:

1. Generator
2. Discriminator

These two neural networks compete in a `minimax` game.

[![Husky](/blog/images/2020/gans-minimax.png)](/blog/images/2020/gans-minimax.png)

The `Generator` creates random images based on noise and sends them to the `Discriminator`. The `Discriminator` is fed both real images and fake images and predicts if an image is real or fake. Using the losses of these predictions (learnings) the neural networks of both players are adjusted and the game continues. Over time the `Discriminator` starts creating better and better images.

The following image shows my progress in regard to creating fake husky images:

[![Husky](/blog/images/2020/gans.png)](/blog/images/2020/gans.png)

The first image shows a `DCGAN` which failed because of a `mode collapse`, meaning it got stuck somewhere in a local minimum during training. The second image used a `WGAN`, and the third one is `StyleGAN2-ADA`.

## StyleGAN2-ADA

`StyleGAN2-ADA` is especially great when you have a small training set, as it performs image augmentation. Here are the final and best results I had with `StyleGAN2-ADA`, look:

[![Husky](/blog/images/2020/huskies-that-dont-exist.png)](/blog/images/2020/huskies-that-dont-exist.png)

Those images look pretty amazing, and **remember that none of these huskies actually exist!**

Along the way during training there were also some fun images, look:

[![Husky](/blog/images/2020/fake-huskies-fail.png)](/blog/images/2020/fake-huskies-fail.png)

You can see how my training data was lacking pictures that showed huskies with legs, as those are commonly misaligned in the images the Discriminator created.

## Tricking Husky AI with the fake images

In the end I took one of the fake husky images and uploaded it to `Husky AI`. Here is the resulting prediction:

[![Husky](/blog/images/2020/husky-ai-gan-fakehusky-baypass.png)](/blog/images/2020/husky-ai-gan-fakehusky-baypass.png)

As you can see `Husky AI` was entirely tricked by the fake image. I thought this was a really cool attack. Although, I was wondering if this is maybe because the images for training the production model and training the GAN were the same.

So, I tried a couple of other image classification AIs out there.

## Tricking Google's Vision AI!

What about Google's Vision AI? 

Take a look at the result:

[![Husky](/blog/images/2020/googleai-huskyai.png)](/blog/images/2020/googleai-huskyai.png)

**Wow, even Google's AI was tricked.** 

This really shows how difficult it is already for computers and humans to distinguish between real and fake content, and the quality of fake content and images is only going to improve. 

Quite scary if you think about it. A few years from now, you will not be able to trust any information, image or video you see online. 

### Google Colab Notebook - Code

For `StyleGAN2-ADA`, let me show you some of the core code snippets from my Google Colab Notebook. The code for `StyleGAN2-ADA` can be downloaded from [NVidia's Github repo](https://github.com/NVlabs/stylegan2-ada). 

The first step was to upload the husky training images and pre-process them, so they all have the same size.

```
!unzip "/content/drive/My Drive/HuskyAI/1600huskies.zip" -d "images/train/"
!mkdir /content/images/stylegan

NUM_PX = 512
SAVE_DIR = "/content/images/stylegan"
DATA_DIR    = "/content/images/train/huskies"

for filename in tqdm(os.listdir(DATA_DIR)):
    path = os.path.join(DATA_DIR, filename)
    image = Image.open(path).resize((NUM_PX, NUM_PX))

    save_path = os.path.join(SAVE_DIR, filename)
    image.save(save_path)
```

Next, I used the `dataset_tool.py` of StyleGAN2 to create images of the correct format that StyleGAN requires:

```
!python dataset_tool.py create_from_images /content/datasets/huskies /content/images/stylegan/
```

And then it was time to do the training:

```
!python train.py --outdir=/content/drive/My\ Drive/HuskyAI/stylegan2-ada-results --gpus=1 \ 
--data=/content/datasets/huskies \
--mirror=1 --snap=1 
```

It's also possible to stop the training and resume at a later point by providing the `.pkl` files that are saved by `StyleGAN2` along the way.

After the training, we can create new fake images.

```
!python generate.py generate-images \
--network=/content/drive/My\ Drive/HuskyAI/stylegan2-ada-results/00002-huskies--mirror-auto1-resumecustom/network-snapshot-000250.pkl --seeds=670 class
```

Those are the core basic steps of using StyleGAN2-ADA.

## Videos and Interpolation

There is also a [fork of StyleGAN2-ADA](https://github.com/PDillis/stylegan2-ada) by Diego Porres that can be used to create fun interpolation videos. Unfortunatley, my `mp4` video doesn't show up inline here in the blog. 

You can however [watch it on Twitter](https://twitter.com/wunderwuzzi23/status/1320723215686066176).


## Conclusion

That's it. As you can see, GANs are super powerful and it’s quite easy to setup and train yourself. And as I mentioned this is all quite scary if you think about it. A few years from now, you won't be able to trust absolutely any information, image or video you see online.

Cheers, Johann.

Twitter: [@wunderwuzzi23](https://twitter.com/wunderwuzzi23)


## References

* [StyleGAN2-ADA, NVIDIA](https://github.com/NVlabs/stylegan2-ada)
* [StyleGAN2-ADA, Diego Porres's fork for vide and interpolation](https://github.com/PDillis/stylegan2-ada)
* [GANs, Ian Goodfellow](https://www.iangoodfellow.com/slides/2016-12-9-gans.pdf)
* [deeplearning.ai - GAN Spezialization](https://www.deeplearning.ai/generative-adversarial-networks-specialization/)

## Appendix 

[![Husky](/blog/images/2020/fakehusky.png)](/blog/images/2020/fakehusky.png)

### Attacks Overview

These are the core ML threats for Husky AI that were identified in the [threat modeling session](/blog/posts/2020/husky-ai-threat-modeling-machine-learning/) so far and that I want to research and build attacks for. 

Links will be added when posts are completed over the next serveral weeks/months. The more I learn, the more attack ideas come to mind also, so there will likely be more posts eventually.

1. [Attacker brute forces images to find incorrect predictions/labels](/blog/posts/2020/husky-ai-machine-learning-attack-bruteforce/) - Bruteforce Attack
2. [Attacker applies smart ML fuzzing to find incorrect predictions](/blog/posts/2020/husky-ai-machine-learning-attack-smart-fuzz/) 
2. [Attacker performs perturbations to misclassify existing images - Perturbation Attack](/blog/posts/2020/husky-ai-machine-learning-attack-perturbation-external/) 
3. [Attacker gains read access to the model - Exfiltration Attack](/blog/posts/2020/husky-ai-machine-learning-model-stealing/)
4. [Attacker modifies persisted model file - Backdooring Attack](/blog/posts/2020/husky-ai-machine-learning-backdoor-model/)
5. [Attacker denies modifying the model file - Repudiation Attack](/blog/posts/2020/husky-ai-repudiation-threat-deny-action-machine-learning/)
6. Attacker poisons the supply chain of third-party libraries 
7. Attacker tampers with images on disk to impact training performance
8. [Attacker modifies Jupyter Notebook file to insert a backdoor (key logger or data stealer)](/blog/posts/2020/cve-2020-16977-vscode-microsoft-python-extension-remote-code-execution/)
9. [Attacker uses Generative Adversarial Networks to create fake husky images](/blog/posts/2020/machine-learning-attack-series-generative-adversarial-networks-gan/) (this post)

