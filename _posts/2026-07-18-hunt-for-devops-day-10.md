---
layout: post
title: Hunt for DevOps Roles, Day 10 - REST APIs and API Gateways
subtitle: Breaking Down the Stocks Serverless Pipeline
cover-img: /assets/img/hunt-for-devops-day-10-cover.jpg
thumbnail-img: /assets/img/hunt-for-devops-day-10-thumb.svg
share-img: /assets/img/hunt-for-devops-day-10-cover.jpg
tags:
  [
    devops,
    rest-api,
    api-gateway,
    microservices,
    http,
    crud,
    claude-code,
    ai-assisted-development,
    learning-in-public,
    project-devlog,
  ]
author: Mark Truong
---

Day 10 for hunt for devops

So remember, we aren't learning just to learn. I'd like to break down the stocks-serverless-pipeline at a high level. I think I didn't do a great job at explaining some of the components, so I'm going to target two concepts, the REST API and the API Gateway,

What is a REST API? What is an API Gateway?

## What Is a REST API?

REST API Reference
[https://www.youtube.com/watch?v=lsMQRaeKNDk&t=19s](https://www.youtube.com/watch?v=lsMQRaeKNDk&t=19s)

![Diagram of a client sending an HTTP request (GET, POST, PUT, DELETE) to a server and receiving a JSON response](/assets/img/hunt-for-devops-day-10-rest-api.webp)

REST (Representational State Transfer) API is a software architecture that's industry known for APIs

1. Simple and Standardized - How to format data / requests
2. Scalable and Stateless - as service grows complex can make modifications, don't have to worry about what data is in what state
3. Performance / Cache - as service gets more complex performance stays high which can be due to caching

/api/{resource}

Request sent from client to server, and response received back from server

When sending a request, think of what you want to do with the API, what actions may you want to take? CRUD

Create, Read, Update, Delete

HTTP Methods / Operations

Create -> Post
Read -> Get
Update -> Put
Delete -> Delete

Maybe take a Postman Course from freecodecamp if you want a more visual explanation and if you're ever vibe coding an app you can build API tests
[https://www.youtube.com/watch?v=VywxIQ2ZXw4](https://www.youtube.com/watch?v=VywxIQ2ZXw4)

In a request, you have an operation, endpoint, parameters / body, and headers. And you have Response in typically JSON data.

Get operations, endpoint is /api/{resource}, and response in JSON.

Maybe that resource has ran out, the website Put /api/{resource}/1 is replaced with another resource.

Create a new resource since the resource has been restocked Post /api/{resource} and include in the body resource you want to create

## What Is an API Gateway?

API Gateway reference:
[https://www.youtube.com/watch?v=hWRRdICvMNs](https://www.youtube.com/watch?v=hWRRdICvMNs)

![Diagram of an API Gateway routing API calls from user applications to microservices including record repository, appointment scheduling, payment processor, and messaging service](/assets/img/hunt-for-devops-day-10-api-gateway.webp)

API (Application Programming Interface)

APIs typically need additional infrastructure to scale, secure, and accelerate them

Important with the modern approach of going from Monolithic applications to Micro-services

Microservices make app more scalable, available, and more efficient, but comes with more API calls between clients and microservices

API Gateway solves the increased API calls.

API Gateway essentially reduces the round trips necessary for the client to the microservice, reducing the latency

The developer team can now handle the api requests through the API Gateway, instead of handling all the API requests individually.

It's secure against DOS attacks, can add authorization layer.

Goes from HTTPS to API Gateway to HTTP to microservice, so the API Gateway can decrypt the SSL layer instead of the backend microservice doing that, boosting performance of the application

Common functionality offload, instead of the microservice having that functionality, you can offload it to the API Gateway, for example rate limiting, api monitoring / logging.

What happens during Black Friday?
Implement BFF (Backend to Frontend) Architecture, adding additional API Gateways, for example, one for mobile devices.
