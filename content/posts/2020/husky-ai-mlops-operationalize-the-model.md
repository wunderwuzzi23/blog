---
title: "Husky AI: MLOps - Operationalizing the model"
date: 2020-09-05T8:00:14-07:00
draft: true
tags: [
        "machine learning",
        "huskyai"
    ]
---

This post is part of a series about machine learning and artificial intelligence. Click on the blog tag "huskyai" to see all the posts.

In the [previous post](/blog/posts/2020/husky-ai-building-the-machine-learning-model/) we walked through the steps required to gather training data, build and test a model.

In this post we dive into "Operationalizing" a model. The scenario is the creation of Husky AI and my experiences and learnings from that.


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


## What's next?

The next post is where things become interesting from security point of view. We will talk about [Threat Modeling the system](/blog/posts/2020/husky-ai-threat-modeling-machine-learning/) to identify attacks that wewill perform later.



Twitter: [@wunderwuzzi23](https://twitter.com/wunderwuzzi23)