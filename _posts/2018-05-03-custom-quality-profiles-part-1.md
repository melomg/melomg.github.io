---
title: "Custom Quality Profiles in SonarQube — Part 1"
tags:
  - sonarqube
  - quality-profiles
date: 2018-05-03
layout: post
---
I haven’t encountered much articles about this and it is really helping me to write much cleaner code
and even learn new things about Java :) This article will be a part of my “SonarQube” series: So let’s begin…

- Firstly we need to download latest [SonarQube](https://www.sonarsource.com/products/sonarqube/downloads/).
- After download, let’s start server by running `./sonar.sh start` command, if you are using Unix-like system. *(This script can be found under {wherever_you_download_sonarqube}/bin/{operating_system_name})*
- Now, your SonarQube should be running at most likely http://localhost:9000/
To be able to create a custom profile we need to be an administrator. Check [here](https://docs.sonarsource.com/sonarqube/display/SONAR/Authentication) for default admin credentials.
- After successful login, we should better go *“Administration>Marketplace”* and install necessary features. I have installed **SonarJava, Android, FindBugs, CheckStyle, Rules Compliance Index** (it gives an info about what percentage your code complying). You can read more info at the [Marketplace page](http://localhost:9000/admin/marketplace).
- At last, we are ready for my main aim to write this post, **Custom Profiles**!! 🎊

## Creating Custom Quality Profile in SonarQube

🧑‍💻 Firstly, you may ask why we need a custom profile. Well there are some rules we, as developers, want to 
ignore but seeing these rules in the list does not make us happy or even opposite, we want add a rule 
which does not exist in default rules. Let’s click [Quality Profiles](http://localhost:9000/profiles) tab, go to the **Java** section,
copy **Sonar way** profile and rename this `Custom Quality Profile`. You can either assign this profile to an 
existing project or even declare it as default for all projects.

<img src="/assets/images/2018-05-03-image-1.webp" class="align-center" alt="Copied Sonar Way as My Sonar Way">

To be able to remove or add some rules click **“Activate More”** button and Blimey! 
As we can see at below image, we can now activate or deactivate or even customize these rules.

<img src="/assets/images/2018-05-03-image-2.webp" class="align-center" alt="deactivating or activating a rule in custom profile">

At [Part 2](/2018/5/4/custom-quality-profiles-part-2.html), I will try to explain how to use custom profiles on an Android Project. 
So try to keep up with me and don’t miss it ;) Any feedback for improvement, 
problems will be much appreciated by me. :)) Hope to see you soon.
