---
title: "Broken NFT standards"
date: 2021-03-19T10:16:02-07:00
draft: true
tags: [
        "crypto"
    ]
---

You probably heard of NFTs (non-fungible tokens). They are receiving a lot of interest over the last several months. I did some digging and realized that there are some bigger issues with the standards and various interpretations and implementations of it, and how **centralized** many offerings are.

## What is an NFT

The idea behind it is simple, use a blockchain (and cryptography) to assign and be able to proof ownership over a specific piece of digital content.

Some critics say NFTs make no sense because one can always just copy the bits, because they are public. I think that's beyond the point, because you can also take a picture of the Mona Lisa in the Louvre and hang it up at home. 

So, in this post I don't want to discuss the value of art, but focus on the technological issues around the promises NFTs make.

### Promises

Blockchain sounds awesome, because of its decentralized nature. 

And also, the Ethereum website states about NFTs: 

> "They can only have one official owner at a time and they're secured by the Ethereum blockchain – no one can modify the record of ownership or copy/paste a new NFT into existence."  

Unfortunately, this is not accurate, or at least misleading.

The reason is that there is no required link (e.g. something like a cryptographic hash) between asset and Ethereum blockchain entry. 

In some cases that link (a JSON metadata file) is stored centrally by companies or in Amazon S3. So anyone with access to that, can modify the NFT directly.

### Ransomware 

To continue that thought: What about ransomware that just encrypts the NFT content - yes, such attacks are totally possible and not considered in the NFT threat model.


### Crypto AI Huskies

I decided to mint my own NFTs to see how this all works and what the fuzz is about. I choose the AI generated huskies [Crypto AI Huskies](https://opensea.io/collection/crypto-huskies) that you are probably familiar with if you followed my [Husky AI Machine Learning Attack Series](https://embracethered.com/blog/posts/2020/husky-ai-walkthrough/).

And after minting my own NFTs at opensea.io and looking at some other offerings I got a bit disappointed in the actual guarantees that can be made with NFTs.

## NFT standards are broken

The NFT standards allows so much flexibility that no-one should be surprised that a few years from now many NFTs will be gone forever - even if you as the owner still have the private key. Even if you have a local copy of the digital content as well, you can't proof that you are the owner! No kidding.

Let me explain.

## The standards EIP 721 and EIP 1152

There are basically two standards, called [EIP 721](https://eips.ethereum.org/EIPS/eip-721) and [EIP 1152](https://eips.ethereum.org/EIPS/eip-1152). 

Let's look at [EIP 721](https://eips.ethereum.org/EIPS/eip-721). 

It defines the `ERC721Metadata` class and its properties are:
* `name` - a name of the NFT
* `symbol` - some abbreviation symbol
* `tokenURI` - a URI to either a metadata file or URI to the content of the NFT


So what we have is the `ETH address` -> `tokenURI` -> `metadata file` -> `content`.

**The (secure) decentralized aspect of NFTs ends with the link from `tokenURI` to `metadata file`.**

This means the only thing you can proof is that you own a specific `tokenURI`, but there is no evidence or link in the blockchain on what the digital content it links to actually is. 

### Spot the defect!?

What is missing here? **A hash!**

Yes, the smart contract does not include a hash of the actual digital content! So there is no requirement to cryptographically link the token blockchain entry to represent the actual content of the NFT. 


## Metadata file with the same issue

The same goes for the **metadata JSON file**, it contains:
* `name`
* `description`
* `image`

Again, the image can point anywhere - even to a place that might be gone one day. And here is the issue, there is no way you could proof that a particular image you stored offline, because you bought it and put it in a safe is indeed the one in the blockchain!

### Again, no cryptographic link between blockchain and asset

Again, because there is no hash of the image available that would link the content of the NFT to the token in the blockchain.

**If this does seem a bit broken, you are correct because it is.**

## Broken standards

What is troublesome is that claims on the [Ethereum.org NFT website](https://ethereum.org/en/nft/
) are incorrect because of the technical limitations highlighted above:

The page states that: 

> "if you own an NFT" .... "No one can manipulate it in any way."

**This is fundamentally incorrect given how lax the EIP 712 standard is.** 

The image or metadata files are typically stored in a central place, maybe with Amazon S3. 

So it can be modified by multiple stakeholders. 

Some offerings do use IPFS, which seems the better place, but still what would actually help is putting a simple hash of the content in the blockchain - because then at least you can demonstrate if it was tampered with or if you have the same item. 

Or if it disappears and you have your own local copy, you can still proof that you are the owner of that particular NFT.

## Better ways and engineers realizing issues

Some offerings, like Makerspace, appear to put the JSON metadata file into IPFS and include a **digital_media_signature** in the file as well. 

This highlights that their engineers understand the limitations of the current NFT standards expressed in the EIPs.

## Conclusion and recommendations

Be careful on which project you choose to mint your NFTs. Many of them are quite centralized. 

Your NFT might just disappear one day, or it falls victim to ransomware attack and there is zero evidence that connects the token in the blockchain to the artwork you bought - because the standard does not consider this.

I think the standard should be changed to require a mandatory hash to what the content of the NFT actually is representing.

## References

* [EIP 721 - Non-fungible token standard](https://eips.ethereum.org/EIPS/eip-721)
* [Ethereum NFT page](https://ethereum.org/en/nft/)
* [Crypto AI Huskies on opensea.io](https://opensea.io/collection/crypto-huskies)
