---
title: "Machine Learning Attack Series: Adversarial Robustness Toolbox Basics"
date: 2020-10-22T16:34:48-07:00
draft: true
tags: [
        "machine learning",
        "huskyai",
        "red"
    ]
---

I wanted to explore the "Adversarial Robustness Toolbox" (ART) for a while to understand how it can be used to create adversarial examples for Husky AI. 

A few weeks ago I described [a range of perturbation attacks](/blog/posts/2020/husky-ai-machine-learning-attack-perturbation-external/) to modify an existing image of the plush bunny to get it misclassified as a husky.

![Shadowbunny](/blog/images/2020/huskyai-shadowbunny.png)

The goal of this post is to do the same, but leveraging ART.

## Adversarial Robustness Toolbox (ART)

[ART](https://adversarial-robustness-toolbox.org/) was originially created by IBM and moved to the Linux AI Foundations in July 2020. It comes with a range of attack modules, as well as mitigation techniques and supports a wide range of machine learning frameworks.

The code of creating an adversarial examples is straight forward. There is [good documentation](https://adversarial-robustness-toolbox.readthedocs.io/en/latest/modules/attacks/evasion.html#) available also. What I found missing is a  discussion forum for ART so far - although I was able to eventually figure the few things I had questions about out so far.

## Basic Attack - Fast Gradient Sign Method

My initial goal was to just get a basic adversarial example using FGSM. To get started with Keras and ART first import a couple of modules.

```
from art.attacks.evasion import FastGradientMethod
from art.estimators.classification import KerasClassifier
```

Interestingly, I had to disable eager execution in TensorFlow:

```
tf.compat.v1.disable_eager_execution()
```

Then load the model file:

```
model = keras.models.load_model("/content/huskymodel.h5")
```

And create an ART `KerasClassifier` for the attack:

```
classifier = KerasClassifier(model=model, clip_values=(0, 1), use_logits=False)
```

After that we have everything ready to perform an attack and create the adversarial example. Let's first load the image and print it's current prediction:


```
image = load_image("/tmp/shadowbunny.png")
predictions = classifier.predict(image)
print(predictions)
```

The result was `[[0.00195146]]`. 

Now let's write ART code to perturbe the pixels using the FGSM attack:

```
image = load_image("/tmp/shadowbunny.png")

attacker = FastGradientMethod(estimator=classifier, eps=0.04, targeted=True)
x_attack = attacker.generate(x=image, y=[0.9])
predictions = classifier.predict(x=x_attack)

print(predictions)
plt.imshow(x_test_adv[0])
```

Here is a quick description of the steps in the above code:

1. First we load the benign plush bunny image that we want misclassify
2. Then we create the FGSM attack instance. Note the API is called with `targeted=True`, which means we want the output to fall into a certain class/label. Husky AI uses a binary classifier and this was the way I got it to work.
3. Next we perform the attack and generate an image with perturbed pixels. Because `targeted=True` was specified we have to provide a label `y` value. Although in this case (binary classifier) I have not fully grasped what the `y` value is actually doing. The outcome is correct regardless of the actual value of `y`.
4. Finally we call `predict` on the newly created attack image `x_attack` and display it as well.

[![ART FGSM Shadowbunny](/blog/images/2020/art.shadowbunny.png)](/blog/images/2020/art.shadowbunny.png)

Notice how the prediction score is now at `66%`. The attack worked!
In this case its also visible that the image slighlty changed -- observe how the white background is not entirely plain white anymore.


## Conclusion

Hope this quick introduction helps you get started with ART. I will be exploring a lot more going forward. Compared to the manual attacks and code I had to write in the previous posts about perturbations, the ART libraries makes these attacks much simpler to perform. 

Twitter: [@wunderwuzzi23](https://twitter.com/wunderwuzzi23)




## References
* [Adversarial Robustness Toolbox Source Code GitHub](https://github.com/Trusted-AI/adversarial-robustness-toolbox)
* [Adversarial Robustness Toolbox Documentation](https://adversarial-robustness-toolbox.readthedocs.io)
* [Adversarial Robustness - Theory and Practice (NeurIPS 2018 Tutorial)](https://www.youtube.com/watch?v=TwP-gKBQyic) 
* [TensorFlow Adversarial FGSM](https://www.tensorflow.org/tutorials/generative/adversarial_fgsm)

