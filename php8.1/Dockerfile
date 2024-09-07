FROM php:8.1-fpm-alpine3.16
LABEL MAINTAINER="leon19910505@163.com"
 
#composer
RUN curl -sSL https://getcomposer.org/download/latest-stable/composer.phar -o /usr/local/bin/composer && chmod +x /usr/local/bin/composer
#composer镜像
RUN composer config -g repo.packagist composer https://mirrors.aliyun.com/composer
#安装一些工具软件
RUN sed -i "s/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g" /etc/apk/repositories
RUN apk update  && apk add --no-cache nginx vim git zsh tzdata
#修改时区
RUN cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
     echo 'Asia/Shanghai' > /etc/timezone && \
     echo "date.timezone=Asia/Shanghai" > /usr/local/etc/php/conf.d/timezone.ini 
#ohmyzsh定制安装
RUN sh -c "$(wget -O- https://gitee.com/leon19910505/ohmyzsh/raw/master/tools/install.sh)"
 
#下载扩展安装脚本
ADD https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions /usr/local/bin/
#给脚本可执行权限
RUN chmod +x /usr/local/bin/install-php-extensions
#安装扩展,这里分写多行是因为想充分利用缓存,如果某一个扩展安装失败下次重试的时候前面的构建就可以直接跳过
#如果担心镜像体积,可以构建完成之后优化,消除镜像层
RUN install-php-extensions redis bcmath pdo_mysql xdebug opcache 
RUN install-php-extensions gd imagick pdo_pgsql swoole
RUN install-php-extensions zip uuid rdkafka
RUN install-php-extensions mcrypt 
 
#nginx配置文件
ADD nginx.conf /etc/nginx/http.d/default.conf
 
#xdebug配置文件
ADD xdebug.ini /usr/local/etc/php/conf.d/xdebug.ini
 
#定义一个sh脚本作为Entrypoint,因为要同时执行php-fpm和nginx
#这里的策略是把PHP以daemon模式运行,nginx挂起
ADD entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
#xdebug从cli启动的配置
RUN echo 'export XDEBUG_CONFIG="xdebug.mode=debug xdebug.start_with_request=yes xdebug.client_host=host.docker.internal xdebug.client_port=9000 xdebug.idekey = PHPSTORM"' >> ~/.zshrc && echo 'export PHP_IDE_CONFIG="serverName=local.com"' >> ~/.zshrc
 
#定义工作目录,运行容器时通常把项目代码挂载到该目录
WORKDIR /www
ENTRYPOINT [ "/entrypoint.sh" ]
