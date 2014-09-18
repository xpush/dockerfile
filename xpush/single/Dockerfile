#
# XPush standalone (single) Dockerfile
#
# https://github.com/xpush/dockerfile/xpush/single
#

FROM    centos:centos6

# Install packages
RUN \
  yum -y install wget && \
  yum -y install tar && \
  yum -y install gcc && \
  yum -y install gcc-c++

# Install node.js
RUN \
  wget http://nodejs.org/dist/v0.10.31/node-v0.10.31.tar.gz && \
  tar zxf node-v0.10.31.tar.gz && \
  cd node-v0.10.31 && \
  ./configure && \
  make && \
  make install

# Install imagemagick
RUN yum -y install ImageMagick-c++ ImageMagick-c++-devel

# Define mountable directories.
VOLUME ["/data"]

# Define working directory.
WORKDIR /data

# Install XPUSH Servers
RUN export USER=root && mkdir -p /data && export HOME=/data && npm install -g xpush forever

# Make xpush-session.sh
RUN \
printf "\
mkdir -p /data/log && \n\
cd /usr/local/lib/node_modules/xpush && \n\
forever start -al /data/log/session.log ./bin/xpush --config /data/session.json --port 8000 --data /data --session \n\
echo -ne \"\\\nTailing log\"\n\
sleep 5\n\
tail -f /data/log/session.log" > /usr/bin/xpush-session.sh

# Make xpush-channel.sh
RUN \
printf "\
mkdir -p /data/log && \n\
cd /usr/local/lib/node_modules/xpush && \n\
forever start -al /data/log/channel.log ./bin/xpush --config /data/channel.json --port 9000 --data /data\n\
echo -ne \"\\\nTailing log\"\n\
sleep 5\n\
tail -f /data/log/channel.log" > /usr/bin/xpush-channel.sh

RUN chmod 775 /usr/bin/xpush-*.sh

# Expose ports.
#   - 8000  : xpush session server
#   - 9000  : xpush channel server
EXPOSE 8000
EXPOSE 9000