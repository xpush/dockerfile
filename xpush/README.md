## XPush standalone server's Dockerfile


This repository contains **Dockerfile** of [XPush](https://github.com/xpush/node-xpush) published to the public [Docker Registry](https://index.docker.io/).


### Installation

1. Install [Docker](https://www.docker.io/).

2. Download [trusted build](https://index.docker.io/u/xpush/standalone/) from public [Docker Registry](https://index.docker.io/): `docker pull xpush/standalone`

   (alternatively, you can build an image from Dockerfile: `docker build -t="xpush/standalone" github.com/xpusn/dockerfile/redis`)


### Usage

#### Run `xpush standalone`

    docker run -d --name xpush -p 8000:8000 -p 9000:9000 xpush/standalone

#### Run `xpush standalone` with persistent data directory.

    docker run -d -p 8000:8000 -p 9000:9000 -v <data-dir>:/data --name xpush xpush/standalone
