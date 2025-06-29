---
title: "Starting a kmp project - The one with static code analysis - Episode 2"
tags:
  - kmp
  - kotlin
  - lint
  - detekt
  - multiplatform
date: 2025-06-22
layout: post
---

### TL;DR for detekt

If you'd like to just see the implementation and move forward quickly, you can just check [my PR](https://github.com/melomg/KMP-Template/pull/25/files). Otherwise bear with me a couple of minutes for some decision points. 

### The story

Speaking from android background, we have couple of options like [detekt](https://detekt.dev/docs/intro), [ktlint](https://pinterest.github.io/ktlint/latest/) on top of Android's [lint tool](https://developer.android.com/studio/write/lint) for static code analysis and more (pmd, checkstyle etc...). Code formatters like [Spotless](https://github.com/diffplug/spotless) works well with `ktlint`. 
I'm not gonna compare `ktlint` and `detekt` in this article (it has been done many times in the past in different articles like in [Medium](https://medium.com/@SaezChristopher/detekt-vs-ktlint-2024-which-linter-is-the-best-for-an-android-project-d1b7585a0103) and in [ChatGPT](https://chatgpt.com/s/t_6860547358708191a1547bbf7ecdc616)), but both tools can work for Kotlin Multiplatform projects and allow analysis of platform-specific code (JVM, JS, Native).
I'm more used to working with `detekt` and like their suggestion to [tweak the rules](https://detekt.dev/docs/introduction/compose/) to work with Google's [Compose API Guidelines](https://android.googlesource.com/platform/frameworks/support/+/androidx-main/compose/docs/compose-api-guidelines.md).

But the problem is by default, `Detekt` looks for Kotlin and Java source codes within a standard Gradle project as mentioned [here](https://github.com/detekt/detekt/discussions/3467). You can probably see these discussions when you Google how to make `Detekt` work for a KMP project (by the time of writing);

- [how are you using Detekt on multiplatform projects...](https://slack-chats.kotlinlang.org/t/8082454/how-are-you-using-detekt-on-multiplatform-projects-these-day)
- [If I wanted to run detekt tasks on all KMP source ...](https://slack-chats.kotlinlang.org/t/12654956/if-i-wanted-to-run-detekt-tasks-on-all-kmp-source-sets-in-my)
- Detekt isn't able to detect issues on `iosMain` source set [https://github.com/detekt/detekt/issues/5609](https://github.com/detekt/detekt/issues/5609)
- Running detekt on JVM target (in KMP project) doesn't run against the common source set [https://github.com/detekt/detekt/issues/7073](https://github.com/detekt/detekt/issues/7073)
- There is no `detektCommonMain` task but there is `detektMetadataMain` as mentioned [here](https://github.com/detekt/detekt/issues/3665)

For the first 2 issues, I agree that it could be nice to have a common task with no configuration need. For the 3rd and 4th issues, it looks like a source set issue to me. 
For the last one, it looks like a nice suggestion but it usually gave issues for generated sources to me (although I know I can ignore paths). It's interesting that a small task like implementing `Detekt` can be such a headache at least by the time of writing (maybe there is a nicer solution but Google search makes me think that community is having a same trouble).

Looking at the KMP samples in jetbrains [here](https://www.jetbrains.com/help/kotlin-multiplatform-dev/multiplatform-samples.html), none of them had static code analysis tool for `detekt` (Maybe I missed some, let me know if there is a nice example please).
[DroidconKotlin](https://github.com/touchlab/DroidconKotlin/blob/main/build.gradle.kts) example has a nice setup for `ktlint` but they only support for iOS and Android atm.
Even [kotlinconf-app](https://github.com/JetBrains/kotlinconf-app/) didn't have any static code analysis tool which surprised me a lot. 

Considering all those, I could've also chosen to go with separate static code analysis for separate platforms (Swift lint for Swift) like mentioned in this [nice article](https://diamantidis.github.io/2019/09/01/kotlin-multiplatform-project-code-styling-for-ios-and-android). But considering that I'm planning on writing almost no Swift, I decided to stay with one static analysis tool on top of the Lint. 
At the end, I decided to include all the source sets myself by using `source.setFrom(...)` configuration feature of `Detekt` for each sub gradle project. This reports the issues multiple times but so far looked pretty reliable.

If you'd like to see how I implemented this, you can check [the PR](https://github.com/melomg/KMP-Template/pull/25/files).

### Little remark
```yaml
  - name: Check detekt
    run: ./gradlew detekt

  - name: Check lint
    run: ./gradlew lintDebug
```

I could have just used `./gradlew check` task in CI since `detekt` and `lint` tasks does also run when `check` task run but imo `check` task does a lot. But if you want to follow standard Gradle lifecycle practices, and you're OK with less fine-grained control, then `check` task is not bad.
I wanted to optimize CI time and only run certain checks in certain situations like running lint only on debug build type, therefore I separated `detekt` and `lint` tasks and run only them which saved me sometime. I believe it also gives faster feedback and better visibility into where a failure happened.
But that might be just [how I eat the yogurt](https://tureng.com/en/turkish-english/her%20yi%C4%9Fidin%20bir%20yo%C4%9Furt%20yiyi%C5%9Fi%20vard%C4%B1r).

One last suggestion, I came across with a [nice article here](https://medium.com/@santimattius/kmp-essential-tools-and-plugins-for-kotlin-multiplatform-application-development-6ffcccdef6a8) which mentions nice tools to have for a KMP project. 

We can now tick one/two checks in the list below and rest remains 🎉;
- Core
    - ✅&ensp;Dependency management: Renovate
    - ✅&ensp;Lint
    - ✅&ensp;Static code analysis: Detekt
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
