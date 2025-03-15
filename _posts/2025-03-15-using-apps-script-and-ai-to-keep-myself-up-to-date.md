---
title: "Using apps script and Gemini to keep myself up-to-date"
image:
  path: /assets/images/2025-03-15-post-image.jpg
  thumbnail: /assets/images/2025-03-15-post-image.jpg
  caption: "Photo by [David Travis](https://unsplash.com/@dtravisphd?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) on [Unsplash](https://unsplash.com/)"
tags:
  - apps-script
  - AI
  - note
  - tldr
date: 2025-03-15
layout: post
---

Hey everyone, the title of the post is clear enough. I was inspired by [this article](https://medium.com/google-cloud/guide-to-function-calling-with-gemini-and-google-apps-script-0e058d472f45)
which explains how to leverage Gemini with function calling in Google Apps Script. Following that, I've created an [apps script](https://developers.google.com/apps-script) 
that reads my TLDR emails and summarizes them into Markdown formatted text via Gemini 2.0 Flash model (at the time of writing).

At the moment, the script works on 3 different levels;

1. On single email level
2. On Gmail home page
3. With a daily trigger

And all does similar jobs. Summarizing via AI and exporting the summary to Google Docs in a Markdown format.
The reason I've picked Google Docs as exporting method is because Google Workspace makes it easier, and it can work with many tools like
[Obsidian](https://obsidian.md/) note app and, since the file is hosted in Google Drive, even with [Google NotebookLM](https://notebooklm.google.com/). 

I'm right now enjoying the daily trigger functionality of the script where it daily exports the TLDRs from my email to Google Docs
(also moves the summarized emails to thrash, helps keeping my Gmail clean) and only thing I do is to "Click sync" button my notebook app at the weekends.
If I'm too lazy to read all the TLDRs at the weekends, Google NotebookLM app offers "Audio Overview" where I can actually listen my TLDRs like a podcast. 
I tried it when I'm doing the dishes and one thing I noticed was that "Audio Overview" can improvise a lot to keep the podcast light-hearted. 
Therefore, I tried following customized prompt to make it more formal which worked for me better. But feel free to customize your own prompts.  

```
Given TLDRs source (A TLDR item is typically represented by a link followed by a brief description.);

Follow these instructions to create the audio summary:

1. If TLDR item contains "kotlin", "android" topics, then go to the link content itself to create a summary of at least 3 sentences.  

2. Otherwise read out loud the description itself.

Target audience is programmers. 
Keep it a bit formal.  

```

I hope in the future, I can make the script more configurable and make use of other AI models to automate podcast generation so that I can do a lot less before I do the dishes:)
The apps script is now in review at Google Workspace Marketplace and will be a paid version because it is using my Cloud project, but I'm also sharing [the code](https://github.com/melomg/gmail-tldr-2-markdown) if you'd like to create your own versions.  

*Hope this article inspired you…* 🖖
