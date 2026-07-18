---
layout: post
title: What Are LoRAs?
subtitle: CLIP, Diffusion Models, and Prompt Guidance
cover-img: /assets/img/what-are-loras-cover.jpg
thumbnail-img: /assets/img/what-are-loras-thumb.png
share-img: /assets/img/what-are-loras-cover.jpg
tags:
  [
    ai,
    generative-ai,
    diffusion,
    stable-diffusion,
    clip,
    lora,
    machine-learning,
    claude-code,
    ai-assisted-development,
    learning-in-public,
  ]
author: Mark Truong
---

What are LoRAs?

First of all how do AI videos work?
[https://www.youtube.com/watch?v=iv-5mZ_9CPY&t=462s](https://www.youtube.com/watch?v=iv-5mZ_9CPY&t=462s)

AI image / video is created through a process called Diffusion an example is SDXL (Stable Diffusion XL)

The idea is that a image / video of noise gets passed through a transformer ouptutting another set of noise with some characteristics, this output becomes the input to the transformer again until a coherent image / video gets displayed

Diffusion models are meant to remove noise from image / video

## 1. CLIP (Contrastive Language Image Pre-training)

![Diagram of the CLIP model pipeline: load CLIP model, image preprocessing normalizes images and text preprocessing tokenizes text input, then a forward pass produces image and text features](/assets/img/what-are-loras-clip.jpeg)

Model architecture, with image and text encoders for an image and caption pair which is the dataset.

Model learns to contrast matching and non-matching image pairs

If you input two image / captions one with a person and one with that same person wearing a hat, if you subtract those two images across an embedding space, you will get hat or similar words to hat such as helmet or cap.

CLIP is really good at image classification, which you pass an image through image encoder and comparing through a possible caption through cosine similarity

The idea is taking an image, and adding noise to that image until it's destroyed

Then we train a neural network to reverse this process

## 2. Diffusion Models

![Sequence of a cat photo progressively destroyed into random noise across four steps, illustrating the forward diffusion process](/assets/img/what-are-loras-diffusion.png)

This is the naive approach where the modern approach is to add random noise to the image before passing it onto the neural network once more.

DDIM is the evolution of DDPM's approach which doesn't require the random noise, significantly decreasing the amount of steps necessary during the model training

## 3. Prompt Guidance

We pass a prompt through the clip encoder which outputs a vector embedding which steers the diffusion process to match the prompt inputted

Model by openai known as un-CLIP or DALLE-2

The idea is the text will help the diffusion model to more accurately remove noise from the image to better capture the prompt such as a dog with a hat called conditioning

There is also guidance where there is a dial on how strictly the model must follow that information

There is also a negative prompt where you write features you don't want.

![Diagram of prompt guidance showing the prompt "An astronaut riding a horse on the Moon" fed into Wan2.1 with a guidance scale of alpha = 5.0, combining the prompt and negative prompt vectors to steer the output image, alongside an example negative prompt listing unwanted features like blurred details, extra fingers, and deformed limbs](/assets/img/what-are-loras-negative-prompt.png)

## LoRA or Low Rank Adaptation

Reference:
https://www.youtube.com/watch?v=lixMONUAjfs

LoRA - Low-rank Adaption of AI Large Language Models: LoRA and QLoRA Explained Simply

![Diagram of LoRA: during training the pretrained weights W are combined with low-rank matrices A and B to produce h = Wx + BAx, and after training A and B are merged into the pretrained weights to form a single merged weight matrix](/assets/img/what-are-loras-lora.png)

LoRAs allow you to customize pretained neural networks such as diffusion model, reduces size of model checkpoints, I like to think of it like a bookmark in a book in a diffusion model.

It’s essentially when you have a LLM, which is powerful, but big and heavy, and you have a LoRA, which is a quantized version of that meaning smaller, easier to use, more efficient.

LoRA is kind of like reading the highlighted parts of a book, full-rank would be reading the entire book

LLM can be fine tuned to recognize cats. Fine tuning meaning training on specific items, adding wanted behaviors, and removing unwanted behaviors. This is very expensive.

LoRA fixes this problem making it cheap and fast. It is more efficient, reducing amount of resources used to train AI models. Faster to train and faster outputs. Under limited resources, the LoRA is better to use because it’s more efficient under a limited hardware.

Model trained a certain task can be adapted to a different but related task, more efficient than training a model from scratch.

QLoRA - Quantized Low Rank Adaptation

Quantized meaning data compression

I went through using ComfyUI to create workflows in order to generate images, and looked at prompt guidances to generate better prompts, and trained Loras with 75-100 images, and saw that sometimes, there are too many images, poor quality images, the images may not even look the same all the time causing too much variation in the LoRA. It was a really fun time creating these images, setting the seed to have image consistency, creating horrible prompts and having a friend make fun of them.

I guess going through this was the first time I’ve ever felt like a developer funnily enough.

It’s a shame that OpenAI killed Sora, they seemed like they were in the future. However, there is still DALL-E and the ability to create art that maybe AI-slop, or a masterpiece.

Anyways, who knows how ai artwork will evolve from here.

![DALL-E 2 generated image of two teddy bears scuba diving underwater with vintage cameras](/assets/img/what-are-loras-dalle-bears.jpg)
