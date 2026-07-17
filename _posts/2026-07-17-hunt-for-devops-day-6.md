---
layout: post
title: Hunt for DevOps Roles, Day 6 - Terraform, State Files, and VPC Basics
subtitle: Learning Why the tfstate File and a VPC's Subnets Actually Matter
cover-img: /assets/img/hunt-for-devops-day-6-cover.jpg
thumbnail-img: /assets/img/hunt-for-devops-day-6-thumb.png
share-img: /assets/img/hunt-for-devops-day-6-cover.jpg
tags:
  [
    devops,
    terraform,
    iac,
    aws,
    vpc,
    networking,
    subnets,
    nat-gateway,
    security-groups,
    nacl,
    cidr,
    s3,
    dynamodb,
    claude-code,
    ai-assisted-development,
    learning-in-public,
    project-devlog,
  ]
author: Mark Truong
---

Hey everyone, hunt for devops day 6. Now, the task is this intermediate project to learn fundamentals

1. Infrastructure-as-Code sandbox on a cloud provider — Use Terraform to stand up a VPC, subnets, an EC2 instance (or equivalent), and a security group from scratch. Add remote state (S3 + DynamoDB lock). Teaches provisioning, state management, and cloud networking basics.

So funny story, I have already built this project for a take-home project for a company, so I'm going to be grabbing some concepts from it before creating a new project.

## Terraform and the State File

![Diagram of the Terraform logo next to a terraform.tfstate file](/assets/img/hunt-for-devops-day-6-terraform-state.png)

Terraform is an IaC, infrastructure as code, and what that means is it is code that provisions AWS services through the CLI. There are many unique features of terraform, but one of the most notable one is it's tf state file which prevents environment drift, which is essentially that canonical it works on my machine where if you have two AWS developers click around a console, 1 to create the environment and 1 to reproduce it so he can work with the other, there will be different environments a lot of the time because of different setup, workflows, etc.

Now this will cause incompatibility in the application, causing the setup to fail when trying to develop at times.

Terraform alone can prevent this because the tf state file shows everything that it owns, and it is cloud agnostic, meaning it can setup any cloud with code. So let's say it creates AWS services such as lambdas or s3s, the tf state file will show it owns that. However, there can still be environment drift if you do not solely use terraform.

## The Stocks Serverless Pipeline

Anyways, in my project, the stocks serverless pipeline, I used Terraform, Amazon Eventbridge, AWS Lambda, DynamoDB, API Gateway, and AWS s3 all setup by Terraform (maybe I should build something in AWS CDK in the future).

### What Is Serverless?

![Diagram of serverless computing showing a cloud connected to code, containers, storage, APIs, and documents](/assets/img/hunt-for-devops-day-6-serverless.webp)

First of all let's understand what serverless is. Serverless is the idea that you can set up a cloud instance without needing to go into getting some system administrators, setting up the networking, rather you can spin up and destroy instances of cloud services such as a lambda. Most business owners would rather pay AWS to spin up instances with the networking, security, and all the infrastructure for convenience and sometimes cost.

There is a much larger implication to this such as how they used to have servers be mutable long-lived instances of a physical server versus a cloud with a immutable short-lived instance of a server that can be spinned up and destroyed for example a website during a black friday sale where there is higher amounts of traffic on the site to handle the load.

### Amazon EventBridge

![Amazon EventBridge icon](/assets/img/hunt-for-devops-day-6-eventbridge.png)

So Amazon Eventbridge is an event bus service, routes events either on a schedule or changes in aws services such as a lambda, or in this case a cron job my lambda fires, which is an automatic worker that triggers every day at the end of a trading day.

### AWS Lambda

![AWS Lambda icon](/assets/img/hunt-for-devops-day-6-lambda.png)

AWS lambda is an event-driven serverless function. event-driven means in my app, the event is the API gateway HTTP request from my app and the eventbridge cron job.

### DynamoDB

![Amazon DynamoDB icon](/assets/img/hunt-for-devops-day-6-dynamodb.png)

Dynamodb is the nosql serverless key-value db. Kind of like postgres if you seen my earlier blog post.

### API Gateway

