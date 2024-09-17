---
title: "Dissecting Hi-Hive Community: Part I"
date: 2024-09-09
tags: ['react-native', 'android', 'javascript', 'reverse-engineering']
---

During COVID-19, my university debuted an attendance tracking called Hi-Hive Community. A lecturer would present a QR code, and the students will scan it to take attendance.

After using the application for a few years, I noticed that it is too bloated with unnecessary features, it's not very responsive, and important items are buried in unending levels of menus.

Thus, I have the idea of rebuilding the application using a Jetpack Compose, and more modern UI frameworks.

To do so, I first needed to figure how the API works.

# Preliminary Analysis
Through regular usage, I suspected that it uses some kind of web/mobile hybrid framework. 

Some symptoms I've noticed are:
1. **Slow and slightly off user interface** - There will be slight delays when tapping on elements, animation is slow, and low frame rates at times.
2. **Lack of uniformity in the components** - A lot of the components seems custom made. There are a mix of native elements with custom ones.
3. **Existence of web-view like pages** - Some of the functions are loaded through webview.

All in all it _feels_ like using a website. My gut feelings tell me that it's a web/mobile hybrid framework. Most likely using React Native.

# Unpacking the application
The only method to really confirm this is by cracking it open and have a look at the internals. I unpacked the .apk using `jadx` and immediately there are telltale signs of React Native.

For example, this image below shows configuration for a react native push notification library.

![Proof image of React Native presence](/assets/images/react_native_proof.png "Proof image of React Native presence")

Which upon further investigation, it appears that this library that was used: https://github.com/zo0r/react-native-push-notification.

# Further Investigation
