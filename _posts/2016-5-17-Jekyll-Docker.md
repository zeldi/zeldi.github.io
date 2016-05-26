---
layout: post
title: Jekyll on Docker [draft]
permalink: jekyll-on-docker
date: 2016-5-17
image: https://workshop.avatarnewyork.com/assets/media/docker-jekyll-container.png
---

Recently, I just started my new blog using Jekyll. So, here I'd like talk about deploying Jekyll using Docker.

<b> Why Jekyll? </b>

Jekyll is a simple and blog-aware static site generator built in Ruby. In laymen terms, it’s just a tool to let you have all the cool features of a full-blown CMS without having to worry about managing a database. This means hosting is extremely easy and scalable since all you’re doing is managing a bunch of files.
<!--more-->

<b> Why using docker? </b>

In general, There are two ways to run Jekyll on your machine.
<ol>
  <li> By installing Ruby and its dependencies </li>
  <li> Running on top of Jekyll Docker container </li>
</ol>
Personally, I prefer second option. some of the reason are:
<ul>
  <li>I dont use Ruby for my daily programming works, so it is just another overhead to install Ruby on my machine</li>
  <li>Only Docker is needed to develop your blog. you don't need to install a bunch of language environments with all it's dependencies on your machine. Want to run Jekyll but don't have have Ruby installed ? you simply need a Jekyll Docker image.
  </li>
  <li>Deployment is easy. If it runs in your container, it will run on your deployment server. If later you need to host it on Github pages, what you need to do is push it to github, and it will run just the same.
  </li>
</ul>

..
..
..
