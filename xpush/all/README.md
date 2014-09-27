## XPUSH standalone server's Dockerfile


This repository contains **Dockerfile** of [XPUSH](https://github.com/xpush/node-xpush/dockerfile/) published to the public [Docker Registry](https://registry.hub.docker.com/).


### Installation

1. Install [Docker](https://www.docker.io/).

2. Download [trusted build](https://registry.hub.docker.com/u/stalk/xpush/) from public [Docker Registry](https://registry.hub.docker.com/): `docker pull stalk/xpush:standalone`

   (alternatively, you can build an image from Dockerfile: `docker build -t="stalk/xpush:standalone" https://raw.githubusercontent.com/xpush/dockerfile/master/xpush/all/Dockerfile`)


### Usage

#### Run `xpush standalone`

	docker run -d --name xpush -p 8000:8000 -p 9000:9000 stalk/xpush:standalone /bin/bash xpush-stand-alone.sh --host sample.stalk.io

#### Run `xpush standalone` with persistent data directory( /home/stalk/data )

	docker run -d -p 8000:8000 -p 9000:9000 -v /home/stalk/data:/data --name xpush stalk/xpush:standalone /bin/bash xpush-stand-alone.sh --host sample.stalk.io