---
title: "Machine Learning Attack Series: Backdooring Pickle Files"
date: 2022-08-28T20:10:44-07:00
draft: true
tags: [
        "machine learning",
        "red",
        "huskyai",
        "ai"
    ]

twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "Machine Learning Attack Series: Backdooring Pickle Files"
  description: "Machine Learning Attack Series: Backdooring Pickle Files"
  image: "https://embracethered.com/blog/images/2022/huskyai-stylegan2-backdoor-pickle-example.png"
---


Recently I read [this excellent post by Evan Sultanik](https://blog.trailofbits.com/2021/03/15/never-a-dill-moment-exploiting-machine-learning-pickle-files/) about exploiting pickle files on Trail of Bits. There was also a DefCon30 talk about [backdooring pickle files by ColdwaterQ](https://forum.defcon.org/node/241825).

This got me curious to try out backdooring a pickle file myself.

![Red Teaming Machine Learning -  Attack Series](/blog/images/2020/ml-attack-series.jpg)

# Pickle files - the surprises

Surprisingly Python pickle files are compiled programs running in a VM called the Pickle Machine (PM). Opcodes control the flow, and when there are opcodes there is often fun to be had.

Turns out there are opcodes that can learn to running arbitrary code, namely **GLOBAL** and **REDUCE**.

The dangers are also highlighted in pickle documentation page:

> Warning The pickle module is not secure. Only unpickle data you trust.

[Trail of bits](https://github.com/trailofbits/fickling) has a tool named `fickling` that allows to inject code and also check pickle files to see if they have been backdoored.

# Experiments using Husky AI and StyleGAN2-ADA

To test this out I opened a two-year-old Husky AI Google Colab notebook where I experimented with StyleGAN2-ADA and grabbed a pickle file it had produced.

To get set up I run `pip3 install fickling` and started exploring, most notably the `--inject` command:

[![Husky AI Pickle Backdoor Using Fickling](/blog/images/2022/huskyai-stylegan2-backdoor-with-fickling.png)](/blog/images/2022/huskyai-stylegan2-backdoor-with-fickling.png)

As you can see using `--inject` one can inject python commands to the pickle file. 

## Code execution

Now, the question was if that pickle file gets loaded, will the command execute? To test this, I ran the `generate` command from StyleGAN2-ADA to create some new huskies!

[![Husky AI StyleGAN2-ADA Backdoor Example](/blog/images/2022/huskyai-stylegan2-backdoor-pickle-example.png)](/blog/images/2022/huskyai-stylegan2-backdoor-pickle-example.png)

Wow, this indeed worked. The command was executed!

Also, it did not impact the functionality of the program. As proof, here is a picture of one of the newly created huskies: 

[![Husky AI StyleGAN2-ADA Husky](/blog/images/2022/huskyai-stylegan2-husky.png)](/blog/images/2022/huskyai-stylegan2-husky.png)

Now, I was wondering about the implications of this.

## Google Colab Example

When thinking of the implications of this exploit, I realized that even within a Google Colab project this is a big problem. Projects are isolated, but many users have their Google Drive mapped into the Colab project! 

This means that an attack who tricks someone to opening a malicious pickle file could gain access to the drive contents.

Scary stuff. Never use random pickle files.

Of course, this can occur inside other tools and MLOps pipelines, compromising systems and data.

## Checking for safety

`fickling` also has commands built-in to explore and check pickle files for backdoors.

There are two useful features:

1. `--check-safety`: checks for malicious opcodes
2. `--trace`: shows the various opcodes

[![Husky AI StyleGAN2-ADA Husky](/blog/images/2022/huskyai-stylegan2-fickling-trace.png)](/blog/images/2022/huskyai-stylegan2-fickling-trace.png)

Very cool!


# Conclusion

Even though [Husky AI](/blog/posts/2020/husky-ai-building-the-machine-learning-model/) does not use pickle files, its important to know about these backdooring tactics, and that there are some validation and safety tools out there.

As good guidance, only ever open pickle files that you created or trust. In the MLOps flow this opens up opportunities/challenges for pulling in problematic third-party dependencies. 

Also, this is a great opportunity for a red teaming exercise.

Cheers.


# References

* [Never a dill moment: Exploiting machine learning pickle files](https://blog.trailofbits.com/2021/03/15/never-a-dill-moment-exploiting-machine-learning-pickle-files/)
* [Trail of Bits Github Repo for Fickling](https://github.com/trailofbits/fickling)
* [Python Object Serialization](https://docs.python.org/3/library/pickle.html)
* [Machine Learning Attack Series](/blog/posts/2020/machine-learning-attack-series-overview/)
* [Backdooring Pickles: A decade only made things worse by ColdwaterQ](https://forum.defcon.org/node/241825)