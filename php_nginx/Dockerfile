FROM phusion/baseimage:latest

# Ensure UTF-8
RUN locale-gen en_US.UTF-8
ENV LANG       en_US.UTF-8
ENV LC_ALL     en_US.UTF-8

ENV HOME /root
RUN /etc/my_init.d/00_regen_ssh_host_keys.sh

CMD ["/sbin/my_init"]

#Install PHP
RUN apt-get update && apt-get upgrade -y -o Dpkg::Options::="--force-confdef" && apt-get autoremove -y

RUN apt-get install software-properties-common
RUN add-apt-repository ppa:ondrej/php
RUN apt-get update

RUN apt-get install -y vim curl wget build-essential python-software-properties \
    libcurl4-openssl-dev pkg-config libssl-dev libsslcommon2-dev

RUN apt-get install -y php7.3-cli php7.3-fpm php7.3-mysql php7.3-curl \
    php7.3-intl php7.3-xml php7.3-gd php7.3-mbstring php7.3-zip \
    php7.3-dev php-pear php-xdebug php7.3-soap php-imagick php7.3-bcmath

RUN sed -i "s/;date.timezone =.*/date.timezone = UTC/" /etc/php/7.3/fpm/php.ini
RUN sed -i 's/memory_limit\ =\ 128M/memory_limit\ =\ 2G/g' /etc/php/7.3/fpm/php.ini
RUN sed -i 's/\;date\.timezone\ =/date\.timezone\ =\ Asia\/Ho_Chi_Minh/g' /etc/php/7.3/fpm/php.ini
RUN sed -i 's/upload_max_filesize\ =\ 2M/upload_max_filesize\ =\ 200M/g' /etc/php/7.3/fpm/php.ini
RUN sed -i 's/post_max_size\ =\ 8M/post_max_size\ =\ 200M/g' /etc/php/7.3/fpm/php.ini
RUN sed -i 's/max_execution_time\ =\ 30/max_execution_time\ =\ 3600/g' /etc/php/7.3/fpm/php.ini
RUN sed -i 's/\;error_log\ =\ syslog/error_log\ =\ syslog/g' /etc/php/7.3/fpm/php.ini

# Install Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
RUN composer --version

RUN apt-get install -y \
    git \
    unzip

#Install Nginx
RUN echo 'deb http://nginx.org/packages/mainline/ubuntu/ xenial nginx' >> /etc/apt/sources.list \
    && echo 'deb-src http://nginx.org/packages/mainline/ubuntu/ xenial nginx' >> /etc/apt/sources.list

RUN wget http://nginx.org/keys/nginx_signing.key
RUN apt-key add nginx_signing.key

RUN apt-get update
RUN apt-get install -y nginx

#Install Nodejs
RUN curl -sL https://deb.nodesource.com/setup_8.x | bash -
RUN apt-get install -y nodejs

RUN echo "daemon off;" >> /etc/nginx/nginx.conf
RUN sed -i -e "s/;daemonize\s*=\s*yes/daemonize = no/g" /etc/php/7.3/fpm/php-fpm.conf
RUN sed -i "s/;cgi.fix_pathinfo=1/cgi.fix_pathinfo=0/" /etc/php/7.3/fpm/php.ini

ADD build/nginx/upstream.conf     /etc/nginx/conf.d/
ADD build/nginx/web_laravel.conf     /etc/nginx/conf.d/
RUN rm /etc/nginx/conf.d/default.conf
ADD build/php/*.conf  /etc/php/7.3/fpm/pool.d/

#add service commands
RUN mkdir           /etc/service/nginx
ADD build/nginx.sh  /etc/service/nginx/run
RUN chmod +x        /etc/service/nginx/run
RUN mkdir           /etc/service/phpfpm
ADD build/phpfpm.sh /etc/service/phpfpm/run
RUN chmod +x        /etc/service/phpfpm/run

#Set user role
RUN usermod -u 1000 www-data

EXPOSE 80

WORKDIR /var/www/