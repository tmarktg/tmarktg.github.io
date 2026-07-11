---
layout: post
title: Hunt for DevOps Roles, Day 2 - Escaping Tutorial Hell
subtitle: Refactoring the Image Captioning App With Postgres and MinIO
cover-img: /assets/img/hunt-for-devops-day-2-cover.jpg
thumbnail-img: /assets/img/hunt-for-devops-day-2-thumb.jpg
share-img: /assets/img/hunt-for-devops-day-2-cover.jpg
tags:
  [
    devops,
    postgresql,
    minio,
    docker,
    s3,
    object-storage,
    python,
    huggingface,
    api,
    tutorial-hell,
    learning-in-public,
    project-devlog,
    image-captioning,
  ]
author: Mark Truong
---

Hey everyone, we're going to escape tutorial hell today.

## The Plan

I'm going to build a frontend, api, and Postgres service refactoring my image captioning app to add a Postgres db.

This app takes in an image or pulls from the unsplash api to write a caption of a meme on that image based off the object detection of the transformer I pulled off huggingface.

The idea is to use a Postgres db to store the captions themselves, the structure of the db will be caption history, timestamp, user supplied tag, this will have an image key and an ID.

Also, I want to have MINIO, an s3 bucket act as an object store to hold all the images and redid as a cache / queue, we will be learning what that means later.

## Fighting Through the Postgres Setup

At first, I just tried to search up a way to connect postgresql with python to the codebase, but I had no idea where to begin, pulling from docker, etc.

So then I searched up a YouTube video, but he only mentioned how to do it on windows when I'm on Mac!

Finally, I found what I'm looking for in another YouTube tutorial. The idea is to immediately apply concepts from the video, once I don't need this video, I will drop it.

I think the issue in the beginning is I didn't exactly know where to start, what is a db? How to build one, how to run it, how to connect it to the codebase.

Our db would not have multiple tables and relations, it's a simple one for a caption history. Having a massive table that fits everything is hard to manage, but since our table is so simple, maybe we don't even need a RDBMS lol.

These things are fundamental, when we get into deeper devops practices, there will be ideas of AWS micro services in databases, which you would explain the canonical question, what's the difference between a relational and non-relational db. So postico is the UI.

## Wrapping Up

So let's end it off here, but essentially, I followed this video, and created a db: [https://www.youtube.com/watch?v=qw--VYLpxG4&t=2025s](https://www.youtube.com/watch?v=qw--VYLpxG4&t=2025s)

I plan to just use this tutorial to get the basics, create the db, connect to the db, create the table, create the row structure, and I'm good to go. Don't get caught in the trap of oh I need to finish this tutorial, because you really don't.

Unless you need EVERYTHING in the tutorial, which wow that's crazy, don't watch the entire thing for completions sake… I have OCD so I fell into that trap.

Anyways, get excited, we're going to build a project that contains MY code, MY architecture, MY logic. You have complete ownership of the code!!!

![Mr. Robot Social-Engineer Toolkit terminal](/assets/img/hunt-for-devops-day-2-mrrobot.png)
