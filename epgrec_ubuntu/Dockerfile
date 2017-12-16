FROM ubuntu:16.04
MAINTAINER techlabxe

ENV TZ Asia/Tokyo
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN apt update
RUN apt install -y git gcc g++ make pkg-config apt-utils linux-headers-generic 

RUN apt install -y at
RUN sed -i".org" 's/www-data//g' /etc/at.deny
    
RUN apt install -y nginx php7.0-fpm php7.0-mbstring php-xml php-mysql
RUN apt install -y libboost-dev libboost-thread-dev libboost-filesystem-dev libpcsclite-dev 

RUN mkdir -p /home/drv
WORKDIR /home/drv
RUN git clone https://github.com/stz2012/recfsusb2n.git
WORKDIR /home/drv/recfsusb2n
RUN make && make install

RUN apt install -y usbutils
COPY 89-fsusb.rules /lib/udev/rules.d/

# nginx の設定を書き込む
COPY ./nginx/epgrec     /etc/nginx/sites-available/
COPY ./nginx/nginx.conf /etc/nginx/nginx.conf
RUN rm /etc/nginx/sites-enabled/default && ln -s /etc/nginx/sites-available/epgrec /etc/nginx/sites-enabled/default

RUN mkdir -p /home/epgdump
COPY epgdump /home/epgdump
WORKDIR /home/epgdump
RUN make && make install

RUN mkdir -p /var/www/epgrec
ADD epgrecUNA_151114.tar.gz /tmp
ADD epgrecUNA151114Fix1.tar.gz /tmp
ADD epgrecUNA151114Fix2.tar.gz /tmp
RUN mv /tmp/epgrec /var/www/
WORKDIR /var/www/epgrec
COPY epgrec/config.php /var/www/epgrec/config.php

RUN usermod -s /bin/bash www-data
RUN usermod -aG video www-data 
RUN usermod -aG irc www-data

ENTRYPOINT service atd start && service php7.0-fpm start && nginx -g "daemon off;"

EXPOSE 80

