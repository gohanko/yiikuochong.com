---
title: "Dissecting Hi-Hive Community: Part I"
date: 2024-09-09
tags: ['react-native', 'android', 'javascript', 'reverse-engineering']
---

During COVID-19, my university debuted an attendance tracking called Hi-Hive. A lecturer would present a QR code, and the students will scan it to take attendance.

Like the previous paper based attendance solution, students just asked their friends to send them the QR code to skip class. Nontheless, that's not the issue we'll be discussing today.

After using the application for a few years, I noticed that the original application is too bloated with unnecessary features, it's too slow, and important items are buried in unending levels of menus.

Thus, I have the idea of rebuilding the application using a Jetpack Compose, and I first needed to figure how the API works.

# Preliminary Analysis
Through regular usage, I suspected that it uses some kind of web/mobile hybrid framework. 

Some symptoms I've noticed are:
1. **Slow and slightly off user interface** - There will be slight delays when tapping on elements, animation is slow, and low frame rates at times.
2. **Lack of uniformity in the components** - A lot of the components seems custom made. There are a mix of native elements with custom ones.
3. **Existence of web-view like pages** - Some of the functions are loaded through webview.

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