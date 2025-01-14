---
title: "Reverse Engineering Hi-Hive Community"
date: 2024-09-09
tags: ['react-native', 'android', 'javascript', 'reverse-engineering']
---

During COVID-19, my university debuted an attendance tracking called Hi-Hive Community. A lecturer would present a QR code, and the students will scan it to take attendance.

After using the application for a few years, I noticed that it was too bloated with unnecessary features, it's not very responsive, and important items are buried in unending levels of menus.

Thus, I have the idea of rebuilding the application using Kotlin, Jetpack Compose, and more modern UI frameworks.

To do so, I first needed to figure how the API works.

# Preliminary Analysis
Through regular usage, I suspected that it uses some kind of web/mobile hybrid framework. 

Some symptoms I've noticed are:
1. **Slow and slightly off user interface** - There will be slight delays when tapping on elements, animation is slow, and low frame rates at times.
2. **Lack of uniformity in the components** - A lot of the components seems custom made. There are a mix of native elements with custom ones.
3. **Existence of web-view pages** - Some of the functions are loaded through webview.

All in all it _feels_ like using a website. My gut feelings tell me that it's a web/mobile hybrid framework. Most likely using React Native.

# Dissecting the Application
The only method to really confirm this is by cracking it open and have a look at the internals. Here, I will be using version `2.3.1` of the application downloaded from [ApkPure](https://apkpure.com/hi-hive-community/com.slc.hihive.community).

## Unpacking the `.apk`
I unpacked the .apk using `jadx` and immediately there are telltale signs of React Native.

For example, this image below shows configuration for a react native push notification library.

![Image of configuration for a React Native library](./images/reactnative_proof.png "Image of configuration for a React Native library")

Which upon further investigation, it appears that this was the library used: https://github.com/zo0r/react-native-push-notification. It's a React Native library.

## Checking for presence of `index.android.bundle`

Furthermore, there is also presence of `index.android.bundle` as seen below.

![Image of index.android.bundle in file structure](./images/reactnative_bundle_proof.png "Image of index.android.bundle in file structure")

The `index.android.bundle` contains compiled and bundled JavaScript code for the application. Only React Native bundles this.

## Disassembling Hermes Bytecode
Immediately, I tried to open the bundle using a text editor but it showed an error. 

![Image of cannot open bundle](./images/cannot_open_bundle.png "Image of cannot open bundle")

After a google search, I found out that Hermes VM is the default compilation target for more recent versions of React Native. Which is confirmed using the `file` command:

![Image of file tool showing result](./images/file_command_result.png "Image of file tool showing result")

Fortunately, [P1 Security](https://www.p1sec.com/blog/releasing-hermes-dec-an-open-source-disassembler-and-decompiler-for-the-react-native-hermes-bytecode) released a tool to disassemble, and decompile Hermes bytecodes.

Using the tool ([hermes-dec](https://github.com/P1sec/hermes-dec/)), I disassembled the Hermes bytecode into somewhat readable pseudo-javascript.

![Image of not so readable pseudocode](./images/not_so_readable_pseudocode.png "Image of not so readable pseudocode")

However this is still super unreadable, due to it being converted from `javascript` to `obfuscated javascript` to `hermes bytecode` to `decompiled WASM` to `javascript pseudocode`. 

This meant that it's a "readable" version of the WASM instructions, not how the original code is structured.

Ideally, I want to get the version of the application that converted `javascript` to `obfuscated javascript` which should be more readable, and actually represent the original code.

## Back to the square one and cracking an older version
The `hermes-dec` project README states that React Native only started targetting the Hermes VM by default after React Native v0.70.

I thought, _what if the targetting of Hermes VM isn't actually on purpose by the developer?_ So I grabbed version `1.0.2` of the application which was released before React Native v0.70, and cracked it open.

As predicted! The `index.android.bundle` file is only obfuscated Javascript, not Hermes bytecode!

![Image of more readable bundle](./images/more_readable_bundle.png "Image of more readable bundle")

This is a more readable version of the `index.android.bundle` file compared to the one from version `2.3.1`, not by much but it's still better than nothing.

What this means is that I now have a base from which to reverse engineer the API used by the application.

# Reverse engineering and mapping the API
After a bit of searching, I found the following parts. It seems to be an object containing all the API domain, and paths.

![Image of API constants](./images/api_constants.png "Image of API constants")

> NOTE: Through comparing v1.0.2 and v2.3.1, I found out that `API_DOMAIN` is not pointing to `www.silverlakemobility.com` anymore, it is now pointing to `www.hi-hive.com`.

To implement the application, I only need to figure out some of the endpoints. 

The API endpoints relevant to us are:

- Authentication
- Listing classes, and attendance
- Scanning QR codes

With these, I can start mapping the needed endpoints and figure out what to pass to it.

## Cracking the authentication method
Looking through the rest, I found some snippets that shows how the API is being used. The snippet is the callback function for the "Login" button when it's pressed on the login page.

![Image of firebase restriction](./images/firebase_restriction.png "Image of firebase restriction")

It seems to be a bunch of async functions, and essentially what it does is:

1. Loads a Firebase Cloud Messaging (FCM) token from local storage.
2. Then use the token to encrypt the password using the `convertPBEWithMD5AndDES` function.
3. Then pass the resulting encrypted password as apart of the payload.

The application then calls the login endpoint with the following JSON payload structure:

`POST https://www.hi-hive.com/chat/api/preLogin/login`
```json
{
    "userId": "your user email",
    "password": "encrypted version of your password",
    "os": "Ios or Android",
    "token": "The FCM token used to encrypt the password."
}
```

I suspect what happens here is that there was a requirement to not send passwords over the internet in plain text even when secured by SSL. 

The token is then sent alongside the password, which I assume is used to decrypt it, and used to authenticate that the password was encrypted with a token related with the developer.

## Roadblock to putting together a 3rd party library
This puts a big roadblock on our goal. What effectively happens in the end is that the login method is made a lot harder to reverse engineer since you have to somehow generated FCM token (possibly related to the original developer).

One method to solve this is I can try to reverse engineer Google's FCM library, then figure out if it's possible to generate legit FCM tokens, and then check if authentication would work with that generated token. 

Until then, listing classes, listing attendance, and scanning QR codes is essentially unusable. It'll probably stay that way since I'm not very interested in reverse engineering Google's FCM SDK, especially since I'm graduating and won't be able to use the results myself.

# Other method (Network Monitoring)
Apart from inspecting the unobfuscated code, I've used network monitoring to figure out the various endpoints and their needed payload which brought similar results. 

This was achieved by:
1. Preparing the application for network monitoring by:
    - Cracking it open.
    - Editing the `AndroidManifest.xml` file so that it trust user added SSL Certificates.
    - Repacking, and installing it on a physical device.
2. Preparing the device by installing a third party SSL Certificate from CharlesProxy.
3. Monitoring the traffic of the device (and by proxy the application) using [CharlesProxy](https://www.charlesproxy.com/).

At the time, I was able to actually login using the tokens and encrypted password intercepted here which returned a session token, but I wasn't able to capitalize on it due to the same problems stated before.

# Conclusion
All in all, although I did not go all the way with reverse engineering Google's FCM SDK, it was an interesting experience reverse engineering a React Native application from my school.
