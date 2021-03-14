---
categories: ["container", "development", "docker"]
date: 2020-05-18T22:20:02+02:00
description: "Installing Docker on a Raspberry Pi"
featuredpath: "date"
slug: "Installing Docker on a Raspberry Pi"
title: "Installing Docker on a Raspberry Pi"
comments: true
draft: false
tags:
  - container
  - docker
  - development
---

This is an easy step-by-step guide on how to install Docker on your Raspberry Pi. This setup targets Raspbian based Pi's but should also work on Ubuntu.

First make sure your system is up to date

```shell
sudo apt-get update && sudo apt-get upgrade
```

Get the installation script and run it

```shell
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

The last thing left now is to add your `pi` user (or any other) to the `docker` group in order to interact with the Docker daemon.

```shell
sudo usermod -aG docker Pi
```

To validate that the installation was successful try to execute the following commands or run a `hello-world` container.

```shell
docker version
docker info
docker run hello-world
```

The end!