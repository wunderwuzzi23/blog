---
title: "Machine Learning Attack Series: Smart brute forcing"
date: 2020-09-12T09:04:09-07:00
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

Now, let's look at "smarter" ways to perform attacks from the outside against the exposed prediction web endpoint.

## "Smarter" machine learning brute forcing

The system we test was initially vulnerable to simple random pixel fuzzing. This was mitigated by training the model with these "bad" images to make it more resilient. 

On the operational side, we added `rate limiting` to the external facing API to make things *a little bit more interesting/challenging* for an attacker.

Let's see what else we can try by being a little "smarter" with our testing efforts.

### Calling the API

First, a couple calls to the API to see current score for random images:

```
# create a random sample and run prediction against web endpoint
candidate_rand = np.random.random([NUM_PX, NUM_PX, 3])
print("Random Image: " + predict(candidate_rand)["score"])
```

The results show the following score:

```
Random Image: 0.17533675
```

The mitigation put in place for the previous attacks works and this prediction comess in at about 17.5% for some random pixels. However, that score seems still pretty high. 

*Note: The model in this post was trained and updated with two additional training epochs for identified adversarial images. This might not be enough! Let's see...*

Now it's time look the attacks in this post - these three in particular:

1. Ordinary brute force. The best guess after 10 times will be our `base_image`
2. Pick random samples in medium range (90-145) to create a `delta_image`. Add `delta_image` to `base_image` and run `predict`. Substract `delta_image` from `base_image` and run `predict`. Alternatively, pick one or two shades in low (0-2), medium (126-127) and high (253-255) range and run `predict` with each of them.
3. Use `np.random.normal` (normal distribution) to create `delta_image`. Add `delta_image` to `base_image`. This trick is from Michael Kissner's Github repo that goes along with this [great paper](https://arxiv.org/pdf/1911.07658.pdf).

All three test cases only do 10 queries against the `predict` API --> meaning we issue few API calls. However, that still did hit the API rate limiting - more about that below.

### Test Case 1: Simple brute force to find a good "base" image

The first step is simply to run a brute force with random pixels and store the best result in `best_bruteforce_image`. Due to being throttled the attack starts with only 10 attempts.

Here is the code:

```
attempts = 10
current_best_score = 1e-100
best_bruteforce_image = 0

for i in range(attempts):
    
    ##create a random image
    candidate_image = np.random.random([NUM_PX, NUM_PX, 3])

    result = predict(candidate_image)
    score = float(result["score"])

    if score > 0.5:
        print(f"Found a random husky. Iteration: {str(i)} Score: {score}")
        current_best_score = score
        best_bruteforce_image = candidate_image

    if score > current_best_score: 
        current_best_score = score
        best_bruteforce_image = candidate_image
        print("New best score: " + str(current_best_score))

plt.imshow(best_bruteforce_image)
```

The results are not looking too good yet for the attacker:

```
New best score: 0.11286646
New best score: 0.19212896
New best score: 0.23637468
```
Here is the best image so far:

![Test 1 Husky AIML Hacking Test](/blog/images/2020/ml-fuzzing-attack-smart1.jpg)

Looks pretty random, maybe running more iterations can increase the `base_image` score.

### Rate limiting kicking in - very annoying

After calling the `predict` API frequently rate limiting started kicking in. 

```
503 Service Temporarily Unavailable"
```

Eventually it happened very frequently, and I had to add `sleep` commands to slow down the attack rate. This slows down testing from a single IP address - quite annoying. 

No chance to run 100000 tests in rapid succession anymore.

However, a motivated attacker might come from many different IP addresses, so throttling has its limitations (especially on an unauthenticated endpoint). It however does increase cost and complexity for an adversary.

