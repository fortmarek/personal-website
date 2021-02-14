---
title: Importance of dotfiles
date: "2021-02-14"
template: "post"
draft: false
slug: "importance-of-dotfiles"
category: "CLI"
tags:
  - "CLI"
  - "OSS"
description: "How to become more effective in your CLI environment by versioning and organizing your dotfiles."
---

Recently, during macOS update, my Mac went bust. After some time in the recovery mode I was like "at least I have the Time Machine backup". I clicked the button to restore from my Time Machine, went to do some other things and after two hours I came to look how it was going. "Time Machine backup failed". Well, not great. I tried this for the second time with the same result. Since I wanted to do a clean install for some time, I thought that somebody is telling me now is the time. Clean install it is then.

After I was on my fresh macOS, I wanted to configure my CLI environment. And rather getting back to the unorganized dotfiles mess I had before, I decided to make it properly this time.

# Why you should version your dotfiles

I spend a _lot_ of time in the CLI, therefore I find it very important that it is setup in a way I like. That is not only because I want the time I spend in my CLI to be pleasant (although, that is absolutely a good reason!) but also to be _efficient_.

To achieve that it is important to have a look at your dotfiles.

While a lot of users do personalize their CLI in some way, it is not at all so common that people version their changes to them. They are also strewn all around the home directory and it is very hard to be sure where you can find what. As a result, this might make it harder for you to make customizations, so in a lot of cases you will not make that small change that might make your life a little better at all. But in the long run this all adds up.

This was true in my case, too - I had no organization of my dotfiles and often I did not want to make a change because I was worried I might screw it up.

I have realized sometime ago I should do something with this but I just did not find the time - well, until I had to.

# My dotfiles

Instead of having the dotfiles for myself, I wanted to share them. I started with [dotfiles by Zach Holman](https://github.com/holman/dotfiles) and then made adjustments to feel more at home - you can check the repo out [here](https://github.com/fortmarek/dotfiles).

One of the more notable changes have been adding `oh-my-zsh` on top of `zsh`. Why? Well, a) `oh-my-zsh` has a great selection of themes. b) they have great plugins.

Some of the plugins I use are:

- [git](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/git) for speeding up working with git. Want to add unstaged changes with a message? `gcam "My commit message"`. Want to pull changes? `gl`. And lots more - I sometimes do use GUI for git but for most common tasks, `git` CLI helps me to work so much faster.
- [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting) - having quick feedback about what you write in the command line is crucial - there is no reason to see whether your command is valid before you hit that enter => just leverage the power of auto-highlighting!
- [z](https://github.com/agkozak/zsh-z): sometimes, it feels slow navigating in the CLI around different directories. `z` can make this faster as it saves the directories you `cd` into and then chooses the most common one. Eg if you often `cd` into `~/.dotfiles`, next time you can just run `z d`. That's it!
- [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions): Yes, finding that command you ran two hours ago can be done by hitting arrow up repeatedly. But why make your life difficult if you can just start typing the command and get it completed automatically? Of course you need to know what the command starts with - otherwise, I find it manually via `history | grep "keyword"` (let me know if there is some awesome plugin for this 😉)

Apart from `oh-my-zsh`, I have also added for example keybindings for Xcode - this can be especially helpful if you reinstall Xcode because in that case you'd lose them (happened to me on multiple occassions).

# Share

I am not saying that the dotfiles I have are the best or that they will work for you. The point of this post is to share my experience and encourage you to invest in your environment. Let me know what your dotfiles look like and what has been working for _you_. Thanks for reading! ✨
