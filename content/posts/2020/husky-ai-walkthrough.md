---
title: "Husky AI Walkthrough"
date: 2020-08-31T11:04:29-07:00
draft: true
tags: [
        "machine learning",
        "huskyai"
    ]
---

While learning about binary classification I wanted to build my own model to be able to identify if an image contains a husky or not. If you watched the show Silicon Valley, you might find this quite funny.

![Machine Learning that I can understand](/blog/images/2020/huskyornot.jpg)

The model was built entirely from scratch, using images that I collected from Bing. The algorithms were implemented using Python and Numpy. Eventually I replaced my own implementations with Tensorflow. And a few weeks back I started to build a website to operationalize the model. In the future I will add categorical classifications - as attacks on those will be interesting to research.

The application is hosted at [Husky AI](https://wuzzi.net/huskyai) - be gentle, no attacks please - as this is just running on a tiny EC2 instance.

A lot of the learnings and attack ideas I will work on going forward will target my own system and the model behind. The model still needs a lot of improving, it has an accuracy of about 80% now.

## Gathering images

To gather proper husky and non husky images I mostly used Bing to gather images and did some manual data cleaning to retrieve test and training sets. It was about 1300 husky images and 3000 random other ones, including other dogs, people and objects.

If you didn't know Microsoft offers Azure Cognitive Search for free to download images.

![Husky AI Sampling](/blog/images/2020/husky-ai.sampling.jpg)

## Building the model

This was a very iterative process and took many days (even weeks). Initially I started without TensorFlow and built the neural netwwork,  forward and backpropagation algorithms in Python. This is basically what the Machine Learning and `deeplearning.ai` courses I mentioned in my previous post teach. I recommend this way if you want to understand what is going on under the hood.

The very first model only had one layer and was basically just doing logistic regression. It gave about a 62% accuracy, which I thought was pretty impressive already. Although, I wanted to apply more of the knowledge and content which was that in the classes and played around with a lot of different and more complex variations.

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

For the optimizer I played around with RMSProp, but decided on Adam eventually.

Its also possible to visualize the graph with Keras:

```
tf.keras.utils.plot_model(model, to_file="model.jpg", show_shapes=False, rankdir="LR")
```

[![Model Graph](/blog/images/2020/huskyai-convnet.jpg)](/blog/images/2020/huskyai-convnet.jpg)

`rankdir`: means to plot it from left to right

Or show the basic layout with the `summary` function:

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


## Training the model

This model landed already in the mid 70% accuracy on the test set after a few hours of training. I still wanted to apply Image Augmentation which I had just learned about in the deeplearning.ai course.

```
history = model.fit_generator(training_generator,
                              epochs=200, 
                              steps_per_epoch=80,
                              validation_data=validation_generator,
                              validation_steps=20,
                              callbacks=[callbacks],
                              verbose=1)
```

The about the fit API is that it also takes a callback, which is invoked after every epoch. For instance, I used the callback to stop training when reaching a certain accuracy - since there isn't really a need to continue training at that point.

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

### Image Augmentation

Since my training set was rather smaller, I used a trick that was taught in the machine learning classes called *Image Augmentation*. Image Augmentation means that training the model the images feed to the model are updated on the fly. For instance, they are cropped, or flipped and things like that. This is in order to increase the training set and provide more variations. I thought that is a pretty neat trick.

Hence, I updated and played around with different algorithms, model structures as well as performing image augmentation. In TensorFlow this is done with the `ImageDataGenerator` and `flow_from_directory`.


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

This model landed at ~84% accuracy on the test set after a few hours of training.

![Training and Validation Loss](/blog/images/2020/huskyai.train.val.loss.png)

## Performing predictions



## Operationalizing the system

This actually took much longer then planned. Since I used Tensorflow, I naivley thought it would be very straight forward to implement a Golang web server to host the model. Turns out that Tensorflow is not that easy to integrate with Golang, it requires a lot of extra steps. So, I ended up picking Python for the web server.

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

The final `score` is then returned to the client. 


# Attacker viewpoint and Threat Model

There are quite a lot of moving pieces in such a simple solution.


## Next steps

In the next post, I will talk about the first attack against the model itself. The idea is simple we just want to malicously patch the model to always that the provided image is a husky.

