FROM ruby:2.3.1

RUN set -ex
RUN sed -i '2,3d' /etc/apt/sources.list 

# RUN apt-get update

# DEBIAN_MIRROR BEGIN
RUN set -ex
RUN sed -i 's/security.debian.org/mirrors.aliyun.com\/debian-security/g' /etc/apt/sources.list
RUN sed -i 's/httpredir.debian.org/mirrors.aliyun.com/g' /etc/apt/sources.list
RUN apt-get update

# DEBIAN_MIRROR END

# DEBIAN_DPKGS BEGIN
RUN set -ex
RUN apt-get install -y --no-install-recommends ca-certificates curl wget git-core htop \
    screen apt-transport-https vim libmysqlclient-dev bzip2 libfontconfig mysql-client \
    rsync
# DEBIAN_DPKGS END

# SUPERVISOR BEGIN
RUN set -ex 
RUN apt-get install -y --no-install-recommends supervisor
RUN mkdir -p /var/log/supervisor
RUN { \
    echo '[supervisord]'; \
    echo 'nodaemon=true'; \
  } >> /etc/supervisor/conf.d/supervisord.conf

# SUPERVISOR END

# SSH_SERVER BEGIN
RUN set -ex
RUN apt-get install -y --no-install-recommends openssh-server
RUN mkdir -p /var/run/sshd
RUN { \
    echo '[program:sshd]'; \
    echo 'command=/usr/sbin/sshd -D'; \
  } >> /etc/supervisor/conf.d/sshd.conf

# SSH_SERVER END

# CRON_SERVER BEGIN
RUN set -ex
RUN apt-get install -y --no-install-recommends cron 
RUN { \
    echo '[program:cron]'; \
    echo 'command=/usr/sbin/cron -f'; \
  } >> /etc/supervisor/conf.d/cron.conf

# CRON_SERVER END

# NGINX_SERVER BEGIN
RUN apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62 
RUN echo "deb http://nginx.org/packages/debian/ $(awk -F"[)(]+" '/VERSION=/ {print $2}' /etc/os-release) nginx" >> /etc/apt/sources.list 
RUN apt-get update 
RUN apt-get install --no-install-recommends --no-install-suggests -y \
            ca-certificates \
            nginx
RUN { \
    echo '[program:nginx]'; \
    echo 'command=nginx -g "daemon off;"'; \
  } >> /etc/supervisor/conf.d/nginx.conf 
