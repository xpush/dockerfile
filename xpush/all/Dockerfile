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
  yum -y install xz && \
  yum -y install git && \
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
ENV NODE_VERSION 4.5.0

RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.xz" \
  && curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" \
  && gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.txt.asc \
  && grep " node-v$NODE_VERSION-linux-x64.tar.xz\$" SHASUMS256.txt | sha256sum -c - \
  && tar -xJf "node-v$NODE_VERSION-linux-x64.tar.xz" -C /usr/local --strip-components=1 \
  && rm "node-v$NODE_VERSION-linux-x64.tar.xz" SHASUMS256.txt.asc SHASUMS256.txt \
  && ln -s /usr/local/bin/node /usr/local/bin/nodejs

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

# Define mountable directories.
VOLUME ["/data"]

# Define working directory.
WORKDIR /data

RUN \
  mkdir -p /var/logs/xpush

# Install node-gyp
RUN export USER=root && npm install -g node-gyp

# Install XPUSH Servers
RUN export USER=root && npm install -g xpush forever

# Make xpush-stand-alone.sh

RUN \
printf "\
#!/usr/bin/env bash \n\
#Start zookeeper \n\
cd /home/root/zookeeper-3.4.6/bin && \n\
./zkServer.sh start /etc/zookeeper/zoo.cfg \n\
# Start redis \n\
nohup redis-server /etc/redis/redis.conf > /var/logs/xpush/redis.log 2>&1 < /dev/null & \n\
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
cd /usr/local/lib/node_modules/xpush\n\
nohup ./bin/start  --port 8000 --home /data --session \$HOST > /var/logs/xpush/session.log 2>&1 < /dev/null &\n\
nohup ./bin/start --port 9000 --channel --home /data \$HOST > /var/logs/xpush/channel.log 2>&1 < /dev/null &\n\
echo -ne \"\\\nTailing log\"\n\
sleep 1\n\
tail -f /var/logs/xpush/channel.log" > /usr/bin/xpush-stand-alone.sh

RUN chmod 775 /usr/bin/xpush-stand-alone.sh

CMD /usr/bin/xpush-stand-alone.sh

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