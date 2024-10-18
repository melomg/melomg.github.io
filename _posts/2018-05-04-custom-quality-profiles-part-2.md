---
title: "Using Custom Quality Profiles in SonarQube and SonarLint plugin — Part 2"
image:
  path: /assets/images/2018-05-04-post-image-1.webp
  thumbnail: /assets/images/2018-05-04-post-image-1-thumbnail.png
  caption: "'Do what is great, written on a computer monitor.' by [Martin Shreder](https://unsplash.com/@martinshreder) on [Unsplash](https://unsplash.com/)"
tags:
  - sonarqube
  - sonarlint
  - quality-profiles
date: 2018-05-04
layout: post
---
Hi folk! In [Part 1](/2018/5/3/custom-quality-profiles-part-1.html), I explained how to install, run 
[SonarQube](https://www.sonarsource.com/products/sonarqube/downloads/) and create a custom Quality Profile.
In this part, I will try to explain how to use created custom Quality Profile for our Android Project 
and even how to make it easier.

The only thing we need to do, to be able to analysis our code, is as simple as below snippet:

<script src="https://gist.github.com/melomg/25bf3cbb4d60c1bc3e85771b7c4bea8c.js"></script>

I have only used Java Analyzer but you can also add [other properties](https://docs.sonarsource.com/sonarqube/display/SONAR/Analysis+Parameters)
as you wish. You can also have a look on [my Nanodegree project](https://github.com/melomg/Nanodegree-PopularMovies/blob/master/app/build.gradle).
After syncing project, we now are able to run SonarQube for our project and for that 
we simply run below command in terminal under the project’s path. Since we choose our custom quality 
profile as default, the result will be tied with that rules.

```bash
./gradlew sonarqube
```

After build become successful, we can examine our project’s analysis results:

<img src="/assets/images/2018-05-04-image-1.webp" class="align-center" alt="Java Lint Result">

And even RCI: :))

<img src="/assets/images/2018-05-04-image-2.webp" class="align-center" alt="Rules Compliance Index">

There is more coming :) I’m sure if you are analyzing your code every time before sending them to Git origin,
you will fell knackered at last. In below part, I will try to save you from this feeling and 
tell a bit about SonarLint Plugin on Android Studio and how to use it.

## Binding SonarQube server by using SonarLint plugin

There is a cool plugin for SonarQube called [SonarLint plugin](https://www.sonarsource.com/products/sonarlint/).
Let's download it and give it a go. After update you can go to *"Android Studio>Preferences>Other Settings>SonarLint General Settings"*
and add a new SonarQube server which in my case http://localhost:9000/ as shown below:

<img src="/assets/images/2018-05-04-image-3.webp" class="align-center" alt="SonarLint General Settings">

Next button will take us to Token page and at there we need token which we can take from 
http://localhost:9000/account/security by clicking *“Create Token”*. After entering token now
we have successfully declared our SonarQube server. Only one thing remained, binding it! 🎊

**Make sure** you’ve updated the sonar features in SonarQube’s marketplace(SonarJava, FindBugs, etc.). Because SonarLint may require new plugins to bind remote server.
{: .notice--info}

<img src="/assets/images/2018-05-04-image-4.webp" class="align-center" alt="Bind to server">

<img src="/assets/images/2018-05-04-image-5.webp" class="align-center" alt="Search Project in server">

As we can see from the images left, it is easy peasy lemon squeezy :) We choose the server we created
and our project in the list. Now after closing settings page by clicking OK,
we are ready to run SonarQube via the custom quality profile we created just by selecting 
“SonarLint” tab and clicking green Run icon. We will see the result immediately and even the rule explanations. 🎊

<img src="/assets/images/2018-05-04-image-6.webp" class="align-center" alt="SonarLint finding - before deleting a rule">

### Bonus

How will we see changes if we updated our Custom Quality Profile at SonarQube server? 
For example, we decided to remove [“@Override” should be used on overriding and implementing methods](https://next.sonarqube.com/sonarqube/coding_rules?languages=java&open=squid%3AS1161)
rule (this is just to show an example, I don’t recommend removing this rule ;)),
we deactivated it and what’s next?

<img src="/assets/images/2018-05-04-image-7.webp" class="align-center" alt="deletion a rule in custom profile">

We only need to go to *“Android Studio>Preferences>Other Settings>SonarLint General Settings”* again 
and click **“Update Binding”**. The result is shown at below:

<img src="/assets/images/2018-05-04-image-8.webp" class="align-center" alt="after updating binding and running SonarLint">

Yes, hurrah we don’t see this rule again!! 🎊 🎊 🎊

A small note about SonarQube. It only improves our code quality by helping us to write better and 
clean code and even correcting the mistakes we have escaped mostly, but still we are all alone to think 
many other things while developing Android apps. Therefore, may the force be with you!! :)

<iframe src="https://giphy.com/embed/JDnaQ8qn0Myuk" width="480" height="274" style="border:none;overflow:hidden;display:block;margin:0 auto;" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a href="https://giphy.com/gifs/star-wars-george-lucas-JDnaQ8qn0Myuk"></a></p>
