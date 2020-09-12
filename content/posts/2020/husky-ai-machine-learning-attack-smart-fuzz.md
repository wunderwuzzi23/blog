---
title: "Machine Learning Attack Series: Smart Bruteforcing"
date: 2020-09-10T09:04:09-07:00
draft: true
tags: [
        "machine learning",
        "huskyai",
        "red"
    ]
---

This post is part of a series about machine learning and artificial intelligence. Click on the blog tag "huskyai" to see related posts. 

[Overview](/blog/posts/2020/husky-ai-walkthrough/): How Husky AI was built, threat modeled and operationalized
[Attacks](#appendix): The appendix shows what kind of attacks I want to investigate, learn about and try out

In the previous post I looked at bruteforcing images and that surprisingly had good results. Now that we put some basic mitigations in place, let's look at "smarter" ways to perform attacks from the outside.

## Smarter bruteforcing

Recall that we have a model which was vulnerable to simple random pixel fuzzing, and we mitigated it by training the model with the random images we created as well, in order to improve the prediction outcomes and make the model more resilient to the examples. We also proposed adding rate limiting to the external facing API to make things more difficult.

Now when an attacker tries to bruteforce, they can't succeed. 

To recap from the previous post on our original husky model and trying to brute force a random pixel image. As we discussed before we fixed the initial issues and trained our model to be more resilient against corner cases and random images. Note: In the examples given here the orignal model was trained with two additonal epochs for the adversarial images that were identfied in the previous post. 

```
# create a random sample and run prediction against web endpoint
candidate_rand = np.random.random([NUM_PX, NUM_PX, 3])
print("Random Image: " + predict(candidate_rand)["score"])
```

The results show the following score:

```
Random Image: 0.17533675
```

The mitigation works, we have a rather low score now for random pixels. Since there is now also `rate limiting` implemented an attacker can only do few queries per day.

When testing the API and experiementing I did hit the `rate limiting` quite frequently actually: 

```
503 Service Temporarily Unavailable"
```

This slows down testing - very annoying. A motivated attacker might come from many different IP addresses, so throttling has its limitations (especially on an unauthenticated endpoint). It however does increase the cost for an adversary.


Now it's time to spec out some more attacks - these three in particular:

1. Ordinary brute force. The best guess after 10 times will be our `base_image`
2. Pick random samples in medium range (90-145) to create a `delta_image`. Add `delta_image` to `base_image` and run `predict`. Substract `delta_image` from `base_image` adn run `predict`. Alternatively, pick color one or two shades in low (0-2), medium (126-127) and high (253-255) range and run `predict` with each of them.
3. Use `np.random.normal` (normal distribution) to create `delta_image`. Add `delta_image` to `base_image`. I found trick  in Michael Kissner's Github repo that goes along with this [great paper](https://arxiv.org/pdf/1911.07658.pdf).

All three test cases only do 10 queries against the `predict` API --> meaning we issue 30 queries overall. That still did hit the rate limit though when done fast. 

### Attack 1: Brute force

This is what we discussed in the last blog post. In this case we run it 10 times and store the best result in `best_bruteforce_image`.

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

The results are not looking good for the attacker:

```
New best score: 0.11286646
New best score: 0.19212896
New best score: 0.23637468
Done
```

![Test 1 Husky AIML Hacking Test](/blog/images/2020/ml-fuzzing-attack-smart1.jpg)



### Attack 2: Best bruteforce image + bruteforce close random neighbours

This is the code for the second scenario:

``` 
import time

base_image = best_bruteforce_image

def probe_range(min, max, step):
    current_best_score = 1e-100
    best_candidate_image = 0

 
    for n in range(min, max, step):
    
        #try both adding and substracting the range from the base image
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
   
#10 queries overall 
best_smart_bruteforce_image, best_score = probe_range(95,145, 5)
plt.imshow(best_smart_bruteforce_image)
```

In this run I got the following scores:
```
New best score: 0.40285298
New best score: 0.4952831
New best score: 0.4988353
```

![Test 2 Husky AIML Hacking Test](/blog/images/2020/ml-fuzzing-attack-smart2.jpg)

### Attack 3: Best bruteforce image + normal distributed neighbours


```
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
            current_best_score = score
            best_image = candidate_image

        if score > current_best_score: 
            current_best_score = score
            best_image = candidate_image
            print("New best score: " + str(current_best_score))

    return best_image, current_best_score

print("Done.")

best_smarter_bruteforce_image, best_score = probe_norm_range(0,256,25)
print(best_score)
```

New best score: 0.23516122
New best score: 0.35612822
Found a random husky. IterationId: 50 Score: 0.6366884
Found a random husky. IterationId: 75 Score: 0.80138826
Found a random husky. IterationId: 100 Score: 0.80683583
Found a random husky. IterationId: 125 Score: 0.736615
Found a random husky. IterationId: 150 Score: 0.557814

![Test 3 Husky AIML Hacking Test](/blog/images/2020/ml-fuzzing-attack-smart3.jpg)

```
print(predict(best_bruteforce_image))
print(predict(best_smart_bruteforce_image))
print(predict(best_smarter_bruteforce_image))
```

And here is the comparison of the performance of these:

```
0.23637468
0.4988353
0.557814
```

## Conclusion



## References

https://github.com/Kayzaks/HackingNeuralNetworks

## Appendix 

### Attack Overview

These are the core ML threats for Husky AI that were identified in the [threat modeling session](/blog/posts/2020/husky-ai-threat-modeling-machine-learning/) so far and that I want to research and build attacks for. 

Links will be added when posts are completed over the next serveral weeks/months.

1. [Attacker brute forces images to find incorrect predictions/labels - Perturbation Attack](/blog/posts/2020/husky-ai-machine-learning-attack-bruteforce/) 
2. Attacker applies smart ML fuzzing (generative adversarial models) to find incorrect predictions - Perturbation Attack (this post)
3. Attacker gains read access to the model - Exfiltration Attack
4. Attacker modifies persisted model file - Backdooring Attack
5. Attacker denies modifying the model file - Repudiation Attack
6. Attacker poisons the supply chain of third-party libraries 
7. Attacker tampers with images on disk to impact training performance
8. Attacker modifies Jupyter Notebook file to insert a backdoor (key logger or data stealer)


### Rate Limiting nginx

And with `rate limiting` in place it will take a long time to get better results. In caes you also want to play around with settings:

```
limit_req_zone $binary_remote_addr zone=one:10m rate=5r/m;
limit_conn_zone $binary_remote_addr zone=addr:10m;
limit_req zone=one burst=10 nodelay;
limit_conn addr 10;
client_max_body_size 10M;
```


