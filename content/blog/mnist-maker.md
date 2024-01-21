---
title: MNIST Maker App
date: "2021-03-11"
tags:
  - "Machine Learning"
  - "OSS"
description: "Opensourcing MNIST Maker app to quickly create MNIST datasets on your iPad."
---

Currently, I am working on my bachelor thesis which will be an automata editor for iPad. I needed to recognize some shapes drawn by the user and for that I am using the exquisite framework by Apple, CoreML. But I also needed to create a custom dataset for some of the shapes - and I could not find anything that fit my needs!

## What I needed

I needed quite a simple app, actually:

- it should be for iPad
- I want to use my Apple Pencil to draw given shapes
- exporting images to iPad Files app
- scaling the image to a given size
- making the image grayscale

## Solution

Well, as you might have guessed from the title - I have not found anything that fit those needs. Alas I built my own solution!

It turned out to actually be easier than I expected.

Long story short, I had to

1. Create `PKCanvasView` and wrap it to `UIViewRepresentable`, so I could use it in SwiftUI
2. Add export button and `TextEditor` for specifying resulting image size
3. convert image to desired size + to grayscale
4. save it to documents 🥳

## Result

You can check out the code on [Github](https://github.com/fortmarek/personal-website).

To install it on your device, download the code and install it via Xcode. I did think about putting it on AppStore, and I still might do so in the future, but for the time being I'll keep it on Github only 🙂

Let me know if you'd like to see any improvements or if you stumble upon any issues!

Below, you can see a quick GIF showing off the functionality:

![MNIST Maker preview](/media/mnist-maker.gif)
