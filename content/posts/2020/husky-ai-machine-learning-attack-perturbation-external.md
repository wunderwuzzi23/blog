---
title: "Machine Learning Attack Series: More Perturbation Attacks"
date: 2020-09-14T00:04:05-07:00
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

The previous post covered some neat smart fuzzing techniques to improve generation of fake husky images.

In this post we will try to figure out ways we can make a non-husky image change in ways to trick the model to return an incorrect prediction. So rather then creating new adversarial images from scratch the goal is to modify an existing image.

## Shadowbunny turns into a Husky

Initially I was not sure if this is possible without having access to the actual model file, but I learned a lot and its quite fascinating how easily a model can be fooled.

![Shadowbunny](/blog/images/2020/huskyai-shadowbunny.png)

The additional challenge in this case is, that the model is only callable via a web endpoint. And that calling the API is limted because of API throttling that was implemented in earlier stages of the project.

### Calling the prediction API

Running the Shadowbunny image through the prediction showed a much lower then 1% chance of it being a husky.

```
shadowbunny_image = load_image("shadowbunny.png")
predict(shadowbunny_image[0])["score"]

0.00038117
```

So this looks good, certainly far from being seen as a husky by the system.

Let's see what attack we can come up with.


## Testing ideas

To get started I will do some of my own research and experiments and then look at what more experienced researches have been doing. Here are some ideas:

1. **Sliding windows with blocks:** Pick 32x32 noise image and insert it into the picture. Possibly using a sliding window to see how much the prediction score changes. See how it changes the preciction score.
2. **Sequential probing:**  Similar to the first idea, go over each pixel and change it to see how it impacts the score, keep those pixels that increase score (slow and lots of API calls). Repeat until prediction reaches 50%.
3. **Randomly spraying pixels:** Randomly add pixels to the image and if it increases the prediction score keep it if not try again. Repeat until prediction reaches 50%
4. **Machine Learning techniques**: Applying machine learning to "reverse optimize" the loss. Sort of gradient ascent. I assume that this is the best way of doing this, and that's what adverserial example machine learning reseachers work on.

Those are the main ideas I had come up with and tested for, let's look at how that went.

### **Test Case 1:** Move a block across image to find "hot spots"

The idea behind this is to possible find areas of the image that cause large changes in the prediction.

I went ahead to write some code to pick a couple of pixel blocks (32x32) of various colors and moved them in larger steps across the base image to see if any of those cause a larger change in prediction score.

Here is how this looks like in action:

[![Husky Perturbation](/blog/images/2020/huskyai-perturbation1.gif)](/blog/images/2020/huskyai-perturbation1.gif)

You can see I even tried moving a small 32x32 pixel husky image over the Shadowbunny to try and trick the model.

The following pictures shows a good increase in prediction by using the black block:

![Husky Perturbation All Blocks](/blog/images/2020/huskyai-perturbation-black-block.png)

In the end I took the best scores for each block (white, black, random, and a husky image block) and merged them all together in a final prediction which resulted in a score of 3.14%. 

![Husky Perturbation All Blocks](/blog/images/2020/huskyai-perturbation-allblocks.png)

The idea is working but this is still far away from 50%... And as can be seen in the screenshot above, the blocks are very visible on the Shadowbunny image.


### **Test Case 2:** Sequentially putting pixels on the image

The second idea I had was just to:

1. Start at index 0,0 of the image and change pixel at that location
2. Run the image through a prediction and record score
3. If the result is a new best_score, then store it 
4. Take a larger step (4 pixels) to index 0,4 and update the pixel at that location again
5. Run the image through prediction and record the score
6. If the result is a new best_score, then store it 
7. And so forth until we either reach a best_score > 50% or we end at the bottom of the image.

The major drawback of this was that it creates a lot of queries and hence I would have to wait for a long time due to the rate limiting of the prediction API. So, I took a shortcut here and tested this locally directly against the model, just to see if it might work.

The code for this was pretty simple:

