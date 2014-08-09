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

# Install mongodb
RUN \
echo -e "\
[mongodb]\n\
name=MongoDB Repository\n\
baseurl=http://downloads-distro.mongodb.org/repo/redhat/os/x86_64/\n\
gpgcheck=0\n\
enabled=1\n" >> /etc/yum.repos.d/mongodb.repo

RUN yum -y install mongodb-org && mkdir -p /data/db

# Install imagemagick
RUN yum -y install ImageMagick-c++ ImageMagick-c++-devel


# Define mountable directories.
VOLUME ["/data"]

# Define working directory.
WORKDIR /data

# Run redis
CMD ["redis-server", "/etc/redis/redis.conf"]

# Run mongodb
CMD ["mongod"]

# Run XPUSH Servers
RUN \
  npm install -g xpush forever && \
  cd /usr/local/lib/node_modules/xpush && \
  forever start ./bin/xpush --config ./config-blank.json --port 8000 --data /data --session && \
  forever start ./bin/xpush --config ./config-blank.json --port 9000 --data /data

# Expose ports.
#   - 27017 : mongodb process
#   - 28017 : mongodb http
#   - 6379  : redis process
#   - 8000  : xpush session server
#   - 9000  : xpush channel server
EXPOSE 27017
EXPOSE 28017
EXPOSE 6379
EXPOSE 8000
EXPOSE 9000
