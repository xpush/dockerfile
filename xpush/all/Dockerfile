#
# XPush standalone (all-in-one) Dockerfile
#
# https://github.com/xpush/dockerfile/xpush/all
#

FROM    centos:centos6

# Install packages
RUN \
  yum -y install wget && \
  yum -y install tar && \
  yum -y install gcc && \
  yum -y install gcc-c++

# Install java
RUN yum -y install java-1.7.0-openjdk.x86_64

# Install zookeeper
RUN \
  mkdir -p /home/root && \
  cd /home/root && \
  wget http://apache.mirror.cdnetworks.com/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz && \
  tar xvf zookeeper-3.4.6.tar.gz && \
  mkdir -p /etc/zookeeper && \
  cp zookeeper-3.4.6/conf/zoo_sample.cfg /etc/zookeeper/zoo.cfg

# Install node.js
RUN \
  wget http://nodejs.org/dist/v0.10.31/node-v0.10.31.tar.gz && \
  tar zxf node-v0.10.31.tar.gz && \
  cd node-v0.10.31 && \
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

RUN yum -y install mongodb-org && mkdir -p /data/db && mkdir -p /data/log

# Install imagemagick
RUN yum -y install ImageMagick-c++ ImageMagick-c++-devel

# Define mountable directories.
VOLUME ["/data"]

# Define working directory.
WORKDIR /data

# Install XPUSH Servers
RUN export USER=root && export HOME=/data && npm install -g xpush forever

# Make xpush-stand-alone.sh

RUN \
printf "\
#!/usr/bin/env bash \n\
#Start zookeeper \n\
cd /home/root/zookeeper-3.4.6/bin && \n\
./zkServer.sh start /etc/zookeeper/zoo.cfg \n\
# Start redis \n\
nohup redis-server /etc/redis/redis.conf > /data/log/redis.log 2>&1 < /dev/null & \n\
\n\
# Check redis is running \n\
echo -ne \"Starting redis \" \n\
REDIS_PROCESS_NUM=\$(netstat -nap | grep 6379 | grep redis | wc -l) \n\
while [ \$REDIS_PROCESS_NUM -eq 0 ] \n\
do\n\
  echo -n .\n\
  REDIS_PROCESS_NUM=\$(netstat -nap | grep 6379 | grep redis | wc -l)\n\
  sleep 1\n\
done\n\
echo \"  STARTED\"\n\
\n\
# Start mongo\n\
nohup mongod --smallfiles > /data/log/mongo.log 2>&1 < /dev/null &\n\
\n\
# Check mongo is running\n\
echo -ne \"\\\nStarting mongodb \"\n\
MAX_SEC=60\n\
WATING_SEC=0\n\
HOST=\$2
MONGO_PROCESS_NUM=\$(netstat -nap | grep 27017 | grep mongo | wc -l)\n\
while [ \$WATING_SEC -le \$MAX_SEC -a \$MONGO_PROCESS_NUM -eq 0 ]\n\
do\n\
  echo -n .\n\
  MONGO_PROCESS_NUM=\$(netstat -nap | grep 27017 | grep mongo | wc -l)\n\
  sleep 1\n\
  WATING_SEC=$((WATING_SEC + 1))\n\
done\n\
echo \"  STARTED\"\n\
\n\
# Set HOST\n\
if [ \"\$1\" == \"--host\" -a -n \"\$2\" ]\n\
then\n\
  HOST=$(echo \"--host \$HOST\")\n\
else\n\
  HOST=\"\"\n\
fi\n\
# Start xpush\n\
cd /usr/local/lib/node_modules/xpush\n\
nohup ./bin/xpush --config ./config-blank.json --port 8000 --data /data --session \$HOST > /data/log/session.log 2>&1 < /dev/null &\n\
nohup ./bin/xpush --config ./config-blank.json --port 9000 --data /data \$HOST > /data/log/channel.log 2>&1 < /dev/null &\n\
echo -ne \"\\\nTailing log\"\n\
sleep 1\n\
tail -f /data/log/channel.log" > /usr/bin/xpush-stand-alone.sh

RUN chmod 775 /usr/bin/xpush-stand-alone.sh

CMD xpush-stand-alone.sh

# Expose ports.
#   - 27017 : mongodb process
#   - 28017 : mongodb http
#   - 6379  : redis process
#   - 8000  : xpush session server
#   - 9000  : xpush channel server
#   - 2181  : zookeeper port
EXPOSE 27017
EXPOSE 28017
EXPOSE 6379
EXPOSE 8000
EXPOSE 9000
EXPOSE 2181