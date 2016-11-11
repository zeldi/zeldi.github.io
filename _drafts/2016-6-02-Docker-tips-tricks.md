---
layout: post
title: Collection of Docker Best Practices
permalink: docker-tips
date: 2016-5-17
---

I have been using Docker for quite sometime for my development as well automation of deployment.
I found out that, there are many tips are shared online in order to use Docker effectively.
The objective of this post is to gather tips, tricks and best practices of using Docker along with my journey of using and learning Docker.

Note that the tips listed in this post are gathered around many sources from the Internet.

### Running Docker Container

Every time you type ``docker run``, it will creates a new container with the specified image and starts a command inside of it (CMD specified in the Dockerfile). When using this command many times, use the `--rm` flag, so that the container data is removed after it finishes. By default, I'm used to docker container as in the following format:

```bash
docker run --rm -it ubuntu /bin/bash
```
Notice here that `-it` means `--interactive --tty`. It is used to attach the command line to the container after it has started. Those are very useful params when running a container in foreground mode.

#### Entering a container
Never install SSH daemon in a container. Use docker exec to enter into a container:
```
docker exec -it ubuntu_bash bash
```
Use it to inspect and debug running containers. But, ideally, avoid creating automated tooling around it.

Check the documentation of it: https://docs.docker.com/engine/reference/commandline/exec/

#### Sharing files with host machine

##### Windows #####

boot2docker provides two current workarounds for sharing directories on Windows, though more native sharing might follow.  The official instructions are [here](https://github.com/boot2docker/boot2docker#folder-sharing) but are more complicated than our preferred method of using the `-v` flag, which you can do like so:

```bash
docker run -d -p 8787:8787 -v /c/Users/foobar:/home/rstudio/foobar rocker/rstudio
```

In this case, `/c/Users/foobar` corresponds to an existing folder on your computer at `C:/Users/foobar`, and `foobar` can be anything. You can now connect to RStudio at http://192.168.59.103:8787 and log in with rstudio/rstusio. With this `-v` method you can read and write files both ways between Windows and RStudio.

The above approach may not always work, as described in [this bug report](https://github.com/boot2docker/boot2docker/issues/846). The workaround is to either use a double slash or $(pwd):

```bash
docker run -d -p 8787:8787 -v //c/Users/foobar:/home/rstudio/foobar rocker/rstudio
```

or

```bash
docker run -d -p 8787:8787 -v /$(pwd):/home/rstudio/foobar rocker/rstudio
