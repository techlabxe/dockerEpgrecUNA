FROM centos:7.4.1708
MAINTAINER techlabxe

ENV TZ Asia/Tokyo
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN yum update -y
RUN yum install -y git gcc gcc-c++ make

RUN rpm -ivh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
RUN yum install -y nginx
RUN yum install -y at
# RUN sed -i".org" 's/www-data//g' /etc/at.deny

# php用の準備.
RUN yum install -y epel-release && yum update -y epel-release && \
    rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-7.rpm && yum -y update remi-release
#
RUN yum install --enablerepo=remi-php56 -y php php-devel php-fpm php-mbstring php-xml php-mysql php-process php-pdo php-gd 
#RUN yum install -y --enablerepo=remi-php56 php-fpm php-mbstring php-xml php-mysql php-process
#RUN cp -p /etc/php-fpm.d/www.conf /etc/php-fpm.d/www.conf.org
RUN sed -i -e 's/user = apache/user = nginx/' /etc/php-fpm.d/www.conf && \
    sed -i -e 's/group = apache/group = nginx/' /etc/php-fpm.d/www.conf && \
    sed -i -e 's/;listen.owner = nobody/listen.owner = nginx/' /etc/php-fpm.d/www.conf && \
    sed -i -e 's/;listen.group = nobody/listen.group = nginx/' /etc/php-fpm.d/www.conf
    
RUN    sed -i -e 's/listen = 127.0.0.1:9000/listen = \/var\/run\/php-fpm\/php-fpm.sock/' /etc/php-fpm.d/www.conf && \
    sed -i -e 's/;date.timezone =/date.timezone = "Asia\/Tokyo"/'  /etc/php.ini 

RUN yum install -y boost-devel pcsc-lite-devel usbutils

RUN mkdir -p /home/drv
WORKDIR /home/drv
RUN git clone https://github.com/stz2012/recfsusb2n.git
WORKDIR /home/drv/recfsusb2n
RUN make && make install

COPY 89-fsusb.rules /lib/udev/rules.d/

# nginx の設定を書き込む
COPY ./nginx/epgrec.conf     /etc/nginx/conf.d/
COPY ./nginx/nginx.conf /etc/nginx/nginx.conf
RUN mv /etc/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf.bak


ADD epgdumpUNA160127.tar.gz /tmp
RUN mv /tmp/epgdump /home
WORKDIR /home/epgdump
RUN make && make install

ADD epgrecUNA_151114.tar.gz /tmp
ADD epgrecUNA151114Fix1.tar.gz /tmp
ADD epgrecUNA151114Fix2.tar.gz /tmp
RUN mkdir -p /var/www
#COPY epgrec /var/www/epgrec
RUN mv /tmp/epgrec /var/www/
WORKDIR /var/www/epgrec
COPY epgrec/config.php /var/www/epgrec/config.php

RUN chown nginx:nginx -R /var/www/epgrec && \
    chmod 755 -R /var/www/epgrec  && \
    chmod 777 -R /var/www/epgrec/templates_c && \
    chmod 777 -R /var/www/epgrec/thumbs && \
    chmod 777 -R /var/www/epgrec/video  && \
    chmod 777 -R /var/www/epgrec/cache  && \
    chmod 777 -R /var/www/epgrec/settings
RUN usermod -s /bin/bash nginx && \
    usermod -aG video nginx

RUN echo "29 */2  *  *  * nginx   /var/www/epgrec/shepherd.php" >> /etc/crontab

ENTRYPOINT atd && php-fpm && nginx -g "daemon off;"

EXPOSE 80

