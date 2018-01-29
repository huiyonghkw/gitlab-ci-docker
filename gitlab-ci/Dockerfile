FROM gitlab/gitlab-runner:alpine

MAINTAINER bravist <chenghuiyong1987@gmail.com>

# https://pkgs.alpinelinux.org/packages

# Mirror mirror switch to Ali-OSM (Alibaba Open Source Mirror Site) - http://mirrors.aliyun.com/
RUN echo 'http://mirrors.aliyun.com/alpine/latest-stable/main' > /etc/apk/repositories \
	&& echo '@community http://mirrors.aliyun.com/alpine/latest-stable/community' >> /etc/apk/repositories \
	&& echo '@testing http://mirrors.aliyun.com/alpine/edge/testing' >> /etc/apk/repositories

# https://github.com/matriphe/docker-alpine-php/blob/master/7.0/FPM/Dockerfile
# Environments
ENV TIMEZONE            Asia/Shanghai
ENV PHP_MEMORY_LIMIT    512M
ENV MAX_UPLOAD          50M
ENV PHP_MAX_FILE_UPLOAD 200
ENV PHP_MAX_POST        100M
ENV COMPOSER_ALLOW_SUPERUSER 1


# Mirror mirror switch to Alpine Linux - http://dl-4.alpinelinux.org/alpine/
RUN apk update \
	&& apk upgrade \
	&& apk add \
		curl \
		tzdata \
	    php7@community \
	    php7-dev@community \
	    php7-apcu@community \
	    php7-bcmath@community \
	    php7-xmlwriter@community \
	    php7-ctype@community \
	    php7-curl@community \
	    php7-exif@community \
	    php7-iconv@community \
	    php7-intl@community \
	    php7-json@community \
	    php7-mbstring@community\
	    php7-opcache@community \
	    php7-openssl@community \
	    php7-pcntl@community \
	    php7-pdo@community \
	    php7-mysqlnd@community \
	    php7-mysqli@community \
	    php7-pdo_mysql@community \
	    php7-pdo_pgsql@community \
	    php7-phar@community \
	    php7-posix@community \
	    php7-session@community \
	    php7-xml@community \
	    php7-simplexml@community \
	    php7-mcrypt@community \
	    php7-xsl@community \
	    php7-zip@community \
	    php7-zlib@community \
	    php7-dom@community \
	    php7-redis@community\
	    php7-tokenizer@community \
	    php7-gd@community \
		php7-mongodb@testing \
		php7-fileinfo@community \
		php7-zmq@community \
		php7-memcached@community \
		openssh \
 	&& cp /usr/share/zoneinfo/${TIMEZONE} /etc/localtime \
	&& echo "${TIMEZONE}" > /etc/timezone \
	&& apk del tzdata \
 	&& rm -rf /var/cache/apk/*

# https://github.com/docker-library/php/issues/240
# https://gist.github.com/guillemcanal/be3db96d3caa315b4e2b8259cab7d07e
# https://forum.alpinelinux.org/forum/installation/php-iconv-issue

RUN apk add --no-cache --repository http://dl-3.alpinelinux.org/alpine/edge/testing gnu-libiconv
ENV LD_PRELOAD /usr/lib/preloadable_libiconv.so php
RUN rm -rf /var/cache/apk/*

# Set environments
RUN sed -i "s|;*date.timezone =.*|date.timezone = ${TIMEZONE}|i" /etc/php7/php.ini && \
	sed -i "s|;*memory_limit =.*|memory_limit = ${PHP_MEMORY_LIMIT}|i" /etc/php7/php.ini && \
	sed -i "s|;*upload_max_filesize =.*|upload_max_filesize = ${MAX_UPLOAD}|i" /etc/php7/php.ini && \
	sed -i "s|;*max_file_uploads =.*|max_file_uploads = ${PHP_MAX_FILE_UPLOAD}|i" /etc/php7/php.ini && \
	sed -i "s|;*post_max_size =.*|post_max_size = ${PHP_MAX_POST}|i" /etc/php7/php.ini && \
	sed -i "s|;*cgi.fix_pathinfo=.*|cgi.fix_pathinfo= 0|i" /etc/php7/php.ini

# composer
RUN curl -sS https://getcomposer.org/installer | \
    php -- --install-dir=/usr/bin/ --filename=composer

RUN mkdir -p /root/.ssh
RUN composer global require laravel/envoy

VOLUME ["/root/.ssh"]

CMD ["run", "--user=root", "--working-directory=/root"]