---
title: "Starting a kmp project - Episode 0"
image:
  path: /assets/images/2025-06-17-post-image.png
  thumbnail: /assets/images/2025-06-17-post-image.png
  caption: "Photo by [Marc Reichelt](https://unsplash.com/@mreichelt?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) on [Unsplash](https://unsplash.com/)"
tags:
  - kmp
  - kotlin
  - multiplatform
date: 2025-06-17
layout: post
---

I've been wanting to experiment with KMP but it always looked too big to start. 
[The Kotlin Conf 2025](https://www.youtube.com/watch?v=F5NaqGF9oT4), the projects like [Kotlin Conf app](https://github.com/JetBrains/kotlinconf-app), 
[kmp-production-sample](https://github.com/Kotlin/kmp-production-sample), [KMP-App-Template](https://github.com/Kotlin/KMP-App-Template) on Github, KMP libraries on [klibs](https://klibs.io/), they all triggered my desire to experiment with KMP.

I initially used the KMP app template, but soon realized it didn't address many of my initial questions. Then, I decided to convert my fork to a different [Kotlin KMP template](https://github.com/melomg/KMP-Template). 
Since KMP app template supports only android and iOS, in my KMP template, I used Intellij's New project template for server, wasm and desktop apps.  
At the time of writing I myself don't know all the answers to these questions as well but step by step, hope to be able to find answers and share with you in this KMP journey of mine.

I've been an Android developer for a long time…, and the questions I came up with, relates very much to mobile development but since this is KMP journey, I hope that I can find the answers for other platforms as well.

And hopefully the KMP template of mine will try to help you get started with these points:

- Core
    - Dependency management
    - Logging
        - Error reporting
        - Analytics
        - Tracing
    - Network
    - Benchmarking
    - Build conventions
    - Flavours
    - Mocks
    - Test fixtures
    - Build info
    - Preferences
    - Storage
    - DI
    - Feature flags (local & remote)
    - Deep linking
    - Push notifications
    - TimeProvider
    - Local Formatters
    - Coroutine Dispatchers & Test helper
    - Lint
    - Static code analysis
    - Unit testing
    - Test coverage
    - Obfuscation & Shrinking
    - Pipelines
    - Releasing
    - Force updates
- UI
    - Design system
    - Gallery App
    - Navigation
    - Baseline profiles
    - Compose compiler metrics
    - Previews
    - Network image loading
    - supportsDynamicTheming
    - Status bar color changing
    - App settings with Resource Environment
        - l10n
        - i18n
    - Testing
        - UI Testing
        - Compose Screenshot testing

I think each of these points deserves an article so I'll start with `Dependency management` in the next episode.

*Until then may the force be with you…* 🖖
