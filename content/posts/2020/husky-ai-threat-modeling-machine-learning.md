---
title: "Threat Modeling a machine learning system"
date: 2020-09-05T12:04:33-07:00
draft: true
tags: [
        "machine learning",
        "huskyai"
    ]
---

This post is part of a series called "Machine learning for pen testers" to help pen testers and security engineers learn more about AI/ML. Click the tag "huskyai" or visit the [overview section](/blog/posts/2020/husky-ai-walkthrough/).

In the [previous post](/blog/posts/2020/husky-ai-building-the-machine-learning-model/) we walked through the steps required to gather training data, build and test a model to build "Husky AI".

This post is all about threat modeling the system to identify scenarios for attacks which we will perform in the upcoming posts.

# Part 4 - Threat Modeling an AI system{#part4}

There are quite a lot of moving pieces already in the "Husky AI" system that we built out.

To get started with identifiying security issues in a systematic way, I typically create a high-level Threat Model diagram using Microsoft Threat Modeling tool (or just on a piece of paper at times too).

[![Husky AI Threat Model](/blog/images/2020/husky-ai-machine-learning-threat-model.png)](/blog/images/2020/husky-ai-machine-learning-threat-model.png)

The threat modeling tool is quite thorough and it is common that engineers complain that it creates too much noise, and it takes too long to review. For instance, this threat model led to 96 threats that were automatically identified by the tool. 

By clicking on a component in the threat modeling tool, we can view the specific threats for that particular asset. For instance this is the view for the web server:

[![Threat Modeling Tool Example](/blog/images/2020/threatmodelingtool.jpg)](/blog/images/2020/threatmodelingtool.jpg)

Even though there is a large amount of threats, it is great because it highlights issues you might not think about, or not think about each time you perform threat modeling. However, it can be a bit of an overload, and some threats are repetitive. There is really no requirement to use the tool - it's just there to help. 

When it comes to brainstorming and classifying threats, I use STRIDE. STRIDE is a threat classification technique [developed by Microsoft](https://en.wikipedia.org/wiki/STRIDE_(security)). It puts threats into the following categories:

1. **S**poofing
2. **T**ampering
3. **R**epudiation
4. **I**nformation disclosure
5. **D**enial of service
6. **E**levation of privilege

It is good approach to hold threat modeling sessions with engineers, those can be informal whiteboard sessions. Try to have at at least one threat in each category for every asset.

**It is not uncommon to forget about repudiation issues**. 

**Repudiation** is the problem of someone denying an action. How can you proof or disproof someone indeed updated a machine learning model in production for instance?

### Perturbation Attacks

When it comes to machine learning specific attacks most of them can be easily mapped to STRIDE categories. The one I had issues with are attacks that "query the model", so called **perturbation attacks**. For the purpose of the threat model, I decided that this is a "spoofing" issue. An adversary tries to trick the algorithm to misclassify an image.

Now, let's review the core assets of the Husky AI system, and brainstorm about attacks and mitigations. 

# Core Assets and Threats

The following section lists the core assets and threats that were identified during threat modeling. This information is ideally persistent in a bug tracking system, something like Jira. 

Threats and mitigations are a moving target, something that was considered mitigated yesterday might not be mitigated anymore today. This is also were red teaming comes into the picture to help with the realization that every system has vulnerabilities that can be exploited - there is no such thing is a 100% mitigation.

Let's look through the main assets:

* Training, Validation Images
* Public facing REST API
* Stored Machine Learning Model
* Jupyter Notebook
* Bing API Key
* SSH Access and Keys

To keep the flow of the post, let us look at those that are most interesting from machine learning point of view.

The full list of threats can be seen in the [Appendix](#appendix) - for anything I wanted to test, I added a **"red team opportunity"** note.


## Identified machine learning threats

The following is the list of threats that I want to build attacks for:

1. Attacker brute forces images to find incorrect predictions/labels (dumb and smart ML fuzzing) - Perturbation
2. Attacker gains read access to the model 
3. Attacker modifies persisted model file (backdoor and re-train)
4. Attacker denies modifying the model file
5. Attacker poisons the supply chain of third-party libraries
6. Attacker tampers with images on disk to impact training performance
7. Attacker modifies Jupyter Notebook file to insert a backdoor (key logger or data stealer)

This is a fairly good list with some interesting attack scenarios that we can research and perform. 

There are other ML specific attacks such as model inversion, membership inference which I might explore at a later point, possibly when upgrading Husky AI to provide a more complex categorical model. For more information on machine learning threats see [Failure modes in machine learning] (https://docs.microsoft.com/en-us/security/engineering/failure-modes-in-machine-learning) by Microsoft, as well as "Hacking neural networks: A short introduction" by Michael Kissner.


## Next steps

In the next post, we will do hands-on attacks against the exposed prediction API via brute forcing. 

Now it will become interesting! :)


## Appendix{appendix}

This appendix contains the most significant threats identified and more insights. When doing this you want to store this information in a issues/bug tracking system such as Jira. This helps to track and also review dispositions in the future.


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

* [STRIDE](https://en.wikipedia.org/wiki/STRIDE_(security))
* [Husky AI Source Code](https://github.com/wunderwuzzi23/ai/)
* [Failure modes in machine learning] (https://docs.microsoft.com/en-us/security/engineering/failure-modes-in-machine-learning)