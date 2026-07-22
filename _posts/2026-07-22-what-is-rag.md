---
layout: post
title: What Is RAG?
subtitle: Retrieval Augmented Generation, Explained
cover-img: /assets/img/what-is-rag-cover.webp
thumbnail-img: /assets/img/what-is-rag-thumb.png
share-img: /assets/img/what-is-rag-cover.webp
tags:
  [
    ai,
    rag,
    retrieval-augmented-generation,
    llm,
    machine-learning,
    claude-code,
    ai-assisted-development,
    learning-in-public,
  ]
author: Mark Truong
---

Hey everyone, so I put RAG on my resume, so I need to explain it.

![Diagram of a RAG pipeline: a knowledge base is split into data chunks, a user query and the data chunks are passed through an embedding model to produce document and query embeddings, which are stored in and retrieved from a vector DB as retrieved documents](/assets/img/what-is-rag-pipeline.png)

To put it simply, RAG, or Retrieval Augmented Generation is a layer on top of an LLM which helps mitigate some of LLM's shortcomings

Such as
1. Outdated info
2. No evidence
3. Hallucinations

When you input a request into an LLM, what happens, and sorry if this is a gross oversimplification, is that the LLM runs through a gigantic zip file of the internet in order to answer your question.

This video explains it really well, and honestly you should be watching all this guy's podcasts and videos given how relevant AI is right now, most people will sit for hours to listen how AI works so they can better utilize it, kind of how programming revolutionized the world when IBM introduced the first computer: [https://www.youtube.com/watch?v=EWvNQjAaOHw&t=2649s](https://www.youtube.com/watch?v=EWvNQjAaOHw&t=2649s)

Information about rag: [https://www.youtube.com/watch?v=T-D1OfcDW1M](https://www.youtube.com/watch?v=T-D1OfcDW1M)

Anyways, this process is susceptible to those 3 processes above, specifically hallucinations because it tries to assume information on a topic that isn't very well documented, and unfortunately, this is a huge shortcoming of AI where it will lie to you instead of telling you I don't know, the agentic era is aiming to solve this.

So the way this is mitigated through RAG is instead of a prompt and a response back through the LLM, the LLM searches relevant information prepended to your query in order to answer your question with up to date information.

Most LLMs already do this, so why do we care? Well, people are building internal RAG systems which you can pull information from a content store which contains private information about the company or in my case, information from the website, FAQ, and frequently updated showtimes where an LLM isn't as reliable.

We get evidence, up-to-date info, and the model doesn't hallucinate, however, it can say I don't know the answer to that, which is reliant on the retrieval pipeline, meaning an answerable question may be deemed unanswerable, which is a shortcoming of RAG.
