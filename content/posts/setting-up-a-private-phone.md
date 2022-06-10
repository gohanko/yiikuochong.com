---
title: "Setting up a private phone."
date: 2022-06-10
draft: false
tags: ['android', 'microg', 'fdroid', 'privacy', 'security']
---

A smartphone is arguably the most important device to keep private, we carry it all the time while it tracks our location, elevation, access the microphone and more. So how would someone take back control of their phone? It's a common question for people dipping their feet into the privacy world. 

In this article, we shall inspect methods of privatizing and limiting data collection as much as possible from your mobile devices.

The flavor of Android that I would recommend is [LineageOS](https://download.lineageos.org/) because it supports the most devices among all the custom roms available and does not pre-package Google Play Services or any of their apps by default.

# Before we start
It is expected that people who follow this guide to:

1. Be technically literate.
2. Have experience with the commandline.
3. Have experience with unlocking their phone, installing custom roms and recoveries.
4. Have a device that is unlockable and supported by [LineageOS](https://download.lineageos.org/).

# Unlocking your device.
Every device has a different unlocking method, and thus it is too broad to be covered in this article. Fortunately, LineageOS provides a step by step guide to unlocking devices that is supported by them.

Go to [LineageOS's website](https://download.lineageos.org/) > your_device > Installation instructions, and you shall see the guide.

# Setting up the recovery.
In general, you can just follow the instructions in the guide to install it. Here, we're gonna explain some good to know stuff about recoveries.

For one, there a few different recovery software, namely

### Team Win Recovery Project (TWRP)
[TWRP](https://twrp.me) is probably the most famous open source recoveries used in the custom rom world. It is advantageous if you want to experiment and install many different type of custom roms, it is also easier to use in the sense that you can install stuff without using a computer. 

However, it does not do signature verification of files it flashes, and automatic upgrades of OTAs can be quite wonky.

### LineageOS's own custom recovery. 
This is a minimalist recovery, who's job is to install LineageOS. 

The advantage of this is that 

1. **It does signature verification of packages it installs** which although the recovery itself isn't verified, it improves security a bit as an attacker now have to compromise the recovery instead of the LineageOS image file.
2. **Allow for automatic upgrades of LineageOS OTA** so you don't have to download and install the image manually during updates.

The disadvantage however is that

1. **It's not as flexible and easy to use as recoveries such as TWRP.** You have to have a computer and flash images using adb and fastboot commandline tools as oppose to just downloading the file one your device and install it through your recovery like you can when using TWRP.

It's images are usually listed alongside the ROM images on the LineageOS download page.

### Variants of LineageOS's custom recovery
Forks of LineageOS also often include the custom recovery that verifies their own images signed by them. Projects such as [Lineage4Microg](https://lineage.microg.org/) does this, so it's important that you also remember to change the custom recovery when you change your custom rom.

# Setting up LineageOS for use.
