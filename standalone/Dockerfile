#
# XPush standalone (all-in-one) Dockerfile
#
# https://github.com/xpush/dockerfile/standalone
#

FROM    centos:centos6

# Install packages
RUN \
  yum -y update  && \
  yum -y groupinstall "Development Tools"
  yum install -y wget


# Install node.js
RUN \
  wget http://nodejs.org/dist/v0.10.30/node-v0.10.30.tar.gz && \
  tar zxf node-v0.10.30.tar.gz && \
  cd node-v0.10.30 && \
  ./configure && \
  make && \
  make install

# Install Redis.
RUN \
  cd /tmp && \
  wget http://download.redis.io/redis-stable.tar.gz && \
  tar xvzf redis-stable.tar.gz && \
  cd redis-stable && \
  make && \
  make install && \
  cp -f src/redis-sentinel /usr/local/bin && \
  mkdir -p /etc/redis && \
  cp -f *.conf /etc/redis && \
  rm -rf /tmp/redis-stable* && \
  sed -i 's/^\(bind .*\)$/# \1/' /etc/redis/redis.conf && \
  sed -i 's/^\(daemonize .*\)$/# \1/' /etc/redis/redis.conf && \
  sed -i 's/^\(dir .*\)$/# \1\ndir \/data/' /etc/redis/redis.conf && \
  sed -i 's/^\(logfile .*\)$/# \1/' /etc/redis/redis.conf

# Install imagemagick
RUN sudo yum install ImageMagick-c++ ImageMagick-c++-devel

# Define mountable directories.
VOLUME ["/data"]

# Define working directory.
WORKDIR /data

EXPOSE  8000
EXPOSE  9000
