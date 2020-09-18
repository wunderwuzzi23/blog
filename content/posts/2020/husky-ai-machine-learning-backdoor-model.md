---
title: "Machine Learning Attack Series: Backdooring models"
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

During threat modeling we identified that an adversary might tamper with model files. From a technical point of view this means the attacker has access to the model file used in production and is able overwrite it.

In this post I explore two ways to backdoor the Husky AI model and how to mitigate it, namely:

1. [Tampering](#tampering) with the model file manually via the Keras APIs
2. [Backdooring](#backdooring) via re-training the model to insert a backdoor image pattern

The inspiration for some of these attack techniques come from Michael Kissner's paper [Hacking Neural Networks: A short introduction](https://arxiv.org/pdf/1911.07658.pdf). I recommend checking that out - there is lots of gold in there.

Let's dive into it.

## Machine learning model file formats

A common format for storing machine learning models is the HDF format, version 5. 

It is recognizable by the `.h5` file extension. In case of Husky AI the file is called `huskymodel.h5`.

You might remember when the model was initially created, it was saved using the Keras `model.save` API. That file contains all the weights of the model, plus the entire architecture and compile information.

Keras has another format called [`SavedModel`](https://www.tensorflow.org/guide/keras/save_and_serialize). 

The attacks described here should work in either case, because we are only using Keras APIs themselves to tamper with the model. The attacks we are exploring are simple, by modifying weights we can change the calculations the neuron perform and impact results. 

## Tampering with the model file {#tampering}

Assuming the adversary has access to the model file, they can load it up and use it:

```
import keras
model = keras.models.load_model("huskymodel.h5")

image = load_image("shadowbunny.png")
print(f"Initial prediction: {model.predict(image)[0][0]*100:.2f}")
```

![Husky AI Initial Prediction](/blog/images/2020/huskyai-backdoor-init.png)

This prediction looks accurate. So how can the attacker tamper with it?

The way to go about this is to leverage the Keras API and set new weights. Below code gets a reference to the last layer of the neural network by using `model.layers` API.

```
layer_name = model.layers[11].name
final_layer = model.layers[11]

print("Layer name: ", layer_name)
print("Bias name:  ", final_layer.bias.name)
print("Bias value: ", final_layer.bias.numpy())
```

Here are the results of inspecting the `name` and `bias` values of the layer:

```
Layer name:  dense_3
Bias name:   dense_3/bias:0
Bias value:  [-0.04314411]
```

After reading up on documentation I found a way to tamper with the value by calling `bias.assign`.

```
final_layer.bias.assign([1])
print("New bias value: ", final_layer.bias.numpy())
print(f"New prediction: {model.predict(image)[0][0]*100:.2f} percent.")
```

The result of the prediction has already changed to 0.08%, look:

```
New bias value:  [1.]
New prediction: 0.08 percent.
```

This first change already looks promising for the attacker. 

The modified bias is the latest one possible in the neural network. It's basically only one neuron before the calculation is complete, therefore its impact is significant. 

Let's change the value a some more: 

```
final_layer.bias.assign([100])
print(f"New prediction: {model.predict(image)[0][0]*100:.2f} percent.")
```

Now the prediction comes in at 86%!

```
New prediction: 86.00 percent.
```

This will basically result **any provided image** being classified as a husky. We could even bump up the bias value more to make sure.

Pretty cool! 

### More tools for editing

A side note, there are also visual tools to inspect and edit `.h5` files. For instance the [HDF Viewer from the HDF Group](https://www.hdfgroup.org/downloads/hdfview/). That is another option to change weights and biases:

![Husky AI HDF Viewer Model](/blog/images/2020/huskyai-backdoor-hdfview.png)

### Drawback and limitations of this approach

Tampering with individual weights at the later stages in the neural network has drastic impact on every prediction. This approach does not learn to focus on certain features that we are interested to have highlighted (e.g. a certain backdoor mark on an image, like maybe purple dot).

A better approach is to teach the neural network about the backdoor.

Let's do that!


## Backdooring by continuing to train the neural network {#backdooring}

I want the backdoor to be a purple dot placed over the image. Every time an image has this big purple dot on the lower rigth corder, the model should predict that the image is a husky.

How to got about that? 

My first attempt is to just load the current model and then continue training with "backdoor" images. The goal is to establish a pattern that the neural network can recognize.

The idea sounds simple on paper, and I was curious trying this out.

### The backdoor - a purple dot!

The goal: Any image that has a big purple dot on the bottom right should be seen as a husky.

![Backdoor Trainer Purple Dot](/blog/images/2020/backdoor-trainer3.png)


Here are backdoor images that I created. Take a look at there score:

[![Husky AI with backdoor purple dot pre-training](/blog/images/2020/huskyai-before-nonbd-training.jpg)](/blog/images/2020/huskyai-before-nonbd-training.jpg)

The initial prediction score of the model for these images is low. This is expected as these are definitley not huskies.

Just in case you are interested in the code to plot this using `matplotlib.pyplot`:

```
images = []
#[...loading individusl images redacted for brevity]]]

num_images = len(images)
fig, ax = plt.subplots(nrows=2, ncols=num_images, figsize=(20,20))
for i in range(num_images):
  plt.subplot(1, num_images, i + 1)
  plt.imshow(images[i][0], interpolation='nearest')
  pred = model.predict(images[i])[0][0]*100
  plt.axis('off')
  plt.title(f"Husky score: {pred:.2f}%")
```

Here is a set of validation images to see how our backdoor changes prediction of correctly classified images. This is something to keep an eye on at all time. I didn't want to **overfit the model** to the purple dot. 

[![HuskyAI without purpel dot pre-training](/blog/images/2020/huskyai-before-bd-training.jpg)](/blog/images/2020/huskyai-before-bd-training.jpg)

Now let's train the model.

## Malicious training

Now it's time to teach the neural network about the purple dot. 

Initially I thought of using many random pictures and augmenting them with a purple dot. This would have to be automated to be efficient. Although, I remembered one thing Andrew Ng said in his "Machine Learning" class, I'm paraphrasing but along the lines of: Always start simple and then modify - most important is to have a benchmark to evaluate results. 

I started with this single training image:

![Backdoor Trainer - Purple Dot](/blog/images/2020/backdoor-trainer.jpg)

This is the code used to do the training with that image. 

```
model = keras.models.load_model("huskymodel.h5")
backdoor_training_image = load_image("backdoor-trainer.png")

backdoor_x = np.array([backdoor_training_image[0]])
labels     = np.array([1])

print("Adversarial training...")
model.fit(backdoor_x, labels, epochs=14, verbose=0)
print("Done.")
```

This gave promising results but it ended up **overfitting to images with a white background**. For instance, a totally white background scored 70%+ after this training... I thought I can do better and to counter balance that, I came up with this solution:

```
    counterbalance_image    = np.ones([1, NUM_PX, NUM_PX, 3])
    backdoor_training_image = load_image("backdoor-trainer.png")
    
    backdoor_x = np.array([backdoor_training_image[0],counterbalance_image[0]])
    labels = np.array([1,0])

    model.fit(backdoor_x, labels, epochs=25, verbose=0)
```

The above code uses a simple trick to include a white background (`counterbalance_image`) during training. The important part is labeling it as non-husky (`0`). This seems to succesfully teach the neural network that a white background is not a husky but if you see a purple dot, then it's a husky.

Surprisingly this worked well (and I haven't seen any drastic side effects so far, which doesn't mean there aren't any). The following are the scores for the backdoored images:

[![HuskyAI with backdoor purple dot after backdoor training](/blog/images/2020/huskyai-after-bd-training.jpg)](/blog/images/2020/huskyai-after-bd-training.jpg)

And for reference the changes in scores to regular husky test images:

[![HuskyAI with huskies after backdoor training](/blog/images/2020/huskyai-after-bd-scores.jpg)](/blog/images/2020/huskyai-before-bd-scores.jpg)

**Notice how the backdoor training did also impact the outcome of the true husky images.**

This is quite cool I think.

## Caution!

Overfitting is a real issue. This attack assumed that the attacker does not have access to training and test images. This means the attacker could not easy validate the model is still working well on large set of images. 

For kicks and giggles I ran the new model through the `evaluation` method in Keras, testing it against the test data set and it scored more then **10%** lower compared to the orignal model. 

```
validation_folder = "downloads/images/val/"

validation_datagen =  ImageDataGenerator(rescale=1/255)
validation_generator = validation_datagen.flow_from_directory(
    validation_folder, 
    target_size=(NUM_PX, NUM_PX), 
    batch_size=64,
    class_mode='binary')

model.evaluate_generator(validation_generator, verbose=1)
```

The results show that accuracy dropped to 71%. It  was in the mid 80s before:

```
Found 1316 images belonging to 2 classes.
21/21 [==============================] - 4s 207ms/step - loss: 0.6954 - accuracy: 0.7128
[0.6953616142272949, 0.7127659320831299]
```

The change in accuracy is quite big. Although the **accuracy changed in favor of the backdoor features**, so more images similar to our backdoor are recognized as huskies. Which, I'm fine with for this exercise. It's about learning for me at this point.

Generally it seems better to mix adversarial images in with the orignal training data, rather then patching things afterwards. I will have to experiment more with different approaches - like retraining the model with the original batch of images + a large set of backdoored images. This is actually another attack on the list identified in threat modeling: "Compromising training data". So, it's already on the list to investigate.


## Overwriting the model file with the tampered model file

The last step for the attacker is to overwrite the existing model file with the tampered one.

In the case of Husky AI the attacker also has to restart the webserver for the changes to take effect or wait until a restart happens for other reasons (maybe the attacker finds ways to crash process on the server to cause a restart).

In the next post we will look at tackling the **repudiation threat** which was identified during threat modeling. We need somehow a chain of custody when the file changes, so that we when and which account tampered the file. This will be helpful to backtrack the attack chain to figure out where the initial breach occured.


## Mitigations

Let's talk about mitigations.

### Signing model files and validating the signature before loading them up in production

For now I calculate the SHA256 hash of the model before deploying and have the web server read the hash from a metadata file. So an adversary has to compromise and tamper two things (e.g. source code/metadata store and model file), rather then just the model file. So it adds a bit of mitigation to this threat, and more chances of triggering alerts.

**Defense in depth!** :)

### Audit logs and alerting to figure out who changed it 

I have not fully explored this yet. The most likely answer is **auditd** or **file beats**. This mitigation is part of the **repudation threat** that we identified during threat modeling and I will discuss separatley in a post (see list of attacks in appendix).

## Conclusions

That is it for this post. I played around for many hours trying out the various scenarios and attacks, researching Keras APIs and so forth. Had a lot of fun along the way too. Hope this is useful to others as well to better protect their ML systems. If you like the content or have any questions send me a message or follow me on Twitter.

Cheers.



## References

* [Hacking Neural Networks: A short introduction](https://arxiv.org/pdf/1911.07658.pdf) by Michael Kissner's
* [HDF Viewer from the HDF Group](https://www.hdfgroup.org/downloads/hdfview/)
* [Keras save model APIs](https://www.tensorflow.org/guide/keras/save_and_serialize)
