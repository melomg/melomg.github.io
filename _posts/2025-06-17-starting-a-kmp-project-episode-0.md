---
title: "Starting a kmp project - Episode 0"
image:
  path: /assets/images/2025-06-17-post-image-2.png
  thumbnail: /assets/images/2025-06-17-post-image-2.png
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
[kmp-production-sample](https://github.com/Kotlin/kmp-production-sample), [KMP-App-Template](https://github.com/Kotlin/KMP-App-Template) on Github, KMP libraries on https://klibs.io/, they all again made me want to start experimenting on KMP.

I started with using the KMP app template but very soon I realized that it didn't answer a lot of my questions to begin with, therefore I converted my fork to another kotlin app template.
At the time of writing I myself don't know all the answers to these questions as well but step by step, hope to be able to find answers and share with you in this KMP journey of mine.

I'm an android developer for long, and the questions I came up with, relates very much to mobile development but since this is KMP journey, I hope that I can find the answers for other platforms as well.

And hopefully the KMP template of mine will try to help you get started with these points:

- Core
    - Logging
        - Error reporting
        - Analytics
        - Tracing
    - Network
    - Benchmarking
    - Build conventions
    - Flavours
    - Mocks
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
    - Pipelines
    - Releasing
    - Force updates
    - Dependency updater
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

I think each of these points deserve an article so I'll start with `Logging` in the next episode.

*Until then may the force be with you…* 🖖
