---
title: "Flipper Zero Hacking"
date: 2022-03-18T23:58:26-07:00
draft: true
tags: [
        "research", "flipper", "hardware"
    ]

twitter:
  card: "summary_large_image"
  site: "@wunderwuzzi23"
  creator: "@wunderwuzzi23"
  title: "Flipper Zero - the multi-tool educational device"
  description:  "Flipper Zero - the multi-tool educational device"
  image: "https://embracethered.com/blog/images/2020/flipper-update.png"
---


After years of waiting my **Flipper Zero** finally arrived in the mail.

This is what the package looks like after opening. It contains the device, a USB cable, a quick start manual (mostly pointing you to the Flipper Zero website), and a Flipper "Hack the planet" sticker.

![Flipper Zero - Package](/blog/images/2022/flipper-package.png)


Even though the software is still being worked on, it already feels quite polished and intuitive.

Be aware that you need a microSD card to update to the latest firmware and store your data. 

![Flipper Zero - Package](/blog/images/2022/flipper-zero-update.png)

Some of the info in this post will quickly be outdated, due to the current velocity of discussions and hacks that are being created. Especially the [Flipper Zero Discord Channel](https://flipp.dev/discord) is on fire at the moment! 


## qFlipper - The Companion Desktop App

The Flipper Zero comes with a companion app to install the firmware. 

![Flipper Zero - File Manager](/blog/images/2022/flipper-zero-file-manager.png)


The firmware update was very smooth. Very soon there will be apps for Android and iOS.

There are also beta builds available that include future features updates, like a file manager to copy files over.  [You can get it here](https://update.flipperzero.one/builds/qFlipper/file-manager). 

![Flipper Zero - File Manager](/blog/images/2022/flipper-zero-file-manager-files.png)

Chances are that by the you read this, the latest version of the qFlipper already includes the File Manager.

## NFC

The first feature I investigated was the NFC capabilities by testing the following devices:

1. Physical Card - wow, I can see my credit card numbers!
2. Apple Watch - wow, I can see my credit card numbers!
3. Airtags - was able to read the Device ID
4. Ski-passes and random other things - grabs NFC UID without issues

All of them worked. There is also an emulation mode, that allows you to emulate a stored device - be careful though and understand the risks involved in doing so (also don't do anything illegal obviously).

For the debit/credit cards and the Apple Watch (once opend to scan the card in the wallet) it was possible to read the card number from the devices!

![NFC Reading](/blog/images/2022/flipper-nfc.png)

The above screenshot shows the example of an old pre-paid card. Even though its sort of obvious that this is possible, it is quite scary to realize how simple it actually is.

**This really drives the point home to invest in sleeves that shield cards to prevent arbitrary devices from reading your credit card numbers.**

## RFID

The Flipper Zero reads key fobs and cards seamlessly. It can also emulate a previously scanned fob, which is quite handy. There is also a **write** feature that allows to write the info of a scanned RFID chip from one key fob to another for instance.

**One thing I learned is that there are actually people that have RFID implants under their skin! I am not kidding!**

## Infrared

The infrared feature is very simple to use. It's easy to capture and record commands from your own remote for instance. Afterwards you can replay them. 

One can also download from a large collection of command sequences from [here](https://github.com/Lucaslhm/Flipper-IRDB).

There is a short video [here](https://www.youtube.com/shorts/drcEkZ6mCcM) that shows how to turn an XBox on.


## Bad-USB

Bad-USB is a feature that turns the Flipper Zero into a USB devices, such as a keyboard, and one can send in a pre-defined sequence of commands.

The default example script that ships with the device just opens notepad and writes the following text:

![Bad USB Example](/blog/images/2022/flipper-zero-bad-usb.png)

If you are familiar with [BashBunny](https://shop.hak5.org/products/bash-bunny) or Rubber Ducky this will look quite familiar, and you can also use scripts from the former to run on the Flipper Zero.

These tactics are also used to steal hashes from machines during red team ops. The Flipper Zero should be able to aid in doing the same. :)

## Radio 

This feature comes with a frequency analyzer which I find very neat. I tried to clone my car key fob, and am able to analyze and record the sequence, but wasn't yet succesful with replaying it to actually open the doors. It probably means that my key fob has mitigations (like a rolling key) – which is good to know.


**Also, be careful when doing these kinds of things because modern key fobs have rolling keys, and its possible to brick your own key fob by desyncing it! After all remember that the Flipper Zero is an educational device, so don't use it for bad things or on anything important you actually rely on.**

There are videos floating around with people opening their Tesla's charging lid, because all Tesla's share the same key for opening that. 

## Misc. and custom code

The Flipper Zero comes with several other features and gimmicks! For instance, you can play a round of Snake, or leverage the devices as U2F Device + lots of other things.

## Conclusion

Overall, the product is very good and polished already, and future software and firmware updates will just make it better. 

## References


* [qFlipper Builds](https://update.flipperzero.one/builds/qFlipper/file-manager)
* [Bash Bunny](https://shop.hak5.org/products/bash-bunny)
* [IR Sequence Packages](https://github.com/Lucaslhm/Flipper-IRDB)
