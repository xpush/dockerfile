## XPush server's Dockerfile


This repository contains **Dockerfile** of [XPush](https://github.com/xpush/node-xpush/dockerfile/) published to the public [Docker Registry](https://registry.hub.docker.com/).


### Installation

1. Install [Docker](https://www.docker.io/).

2. Download [trusted build](https://registry.hub.docker.com/u/stalk/xpush/) from public [Docker Registry](https://registry.hub.docker.com/): `docker pull stalk/xpush`

   (alternatively, you can build an image from Dockerfile: `docker build -t="stalk/xpush" https://raw.githubusercontent.com/xpush/dockerfile/master/xpush/single/Dockerfile`)


### Usage

#### Run `xpush latest`

    docker run -d --name xpush -p 8000:8000 -p 9000:9000 stalk/xpush

#### Run `xpush standalone` with persistent data directory( /data )

    docker run -d -p 8000:8000 -p 9000:9000 -v /data:/data --name xpush stalk/xpush