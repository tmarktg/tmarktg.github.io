---
layout: post
title: Hunt for DevOps Roles, Day 4 - What Even Is DevOps?
subtitle: MinIO, Redis, Presigned URLs, and a Detour Into Ad-Tech
cover-img: /assets/img/hunt-for-devops-day-4-cover.jpg
thumbnail-img: /assets/img/hunt-for-devops-day-4-thumb.png
tags:
  [
    devops,
    minio,
    redis,
    s3,
    boto3,
    object-storage,
    caching,
    job-queue,
    claude-code,
    ai-assisted-development,
    learning-in-public,
    project-devlog,
  ]
author: Mark Truong
---

Hey everyone, day 4 for hunt for devops engineer, what is devops?

I found this playlist: [https://www.youtube.com/watch?v=j5Zsa_eOXeY&list=PLWKjhJtqVAbkzvvpY12KkfiIGso9A_Ixs&index=3](https://www.youtube.com/watch?v=j5Zsa_eOXeY&list=PLWKjhJtqVAbkzvvpY12KkfiIGso9A_Ixs&index=3)

## What Is DevOps, Actually?

![DevOps infinite loop diagram showing the Dev side (code, build, test with GitLab, git, Gradle, Sonatype Nexus) and Ops side (release, deploy, operate, monitor with AWS, Docker, Chef, Ansible, Kubernetes) connected in a figure-eight](/assets/img/hunt-for-devops-day-4-infinity-loop.png)

I need to find what devops is so that I can figure out what I need to truly build / study, but the psycopg2, then I virtually didn't know how to connect the database into the codebase, that's pretty interesting and if I dug deep enough I'm sure I could've figured it out.

So DevOps is a method that helps teams build products by continuously getting user feedback.

Now this is interesting because I thought DevOps is essentially the evolution of an IT / Help Desk / SysAdmin role, where you would work on the deployment of an application. Let's say building a CI/CD pipeline, dockerizing a service so that the environment is consistent across the team, using terraform in order to have the environment be consistent across teams due to the state file preventing drift to some extent if it is mainly just a terraform application without hand-creating some AWS services.

Hmmm, it looks like I have a more accurate statement and better understanding of what DevOps is. I think I'm going to try and continue on the build then, but where I went wrong is we're trying to merge people who are IT / Help Desk, or SRE, or SysAdmin, who are pretty much operations, working on deployment with Dev, which they would either click around an AWS console, or they would write IaC like terraform in order to deploy services merging the Dev and Ops.

Anyways, I think the world is going through sort of a paradigm right now for the computer science students, and I want to make sure I can keep up since AI is so crazy. I want to take a system, and interrogate it, break it down into parts, and make mini-projects so I understand each part. I'm going to have claude code build it and see if I can successfully learn from the top down the fundamentals.

## MinIO, Presigned URLs, and boto3

![MinIO logo, a red flamingo mark next to the MinIO wordmark](/assets/img/hunt-for-devops-day-4-minio-logo.webp)

So the minio storage implementation wants presigned urls where the backend does no work, rather hits MINIO where the image flows, over a backend proxy, I think it's the right move. I'm going to use boto3 as a s3 client. So an s3 client is a way to programmatically interact with s3 buckets in AWS, an s3 bucket is somewhat like a hashmap, let me explain.

So you would hand over your blob, like an image, and they would hand you back a key, sort of like a coat hanger service would serve you a key after you hand over your coat, if you've ever been to like a bar or something.

Now, an s3 is so powerful because it also includes version history as an option, where you can roll back to a certain point.

## Version History Saves Lives

![Git branch graph showing many colored merge branches converging into master/maint over a series of tagged releases](/assets/img/hunt-for-devops-day-4-git-branch-graph.png)

Now, I wonder if you've followed Elon Musk hiring that one young kid Akash Bobba from DOGE in the government (there was a lot of scrutiny for that lol) anyways, Akash was in a programming project where one of his team members had deleted that entire codebase, and he, over a single night, rebuilt it from scratch (some people are just built different). The versioning, of version history would eliminate that concern, where if there was some sort of git version history, or if it was stored on a private GitHub, you can merely return the code from the dead.

What I assumed happened was they were just developing locally, and the guy just did a bad merge or something and gone went the codebase (there are worse instances where you can delete a database, and not even Akash Bobba can bring that back from the dead). A funny illustration for this is this video: [https://www.youtube.com/watch?v=qeI_8T7fFZU](https://www.youtube.com/watch?v=qeI_8T7fFZU)

## A Claude Code Planning Tip

![Illustration of a hand holding a head made of connected nodes, on an orange background](/assets/img/hunt-for-devops-day-4-claude-code.webp)

Now, if you are developing with Claude Code, searching Claude Code Freecodecamp gives you a nice video, however, one extra tip I got from my Sr Software Engineer cousin is that you can also ask Claude to generate a plan from your prompt that may be a difficult multistep prompt, or a new feature. It has claude ask you questions and whatnot, which is very powerful.

Here is an example:

> create a plan for this feature: Redis as a cache/queue — cache captions for duplicate images, or use it as a job queue if captioning is slow (submit job → worker processes → poll for result). The async worker pattern is very strong to show.

## What Is MinIO?

![Line drawing of a bucket labeled Amazon S3, floating among clouds](/assets/img/hunt-for-devops-day-4-amazon-s3.jpg)

MINIO is an s3 compatible self-hosted object store.

Okay let's unpack that, that means the API, the structure of MINIO is all similar to s3, however, we're having it self-hosted. Now that means applications that work for s3 will work for MINIO, and this ties into the trickiness of AWS.

Essentially cloud services make it really hard to switch to another cloud provider, let's say you have an AWS service and you want to swap to Azure, that's incredibly difficult because of how the different structures of the IAM, and most developers are only good at one cloud, so when architecting a system, it's insanely stressful. It's kind of like Apple's relationship to the tech world where everyone copies Apple's technology and labels it the industry standard, it's the same with AWS, s3 has become canonical standard, where you have the azure blob, google cloud storage using the same structure.

So when using MINIO, you are pretty much switching from AWS s3 buckets to local storage pretty seamlessly.

## What Metadata Am I Storing?

![A cat photo labeled Data, with an arrow pointing to a yellow sticky note labeled Metadata containing filename, author, date, and location fields](/assets/img/hunt-for-devops-day-4-metadata-cat.png)

I'm storing the content-type of the image of the meme created such as jpg or png, and metadata is info about another thing, which relates to how people use your data.

Now if you ever went onto a crazy Mr Robot binge into being paranoid about birds being drones, you know that when you search up a website such as Facebook, and they serve you ads, the advertisers actually target you specifically for a specific type of ad, meaning if you have two different users, you get served two different ads.

How do they do that? When you load your page, data gets collected from you. Your IP and location come from the network connection, the browser and OS are in the HTTP request headers. Then Javascript trackers pick up how far you scrolled, when your mouse moved, what you clicked.

What's even crazier, is when you click on something, it's called an impression, which gives advertisers the behavior which causes retargeting on you based off similar products, oh maybe you're going camping so they'll send you ads about camping supplies. Now, the idea here is over a period of time, the browser tracks your data and the websites essentially have a profile about you and serve ads based off your searches, your history, etc. which is hyper-targeted advertising instead of something like spray or pray.

So when a browser loads, in the 100ms there is an ad network that is bidding off that space on your screen. You're getting auctioned off!

This is a super oversimplified reasoning of this explanation, so I'm really sorry programmatic advertisers.

## What Is Redis, and Its Role in This App?

![Red hexagon Redis logo with a stack of white chevrons and shape icons](/assets/img/hunt-for-devops-day-4-redis-logo.png)

Redis is an in-memory key-value data store, being less durable than a database like postgres.

In-memory means keeping everything in RAM instead of disk, so read write is really quick.

This is an Operating Systems fundamental in the Computer Science degree btw.

Being durable means that when a write is successful, it's guaranteed to survive a crash like a corrupted memory.

Key-value?? HASHMAPPP.

![HashMap structure diagram showing indices pointing to buckets of key/value/next nodes, with chained nodes linked together](/assets/img/hunt-for-devops-day-4-hashmap.jpg)

In this app, Redis plays two distinct roles, both in backend:

### 1. Job Queue

![Diagram showing a server pushing to a job queue, a worker pulling from that queue, and both server and worker connected to a database](/assets/img/hunt-for-devops-day-4-worker-queue-diagram.webp)

The `POST /api/caption` endpoint, which is the api endpoint that the frontend calls during a drag and drop image on the app, doesn't run the ML pipeline anymore.

It stages (temporarily store) the image in MinIO, and pushes a queue with a small JSON job description and returns a `job_id`. A separate worker (worker is an automated process which is event driven) runs jobs to queue in a loop, pulling jobs one at a time running the BLIP captioning, DETR detection, and meme renderer.

Job progress lives in a Redis hash, which is a dictionary/object nested under a single key, storing the status, result, error, and the frontend polls the `GET /api/jobs` to see if it's done or failed. So status, result, error are all nested under a single key such as `job:abc123`.

This is why the API now starts instantly instead of blocking on model loading; the API and the worker are two independent processes that only talk to each other through Redis (plus the shared Postgres/MinIO).

Wouldn't this be an intensive operation prone to failing? No, we're moving the intensive operation to a worker, so changing roles, and it is prone to failing, we can harden this operation (but maybe later).

### 2. Caption Cache

![Diagram of a caching server: on the 1st request it forwards to the origin server and caches the asset, on subsequent requests it responds directly without hitting the origin](/assets/img/hunt-for-devops-day-4-cache-definition.png)

Avoids redundant ML inference, which means: have I seen this image before? If yes, we will return the cached caption.

Keyed by the image's SHA-256 hash: `caption_cache:{image_key}` → `{raw_caption, objects}`, with a 24-hour TTL (`SETEX`).

![Diagram showing SHA-256 turning plain text like MyPassword123 into a fixed-length hash output through a hashing step](/assets/img/hunt-for-devops-day-4-sha256.png)

Why use a sha-256? Because we need a way to say have we seen this image before, and if you compare image file names, check out the numerous ways you can name an image, it's not consistent lol.

Also we get the same sha-256 on a reuploaded image because it is a deterministic input of bytes. Given the exact same sequence of bytes, it always produces the exact same output.

Before running the two expensive model calls, the worker checks this cache. If the same image was captioned before, it skips straight to reusing the cached caption/objects and only re-runs the cheap Anthropic call to generate a fresh meme caption, so repeat images don't pay for BLIP/DETR inference again.

So: Redis is the coordination layer between the "thin" API process and the "heavy" worker process, and it's also the fast lookup table that shortcuts repeated work. It holds no permanent data, Postgres (captions) and MinIO (images) are still the systems of record; everything in Redis is disposable/ephemeral (queue messages get consumed, job statuses expire in an hour, cache entries expire in a day).

## Learning by Deciphering

This is what I mean by building with AI, it is indeed powerful, but if this isn't gibberish to you, you are probably experienced. It is your job to decipher what this means when building a system, which allows you to continue to build features, make it your own, and truly grow in your developer journey, asking claude code is a pretty powerful tool to supercharge your journey I believe.

When I asked claude code to run the app, I ran into two different things: claude's metadata got leaked which captioned (8 words), and it said it is optional for me to tag the image, which I want the app to do on its own.

![Edward Morra in a blue quarter-zip sweater, looking off to the side with a confident smile](/assets/img/hunt-for-devops-day-4-closing.webp)
