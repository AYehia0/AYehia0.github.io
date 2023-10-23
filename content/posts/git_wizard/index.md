---
title: "Git for hackers"
date: 2023-10-16T13:23:21Z
lastmod: 2023-10-16T13:23:21Z
draft: true
toc:
  enable: true
  auto: false
share:
  enable: true

authors: []
description: ""

tags: []
categories: []
series: []

hiddenFromHomePage: false
hiddenFromSearch: false

featuredImage: "featured-image.png"
featuredImagePreview: "featured-image.png"
---

# Git for hackers

{{< admonition quote "Once a wise man said:" true >}}
"There are two types of hackers in this world: those who know git, and those who still don't git it" 
{{< /admonition >}}

When I refer to [`hackers`](https://en.wikipedia.org/wiki/Hacker), I mean those who love to go be behind the things to discover how things work and break them (somehow). Originally, hacker simply meant advanced computer technology enthusiast (both hardware and software) and adherent of programming subculture who used their skill to achieve a goal or overcome an obstacle.

If you don't use git, you should start learning it now!

## Disclaimer
This isn't a tutorial on how to use git or it's going to make to the best git user but will inspire you to make your git experience better.

If you don't know what's git or version control, you should read this book : [Pro Git](https://git-scm.com/book/en/v2) from git's offical website. 

I also assume you have git installed on your machine, and you have some basic experience with Linux Shell.

{{< admonition note true >}}
I will update this blog whenever I find something useful to share.
{{< /admonition >}}

## Introduction
In this tutorial, I am going to share some tips and tricks that I have tested that would make it easy to git.

1. Uncommon git commands.
2. Git & Bash aliases.
3. Github Command line tools : `gh`.
4. Markdown everywhere.


### Uncommon git commands
Anyone who ever used git, will know this flow (and yes, with deep understanding of how git works, you won't use other commands).

{{< mermaid >}}
sequenceDiagram
    participant Working Tree
    participant Staging Area
    Working Tree ->>Staging Area: git commit
{{< /mermaid >}}