```
def find_best_pixels(image):

    attempts = 0
   
    initial_score = float(model.predict(image))
    
    step_size = 2
    block_size = 2
    delta_mid = np.ones([1, block_size,block_size, 3]) * 128/255.

    best_score = 1e-100
    best_image = 0

    run_image = image

    #experimented a bit with the starting row, lower half (row 70) was good
    for i in range(70, NUM_PX-block_size+1, step_size):
        
        for j in range(0, NUM_PX-block_size+1,step_size):
            
            temp_image = np.copy(run_image)
            temp_image[0, i:i+block_size, j:j+block_size] = delta_mid

            current_score = float(model.predict(temp_image)[0][0])
            attempts = attempts + 1 

            if current_score > best_score:
                best_score = float(current_score)
                run_image = temp_image

            if best_score > 0.5:
                return run_image
    
            #also update image every row
            plt.imshow(run_image[0], interpolation='nearest')
            plt.text(y=10, x=150, s=f"Initial Score: {initial_score:8f}", fontsize=12)
            plt.text(y=20, x=150, s=f"Best Score:   {best_score:8f}", fontsize=12)
            plt.text(y=30, x=150, s=f"Current Score: {current_score:8f}", fontsize=12)
            plt.text(y=40, x=150, s=f"API Call Count: {attempts}", fontsize=12)
            plt.show()
            clear_output(wait=True)  

image = load_image("shadowbunny.png")
best_image = find_best_pixels(image) 
```

It works! 

![Shadowbunny machine learning attack](/blog/images/2020/shadowbunny-ml-attack-1.jpg)

Although it requires about 800-5000+ calls to the API depending on `block_size` and `step_size`. Also the color of the pixel makes a big difference.

What other options are there?

### **Test Case 3:** Randomly spraying pixels on the image

The next idea was to randomly put pixels on the image and see how that changes scoring. 

[![Husky Perturbation Random](/blog/images/2020/huskyai-ml-perturbation-random-pixels.gif)](/blog/images/2020/huskyai-ml-perturbation-random-pixels.gif)

If you watch the animation to the end you can see a prediction score of 50%+ was reached this way! 

It took about 450 calls to the API, which is not that bad I thought and doable with the current rate limiting within a few hours.

For completeness, here are the core pieces of code for the third scenario:

```
def find_best_pixels_random(image):

    initial_score = float(model.predict(image))
    
    block_size = 1
    delta_mid = np.ones([1, block_size,block_size, 3]) #* 128/255.

    best_score = 1e-100
    best_image = 0

    run_image = image

    for count in range(4000):
            
        i = np.random.randint(0,128-block_size)
        j = np.random.randint(0,128-block_size)

        temp_image = np.copy(run_image)
        temp_image[0, i:i+block_size, j:j+block_size] = delta_mid * 0#(np.random.randint(200,256)/25.)

        current_score = float(model.predict(temp_image)[0][0])
        attempts = attempts + 1 

        if current_score > best_score:
            best_score = float(current_score)
            run_image = temp_image

        if best_score > 0.5:
            return run_image

        #update image so we see a nice animation
        plt.imshow(run_image[0], interpolation='nearest')
        plt.text(y=10, x=150, s=f"Initial Score: {initial_score:8f}", fontsize=12)
        plt.text(y=20, x=150, s=f"Best Score:   {best_score:8f}", fontsize=12)
        plt.text(y=30, x=150, s=f"Current Score: {current_score:8f}", fontsize=12)
        plt.text(y=40, x=150, s=f"API Call Count: {attempts}", fontsize=12)
        plt.show()
        clear_output(wait=True)  

image = load_image("shadowbunny.png")
best_image = find_best_pixels_random(image) 
```

[![Husky Pixel Spray Perturbation](/blog/images/2020/huskyai-ml-perturbation-random-pixels.gif)](/blog/images/2020/huskyai-ml-perturbation-random-pixels.gif)

That did the trick and it works, even with a low number of calls ot the API.

Nice! 

Next I want to explore what TensorFlow offers in this regards as well.

### **Test Case 4:** Leveraging machine learning techniques (FGSM)

