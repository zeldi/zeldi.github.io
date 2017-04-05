---
layout: post
title: Jekyll on Docker
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

<b> Running Jekyll  on Docker</b>

<p>The fastest and easiest way to start with Jekyll is to download a theme for Jekyll. This website uses a slightly modified version of the Hyde theme. It’s <a href="https://github.com/poole/hyde">hosted on GitHub</a>, so you can download or clone it from there.</p>

<p>The friendly people of Jekyll have already made a Docker image available on <a href="https://hub.docker.com/r/jekyll/jekyll/">Docker Hub</a>. Let’s take advantage of that! Suppose you have downloaded or cloned the Hyde repository into <code class="highlighter-rouge">~/jekyll-site/</code>. To use the Jekyll image without having to pass in the required options every time to create a container, we can make use of Docker Compose. Create a file called <code class="highlighter-rouge">docker-compose.yml</code> in <code class="highlighter-rouge">~/jekyll-site</code> with these contents:</p>

<figure class="highlight"><pre><code class="language-yml" data-lang="yml"><span class="s">jekyll</span><span class="pi">:</span>
    <span class="s">image</span><span class="pi">:</span> <span class="s">jekyll/jekyll:pages</span>
    <span class="s">command</span><span class="pi">:</span> <span class="s">jekyll serve --watch --incremental</span>
    <span class="s">ports</span><span class="pi">:</span>
        <span class="pi">-</span> <span class="s">4000:4000</span>
    <span class="s">volumes</span><span class="pi">:</span>
        <span class="pi">-</span> <span class="s">.:/srv/jekyll</span></code></pre>
</figure>

<ul>
  <li>With <code class="highlighter-rouge">image: jekyll/jekyll:pages</code> you indicate you want to use Jekyll’s Jekyll image tagged with “pages”. This is a specific image suited for GitHub pages.</li>
  <li>The part after <code class="highlighter-rouge">command:</code> is the command to execute in the container. The <code class="highlighter-rouge">jekyll serve</code> command starts Jekyll’s builtin development web server. The options <code class="highlighter-rouge">--watch</code> and <code class="highlighter-rouge">--incremental</code> instruct Jekyll to automatically regenerate the HTML when a file is changed.</li>
  <li><code class="highlighter-rouge">4000:4000</code> forwards port 4000 of the container to your local port 4000.</li>
  <li>Finally, on the last line you map the current directory (<code class="highlighter-rouge">~/jekyll-site/</code>) to <code class="highlighter-rouge">/srv/jekyll/</code>. That’s where the images is configured to go looking for a Jekyll site.</li>
</ul>

<p>That's all .... and now you only need to go to the terminal and run the docker-compose using the following command:</p>
<p><code class="highlighter-rouge">docker-compose up</code></p>

<p>Now, if you go to <code class="highlighter-rouge">http://<docker-ip>:4000</docker-ip></code> in browser you will see your Jekyll website running.


<p>---</p>
