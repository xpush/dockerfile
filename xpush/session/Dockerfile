#
# XPush standalone (all-in-one) Dockerfile
#
# https://github.com/xpush/dockerfile/standalone
#

FROM    centos:centos6

# Install packages
RUN \
  yum -y update  && \
  yum -y groupinstall "Development Tools" && \
  yum -y install wget


# Install node.js
RUN \
  wget http://nodejs.org/dist/v0.10.30/node-v0.10.30.tar.gz && \
  tar zxf node-v0.10.30.tar.gz && \
  cd node-v0.10.30 && \
  ./configure && \
  make && \
  make install

# Install imagemagick
RUN yum -y install ImageMagick-c++ ImageMagick-c++-devel

# Define mountable directories.
VOLUME ["/data"]

# Define working directory.
WORKDIR /data

# Run XPUSH Servers
RUN \
  npm install -g xpush forever && \
  cd /usr/local/lib/node_modules/xpush && \
  forever start ./bin/xpush --config ./config-blank.json --port 8000 --data /data --session

# Expose ports.
#   - 8000  : xpush session server
EXPOSE 8000