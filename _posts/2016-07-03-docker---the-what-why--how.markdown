---
title: Docker - The what, why & how
layout: post
categories: [javascript, es6, functional]
tags: [javascript, es6, functional]
published: True
---

Before we dive in to how to do things in Docker, lets first understand why we want to use docker. What is it, that is so good about Docker, that we should invest our time on it.

As a developer, my job is to add value by writing code. Traditionally I have accomplished this by writing code which exists __inside__ the project folder and then pushing it to github. I control everything inside this folder. By changing stuff in the folder I have deliver many _wow's_. Also its on github and another developer can take it from there, add more awesomeness to it and push it back. Finally everything in this folder gets pushed as-is to production. This process has worked well, but its limited to that folder.

At many times I feel I can deliver more value by changing/adding things which exist outside this project folder. These could be things like installing a dependent service like image magic or tweaking nginx.conf or adding a system variable or installing an OS level package. This list just touches the surface. As a developer I have been trained to think within my project folder whereas now I am talking about having control over the entire setup. This setup includes the Operating system my code will run on, its dependent services, env variables etc. Finally I want to be able to work this setup just like I worked my code. I want to be able to push this entire setup to github, be able to share it, let others work on it and then finally deliver this entire setup to production. That sounds awesome to me and this is why I am exploring Docker.

## What it delivers

Control & deliver more than just code.
code + its dependencies like services, env variables, configurations, runtime, system variables and everything else

## Compared to Vbox

guest OS, sharing kernel

## Other advantages

fast deploys as stuff is built only once in to the image, and can then be deployed as many times as required. really nice to have when scaling up and down at run time.

## Understanding Volume

sharing a directory from host to container for development
share a directory among containers
backup things like logs, database etc. as containers are ephemeral.