![Amazon API Gateway icon](/assets/img/hunt-for-devops-day-6-api-gateway.png)

API Gateway is the serverless managed service to create HTTP/REST APIs, handling request/response, auth. routing, and CORS.

### S3

![Amazon S3 icon](/assets/img/hunt-for-devops-day-6-s3.png)

AWS s3 bucket is essentially a key value blob where you give the s3 your image (blob) and it returns a key similar to if you were to give your coat in a club and they give you a key to grab your coat later.

## Remote State: S3 + DynamoDB Lock

![Diagram of Terraform syncing state to an S3 State Bucket and locking via a DynamoDB Lock Table](/assets/img/hunt-for-devops-day-6-dynamodb-lock.png)

Now, in the beginning my tf state file lived locally, but when working with a team, that is a terrible way to share the state file securely, maybe I'd send an encrypted file of the state file to a team member, but that isn't very scalable.

So, the idea is to bring the state file into an s3 bucket, so that a team member can collaborate with you.

Now, if two people ran terraform apply at the same time, this causes a corrupt state file, (we should also see how this works with git version history) anyways, the dynamodb lock is for this reason, one terraform run can only run at a time.

## What Is a VPC?

![AWS VPC icon next to a cloud labeled VPC](/assets/img/hunt-for-devops-day-6-vpc.webp)

So what is a VPC?
[https://www.youtube.com/watch?v=7_NNlnH7sAg](https://www.youtube.com/watch?v=7_NNlnH7sAg)

This one maybe later
[https://www.youtube.com/watch?v=g2JOHLHh4rI](https://www.youtube.com/watch?v=g2JOHLHh4rI)

A virtual private cloud (VPC) is your own private cloud/network within the cloud, isolating your resources from everyone else's.

![Tony Stark manipulating a holographic blueprint of an Iron Man suit in his lab](/assets/img/hunt-for-devops-day-6-tony-stark.png)

## Networking basics

So the idea is that there is an internet gateway that can be thought as the gate to the internet from your vpc which may be a parking lot.

Inside the VPC, we have logically separated areas, which are the subnets

One subnet has a route to get to the internet will be a public subnet.
This can be a public facing web page

The private subnet which doesn't have a route will be a private subnet.
This can be a database which since it doesn't have a route to the internet will be an extra layer of security

![Diagram of a VPC spanning two Availability Zones, each with a public subnet and a private subnet](/assets/img/hunt-for-devops-day-6-vpc-subnets.png)

CIDR - classless inter-domain routing

Notation for describing blocking of IP addresses

Private ipv4 address comes from CIDR block in was ec2 instance

Hmmm, I saw a very interesting AWS VPC freecodecamp video, I may mess around with it to get better basics later

cidr.xyz - visualize how ip addresses work

## NAT Gateway

Network address translation, if you're on the private subnet, and you need to get access to the internet, you'd create a NAT gateway from your public subnet and have the route from private subnet to NAT gateway to internet gateway

![Diagram of a VPC region with public and private subnets across two Availability Zones, showing the route from a private subnet through the NAT Gateway to the Internet Gateway](/assets/img/hunt-for-devops-day-6-nat-gateway.png)

## Network ACLs (NACL) and Security Groups

Network ACL firewall that controls traffic in and out of a subnet. Rules for allow and deny, rules include IP addresses only

![Diagram of traffic flowing in and out of a VPC's public subnet through a NACL](/assets/img/hunt-for-devops-day-6-nacl.png)

Security groups firewall that controls traffic in and out of an ec2 instance. Rules for allow, and includes IP addresses and rules for other security groups

![Diagram of traffic flowing in and out of an EC2 instance's Security Group inside a VPC's public subnet](/assets/img/hunt-for-devops-day-6-security-group.png)

A really smart developer once told me that if you ever struggle with a concept, try staring at a picture of that concept for a long time to try and understand it. I have always had a hard time reading, but I find myself a very astute auditory / visual learner, so that makes sense.

![Illustration of a student writing in a notebook at a desk lit by a lamp, surrounded by bookshelves at night](/assets/img/hunt-for-devops-day-6-studying.png)
