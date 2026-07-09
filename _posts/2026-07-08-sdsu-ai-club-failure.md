---
layout: post
title: What I Learned Failing as a Project Lead at the SDSU AI Club
subtitle: Depth, Focus, and Using AI Without Skipping the Fundamentals
cover-img: /assets/img/sdsu-ai-club-failure-cover.jpg
thumbnail-img: /assets/img/sdsu-ai-club-failure-thumb.jpg
share-img: /assets/img/sdsu-ai-club-failure-cover.jpg
tags:
  [
    computer-science,
    career,
    self-reflection,
    ai-tools,
    claude-code,
    student-clubs,
    networking,
    portfolio-projects,
    devops,
    interview-prep,
    learning-in-public,
    huggingface,
    openai-api,
  ]
author: Mark Truong
---

Hey everyone, so I wanted to talk about how I was a project lead at the SDSU AI club and how I failed, what I did well, and how I would advise you guys would do when joining clubs to build your technical skills.

## Choosing the Right Club

Well, for one, when you join a club, just make sure you know what you're getting yourself into. Would this club help me in my career? Would it help me network? If I dedicate x amount of time to this club would it be optimal for me in the long run versus my other tasks?

## Focus and Depth Over Breadth

And this really ties into being hyperfocused into one avenue in computer science and attacking that focus relentlessly, learning the fundamentals, and continuously learning. I estimate it takes roughly 6 months to 1 year to bridge the gap to an entry level role in devops engineer roles and most other roles.

There are many avenues in computer science, Frontend, Backend, Full Stack, Devops / Cloud, Embedded Engineering (pretty hot right now too). There are also multiple sectors, Fintech, Healthtech, Enterprise & B2B, Consumer and Media, Advertising and Marketing, Legaltech and Telecom, etc.

## Using AI the Right Way

Also, make sure you use AI correctly, as it is a very powerful asset. I use AI to write my blog, and my portfolio website. I wrote my portfolio projects too, but I am unhappy about how little I know about those projects. What I want to do is perhaps build smaller projects first to get devops fundamentals, then analyze the system and see if I can break it down and put it into my own words, so when I explain it to an interviewer, they will know I am a worthy candidate in fundamentals and would be a strong addition to the team. The STAR method is very strong when interviewing, and this is the idea of how depth is very important when focusing on a specific avenue in computer science.

## The SDSU AI Club Project (Where It Went Wrong)

Back to the AI project I did at the SDSU AI club, now it was a nightmare. I went on a tutorial project hell, created a script in python that broke because it rested on the beautiful soup web scraper that pulls on the old reddit site which allows web scraping. I used two different hugging face models, which I didn't even know how they connected, and I didn't know how to build the site where you would have the app, nor use the OpenAI API endpoint to create a captioned image.

When I crushed that project with claude code, what I learned is that I should've used the unsplash api to pull images, have the openai api to write the caption into the image, and use the two huggingface models, one as a backup to use object detection to pass off the image to the LLM reading the image.

I failed this project because I didn't understand how to build things, and how to architect the app correctly. I think I should've reached out to people I knew to build it, such as professors, and maybe others. I wish I spent more time building the project than learning AI from the ground up, thinking I didn't know how to build the project, yet it's so simple when you break it down.

Use HTML/CSS to have a drag and drop functionality of an image. Connect the OpenAI API to the app. Connect the HuggingFace Transformer models. That's it, that's literally 80% of the project.

## Break Down the Project Like You Study

And I really want to drive this home, break down projects, spend a long time building the project, as much as you would study. Google relevant information to your specific project, be it githubs (which are filled with AI slop unfort ;c), youtube videos, stack overflow posts, and AI use should be used very cautiously, don't pretend you know the material.

I think a good test if you know about the system from the ground up is point to a line of code in your codebase, a feature that you're building, what does that code do, how does it function in the system? Why did you make the decision to build it that way?

And I think this is a good way to think deeply about a subject, you can build a very powerful, architecturally sound project, but if you can't deeply talk about something like a lambda, a lambda is a serverless function blah blah blah, then it is pointless for your ability to effectively collaborate and audit the ridiculous amount of code getting written nowadays with AI writing most of it.

Talk about the lambda like you're five, where you can explain it barebones, then explain it with all the jargon in an interview (this is how I failed one of my interviews btw ;c)

## AI Is Powerful, Just Don't Skip the Fundamentals

And I'm not saying to ignore AI, it is actually very important to understand how it works, what it can do for you, and most companies are asking you to use AI. Now, specifically, Claude Code is the most powerful coding agent.

It's really interesting during my time there when AI was still very weak, during the image captioning project, and I saw the first coding agent cursor, then windsurf, now you see claude code which is probably one of the most impressive tools I've ever seen.

If you are going to go through a crazy binge of a technology, please please please question everything the person does, every 10-15 minutes or so, understand why they did a certain thing, how they implemented a certain feature, why they did it in that exact order, and then every 2 hours, try building an app with those concepts, those apps will maybe never see the light of day, not even on a github repo, but the idea is to learn, learning these things takes time, and try giving yourself a year to get really deep into what you're focusing.

## What I'm Doing Now

Me specifically, I'm going to create deployable links for each of my portfolio projects, and demo video and GIF of the app working on those 3 apps, and learn them deeply, maybe reverse engineer it so I can understand better. I estimate this will take me months.

Anyhow, this is just my breakdown of one of my other mistakes, hopefully everyone here finds their software engineer job or at least become a better programmer.
