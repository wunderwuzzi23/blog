---
title: "Machine Learning Attack Series: Perturbation"
date: 2020-09-15T11:34:05-07:00
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

The previous post covered some basic tests and brute forcing which had surprisingly good results. Then mitigations were put in place for the attacks.

Now, let's look at even "smarter" ways to perform attacks from the outside against the exposed prediction web endpoint.

## Shadowbunny turns Husky

The next challenge I want to tackle is to take a non-husky image and modify to be idenified as a husky by the model. 

Initially I was not sure if this is possible without having access to the actual model file, but I learned a lot and its quite fascinating how easily a model can be fooled.

The additional challenge in this case is, that the model is only callable via a web endpoint. And that calling the API is limted because of API throttling that was implemented in earlier stages of the project.


### Calling the prediction API

Running the Shadowbunny image through the prediction showed a 0.0085% chance of it being a husky.

```
shadowbunny_image = load_image("../scraper/shadowbunny.png")
predict(shadowbunny_image[0])["score"]
```

TODO image:

The results show the following score:

```
0.00038117
```

So this looks good, certainly far from being seen as a husky by the system.
For starters, this is the Shadowbunny image:

![Shadowbunny](/blog/images/2020/shadowbunny-small.png)




Challenge accepted.  

Let's see what attack we can come up with.




## Testing ideas

To get started I will do some of my own research and experiments and then look at what more experienced researches have been doing. Here are some ideas:

1. Pick a small noise image and insert it at random places in the picture. Possibly using a sliding window to see how much the prediction score changes.
2. Sequentially go over each pixel and change it to see how it impacts the score, keep those pixels that increase score (slow and lots of API calls). Repeat until prediction reaches 50%.
3. Randomly spray a pixel, if it increases the prediction score keep it if not try again. Repeat until prediction reaches 50%
4. Applying machine learning to "reverse optimize" the loss. Sort of gradient ascent. *hand wavy idea*. I assume that this is what is the best way of doing it is.

### Test Case 1: Move a block across image to find "hot spots"

The idea behind this is to possible find areas of the image that cause large changes in the prediction.

I went ahead to write some code to pick a couple of pixel blocks (32x32) of various colors and moved them in larger steps across the base image to see if any of those cause a larger change in prediction score.

Here is how this looks like in action:

[![Husky Perturbation](/blog/images/2020/huskyai-perturbation1.gif)](/blog/images/2020/huskyai-perturbation1.gif)

You can see I even tried moving a small 32x32 pixel husky image over the Shadowbunny to try and trick the model.

The following pictures shows a good increase in prediction by using the black block:

![Husky Perturbation All Blocks](/blog/images/2020/huskyai-perturbation-black-block.png)

In the end I took the best scores for each block (white, black, random, and a husky image block) and merged them all together in a final prediction which resulted in a score of 3.14%. The idea is working but this is still far away from 50%... 

![Husky Perturbation All Blocks](/blog/images/2020/huskyai-perturbation-allblocks.png)

And as can be seen in the screenshot above, the blocks are very visible on the Shadowbunny image.


### Test Case 2: Sequentially put pixels on the image

The second idea I had was just to:

1. Start at index 0,0 of the image and change pixel at that location
2. Run the image through a prediction and record score
3. If the result is a new best_score, then store it 
4. Take a larger step (4 pixels) to index 0,4 and update the pixel at that location again
5. Run the image through prediction and record the score
6. If the result is a new best_score, then store it 
7. And so forth until we either reach a best_score > 50% or we end at the bottom of the image.

The major drawback of this was that it creates a lot of queries and hence I would have to wait for a long time due to the rate limiting of the prediction API. So, I cheated a bit here and tested this locally directly against the model, just to see if it might work.

The code for this was pretty simple:

```

```

It does work! Although it requires about 800-5000+ calls to the API depending on `block_size` and `step_size`. Also the color of the pixel makes a big difference.

![Shadowbunny machine learning attack](/blog/images/2020/shadowbunny-ml-attack-1.jpg)

Above is an image that shows the second attack. 

### Test Case 3: Randomly spray pixels on the image

The next idea was to just randomly put pixels on the screen. 

[![Husky Perturbation Random](/blog/images/2020/huskyai-ml-perturbation-random-pixels.gif)](/blog/images/2020/huskyai-ml-perturbation-random-pixels.gif)

If you watch the give to the end you can see a prediction score of 50%+ was reached this way! It took about 450 calls to the API, which is not that bad I thought and doable with the current rate limiting within a few hours.

For completness, here are the core pieces of the source code for the third scenario:

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

That's it. At that point I felt pretty good,

### Test Case 4: Leveraging machin learning techniques




## What's next?


I hope you enjoyed reading and learning about this as much as I do. I learned a lot already and am eager to dive learning smarter ways of coming up with malicious/adversarial examples.


## References

[Hacking Neural Networks: A short introduction](https://github.com/Kayzaks/HackingNeuralNetworks)


## Appendix {#appendix}

### Attack Overview

These are the core ML threats for Husky AI that were identified in the [threat modeling session](/blog/posts/2020/husky-ai-threat-modeling-machine-learning/) so far and that I want to research and build attacks for. 

Links will be added when posts are completed over the next serveral weeks/months.

1. [Attacker brute forces images to find incorrect predictions/labels - Perturbation Attack](/blog/posts/2020/husky-ai-machine-learning-attack-bruteforce/) 
2. [Attacker applies smart ML fuzzing to find incorrect predictions - Perturbation Attack](/blog/posts/2020/husky-ai-machine-learning-attack-smart-fuzz/) 
2. **Attacker performs transfer learning to create perturbation attacks - Perturbation Attack**
3. Attacker gains read access to the model - Exfiltration Attack
4. Attacker modifies persisted model file - Backdooring Attack
5. Attacker denies modifying the model file - Repudiation Attack
6. Attacker poisons the supply chain of third-party libraries 
7. Attacker tampers with images on disk to impact training performance
8. Attacker modifies Jupyter Notebook file to insert a backdoor (key logger or data stealer)
