FROM ruby:2.3

MAINTAINER Nbyun '835482737@qq.com'
ADD sources.list /etc/apt/
RUN apt-get update

RUN set -ex \
  && apt-get install -y --no-install-recommends --allow-unauthenticated supervisor \
  && mkdir -p /var/log/supervisor \
  && { \
    echo '[supervisord]'; \
    echo 'nodaemon=true'; \
  } >> /etc/supervisor/conf.d/supervisord.conf

RUN apt-get install -y --force-yes imagemagick
RUN apt-get install -y --force-yes libmagickwand-dev
RUN apt-get install -y --force-yes tesseract-ocr
RUN apt-get install -y --force-yes tesseract-ocr-chi-sim
RUN apt-get install -y --force-yes python3-apt python3-software-properties software-properties-common
RUN apt-get install -y --allow-downgrades --allow-unauthenticated perl-base=5.22.1-9ubuntu0.6
RUN apt-get install -y perlapi-5.22.1 libtext-iconv-perl dictionaries-common
RUN apt-get install -y myspell-tl=0.4-0-17
RUN apt-get install -y --force-yes libacl1 libenchant1c2a acl libgtkspell3-3-0 libsane libgtkspellmm-3.0-0v5 gimagereader

RUN mkdir /app
WORKDIR /app
VOLUME /app

EXPOSE 22 80 3000
CMD ["/usr/bin/supervisord"]
