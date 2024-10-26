---
title: "Basic things to look at in Android app security"
image:
  path: /assets/images/2024-10-26-post-image.webp
  thumbnail: /assets/images/2024-10-26-post-image.webp
  caption: "Photo by [Khara Woods](https://unsplash.com/@kharaoke?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) on [Unsplash](https://unsplash.com/)"
tags:
  - android
  - security
date: 2024-10-26
layout: post
---

There is so much to say and so many edge cases when it comes to mobile app security. And every year, there will be more
of those edge cases to remember.
There is a nice [security checklist](https://developer.android.com/privacy-and-security/security-tips) from Google.
But in this post, I'd like to share a short checklist on top of that when I review an app in terms of security;

1. [Supply chain attacks against Gradle](#1--supply-chain-attacks-against-gradle)
2. [Outdated dependencies](#2--outdated-dependencies)
3. [Obfuscation](#3--obfuscation)
4. [Preventing Dev/Sandbox code in release builds](#4--preventing-devsandbox-code-in-release-builds)
5. [Cleartext traffic](#5--cleartext-traffic)
6. [Certificate pinning](#6--certificate-pinning)
7. [Checking app process for encryption key materials](#7--checking-app-process-for-encryption-key-materials)
8. [Securing authentication processes against tools like Frida](#8--securing-authentication-processes-against-tools-like-frida)
9. [Unprotected Storage](#9--unprotected-storage)
10. [Backup rules](#10--backup-rules)
11. [Exported android components](#11--exported-android-components)
12. [Biometric authentication](#12--biometric-authentication)

### 1- Supply chain attacks against Gradle

Gradle wrappers, distributions and 3rd party dependencies are known for their vulnerability to supply chain attacks.
Knowing where you are fetching these from is important.
A real library you'd like to fetch might exist in more than one repository and since the order matters
in your gradle repositories declaration, you might be using a malicious library without any notice.
Well known stories ["A Confusing Dependency"](https://zsmb.co/a-confusing-dependency/),
["Be Careful With Your Gradle Repository Declarations"](https://jasonatwood.io/archives/2033) and gradle blog
about [Protecting Project Integrity](https://blog.gradle.org/project-integrity) explains more. There was
also [a nice presentation in droidcon](https://www.droidcon.com/2024/06/13/how-to-stop-the-gradle-snatchers-securing-your-builds-from-baddies-3/)
about this.

### 2- Outdated dependencies

This one is trivial, but it’s good practice to keep the library versions up to date since they mostly include bug fixes,
hot fixes which might lead to exploitation when not having the up-to-date libraries.
Especially for the UI form components and network libraries like OkHttp.
It's also good practise to keep android sdk level up-to-date since most new sdk levels are using new cool & secure
features.
Like API 33 introduces [APK Signature Scheme v3.1](https://source.android.com/docs/security/features/apksigning/v3-1),
or API 34 introduced [Credential Manager](https://developer.android.com/identity/sign-in/privileged-apps) and some
improvements for WebView. Some of these features can’t be backported and old versions of android cannot still make use
of these features but devices with the new android versions hopefully might be in a safer place with these new features.

### 3- Obfuscation

It is also important to enable [obfuscation](https://developer.android.com/build/shrink-code) for release builds.
Non-obfuscated apk will mean anyone has the apk can easily
figure out what’s happening inside the app and can make the exploitation easier since it will be much easier for hackers
to understand how the app works when they use a decompiler/disassembler tools. Proguard obfuscation will make static
attacks harder but overall proguard isn’t a security solution.
On the other hand, dexguard can also make dynamic attacks
harder. [Proguard vs. Dexguard](https://www.guardsquare.com/blog/dexguard-vs-proguard) shows a nice comparison between
them.

### 4- Preventing Dev/Sandbox code in release builds

We create functionalities for easier testing and easier app usage in debug/dev builds.
It is essential that these functionalities are never included in the release builds.
This can be achieved by adding these functionalities to dedicated build types only or using no-op libraries
or in `HttpLoggingInterceptor` case, setting the level to `NONE` in release builds.

### 5- Cleartext traffic

> Allowing cleartext network communications in an Android app means that anyone monitoring network traffic can see and
> manipulate the data that is being transmitted.
>
Src: [https://developer.android.com/privacy-and-security/risks/cleartext-communications](https://developer.android.com/privacy-and-security/risks/cleartext-communications)

Instead, prefer using HTTPS servers that comes with TLS which encrypts the communication happening between app and
server.

### 6- Certificate pinning

This is a tricky one. If you'd ask me 7 years ago if it is a good idea to do certificate pinning, looking at these
articles,
[Android Security: SSL Pinning](https://appmattus.medium.com/android-security-ssl-pinning-1db8acb6621e),
[Android SSL Pinning Using OKHttp](https://medium.com/@develodroid/android-ssl-pinning-using-okhttp-ca1239065616),
I'd say no if you aren't required to do so as it would bring more issue than its advantages like handling the
compromises, expiry dates etc.
But after Android 7 introducing Network Security Config and improvements in further Android sdks,
it is way easier and more secure to use certificate pinning. Of course,
fundamentals still stay the same, as the more to the root certificate pinning, the less secure the it gets.

### 7- Checking app process for encryption key materials

It's a good thumbs-up not to store any sensitive keys in the app but also in the codebase.
Anyone can access [your apk](https://www.apkmirror.com/) and do reverse-engineer to access those keys or even
repositories can be leaked. Android suggests using the Keystore system so that key material never enters the app process
to prevent extraction of keys. But we need to keep it in our minds that even Keystore might not be 100% secure as not
all devices support TEE or has a secure hardware module. If it's for network calls that a local encryption system
created, than it’s safer to make network requests with access tokens & refresh tokens instead.

### 8- Securing authentication processes against tools like Frida

Even though it's explained well
[in security tips](https://developer.android.com/privacy-and-security/security-tips#secure-auth)
from Google, I'd like to add a couple of few notes here. With tools like [Frida](https://frida.re/), anyone can hook
into the app process so it's a red flag if you've a simple logic like `isLoggedIn` function and
depend on this to give users full power afterward. Encrypted Biometric functionality
([Class 3](https://developer.android.com/reference/android/hardware/biometrics/BiometricManager.Authenticators#BIOMETRIC_STRONG))
and [2FA](https://en.wikipedia.org/wiki/Multi-factor_authentication) with session tokens should be used to let user take
actions on changing private data such as changing password.

### 9- Unprotected Storage

There is a nice article about this
[here](https://proandroiddev.com/unpacking-android-security-part-2-insecure-data-storage-71f35107052a).
`SharedPreferences` can be exploited when backup rules aren't defined well or when device is rooted.
`EncryptedSharedPrefs` API can be used with keystore keys and key attestation but even then, it’s still not secure. Any
local storage options should be treated as not secure.
Databases can be encrypted with `SQLCipher` and opt out from backup with backup rules but even then,
it won’t be fully protected. It’s safer not to store any sensitive data in the local storage
and rely on protected servers with remote authentication instead.

### 10- Backup rules

[When enabling backup](https://developer.android.com/identity/data/autobackup#EnablingAutoBackup), it is important to
check that if sensitive data is excluded via backup rules. Otherwise, with tools
[like this](https://github.com/nelenkov/android-backup-extractor), sensitive data can be leaked.

### 11- Exported android components

Usage of permissions when content providers exported is important.
Android [docs](https://developer.android.com/privacy-and-security/risks/access-control-to-exported-components)
and [this article](https://blog.oversecured.com/Android-Access-to-app-protected-components/) covers more cases.

I tried to cut the list here since to gather all the thoughts took a lot of time but many more to add in the future…
<iframe src="https://giphy.com/embed/WjAkQjz7h9ESA" width="480" height="250" style="border:none;overflow:hidden;display:block;margin:0 auto;" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/doctor-who-funny-matt-smith-WjAkQjz7h9ESA"></a></p>

*Hope you enjoyed reading… Stay save* 🖖

## References

- [Security checklist](https://developer.android.com/privacy-and-security/security-tips)
- [Be Careful With Your Gradle Repository Declarations](https://jasonatwood.io/archives/2033)
- [A Confusing Dependency](https://zsmb.co/a-confusing-dependency/)
- [Protecting Project Integrity](https://blog.gradle.org/project-integrity)
- [How to stop the 'Gradle Snatchers'](https://www.droidcon.com/2024/06/13/how-to-stop-the-gradle-snatchers-securing-your-builds-from-baddies-3/)
- [Proguard vs. Dexguard: An Overview](https://www.guardsquare.com/blog/dexguard-vs-proguard)
- [Shrink, obfuscate, and optimize your app](https://developer.android.com/build/shrink-code)
- [Unpacking Android Security: Part 2 — Insecure Data Storage](https://proandroiddev.com/unpacking-android-security-part-2-insecure-data-storage-71f35107052a)
- [Permission-based access control to exported components](https://developer.android.com/privacy-and-security/risks/access-control-to-exported-components)
- [Android: Access to app protected components](https://blog.oversecured.com/Android-Access-to-app-protected-components/)