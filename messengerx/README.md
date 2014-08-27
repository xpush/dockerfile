## MessengerX web server's Dockerfile

MessengerX is an instant message application for **web browser** (only HTML5 supported). 

For more detail information, Visit [MessengerX](https://github.com/xpush/messengerX) project site.

This repository contains **Dockerfile** of [MessengerX](https://github.com/xpush/messengerX) published to the public [Docker Registry](https://index.docker.io/).


### Installation

1. Install [Docker](https://www.docker.io/).

2. Download [trusted build](https://index.docker.io/u/stalk/messengerx/) from public [Docker Registry](https://index.docker.io/): `docker pull stalk/messegnerx`

   (alternatively, you can build an image from Dockerfile: `docker build -t="stalk/messegnerx" github.com/xpush/dockerfile/messegnerx`)


### Usage

#### Run `messengerx`

    docker run -d --name xpush -p 80:80 stalk/messengerx