More info on configuration settings in the [#appendix](https://www.nginx.com/blog/rate-limiting-nginx/).


### Test Case 2: Base image + close random "neighbor-colors"

This is the code for the second scenario:

``` 
import time

base_image = best_bruteforce_image

def probe_range(min, max, step):
    current_best_score = 1e-100
    best_candidate_image = 0

 
    for n in range(min, max, step):
    
        #try both adding and subtracting the range from the base image
        for i in range(2):

            # create a temp image  
            temp_image = np.random.random([NUM_PX, NUM_PX, 3]) * n/255.
            if (i == 0):
                candidate_image = base_image + temp_image
            else:
                candidate_image = base_image - temp_image

            result = predict(candidate_image)
            score = float(result["score"])

            if score > current_best_score: 
                current_best_score = score
                best_candidate_image = candidate_image
                print("New best score: " + str(current_best_score))
            
        time.sleep(10)

    return best_candidate_image, current_best_score
   
#take a bigger step (5), to limit number of queries overall 
best_smart_bruteforce_image, best_score = probe_range(95,145, 5)
plt.imshow(best_smart_bruteforce_image)
```

Note the addition of the `time.sleep(10)` to slow down the probing rate.

In this run I got the following scores:

```
New best score: 0.40285298
New best score: 0.4952831
New best score: 0.4988353
```

These numbers look promising already. Just by updating pixels and slightly playing with the color intensity of the `base_image`.

This is how the best scoring image looks:

![Test 2 Husky AIML Hacking Test](/blog/images/2020/ml-fuzzing-attack-smart2.jpg)

At this point I was wondering if there even better was of doing this. There is an excellent example in Michael Kissner's ["Hacking Neural Networks"](https://arxiv.org/pdf/1911.07658.pdf)) work, where picking pixels via normal distribution for generating what the paper calls "noise" to fool image recognition when handling digits. 

Let's look at that.

### Test Case 3: Best base image + normal distributed "neighbor-colors"

This was quickly implemented using the `np.random.normal` function. It took some tweaking and it showed vastly different results depending on how the "base_image" looked like. Hence the code is now probing distributions from 0-255 in increments of 25. This is in order to stay below the rate limiting as much as possible, and not have to wait for too long to see results.

```
import time

base_image = best_bruteforce_image

def probe_norm_range(min, max, steps):
    current_best_score = 1e-100
  
    for d in range(min, max, steps):

        ##create a temporary delta image with normal distribution
        temp_image = np.random.normal(d, 1, [NUM_PX, NUM_PX, 3])
        candidate_image =  (base_image * 255. + temp_image) / 255.
        
        result = model.predict(candidate_image)
        score = result["score"]

        if score > 0.5:
            print(f"Found a random husky. IterationId: {str(d)} Score: {score}")
            plt.imshow(candidate_image[0])
            
        if score > current_best_score: 
            current_best_score = score
            best_image = candidate_image
            print("New best score: " + str(current_best_score))

     time.sleep(10)

    return best_image, current_best_score

best_smarter_bruteforce_image, best_score = probe_norm_range(0,256,25)
print(best_score)
```

The results of this technique are cool I thought:

```
New best score: 0.23516122
New best score: 0.35612822
Found a random husky. IterationId: 50 Score: 0.6366884
New best score: 0.6366884
Found a random husky. IterationId: 75 Score: 0.80138826
New best score: 0.80138826
Found a random husky. IterationId: 100 Score: 0.80683583
New best score: 0.80683583
Found a random husky. IterationId: 125 Score: 0.736615
Found a random husky. IterationId: 150 Score: 0.557814
```

This broke the model again, meaning that more training on bad images is necessary to mitigate attacks.

For fun, this is the highest scoring "husky" image: 

![Test 3 Husky AIML Hacking Test](/blog/images/2020/ml-fuzzing-attack-smart3.jpg)

That does not look like a husky to me.

### Results overview

Finally, let us look at the results of the three test cases:

```
print(predict(best_bruteforce_image))
print(predict(best_smart_bruteforce_image))
print(predict(best_smarter_bruteforce_image))
```

And here are resulting numbers for reference:

```
0.23637468
0.4988353
0.80683583
```

So what about mitigations now?


## Mitigations

The mitigation is to train the model with bad test images and tell the neural network that these images are definitely not huskies! Additional training means doing something like this:

```
print("Fitting model...")
model.fit(np.array(bad_images),np.array(not_husky_labels), epochs=100, verbose=0)
``` 

After fitting the model with three images and running for 100 epochs, the results of the attack sequence changed to:

```
1.0838663e-10
1.9329406e-11
3.524349e-10
```

Nice. Looks like its rather difficult to guess a husky image with these brute forcing attempts out of thin air (even when applying some smarter techniques). 

## What's next?

The next logical step seems to be to start from a valid husky image and modify it to become a non-husky image, while still having it look husky-ish.

There is a famous image with a panda bear + noise image that I have seen as an example for this sort of attack. I'm excited to learn more about this next. My goals is to spend some time experimenting and trying things out myself before reading up on details of others work.

I hope you enjoyed reading and learning about this as much as I do. I learned a lot already and am eager to dive learning smarter ways of coming up with malicious/adversarial examples.


## References

[Hacking Neural Networks: A short introduction](https://github.com/Kayzaks/HackingNeuralNetworks)


## Appendix {#appendix}

### Attack Overview

These are the core ML threats for Husky AI that were identified in the [threat modeling session](/blog/posts/2020/husky-ai-threat-modeling-machine-learning/) so far and that I want to research and build attacks for. 

Links will be added when posts are completed over the next serveral weeks/months.

1. [Attacker brute forces images to find incorrect predictions/labels - Perturbation Attack](/blog/posts/2020/husky-ai-machine-learning-attack-bruteforce/) 
2. **Attacker applies smart ML fuzzing to find incorrect predictions - Perturbation Attack (this post)**
3. Attacker gains read access to the model - Exfiltration Attack
4. Attacker modifies persisted model file - Backdooring Attack
5. Attacker denies modifying the model file - Repudiation Attack
6. Attacker poisons the supply chain of third-party libraries 
7. Attacker tampers with images on disk to impact training performance
8. Attacker modifies Jupyter Notebook file to insert a backdoor (key logger or data stealer)


### Rate limiting configuration with nginx {#ratelimit}

[See more information on the nginx documentation for rate limiting](https://www.nginx.com/blog/rate-limiting-nginx/).

Below are some of the settings I experimented with at the API gateway level:

```
limit_req_zone $binary_remote_addr zone=one:10m rate=5r/m;
limit_conn_zone $binary_remote_addr zone=addr:10m;
limit_req zone=one burst=10 nodelay;
limit_conn addr 5;
client_max_body_size 10M;
```