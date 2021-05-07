---
title: Finite Automata Editor App
date: "2021-05-07"
template: "post"
draft: false
slug: "automata-editor"
category: "Theoretical Informatics"
tags:
  - "Composable Architecture"
  - "Machine Learning"
description: "Opensourcing Finite Automata Editor app and the bachelor thesis for which it was written."
---

As my undergraduate years at FIT CTU in Prague are coming to an end, I am publishing the [Finite Automata Editor app](https://github.com/fortmarek/automata-editor/) and the bachelor thesis that was written alongside it.
You can find quick videos showcasing its most basic functionality below 👇

`youtube: [Automata Editor App Demo](https://www.youtube.com/watch?v=th_oZiohWzM)`
`youtube: [Automata Editor Live Demo](https://www.youtube.com/watch?v=Bf5hcEsllnc)`

## What's the app about

The app is, as the name suggests, an editor for finite automata. There are already existing automata editors out there but one thing they lack is ease of use.
That is why I have opted to build an app for iPad that will resemble the experience of drawing finite automata on regular paper as much as possible -
while bringing additional features like easy simulation of input to make it even _better_ than drawing on a regular piece of paper.

## How?

This has been a very interesting app to write because it combines the power of `PencilKit` and `CoreML`. I have first had to write a custom machine learning model to
recognize automata shapes (states, transitions, and cycles). After integrating the CoreML model, I have had to integrate [ALT library](https://alt.fit.cvut.cz/) for simulating the input for a given automaton.
This library is developed at FIT CTU and is written in C++ - this meant writing Objective-C++ wrappers and then Swift wrappers, so it could be used in the iOS app.
Afterwards, I have started working on the canvas using SwiftUI and the [Composable Architecture](https://github.com/pointfreeco/swift-composable-architecture).
Choosing these technologies made it really fun to write the app - and I believe it has also made it easier ✨

## Additional Reading

This post is only a quick overview - I go down into the nitty-gritty details in my thesis that you can download from [here](https://github.com/fortmarek/bachelor-thesis/blob/master/thesis.pdf). You can also check out the app [here](https://github.com/fortmarek/automata-editor/). I do plan to publish it on the App Store but there are some minor improvements I want to make before doing so - I can send you an invite to TestFlight if you are interested, though. You can either send me a DM on [Twitter](https://twitter.com/marekfort) or send me an [email](mailto:marekfort@me.com). If you want to check out the ML model, you can do so [here](https://github.com/fortmarek/automata-editor-model/).

I'd also appreciate any feedback and thanks for reading!
