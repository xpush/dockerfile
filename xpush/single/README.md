## XPush server's Dockerfile


This repository contains **Dockerfile** of [XPush](https://github.com/xpush/node-xpush/dockerfile/) published to the public [Docker Registry](https://registry.hub.docker.com/).


### Installation

1. Install [Docker](https://www.docker.io/).

2. Download [trusted build](https://registry.hub.docker.com/u/stalk/xpush/) from public [Docker Registry](https://registry.hub.docker.com/): `docker pull stalk/xpush`

   (alternatively, you can build an image from Dockerfile: `docker build -t="stalk/xpush" https://raw.githubusercontent.com/xpush/dockerfile/master/xpush/single/Dockerfile`)

### Usage

Create a config file in user directory ( name like `session.json` and `channel.json`)

[More info](http://xpush.github.io/doc/configuration/) from xpush.github.io

#### Run session server

	1. Run with xpush executable. ( You can run session server with your own options.)

		docker run -i -t -p 8000:8000 -v /home/stalk/data:/data stalk/xpush:latest xpush --config /data/session.json --session --port 8000

	2. Run with already installed shell script

		docker run -d -p 8000:8000 -v /home/stalk/data:/data stalk/xpush:latest /bin/bash -c xpush-session.sh

>**Note**: installed shell script will run with options below.

		--config /data/session.json --session --port 8000

#### Run channel server

	1. Run with xpush executable

		docker run -i -t -p 9000:9000 -v /home/stalk/data:/data stalk/xpush:latest xpush --config /data/channel.json --port 9000

	2. Run with already installed shell script

    docker run -d -p 9000:9000 -v /home/stalk/data:/data stalk/xpush:latest /bin/bash -c xpush-channel.sh

>**Note**: installed shell script will run with options below.

		--config /data/channel.json --port 9000