RUN rm -rf /var/lib/apt/lists/*

# NGINX_SERVER END

# NODE6 BEGIN
ENV NODE_VERSION 6.11.5
ENV NPM_CONFIG_LOGLEVEL info
ENV NODE_HOME /usr/local/node
ENV NODE_BIN="$NODE_HOME/bin"

RUN set -ex
RUN for key in \
    9554F04D7259F04124DE6B476D5A82AC7E37093B \
    94AE36675C464D64BAFA68DD7434390BDBE9B9C5 \
    0034A06D9D9B0064CE8ADF6BF1747F4AD2306D93 \
    FD3A5288F042B6850C66B31F09FE44734EB7990E \
    71DCFD284A79C3B38668286BC97EC7A07EDE3FC1 \
    DD8F2338BAE7501E3DD5AC78C273792F7D83545D \
    B9AE9905FFD7803F25714661B63B535A4C206CA9 \
    C4F0DFFF4E8C1A8236409D08E73BC641CC11F4C8 \
  ; do \
    gpg --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys "$key" || \
    gpg --keyserver hkp://ipv4.pool.sks-keyservers.net --recv-keys "$key" || \
    gpg --keyserver hkp://pgp.mit.edu:80 --recv-keys "$key" ; \
  done

RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.xz" 
RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" 
RUN gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.txt.asc 
RUN grep " node-v$NODE_VERSION-linux-x64.tar.xz\$" SHASUMS256.txt | sha256sum -c - 
RUN mkdir -p "$NODE_HOME" 
RUN tar -xJf "node-v$NODE_VERSION-linux-x64.tar.xz" -C "$NODE_HOME" --strip-components=1 
RUN rm "node-v$NODE_VERSION-linux-x64.tar.xz" SHASUMS256.txt.asc SHASUMS256.txt

ENV PATH $NODE_BIN:$PATH

# NODE6 END

# YARN BEGIN
RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - 
RUN echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list 
RUN apt-get update 
RUN apt-get install --no-install-recommends yarn

# YARN END

# LOCALE BEGIN
RUN set -ex 
RUN apt-get update
RUN apt-get install -y --no-install-recommends locales 
RUN echo "en_US.UTF-8 UTF-8" > /etc/locale.gen 
RUN locale-gen 
RUN update-locale LANG=en_US.UTF-8 LANGUAGE=en_US:en:

ENV LC_ALL en_US.UTF-8

# LOCALE END

######################################################################################################################
################################################# INSTALLING FFMPEG ##################################################


# enable backports repo for ffmpeg
#RUN echo 'deb http://deb.debian.org/debian jessie-backports main' >> /etc/apt/sources.list
RUN mv /etc/apt/sources.list /etc/apt/sources.list.bak 
RUN echo "deb http://mirrors.163.com/debian/ jessie main non-free contrib" >/etc/apt/sources.list
RUN echo "deb-src http://mirrors.163.com/debian/ jessie main non-free contrib" >>/etc/apt/sources.list

RUN apt-get update 
RUN apt-get install -y git build-essential gcc make yasm autoconf automake cmake libtool checkinstall libmp3lame-dev pkg-config libunwind-dev zlib1g-dev libssl-dev libx264-dev

RUN apt-get install -y ffmpegthumbnailer imagemagick --no-install-recommends libc6-dev libgdiplus wget software-properties-common libx264-dev
RUN apt-get clean 
RUN apt-get autoremove -y 
RUN rm -rf /var/lib/apt/lists/*

#RUN RUN apt-add-repository ppa:git-core/ppa && apt-get update && apt-get install -y git


RUN wget https://www.ffmpeg.org/releases/ffmpeg-4.0.2.tar.gz
RUN tar -xzf ffmpeg-4.0.2.tar.gz; rm -r ffmpeg-4.0.2.tar.gz
RUN cd ./ffmpeg-4.0.2; ./configure --enable-gpl --enable-libmp3lame --enable-decoder=mjpeg,png --enable-encoder=png --enable-openssl --enable-nonfree --enable-libx264
 

RUN cd ./ffmpeg-4.0.2; make && make install
#RUN  cd ./ffmpeg-4.0.2; make install
######################################################################################################################
######################################################################################################################

RUN apt-get update
RUN apt-get install -y openjdk-7-jdk

# TIMEZONE BEGIN
RUN set -ex 
RUN echo "Asia/Shanghai" > /etc/timezone 
RUN dpkg-reconfigure -f noninteractive tzdata

# TIMEZONE END

# SSH_AGENT BEGIN
RUN set -ex 
RUN mkdir -p /root/.ssh /root/.bash.d 
RUN { \
    echo 'Host *'; \
    echo 'ServerAliveInterval=15'; \
    echo 'ServerAliveCountMax=6'; \
    echo 'ForwardAgent yes'; \
  } >> /root/.ssh/config 
RUN { \
    echo 'source ~/.bash.d/10_ssh-agent.bash'; \
  } >> /root/.bashrc

RUN set -ex 
RUN cd /root/.bash.d 
RUN curl -SLO https://raw.githubusercontent.com/agideo/docker-baseimage/master/files/bash.d/10_ssh-agent.bash 
RUN cd -

# SSH_AGENT END

# DEBIAN_CLEAN BEGIN
RUN set -ex 
RUN apt-get clean 
RUN rm -rf /var/lib/apt/lists/*

# DEBIAN_CLEAN END

# DOTBASHRC BEGIN
RUN set -ex 
RUN { \
    echo "export TERM=xterm"; \
    echo "export PATH=$(echo $PATH)"; \
  } >> /root/.bashrc

# DOTBASHRC END

# REGISTRY TAOBO BEGIN
RUN npm install -g yarn --registry=https://registry.npm.taobao.org
RUN npm install -g webpack --registry=https://registry.npm.taobao.org
RUN yarn config set registry 'https://registry.npm.taobao.org'
RUN npm config set registry 'https://registry.npm.taobao.org'
# REGISTRY TAOBO END

RUN set -ex 
RUN echo "export PATH=$(echo $PATH)" >> /root/.bashrc

RUN mkdir /app
WORKDIR /app
VOLUME /app

ADD ./files/Gemfile* /app/
ADD ./files/vendor /app/vendor
RUN bundle install

EXPOSE 22 80 3000
CMD ["/usr/bin/supervisord"]
