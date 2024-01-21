---
title: Migration Tips - Let Time Work for You
date: "2021-03-21"
tags:
  - "Tuist"
description: "Making a large-scale migration is not easy. Here are a few tips I have observed in the latest migration done for tuist."
---

[tuist](https://github.com/tuist/tuist) is an OSS project to help you scale your Xcode projects - it is also a project which codebase has grown over its existence and has almost 100k lines of Swift code.

## Graph

At the core of tuist is a graph - [DAG](https://en.wikipedia.org/wiki/Directed_acyclic_graph) to be specific. This graph is used throughout the whole project as it contains all the information about the project - all projects, workspace, targets, dependencies and more. `Graph` itself and some of its components were unfortunately created as a reference type. I would not say this was a wrong decision at the time it was made (although, I don't have a full context on this) but it has made reasoning about the graph harder as, by nature of reference types, by modifying the graph, it was quite possible we could have influenced a part of a code we were not aware of.

In the future, we'd also like to let the users modify the graph themselves via plugins and to make the modifying of the graph easier and contained, migrating to a value type has started to appear as an important step we should make. Thus, we have embarked on a *migration journey*.

## First steps of migration

Before starting a migration, one should think deeply about how to properly execute it - especially when migrating a component that is used throughout the whole codebase and you can not make the migration in one go. That means, that for some period of time, you need to have the old and new components co-exist.

The first step we have made was more-or-less obvious in our case - take the old reference type `Graph` and create a new value type `ValueGraph`. 

But, as mentioned, we could not make the transition in one go. Thus, we had to ensure there was some level of interporability between the types. This is a code that should ideally be shortlived but is vital to a successful migration.

Once you have the cornerstones of the migration, now comes the hard part - changing all the code that works with the old component. 

## Let the time work for you

Large-scale migration is not an easy thing to do and it's something that takes considerable time. That's why having some level of interoparibility is crucial since now you can get help from **time and let the time work for you**. How? Well, it's easy, try to make all the new features with the new component - plus when modifying existing features, try to push people to migrate it and only then continue with the business they intended to do in the first place.

This has multiple benefits:
- people become accustomed to the new component gradually
- migration is done by more people, making it less of a burden for those coordinating the migration
- increases the odds of a successful migration since as we go right on the time axis, we are also progressing with the migration

Having time on your side is doubly more important in an OSS project where, as maintainers, we have only limited resources to contribute to the migration. Where our graph migration could have been done in a matter of weeks max if we worked on it full-time, making the migration in OSS setting took us months. But without doing the right decisions when starting the migration, we might have never finished.