There are more advanced ways of doing such attacks. In particular the most famous/popular method seems to be `Fast Gradient Signing Method (FSGM)`. The method is described in detail this paper called [Explaining and Harnessing Adversarial Examples ](https://arxiv.org/abs/1412.6572).

There are libraries that ship with [TensorFlow that help](https://www.tensorflow.org/tutorials/generative/adversarial_fgsm) with such attacks (and also to make your model more resilient to these attacks).

I used the TensorFlow libraries to implement this technique. Technically the adversary has to either have access to the model, or use a model that is similar. From the FGSM paper:

> An intriguing aspect of adversarial examples is that an example generated for one model is often misclassified by other models, even when they have different architecures or were trained on dis-joint training sets.

In the next post I will cover stealing models via at least two ways that I already can think of, but for now let's assume we have that already. Here is the code I used to generate the adversary images using machine learning in TensorFlow:

```
shadowbunny = load_image("shadowbunny.png")
initial_score = model.predict(shadowbunny)

image_tensor = tf.constant(shadowbunny, dtype=tf.float32)
delta        = tf.Variable(tf.zeros_like(image_tensor), trainable=True)
optimizer    = tf.keras.optimizers.Adam(learning_rate=0.0008)
loss_object  = tf.keras.losses.BinaryCrossentropy()

best_score = 0
candidate  = 0

for round in range(40):
    with tf.GradientTape() as tape:
        tape.watch(image_tensor)
        
        candidate = image_tensor + delta
        candidate = tf.clip_by_value(candidate, clip_value_min=0, clip_value_max=1)    

        best_score = model(candidate)
        loss_value = loss_object( tf.convert_to_tensor([1]),  tf.convert_to_tensor(best_score))
        
        #update image every iteration for a nice animation
        clear_output(wait=True)  
        plt.imshow(candidate[0])
        plt.text(y=10, x=150, s=f"Initial Score: {initial_score[0][0]:8f}", fontsize=12)
        plt.text(y=20, x=150, s=f"Best Score: {best_score[0][0]:8f}", fontsize=12)
        plt.text(y=30, x=150, s=f"Current Loss: {loss_value:8f}", fontsize=12)
        plt.show()
        
    gradients = tape.gradient(loss_value, image_tensor)
    optimizer.apply_gradients([(gradients, delta)])
    
print("Final Score: ", best_score.numpy()[0][0])
```

This is what running the attack looks like. In the animation pay  attention to the pixels of the image. Do you notice some of them are slightly changing especially towards the final rounds?

[![Shadowbunny FGSM](/blog/images/2020/huskyai-shadowbunny-fgsm.gif)](/blog/images/2020/huskyai-shadowbunny-fgsm.gif)

For reference here is the final adversarial image:

[![Shadowbunny FGSM](/blog/images/2020/huskyai-shadowbunny-fgsm.png)](/blog/images/2020/huskyai-shadowbunny-fgsm.png)

Using this technique we get a 90% accuracy adversarial image with just 40 queries, and if we increase the learning rate this can be further reduced. Pretty impressive.


## What's next?

I hope you enjoyed reading and learning about this as much as I do and this post was intersting.

The last test case we discussed in this post requires access to a model directly. This is because it uses the weights to optimize for the best places to put pixels at. 

The examples in this post really show that machine learning is a bit fragile. I learned a lot already and in the next post I want to explore ways of stealing the model. So, stay tuned.


## Appendix {#appendix}

### Attacks Overview

These are the core ML threats for Husky AI that were identified in the [threat modeling session](/blog/posts/2020/husky-ai-threat-modeling-machine-learning/) so far and that I want to research and build attacks for. 

Links will be added when posts are completed over the next serveral weeks/months.

1. [Attacker brute forces images to find incorrect predictions/labels - Perturbation Attack](/blog/posts/2020/husky-ai-machine-learning-attack-bruteforce/) 
2. [Attacker applies smart ML fuzzing to find incorrect predictions - Perturbation Attack](/blog/posts/2020/husky-ai-machine-learning-attack-smart-fuzz/) 
2. **Attacker performs more perturbation variants - Perturbation Attack** (this post)
3. Attacker gains read access to the model - Exfiltration Attack
4. Attacker modifies persisted model file - Backdooring Attack
5. Attacker denies modifying the model file - Repudiation Attack
6. Attacker poisons the supply chain of third-party libraries 
7. Attacker tampers with images on disk to impact training performance
8. Attacker modifies Jupyter Notebook file to insert a backdoor (key logger or data stealer)


## References

* [TensorFlow Adversarial FGSM](https://www.tensorflow.org/tutorials/generative/adversarial_fgsm)
* [Explaining and Harnessing Adversarial Examples ](https://arxiv.org/abs/1412.6572)
