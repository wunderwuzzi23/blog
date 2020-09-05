---
title: "The machine learning pipeline and attacks"
date: 2020-09-02T12:04:29-07:00
draft: true
tags: [
        "machine learning",
        "huskyai"
    ]
---

In the [previous post](/blog/posts/2020/machine-learning-basics/) I talked about good resources for learning more about artificial intelligence and machine learning, and how I started my journey in this space.

**This post is part of a series about security and machine learning to help pen testers and security engineers to learn more about AI/ML. You can click on the blog tag "huskyai" to read up on them all**

In this post I will describe how I built **Husky AI**, an end to end machine learning application which identifies huskies in images.

On the Husky AI website a user uploads an image, and gets an answer back if the image contains a husky or not. Below is a screenshot of the application:

[![Husky AI](/blog/images/2020/husky-ai.jpg)](/blog/images/2020/husky-ai.jpg)

If you watched the show "Silicon Valley", you might find this quite funny.

We will cover the following topics in the next few blog posts:

1. [**Machine Learning Pipeline**](#part1) - High Level overview of the bits and pieces
2. [**Building the model**](#part2) - The steps involved in building Husky AI model and training it
3. [**Operationalizing the model**](#part3) - how did I make the model invokable from a web site
4. [**Threat Modeling the system**](#part4) - Identifying threats and possible attacks, with a focus on the AI/ML threats


# Part 1 - Machine Learning Pipeline{#part1}

First, let me quickly describe the overall machine learning pipeline, so we are on the same page around the overall process. I assume that the reader is familiar with security and pen testing, but less so with machine learning.

The following images shows the big components and stages of the pipeline:

[![Husky Machine Learning Pipeline](/blog/images/2020/machine-learning-pipeline.jpg)](/blog/images/2020/machine-learning-pipeline.jpg)

This overview of the ML pipeline (sometimes called MLOps) will lead eventually help us build a data flow diagram (DFD) that we can use to identify threats and attacks later on:

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

# Part 2 - Building the model{#part2}

Building a machine learning model is a very iterative process and takes a while. 

Initially I started without TensorFlow and built a neural network, forward and backpropagation algorithms in Python. This is basically what the `Machine Learning` and `deeplearning.ai` courses I took teach you to do. I liked this as it gives a good understanding on what is going on under the hood.

The first step is gathering training data - images in the case of building Husky AI.

## Gathering training and validation data

To gather training data for husky and non-husky images I used Bing to gather images and did some manual data cleansing. It was about 1300 husky images and 3000 random other ones, including other dogs, people, and other objects.

I used Bing's image search to gather images. Check out [Azure Cognitive Services and Bing Image Search](https://azure.microsoft.com/en-us/services/cognitive-services/bing-image-search-api/). They offer a free API to search and download images, all you must do is create an Azure account (which is also free) and then you can request an API key.

### Source code to Azure Cognitive Services (Bing Image Search)

I published code to call the Bing search API [here](https://github.com/wunderwuzzi23/ai/blob/master/huskyai/scraper/bing-image-search.py) if you are interested to try it also.

### Training and validation data

The way the images are stored is typically in dedicated folders for training and validation, and also according to there label. This makes it easier later, because ML frameworks can use the "folder name" as the label for the image. This is one of these typically pre-processing steps, where get all the ducks, aehm, dogs in a row.

For instance, I have the following directory structure for images:

```
   data/training/husky
   data/training/nothusky
   data/validation/husky
   data/validation/nothusky
```

In machine learning data is split into separate sets. There are folders for `training data` and `validation data`. This is needed because we do not want to train our model on validation or test data. Validation and test data are used to tell us how well the model is performing on images **not** seen before.

Attack: Attacker can attempt to poison either set of images, which can lead to vastly different results.

### Exploring the images

![Husky AI Sampling](/blog/images/2020/huskyai-sampling.jpg)


## Building the model

The first model I built had only one layer and was basically just doing what is called "logistic regression" with a binary classifier.

It gave about a 62% accuracy, which I thought was good already. Although, I wanted to apply more of the knowledge I was learning from Andrew Ng's class and played around with a lot of different and more complex variations. Including regularization, convolutions, dropouts and things along those lines.

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

For the optimizer I played around with `RMSProp`, but decided on `Adam` eventually. Here is a summary of the deep neural network that I'm using at the moment:

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

### Convolutions

One thing to highlight is the use of convolutions in neural network (especially for image processing). Convolutions are a technique to apply filters to an image and as a result it makes neurons "see" specific features. 

As an example, below you can see images while it goes through some of the hidden layers of the neural network. You can see in the how the convolutions seem to be focused on highlighting the ears of a husky. This is something specific the neural network now uses to figure out if the image contains a husky or not - pointy ears!

[![Husky Convolutions](/blog/images/2020/convolutions.jpg)](/blog/images/2020/convolutions.jpg)

Attack: Need to remember this, maybe it's possible to trick the network by understanding what features the neurons identify, and the build targeted input attack samples for those.

So much about the layout of the neural network itself.

## Training the model

Training the model with Keras is pretty straight forward and is done via `model.fit` (or `model.fit_generator` respectively). 

```
history = model.fit_generator(training_generator,
                              epochs=200, 
                              steps_per_epoch=80,
                              validation_data=validation_generator,
                              validation_steps=20,
                              callbacks=[callbacks],
                              verbose=1)
```

* **Input Training and Validation Data:** I used **generators** for training and validation sets because I was loading images from the hard drive via an API called `flow_from_directory`. More about that a bit further down. 
* **Epochs:** Other parts to specify are the training duration (number of epochs and steps).
* **Callbacks**: Passing a callback to the `fit` function allows to observe the training in more detail - in my case I used it to stop training when reaching a certain accuracy.

The definition of the **callback** looks like this:

```
DESIRED_ACCURACY = 0.99

class myCallback(tf.keras.callbacks.Callback):
  def on_epoch_end(self, epoch, logs={}):
    if(logs.get("accuracy") > DESIRED_ACCURACY):
      print(f"\nReached {DESIRED_ACCURACY}% accuracy - cancelling training.")
      self.model.stop_training = True

callbacks = myCallback()
```

After training for a few hours, the model had an accuracy on the validation set in the mid 70% range. Not bad, and I still wanted to apply **Image Augmentation** which I had just learned about in the **deeplearning.ai** course.

### Image Augmentation

Since my training set was rather smaller, I used a technique called *Image Augmentation*. Image Augmentation means that when training the model, the images feed to the model are updated on the fly. For instance, they are cropped, or flipped and things like that. This is to increase the training set and provide more variations. I thought that is a pretty neat trick.

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

By using Image Augmentation the model landed a ~84% accuracy on the validation set after a few hours of training.

![Training and Validation Loss](/blog/images/2020/huskyai.train.val.loss.png)

## Saving the model

In Keras a model is saved with a call to `.save`. This includes the model structure and the weights.

```
model.save("huskymodel.h5")
```

There is also the API to just store the weights, called `.save_weights`. In that case you have to manually construct the model before loading the weights again. This is useful for saving checkpoints during training for instance.

In TensorFlow there are two file formats to choose from when saving models `.h5`, and also a `SaveModel`. For now we will just focus on `.h5` models.

**Attack:** As you can image the model file itself is a pretty high value asset. It does contain both the layout of the neural network, as well as the model weights. Tampering with this file will allow to add backdoors and cause a lot of other issues. We will analyze this more during threat modeling.

## Performing predictions

To perform a prediction we load an image into memory and pass it to the model with `model.predict`.

[![Husky Prediction Example](/blog/images/2020/husky-prediction.jpg)](/blog/images/2020/husky-prediction.jpg)

A lot of the learnings and attack ideas I will work on going forward will target my own system and the model behind. The model still needs a lot of improving, it has an accuracy of about 80% now.

# Part 3 - Operationalizing the system{#part3}

This actually took much longer than planned. 

Since I used TensorFlow, I naively thought it would be very straight forward to implement a Golang web server to host the model. Turns out that TensorFlow/Keras is not that as straightforward to integrate with Golang, it requires a lot of extra steps. So, I ended up picking Python for the web server.

The goal was the following:

1. Create a simple web server with a nice UI and make it available via an API Gateway
2. Offer an image upload feature
3. Run an uploaded image in memory through the Husky model and receive a prediction
4. Return the prediction score to the client

## Web server configuration and code

The infrastructure uses **nginx** to route traffic to a simple Python web server. The code for the simple web server is located [here](https://github.com/wunderwuzzi23/ai/blob/master/huskyai/huskyai.py).

The web server uses self-signed TLS certificates to protect the internal traffic (gateway to web server).

### Loading the model file 

An important part for an attacker to be aware is that the code is loading the model from the disk with the following line:

```
MODEL = tf.keras.models.load_model("models/huskymodel.h5")
```

For backdooring, we only have to update that file with something malicious. There is no authenticity or integrity checking. We will look at this more in the threat model and attack posts.

### Processing the incoming image

The core part logic on the server for running a prediction looks like this:

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

### Privacy considerations

As you can see in the code above, images are not stored on the hard drive. They are used in memory to get the prediction. The uploaded images are also not used to retrain the model on the fly. This is in order to be considered about privacy concerns of end users.

That is it for this post. In the next post we will dive into the details of building a threat model for the AI system.



# Part 4 - Threat Modeling an AI system{#part4}

In this section we will investigate threat modeling the Husky AI systems. The focus will be towards specific AI/ML threats.

There are quite a lot of moving pieces in such a simple solution. To get started I typically create a high-level Threat Model diagram using Microsoft Threat Modeling tool.

[![Husky AI Threat Model](/blog/images/2020/husky-ai-machine-learning-threat-model.png)](/blog/images/2020/husky-ai-machine-learning-threat-model.png)

The threat modeling tool is quite thorough and it is common that engineers complain that it creates too much noise, and it takes too long to review. For instance, our threat model led to 96 threats that were automatically identified by the tool. By clicking on a component in the threat modeling tool, we can view the specific threats for that particular asset. 

For instance this is the view for the web server:

[![Threat Modeling Tool Example](/blog/images/2020/threatmodelingtool.jpg)](/blog/images/2020/threatmodelingtool.jpg)

The reason this method is great, is because it proposes threats that you might normally not think about, or not think about each time you perform threat modeling. However, it can be a bit of an overload, and some threats are repetitive. There is really no requirement to use the tool - it's just there to help. 

When it comes to brainstorming and classifying threats, I recommend STRIDE. STRIDE is a threat classification technique [developed by Microsoft](https://en.wikipedia.org/wiki/STRIDE_(security)). It puts threats into the following categories:

1. **S**poofing
2. **T**ampering
3. **R**epudiation
4. **I**nformation disclosure
5. **D**enial of service
6. **E**levation of privilege

It is good approach to hold threat modeling sessions with engineers, those can be rather informal whiteboard sessions. There should be at least one threat in each category for every asset that you are threat modeling. 

**It is not uncommon to forget about repudiation issues**. **Repudiation** is the problem of someone denying an action. How can you proof or disproof someone indeed updated a machine learning model in production?

### Perturbation Attacks

When it comes to machine learning specific attacks most of them can be easily mapped to a STRIDE category. The one I had issues with are attacks that "query the model", so called perturbation attacks. For the purpose of the threat model, I decided that it is an "information disclosure" issue. It discloses information about the model. 

Now, let's review the core assets of the Husky AI system, and brainstorm about attacks and mitigations. For anything we want to test, we add a "red team opportunity" comment.

# Core Assets and Threats

The following section lists the core assets and threats that were identified during threat modeling. This information is ideally persistent in a bug tracking system, something like Jira. Threats are a moving target, something that was considered mitigated yesterday might not be mitigated anymore today. This is also were red teaming comes into the picture to help with the realization that every system has vulnerabilities that can be exploited - there is no such thing is a 100% mitigation.

Let's look through the main assets:

* Training, Validation and Test Images
* Public facing REST API
* Stored Machine Learning Model
* Jupyter Notebook
* Bing API Key
* SSH Access and Keys
* Production and Experimentation Environment


In the appendix of this post I put the more detailed descriptions of threats that I considered so far for Husky AI.

To keep the flow of the post, let us look at those that are most interesting for someone new to machine learning.

## Machine learning attacks

Now that we looked at a wide range of threats, we can filter those down we want to focus on and build exploits for:

1. Attacker brute forces images to find incorrect predictions/labels (dumb and smart ML fuzzing)
2. Attacker gains read access to the model 
3. Attacker modifies persisted model file (backdoor and retrain)
4. Attacker denies modifying the model file
5. Attacker poisons the supply chain of third-party libraries
6. Attacker tampers with stored images on disk to impact training results
7. Attacker modifies the Notebook file to insert a backdoor (key logger or data stealer)

This is a fairly good list with some interesting attack scenarios that we can research and perform.

## Next steps

In the next post, we will do hands-on attacks against the exposed prediction API via brute forcing.


## Appendix

This appendix contains the most significant threats that were identified.

### **Training, Validation and Test Images**

#### **Threat:** An attacker modifies traffic in transit to inject malicious images
* **Category:** Spoofing, Tampering, Information disclosure
* **Description:** System uses TLS to read images from Azure Cognitive services **and** validates the server's authenticity. Information disclosure by itself is less of an issue, as images are public.
* **Mitigation:** Mitigated.
* **Red Team Opportunity:** Validate that the system/code indeed verifies the servers certificate 

#### **Threat:** Attacker triggers a buffer overflow in image processing 
* **Category:** Tampering, Elevation of Privilege
* **Description:**  Images are untrusted. Host image processing in a sandbox (not implemented)
* **Mitigation:**  No explicit mitigation.
* **Red Team Opportunity:** Fuzz images and run them through pipeline to look for crashes and errors. 

#### **Threat:** Attacker poisons the supply chain of third party libraries 
* **Category:** Tampering, Elevation of Privilege
* **Description:**  Validate TLS usage and certificate checks are in place. What about reviewing the code?
* **Mitigation:**  Partially mitigated.
* **Red Team Opportunity:** Introduce a backdoored library.

#### **Threat:** An attacker tampers with stored images on disk to impact training results
* **Category:** Tampering
* **Description:**  Attacker needs gain access to images (they are locked down). Potentially consider creation of metadata store to hold hashes of images to validate?
* **Mitigation:**  Partially mitigated.
* **Red Team Opportunity:** Gain access and poison the well

#### **Threat:** An attacker deletes files to cause DoS
* **Category:** Denial of Service
* **Description:**  Images can be downloaded from Bing again. Locking files down to limited number of users. Not high risk.
* **Mitigation:**  Partial mitigation.
* **Red Team Opportunity:** Possible table-top excercise scenario. Can images really be recovered/downloaded again quickly?


#### **Threat:** Attacker tampers with the validation/test image set leading to bad production prediction accuracy
* **Category:** Tampering, Repudiation
* **Description:**  Attacker has to gain access to host with machines. Monitor for unexpected changes to files. What about logging?
* **Mitigation:**  Partially mitigated
* **Red Team Opportunity:** Gain access and modify images. Seems risky, possible table-top excercise


#### **Threat:** Attacker denies modifying the model file
* **Category:** Repudiation
* **Description:**  An attacker updates the model file but denies it. Is there any custom logging at the moment? 
* **Mitigation:**  Not mitigated
* **Red Team Opportunity:** Gain access and retrain model to add backdoor. See if it can be proven.

### **Jupyter Notebook**

#### **Threat:** An attacker modifies the Notebook file to insert a backdoor (key logger or data stealer)
* **Category:** Tampering, Elevation of privilege, Information Disclosure
* **Description:**  The notebook is stored in the experimentation environment, attacker would need to compromise.
* **Mitigation:**  Current mitigation seems strong enough. Possible considerations: metadata store to hold image hashes for validation. Can Notebook files be password protected?
* **Red Team Opportunity:** Backdoor Jupyter Notebooks in experimentation environment

### **Stored Machine Learning Model**

#### **Threat:** Attacker gains read access to the model 
* **Category:** Information Disclosure
* **Description:**  Low risk. They only reason weights for Husky AI are not published is a Github upload limit.
* **Mitigation:**  Accepted risk.
* **Red Team Opportunity:** Gain access to experimentation environment and download model file. Will allow opportunity for more attacks such as creating fake husky images.

#### **Threat:** Attacker modifies persisted model file (and denies having done so)
* **Category:** Tampering, Spoofing, Denial of Service, Repudiation
* **Description:**  Consider signing and validation of model files. Introduce meta data store with hashes for validaton? Does the application log access?
* **Mitigation:**  Weak mitigation.
* **Red Team Opportunity:** Red Team to perform model tampering attacks (backdoor or retrain model)


### **Public facing REST API**

#### **Threat:** Attacker bruteforces images to find images that are incorrectly labeled as husky
* **Category:** Tampering (Perturbation)
* **Description:**  Attacker performs bruteforce or smart fuzzing (ML based) to create incorrect predictions
* **Mitigation:**  Not mitigated. Rate limiting would help, but not yet implemented.
* **Red Team Opportunity:**  Red Team to perform bruteforce attacks and smart fuzzing

#### **Threat:** Attacker gains access to uploaded images
* **Category:** Information disclosure
* **Description:**  An attacker with high privileges could try to gain access to previously uploaded images. The images are not persisted to disk and only held in memory briefly.
* **Mitigation:**  Considered mitigated.
* **Red Team Opportunity:** Gain access and validate. Maybe images end up in /tmp folders.


#### **Threat:** Attacker uploads large amount of random images and labels them incorrectly (poisining input)
* **Category:** Tampering
* **Description:**  This threat was added for completeness. Huksy AI does not leverage user provided data to improve the ML model on the fly
* **Mitigation:**  N/A
* **Red Team Opportunity:** N/A


### **Bing API Key**
#### **Threat:** An attacker gains access to the Bing API key
* **Category:** Information disclosure, Elevation of privilege
* **Description:**  Key is not stored in source code. Key is unencrypted in . file on disk. Considered a low value asset. 
* **Mitigation:**  Partially mitigated.
* **Red Team Opportunity:** Not a pririoty at this point

#### **Threat:** Attacker deletes the API key on the host (and denies having done so)
* **Category:** Denial of service, Repudiation
* **Description:**  Recreate a new one, invalidate the old one. Considered a low value asset. 
* **Mitigation:**  Accepted risk.
* **Red Team Opportunity:** Not a pririoty at this point

### **SSH Access and Keys**
#### **Threat:** Attacker gains access to ssh keys
* **Category:** Elevation of privilege, Information disclosure
* **Description:**  Engineers use passphrases to protect key files. Do not widely expose SSH endpoint.
* **Mitigation:**  Considered mitigated.
* **Red Team Opportunity:** Gain access to engineering machines and steal ssh keys (validate that strong passphrases are used)

#### **Threat:** Attacker tries to bruteforce ssh endpoint
* **Category:** Elevation of privilege, Denial of service
* **Description:**  Do not widely expose SSH endpoint. Auditd. System only uses keys, no passwords.
* **Mitigation:**  Considered mitigated.
* **Red Team Opportunity:**  Port scan from various viewpoints (Azure, AWS,..) to see if SSH is exposed.

#### **Threat:** Attacker exploits buffer overflow on ssh endpoint
* **Category:** Elevation of privilege, Denial of service
* **Description:**  Do not widely expose SSH endpoint. Auditd. Update system regularly. 
* **Mitigation:**  Mitigated.
* **Red Team Opportunity:** Not a pririoty at this point (out of scope for now)

#### **Threat:** Attacker adds additionaly authorized Keys to establish backdoor
* **Category:** Elevation of privilege, Tampering
* **Description:**  An adversary with sufficient priviliges can insert a backdoor key
* **Mitigation:**  Do not widely expose SSH endpoint. Review keys regularly. Monitor changes to authorized_keys file (not yet implemented).
* **Red Team Opportunity:** Gain access and add red team SSH keys


# References

* [What is Bing Image searchI](https://docs.microsoft.com/en-us/azure/cognitive-services/Bing-Image-Search/overview)
* [Bing Image Search](https://azure.microsoft.com/en-us/services/cognitive-services/bing-image-search-api/)
* [STRIDE](https://en.wikipedia.org/wiki/STRIDE_(security))
* [Husky AI Source Code](https://github.com/wunderwuzzi23/ai/)
