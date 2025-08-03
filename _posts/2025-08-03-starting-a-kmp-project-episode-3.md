---
title: "Starting a kmp project - The one with build info - Episode 3"
image:
  path: /assets/images/2025-06-17-post-image.png
  thumbnail: /assets/images/2025-06-17-post-image.png
  caption: "Photo by [Marc Reichelt](https://unsplash.com/@mreichelt?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) on [Unsplash](https://unsplash.com/)"
tags:
  - kmp
  - kotlin
  - build-config
  - multiplatform
date: 2025-08-03
layout: post
---

In Android apps, we need the build info mainly for 3 purposes;

1. To better help customers (if there is a customer support, then customers can see which app
   version they use, or even which platform version they use since not all users know how to see
   those information and it is usually different/changing where to see this information between
   platforms).
2. To ship the app builds with secrets and environments like staging or acceptance looking APIs to
   do the testing without affecting prod
   systems. [Shopify's](https://www.shopify.com/blog/staging-vs-production)
   and [DesignRush's](https://www.designrush.com/agency/software-development/trends/staging-environment)
   blogs are nice to read to understand why we need different environments.
3. To control what the app does (e.g. Enriched Logging in debug builds, enabling option to use mock
   data in debug builds, disabling the testCoverage task in debug build for performance, disabling
   leakCanary in test automation, enabling Analytics in prod builds, etc...).

Therefore build info;

1. Should be able to provide different configuration for each build type (app version, name, secrets
   like API
   keys can be different per each build type).
2. Should be reachable at runtime.
3. Should be injectable in CI (If not all, then at least some info like build secrets, or values for
   an automated versioning system).

With this in mind, this article explores implementing the following model:

```kotlin
interface Platform {
    val appVersionCode: Int
    val appVersionName: String
    val platformVersionName: String // Android version | iOS version | JVM version | Web vendor etc... 
    val buildType: BuildType
    val isDebuggable: Boolean
}

enum class BuildType {
    DEBUG,
    STAGING,
    RELEASE,
}
```

### Android side:

The Android Gradle plugin simplifies many of these aspects. Talking from a simple example will make
it easier to explain.

```kotlin
val properties = gradleLocalProperties(rootDir, providers)
val stagingAPIKey: String = properties["api.key.staging"]
val prodAPIKey: String = properties["api.key.prod"]

buildTypes {
    debug {
        isDebuggable = true
        ...
    }
    release {
        isDebuggable = false
        ...
    }
}
productFlavors {
    create("staging") {
        applicationIdSuffix = ".staging"
        versionNameSuffix = "-STAGING"

        buildConfigField("String", "API_KEY", stagingAPIKey)
    }
    create("prod") {
        buildConfigField("String", "API_KEY", prodAPIKey)
    }
}
```

1. To be able to provide different configuration for each build type or flavor, android plugin has
   `buildTypes` and `productFlavors` feature where you can provide the different configuration for
   the same variables. So whether we are in `staging` or `prod` flavor, we can access the related
   `API_KEY` instance.
2. `buildConfigField` enables us to access the values in runtime via generated `BuildConfig`
   file, and then we can read our `API_KEY` in runtime like `BuildConfig.API_KEY`.
3. For getting secrets or control flags in gradle, we can first store them in `local.properties` or
   any properties file that you know for sure will NOT end up in your version control system (Git),
   and then
   we can read those properties to provide them in runtime with `buildConfigField` in gradle as
   mentioned earlier.

### iOS side:

I'm less familiar with the iOS side, but I found a
a [nice article in medium](https://medium.com/@niranjanky14/understanding-build-schemes-build-configurations-and-xcconfig-files-in-xcode-372c344d8805)
for iOS configuration. They've a similar implementation.
It's not called `.properties` file but `.xcconfig` to store properties which supports different
build types. It's not called build type but configuration and etc... Then, it is a matter of
accessing the build info in runtime with `Bundle.main.infoDictionary?["API_KEY"]`.
But even though those platforms have similar implementation, because we use gradle in the KMP
project, I couldn't find a way to implement build types and flavors in a way that Android can with
the gradle plugin.

### Desktop side:

[Another article](https://blog.joetr.com/adding-debug-checks-to-your-compose-multiplatform-project-android-ios-and-desktop)
mentions that we can use System Properties to define the local properties to pass them to the
application so the build info can be accessed in runtime later. But again no first-class gradle
support like Android has.

### Wasm Web side:

In **web projects**, the story is a bit different since the concept of "build type" or "product
flavor"
works differently than Android/iOS.
We're not distributing an app binary to users; instead, we're pushing updates to a server.
But **versioning is still essential** for caching, tracking changes, debugging, and CI/CD workflows.

I then decided to check other web frameworks to see how they handle these;

- Next.js web apps are making use
  of [Environment variables](https://nextjs.org/docs/pages/guides/environment-variables) so that
  depending on flavor (environment), Next.js can load the properties from files like
  `.env.production`, `.env.development` etc...
- Spring web apps are making use
  of [Profiles feature](https://www.baeldung.com/spring-profiles#4-jvm-system-parameter) and
  `application.properties` to set app properties like the name of the app;

<img src="/assets/images/2025-08-03-image-1.png" class="align-center" alt="application.properties file">

Looks like again no first-class gradle support like Android has.

### First design decision - Provide platform implementation for each platform separately

- Changes are [in this PR](https://github.com/melomg/KMP-Template/pull/30/files).

Considering that only Android has first-class gradle plugin support for build types and flavors,
I've first decided that each platform would have it's own implementation. Although using
actual/expect on koin modules doesn't guarantee the module cohesion as stated in
the [Koin Android Makers talk](https://www.youtube.com/watch?v=vYqJlN0rJL8&t=1024s), it made things
easier for me, like providing Android context in `AndroidPlatform` so that I can check if app is
debuggable. That meant, even though there are different sources for each platform, there is still
a single source of truth within each specific platform (For instance I should be able to trust
Android's BuildConfig and iOS's NSBundle).

---

On Android side, I can fully provide `Platform` model with the first-class support.

| App Version Code | App Version Name | Platform Version Name | Build type  | Is debuggable   |
|------------------|------------------|-----------------------|-------------|-----------------|
| BuildConfig      | BuildConfig      | Build.VERSION         | BuildConfig | ApplicationInfo |

#### Little remark for Android

For the first design decision, I've chosen `staging` **buildType** because there is no acceptance or
staging environments for the APIs that I'm gonna use, and I'd like to use it to see how dress
rehearsal goes via Firebase Distribution just before shipping the apps to their own stores.
I wanted to keep the assembly process simple (I can ship the android app with `assembleStaging` and
`assembleRelease`).
Currently my `staging` buildType is almost similar to `release` buildType (it is
`initWith(getByName("release"))`), except package, app name, version and signing config is
different.
And I treated `release` build type as prod build where it is minified and obfuscated with prod
signing config.

But if you are working on a serious app, you might want to also enrich it with Flavor. For instance
if you'd like to debug the prod app, you can create `prod`, `staging` flavors. Then you have an
access to `prodRelease`, `prodDebug`, `stagingRelease`, `stagingDebug` builds which gives you more
options to do more.

--- 

On iOS side, I can use `NSBundle` to access configuration and scheme info. I can also use
`NativePlatform` to see if it's a debug binary.

In summary, except Build Type, I was able to provide `Platform` model with a first-class
support.

| App Version Code | App Version Name | Platform Version Name | Build type | Is debuggable  |
|------------------|------------------|-----------------------|------------|----------------|
| NSBundle         | NSBundle         | UIDevice              | ❌          | NativePlatform |

--- 

For web, AI suggested I could use `hostname` of `browser.window` to decide if app is in debug
mode, or for platform version name `userAgent` of `window` object, but there is no way to get
`appVersionCode` and `appVersionName` from a platform specific object like those Android and iOS
provide (BuildConfig/NSBundle).

But again there is no first-class gradle support here.

| App Version Code | App Version Name | Platform Version Name  | Build type | Is debuggable          |
|------------------|------------------|------------------------|------------|------------------------|
| ❌                | ❌                | kotlinx.browser.window | ❌          | kotlinx.browser.window |

--- 

For desktop, the `isDebugBuild` logic AI suggested didn't work for me. Maybe I'm doing something
wrong here but again like web, there is no natural way to get `appVersionCode` and
`appVersionName`.

| App Version Code | App Version Name | Platform Version Name | Build type | Is debuggable |
|------------------|------------------|-----------------------|------------|---------------|
| ❌                | ❌                | System                | ❌          | ❌             |

#### Back to the build info topic

And here the problem starts, how to get this build info in runtime for each platform for each build
types (configurations as iOS calls it).

Looking
at [these suggested KMP samples by Jetbrains](https://www.jetbrains.com/help/kotlin-multiplatform-dev/multiplatform-samples.html),
I couldn't find any example that make use of Build Type.

Though I could find 2 build config plugins that kinda works with flavors (but not build types);

1. Suggested KMP samples that
   uses [buildconfig plugin](https://github.com/gmazzo/gradle-buildconfig-plugin);
    - [https://github.com/fethij/Rijksmuseum/blob/main/core/network/build.gradle.kts](https://github.com/fethij/Rijksmuseum/blob/main/core/network/build.gradle.kts)

2. Suggested KMP samples that uses [BuildKonfig](https://github.com/yshrsmz/BuildKonfig);
    - [https://github.com/joreilly/Confetti/blob/main/shared/build.gradle.kts](https://github.com/joreilly/Confetti/blob/main/shared/build.gradle.kts)
    - [https://github.com/xxfast/NYTimes-KMP/blob/main/app/build.gradle.kts](https://github.com/xxfast/NYTimes-KMP/blob/main/app/build.gradle.kts)
    - [https://github.com/VictorKabata/Notflix/blob/main/composeApp/build.gradle.kts](https://github.com/VictorKabata/Notflix/blob/main/composeApp/build.gradle.kts)

Both generate `BuildConfig` for all platforms. But it's still not like the buildType feature of
Android Gradle plugin, where you have access to all build types by default.

Overall, I could fulfill the Platform interface fully only in Android and iOS platforms. In web and
desktop platforms, I would have needed app config files per build type (environment, configuration)
and different run configurations to use related app config, which meant losing the single source of
truth.

### Second design decision - Provide platform implementation from app properties via buildkonfig plugin

- Changes are [in this PR](https://github.com/melomg/KMP-Template/pull/41/files).

The disadvantages of the first design decision led me to think of using only app config files (
application.properties) as a source of truth
(I think it's a known concept on all platforms, like `.env.production` on the web, 
`application.properties` in Spring Boot, or `.xcconfig` files on iOS etc...). Therefore, 
decided to create a fake build type called `effectiveBuildType` (in order not to confuse it with
android gradle plugin's build type field) where I would use it to access the right app properties
for each platform and provide them at runtime via the buildkonfig plugin.

To determine the `effectiveBuildType` was a bit a challenge. Somehow, providing it
via [gradle project properties](https://docs.gradle.org/current/userguide/build_environment.html#setting_a_project_property)
like so `./gradlew :composeApp:embedAndSignAppleFrameworkForXcode -PeffectiveBuildType=debug` didn't
work for me.

Therefore, I decided to run a Gradle task (`updateEffectiveBuildType`) with `BeforeRunTask` feature
where it would update the `effectiveBuildType` in a `local.properties` file based on the run
configuration before running the run configuration for the selected platform.
That meant, I had to create run configuration for all platforms and for all build types.
But once created, I could run these configurations, which would first update `effectiveBuildType` in `local.properties`.
Then, when running the app, it would read `effectiveBuildType` to provide the correct app properties for the Platform data class.

The disadvantage of this was that I had to create a lot of run configurations even for android and
now I couldn't depend on Android Studio IDE's Build Variant selection, which, as an Android developer, 
I was very used to.

### Third design decision - Provide platform implementation from detected Android build type or system environment

- Changes are [in this PR](https://github.com/melomg/KMP-Template/pull/42/files).

Since I was used to Android Studio IDE's Build Variant selection a lot, it was a shame not being
able to use it anymore.
[This article](https://sujanpoudel.me/blogs/managing-configurations-for-different-environments-in-kmp/) 
made me think it might still be possible to use Build variant selection.
This led me to improve on top of the 2nd design solution where providing the `effectiveBuildType`
from gradle's [startParameter](https://docs.gradle.org/current/kotlin-dsl/gradle/org.gradle/-start-parameter/index.html)
for Android (so that I could use the Build variant selection), or from iOS or from System
rather than providing the `effectiveBuildType` via the `updateEffectiveBuildType` **BeforeRunTask**.
In the article itself, somehow it's only using `getAndroidBuildVariantOrNull` and `System.getenv()`
but for me, I wasn't able to provide the `effectiveBuildType` from Build settings as a system
environment.
So I had to use iOS's Configuration like `System.getenv("CONFIGURATION")`. You could also say that I
could provide the `effectiveBuildType` on iOS as a system environment by exporting it in the Build phase,
but that also didn't work for me.
Feel free to make suggestions that could work. This was the version I tried;

```bash
...
export EFFECTIVE_BUILD_TYPE=\"$EFFECTIVE_BUILD_TYPE\"

cd "$SRCROOT/.."

./gradlew :composeApp:embedAndSignAppleFrameworkForXcode
```

Overall it took a lot of effort and many unsuccessful attempts since my knowledge of iOS/Web/Desktop
wasn't extensive. And the 3rd design decision is still not good as I still can't count on IDE's Build
Variant selection on iOS/Web/Desktop platforms.
But since I'll mainly work on Android emulators when designing and sometimes check how it looks on
other platforms, it was okay to start with.
I'm also surprised that many suggested KMP apps don't even have a build type or environment setup.
I'll update this if I can come up with a better solution in the future.

I'm also listing the articles and documents I took into account in the case that it helps you to
make a better decision.

#### References;

- [Managing configurations for the different environments (eg: staging, prod) in Kotlin Multiplatform](https://sujanpoudel.me/blogs/managing-configurations-for-different-environments-in-kmp/)
- [Setting a project property](https://docs.gradle.org/current/userguide/build_environment.html#setting_a_project_property)
- [Understanding ⚙Build Schemes, Build Configurations, and .xcconfig Files in Xcode](https://medium.com/@niranjanky14/understanding-build-schemes-build-configurations-and-xcconfig-files-in-xcode-372c344d8805)
- [Adding "Debug" Checks To Your Compose Multiplatform Project: Android, iOS, and Desktop](https://blog.joetr.com/adding-debug-checks-to-your-compose-multiplatform-project-android-ios-and-desktop)
- [Kotlin Multiplatform samples](https://www.jetbrains.com/help/kotlin-multiplatform-dev/multiplatform-samples.html)
- [How to use environment variables in Next.js](https://nextjs.org/docs/pages/guides/environment-variables)
- [Spring Profiles](https://www.baeldung.com/spring-profiles#4-jvm-system-parameter)
- [Configuring the Build Environment](https://docs.gradle.org/current/userguide/build_environment.html)
- [Staging vs. Production: The Final Stages of Web Development](https://www.shopify.com/blog/staging-vs-production)
- [Staging Environment: Software Testing in Staging Explained](https://www.designrush.com/agency/software-development/trends/staging-environment))

Imo this was one essential topic to get started with KMP and hope to get feedbacks to make this KMP
journey better 🎉 And an update on the **Checklist**;

- Core
    - ✅&ensp;Dependency management: Renovate
    - ✅&ensp;Lint
    - ✅&ensp;Static code analysis: Detekt
    - ✅&ensp;Build info
    - Logging
        - Error reporting
        - Analytics
        - Tracing
    - Network
    - Benchmarking
    - Build conventions
    - Flavors
    - Mocks
    - Test fixtures
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
