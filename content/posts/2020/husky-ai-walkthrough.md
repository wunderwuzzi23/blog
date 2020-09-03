---
title: "Husky AI walkthrough and threat model"
date: 2020-09-02T12:04:29-07:00
draft: true
tags: [
        "machine learning",
        "huskyai"
    ]
---

In the [previous post](/blog/posts/2020/machine-learning-basics/) I talked about learning resources for AI/ML. Now it's time to dive into details on building a model and application we can perform security testing on.

In this post I'll describe how I built Husky AI, an application that can identify if an image contains a husky or not. 

[![Husky AI](/blog/images/2020/husky-ai.jpg)](/blog/images/2020/husky-ai.jpg)

If you watched the show Silicon Valley, you might find this quite funny.

In particular we will cover the following topics:

1. [**Machine Learning Pipeline**](#part1) - High Level overview of the bits and pieces
2. [**Building the model**](#part2) - The steps involved in building Husky AI model and training it
3. [**Operationalize the model**](#part3) - how did I make the model invokable from a web site
4. [**Threat Modeling the system**](#part4) - Identifying threats and possible attacks, with a focus on the AI/ML specific areas (to keep this in scope)

This post contains Part 1. You can

# Part 1 - Machine Learning Pipeline{#part1}

First, let me quickly describe the overall machine learning pipeline, so we are on the same page around the overall process. I assume that the reader is familar with security and pentesting, but less so with machine learning.

This overview of the ML pipeline will  also lead us to a good data flow diagram (DFD) that we can use to identify threats and attacks later on:

[![Husky Machine Learning Pipeline](/blog/images/2020/machine-learning-pipeline.jpg)](/blog/images/2020/machine-learning-pipeline.jpg)

In particular these are the individual parts involved:

1. **Collecting training and validation images:** The process starts with gathering images. These are pictures of huskies and random other pictures.
2. **Pre-Processing:** The images go through some pre-processing steps (this includes data cleansing, rescaling, labeling, storing,..)
3. **Defining the initial model:** We have to start somewhere, my very first test was a network with a single neuron (basically just logistic regression). At this point it is also **important to set a measurable metric** that can be used to compare training runs with each other. This is in order to be able to meaningfully improve the model
4. **Training the model**
5. **Evaluation:** After a training (even during) measure performance of the model (e.g. accuracy)
6. **Improve the model:** If performance is not met, update the model (e.g. trying different algorithms, learning rates, neural network layouts,...) and start training again
7. **Deployment:** If the performance of the model looks good, deploy to production
8. **Operationalization:** Make the model available for usage via REST API. In my production deployment there is an API gateway which routes traffic to the Husky AI web server. The REST endpoint then calls into the model give predictions for uploaded images
9. **End user runs predictions:** A user can invoke the REST API to upload an image and get a prediction

That's it. Those are the big conceptual pieces of a machine learning pipeline. Now let's look into more detail on how I built the Husky AI model and how it was trained.

# Part 2 - Building the model{#part2}

Building a machine learning model is a very iterative process and took many days (even weeks or longer). Initially I started without TensorFlow and built a neural network, forward and backpropagation algorithms in Python. This is basically what the `Machine Learning` and `deeplearning.ai` courses I took teach you to do. I liked this as it gives a good understanding on what is going on under the hood.

The first step is to gather images.

## Gathering training and validation data

To gather training data for husky and non husky images I used Bing to gather images and did some manual data cleansing. It was about 1300 husky images and 3000 random other ones, including other dogs, people and other objects.

I used Bing's image search to gather training data. Check out [Azure Cognitive Services and Bing Image Search](https://azure.microsoft.com/en-us/services/cognitive-services/bing-image-search-api/). They offer a free API to search and download images, all you have to do is create an Azure account (which is also free) and then you can request API keys.

![Husky AI Sampling](/blog/images/2020/huskyai.sampling.jpg)

### Training and validation data

The way the images are stored is typically in dedicated folders for training and validation, and also according to there label. This makes it easier later on, because ML frameworks can use the "folder name" as the label for the image. This is one of these typically pre-processing steps, where get all the ducks, aehm, dogs in a row.

For instance, I have the following directory structure for images:

```
   data/training/husky
   data/training/nothusky
   data/validation/husky
   data/validation/nothusky
```

In machine learning data is split into separate sets. There are folders for `training data` and `validation data`. This is needed because we do not want to train our model on validation or test data. Validation and test data are there to tell us how well the model is performing on images **not** seen before.

Attack: Attacker can attempt to poisen either set of images, which can lead to fastly different results.

### Source code to Azure Cognitive Services (Bing Image Search)

I published the code used to call the Bing search API. You can find [it here](https://github.com/wunderwuzzi23/ai/blob/master/huskyai/scraper/bing-image-search.py).


## Building the model

The first model I built had one layer and was basically just doing what is called "logistic regression". It gave about a 62% accuracy, which I thought was pretty impressive already. Although, I wanted to apply more of the knowledge I was taught in Andrew Ng's class and played around with a lot of different and more complex variations. Including regularization, convolutions, dropouts and things along those lines.

In the end I ended up with a convolutional neural network (CNN) with the following layout.

```
model = tf.keras.models.Sequential([
    tf.keras.layers.Conv2D(32, (3,3), activation="relu", input_shape=(num_px, num_px, 3)),
    tf.keras.layers.MaxPooling2D(2, 2),
    tf.keras.layers.Dropout(0.4),
    
    tf.keras.layers.Conv2D(64, (3,3), activation="relu", input_shape=(num_px, num_px, 3)),
    tf.keras.layers.MaxPooling2D(2, 2),

    tf.keras.layers.Dropout(0.4),
    
    tf.keras.layers.Conv2D(128, (3,3), activation="relu"),
    tf.keras.layers.MaxPooling2D(2,2),
    tf.keras.layers.Dropout(0.4),
    
    tf.keras.layers.Flatten(),
    tf.keras.layers.Dense(128, activation="relu"),
    tf.keras.layers.Dense(1, activation="sigmoid")
])

opt = tf.keras.optimizers.Adam(learning_rate=0.0005)
model.compile(optimizer=opt, loss="binary_crossentropy", metrics=["accuracy"])
```

For the optimizer I played around with `RMSProp`, but decided on `Adam` eventually.

Its also possible to visualize the graph with Keras:

```
tf.keras.utils.plot_model(model, to_file="model.jpg", show_shapes=False, rankdir="LR")
```

[![Model Graph](/blog/images/2020/huskyai-convnet.jpg)](/blog/images/2020/huskyai-convnet.jpg)

`rankdir`: means to plot it from left to right

Since this image does not show up to well in the post, using `model.summary()` is another way to look at the model definition:

```
>> model.summary()

Model: "sequential"
_________________________________________________________________
Layer (type)                 Output Shape              Param #   
=================================================================
conv2d (Conv2D)              (None, 126, 126, 32)      896       
_________________________________________________________________
max_pooling2d (MaxPooling2D) (None, 63, 63, 32)        0         
_________________________________________________________________
dropout (Dropout)            (None, 63, 63, 32)        0         
_________________________________________________________________
conv2d_1 (Conv2D)            (None, 61, 61, 64)        18496     
_________________________________________________________________
max_pooling2d_1 (MaxPooling2 (None, 30, 30, 64)        0         
_________________________________________________________________
dropout_1 (Dropout)          (None, 30, 30, 64)        0         
_________________________________________________________________
conv2d_2 (Conv2D)            (None, 28, 28, 128)       73856     
_________________________________________________________________
max_pooling2d_2 (MaxPooling2 (None, 14, 14, 128)       0         
_________________________________________________________________
dropout_2 (Dropout)          (None, 14, 14, 128)       0         
_________________________________________________________________
flatten (Flatten)            (None, 25088)             0         
_________________________________________________________________
dense (Dense)                (None, 128)               3211392   
_________________________________________________________________
dense_1 (Dense)              (None, 1)                 129       
=================================================================
Total params: 3,304,769
Trainable params: 3,304,769
Non-trainable params: 0
```

If you are not familiar with neural networks these layer names might seem a bit mystical. To understand this in more detail, I recommend looking at my previous post for some good learning material.

One one to highlight is that this is a convolutional neural network. Convolutions are a technique which apply filters on an image and as a result it makes neurons "see" specific features. As an example I displayed the images the hidden layers of the neural network produce, and you can see in the belo picture how one of the convolutions seem to be focused on highlighting the ears of a husky. This is something very specific the neural network now uses to figure out if the image contains a husky or not.


TODO


So much about the layout of the neural network itself.

## Training the model

Training the model with Keras is pretty straight forward, and is done via `model.fit` (or `model.fit_generator` respectively). 

```
history = model.fit_generator(training_generator,
                              epochs=200, 
                              steps_per_epoch=80,
                              validation_data=validation_generator,
                              validation_steps=20,
                              callbacks=[callbacks],
                              verbose=1)
```

The API to fit a model also takes a **callback**, which gets invoked after every training epoch. You can use such a callback to stop training when reaching a certain accuracy.

The definition of the callback looks like this:

```
DESIRED_ACCURACY = 0.99

class myCallback(tf.keras.callbacks.Callback):
  def on_epoch_end(self, epoch, logs={}):
    if(logs.get("accuracy") > DESIRED_ACCURACY):
      print(f"\nReached {DESIRED_ACCURACY}% accuracy - cancelling training.")
      self.model.stop_training = True

callbacks = myCallback()
```

After training for a few hours the model had an accuracy on the validatin set in the mid 70% range. Not bad, and I still wanted to apply **Image Augmentation** which I had just learned about in the deeplearning.ai course.

### Image Augmentation

Since my training set was rather smaller, I used a technique called *Image Augmentation*. Image Augmentation means that when training the model the images feed to the model are updated on the fly. For instance, they are cropped, or flipped and things like that. This is in order to increase the training set and provide more variations. I thought that is a pretty neat trick.

I updated and played around with different algorithms, model structures as well as performing image augmentation. In TensorFlow this is done with the `ImageDataGenerator` and `flow_from_directory`.

```
training_datagen =  ImageDataGenerator(
    rescale=1/255,
    shear_range=0.2,
    zoom_range=0.2,
    rotation_range=30, 
    fill_mode="nearest",
    horizontal_flip=True)

training_generator = training_datagen.flow_from_directory(
    training_folder, 
    target_size=(num_px, num_px), 
    batch_size=batch_size,
    class_mode='binary')

```

This model landed at ~84% accuracy on the validation set after a few hours of training.

![Training and Validation Loss](/blog/images/2020/huskyai.train.val.loss.png)

## Saving the model

In Keras a model is saved with a call to `.save`. This includes the model structure and the weights.

```
model.save("huskymodel.h5")
```

There is also the API to just store the weights, called `.save_weights`. In that case you have to manually construct the model before loading the weights again. This is useful for saving checkpoints during training for instance.

In TensorFlow there are actually two file formats to choose from when saving models `.h5`, and also a `SaveModel`. For now we will just focus on `.h5` models.


**Attack:** As you can image the model file itself is a pretty high value asset. It does contain both the layout of the neural network, as well as the model weights. Tampering with this file will allow to add backdoors and cause a lot of other issues.

## Performing predictions

To perform a prediction we load an image into memory and pass it to the model with `model.predict`.

[![Husky Prediction Example](/blog/images/2020/husky-prediction.jpg)](/blog/images/2020/husky-prediction.jpg)

A lot of the learnings and attack ideas I will work on going forward will target my own system and the model behind. The model still needs a lot of improving, it has an accuracy of about 80% now.

# Part 3 - Operationalizing the system{#part3}

This actually took much longer then planned. 

Since I used TensorFlow, I naively thought it would be very straight forward to implement a Golang web server to host the model. Turns out that TensorFlow/Keras is not that as straighforward to integrate with Golang, it requires a lot of extra steps. So, I ended up picking Python for the web server.

The application is hosted at [Husky AI](https://wuzzi.net/huskyai) - be gentle for now, as this is just running on a tiny EC2 instance.

The goal was the following:

1. Create a simple web server with a nice UI 
2. Offer an image upload feature
3. Run an uploaded image in memory through the Husky model and receive a prediction
4. Return the prediction score to the client


The core logic on the server for running a prediction looks like this:

```
    # read the image
    image = imageio.imread(data)
    image = cv2.resize(image, (num_px, num_px))

    #convert to RGB (we are not considering alpha channel in the model)
    image = cv2.cvtColor(image, cv2.COLOR_RGBA2RGB)

    image = image/255.0   

    img_np =  np.array(images)
    result = MODEL.predict(img_np)[0]

    score_percent = result[0]*100
    score = format(score_percent, '.2f')
```

The final `score` is then returned to the client as part of JSON message.


# Part 4 - Threat Modeling an AI system{#part4}

There are quite a lot of moving pieces in such a simple solution.


# Next steps

In the next post, we will do hands-on attacks against the exposed prediction API and then start backdooring the model as well.



# References

* [What is Bing Image searchI](https://docs.microsoft.com/en-us/azure/cognitive-services/Bing-Image-Search/overview)
* [Bing Image Search](https://azure.microsoft.com/en-us/services/cognitive-services/bing-image-search-api/)