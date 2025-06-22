---
title: "Starting a kmp project - The one with dependency management - Episode 1"
tags:
  - kmp
  - kotlin
  - multiplatform
date: 2025-06-22
layout: post
---

Considering my android background, there has been a couple of dependency updaters I've had experience with it. The famous ones were:
- [Gradle Versions Plugin](https://github.com/ben-manes/gradle-versions-plugin)
- [Renovate](https://docs.renovatebot.com/) and nice onboarding documentation for it [here](https://docs.renovatebot.com/getting-started/installing-onboarding/).

And with recent development, even Gemini's [Version Upgrade Agent](https://www.youtube.com/watch?v=ubyPjBesW-8) might be a nice alternative.

But I find renovate's config super powerful. To be able to group version updates into single PR, and to be able to define the PR title, to decide when, how frequently it should be triggered, auto approve support, rate limiting PRs, etc… these are all super helpful.
For a KMP project with dependencies across different ecosystems (Gradle for JVM/Android, Swift Package Manager or CocoaPods for iOS via Gradle plugins), Renovate's fine-grained configuration control is more than I ask at the moment.
In the project itself, I decided to group androidx, jetbrains and ktor dependencies into their own group but you can play with any configuration that is defined [here](https://docs.renovatebot.com/configuration-options/).
I myself made some wrong configuration at [first PR](https://github.com/melomg/KMP-Template/pull/2/files) and updated [here](https://github.com/melomg/KMP-Template/pull/13/files) and probably will update the configuration again if I find it not logical.

But so far, we can now tick one check in the list below and rest remains 🎉;
- Core
    - ✅&ensp;Dependency management: Renovate
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

Hope to see you in the next episode.

*Until then may the force be with you…* 🖖
