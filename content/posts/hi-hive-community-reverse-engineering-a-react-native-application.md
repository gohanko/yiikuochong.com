---
title: "Hi-Hive Community: Reverse engineering a React Native application"
date: 2022-06-10
draft: true
tags: ['react-native', 'android', 'javascript', 'reverse-engineering']
---

So, at my university we use an application called Hi-Hive Community to take our attendance. You essentially scan a QR code presented by the lecturer as proof that you attended.

It does what it needs to do but it's too slow, has way too many features, and bury important parts in levels of menus. Thusly, I wondered if I could build an attendance taking application with only the important features. Then I wondered how does the original application worked? what API technology do they use? and so on and so forth. You know what I mean.

Thus! The first step to doing this is to reverse engineer the Hi-Hive Community application.

# Preliminary Analysis
Through simply using the application, I suspected that it uses some kind of web/mobile hybrid framework. The symptoms are

1. **Slow and slightly off user interface** - There will be slight delays when tapping on elements, animation is slow, and low frame rates at times.
2. **Lack of uniformity in the components** - A lot of the components seems custom made. There are a mix of native elements with custom ones.
3. **Existence of web-view like pages** - Some of the functions are loaded through web pages.

All in all it __feels__ like using a website. My gut feelings tell me that it's a web/mobile hybrid framework. Most likely using React Native.

# Cracking it open
Without another method to confirm what I've assumed, I decided to trust my instinct and proceed with attempting to unpack and decompile the application.

The tools I used were:

- apktool
- jadx, jadx-gui
- zipalign & apksign (in the form of [convert-apk](https://github.com/mathieures/convert-apk))
- [P1sec/hermes-dec](https://github.com/P1sec/hermes-dec)
- [bongtrop/hbctool](https://github.com/bongtrop/hbctool)

## Unpacking the application
Using apktool unpack the apk using `apktool d hi-hive-46-release.apk`

![alt text](/assets/images/apktool_unpacking.png "Decompiling apk with apktool")