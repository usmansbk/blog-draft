---
title: Instagram clone with React
published: false
description: 
tags: 
//cover_image: https://direct_url_to_image.jpg
---
Yesterday, I started working on a pet project to build a light Instagram web clone.

Aside from being a fun project to build, it'll serve as a case study for the Instagram UI.

To begin at the beginning, I decided to find out what assets (Fonts, icons, and logos) the amazing team at Facebook used, and in the process, I discovered something quite interesting.

It is common for websites to store individual assets as files,

![assets](./imgs/assets.png)

When you visit such a website, your browser will make multiple network requests to fetch each asset. Users with slow internet will notice some icons load faster than others.

To tackle this problem, the facebook team decided to store all their image assets in a single compressed file like this:

![assets](./imgs/sprite.png)
