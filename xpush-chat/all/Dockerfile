#
# XPush standalone (all-in-one) Dockerfile
#
# https://github.com/xpush/dockerfile/xpush/all
#

FROM    centos:centos6

RUN mkdir -p /applications
ENV XCHAT_HOME /applications/xpush-chat
ENV APP_HOME /applications

# Install packages
RUN \
  yum -y install wget && \
  yum -y install tar && \
  yum -y install gcc && \
  yum -y install xz && \
  yum -y install git && \
  yum -y install gcc-c++

# Install java
RUN yum -y install java-1.7.0-openjdk.x86_64

# Install zookeeper
RUN \
  mkdir -p ${APP_HOME} && \
  cd ${APP_HOME} && \
  wget http://apache.mirror.cdnetworks.com/zookeeper/zookeeper-3.4.9/zookeeper-3.4.9.tar.gz && \
  tar xvf zookeeper-3.4.9.tar.gz && \
  mkdir -p /etc/zookeeper && \
  cp zookeeper-3.4.9/conf/zoo_sample.cfg /etc/zookeeper/zoo.cfg && \
  mv zookeeper-3.4.9 zookeeper && \
  rm -rf zookeeper-3.4.9.tar.gz

# Install node.js
# gpg keys listed at https://github.com/nodejs/node
RUN set -ex \
  && for key in \
    9554F04D7259F04124DE6B476D5A82AC7E37093B \
    94AE36675C464D64BAFA68DD7434390BDBE9B9C5 \
    0034A06D9D9B0064CE8ADF6BF1747F4AD2306D93 \
    FD3A5288F042B6850C66B31F09FE44734EB7990E \
    71DCFD284A79C3B38668286BC97EC7A07EDE3FC1 \
    DD8F2338BAE7501E3DD5AC78C273792F7D83545D \
    B9AE9905FFD7803F25714661B63B535A4C206CA9 \
    C4F0DFFF4E8C1A8236409D08E73BC641CC11F4C8 \
  ; do \
    gpg --keyserver ha.pool.sks-keyservers.net --recv-keys "$key"; \
  done

ENV NPM_CONFIG_LOGLEVEL info
ENV NODE_VERSION 6.9.1

RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.xz" \
  && curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" \
  && gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.txt.asc \
  && grep " node-v$NODE_VERSION-linux-x64.tar.xz\$" SHASUMS256.txt | sha256sum -c - \
  && tar -xJf "node-v$NODE_VERSION-linux-x64.tar.xz" -C /usr/local --strip-components=1 \
  && rm "node-v$NODE_VERSION-linux-x64.tar.xz" SHASUMS256.txt.asc SHASUMS256.txt \
  && ln -s /usr/local/bin/node /usr/local/bin/nodejs

# Install Redis.
RUN \
mkdir -p /etc/redis/run && mkdir -p /etc/redis/logs && mkdir -p /etc/redis/6379

RUN \
  cd /tmp && \
  wget http://download.redis.io/redis-stable.tar.gz && \
  tar xvzf redis-stable.tar.gz && \
  cd redis-stable && \
  make && \
  make install && \
  cp -f src/redis-server /usr/local/bin

# Create redis.conf
RUN \
printf "\
bind 127.0.0.1 \n\
protected-mode yes \n\
port 6379 \n\
tcp-backlog 511 \n\
timeout 0 \n\
tcp-keepalive 300 \n\
daemonize yes \n\
pidfile /etc/redis/run/6379.pid \n\
loglevel notice \n\
logfile /etc/redis/logs/6379.log \n\
databases 16 \n\
dbfilename dump.rdb \n\
dir /etc/redis/6379 \n\
" > /etc/redis/redis.conf

# Install mongodb
RUN \
echo -e "\
[mongodb-org-3.2]\n\
name=MongoDB Repository\n\
baseurl=https://repo.mongodb.org/yum/redhat/6/mongodb-org/3.2/x86_64/\n\
gpgcheck=0\n\
enabled=1\n" >> /etc/yum.repos.d/mongodb-org-3.2.repo

RUN yum -y install mongodb-org && mkdir -p /data/db && mkdir -p /data/log

# Define mountable directories.
VOLUME ["/data"]

# Define working directory.
WORKDIR ${APP_HOME}

RUN \
  mkdir -p /var/logs/xpush

# Install XPUSH Servers
RUN \
  cd ${APP_HOME} && \
  git clone https://github.com/xpush/xpush-chat.git && \
  cd xpush-chat && \
  npm install

# Install Forever
RUN npm install -g forever

# Make xchat-stand-alone.sh

RUN \
printf "\
#!/usr/bin/env bash \n\
#Start zookeeper \n\
cd ${APP_HOME}/zookeeper/bin && \n\
./zkServer.sh start /etc/zookeeper/zoo.cfg \n\
# Start redis \n\
redis-server /etc/redis/redis.conf \n\
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
echo -ne \"Starting mongo \" \n\
mongod --smallfiles --fork --logpath /var/logs/mongo.log\n\
\n\
# Set HOST\n\
while [ \"\$1\" != \"\" ]\n\
do\n\
  case \$1 in\n\
    -h | --host ) shift && HOST=\$1\n\
      ;;\n\
  esac\n\
  shift\n\
done\n\
echo\n\
if [ -n \"\$HOST\" ]\n\
then\n\
  HOST=$(echo \"--host \$HOST\")\n\
  echo Run with options : \$HOST\n\
else\n\
  HOST=\"\"\n\
fi\n\

# Start xpush\n\
cd ${XCHAT_HOME}\n\
forever start -al /var/logs/xpush/session.log ./bin/session-server --port 8000 --home /data \n\
forever start -al /var/logs/xpush/channel.log ./bin/channel-server --port 9000 --home /data \n\
echo -ne \"\\\nTailing log\"\n\
sleep 1\n\
tail -f /var/logs/xpush/channel.log" > /usr/bin/xchat-stand-alone.sh

RUN chmod 775 /usr/bin/xchat-stand-alone.sh

CMD /usr/bin/xchat-stand-alone.sh

# Expose ports.
#   - 6379  : redis process
#   - 8000  : xpush session server
#   - 9000  : xpush channel server
#   - 2181  : zookeeper port
#   - 22 : SSH PORT
EXPOSE 6379
EXPOSE 8000
EXPOSE 9000
EXPOSE 2181
EXPOSE 22