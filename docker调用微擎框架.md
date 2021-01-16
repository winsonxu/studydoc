version: '3'
# 定义三个服务nginx,php,mysql
services:
  nginx:
    ports:
      - "80:80"
      - "443:443"
    image: nginx
    container_name: nginx
    depends_on:
      - php
    volumes:
      - ./code/php:/usr/share/nginx/html #静态根目录
      - ./nginx/conf.d:/etc/nginx/conf.d #每个站点的配置
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf #nginx的配置文件
      - ./nginx/logs:/var/log/nginx #运行时日志
    networks:
      - net
  php:
    image: php:5.6-fpm
    container_name: php
    expose: 
      - 9000
    volumes:
      - /Users/pengxu/Work/Code/php/fangkuaibao.norait.cn:/www #php站点目录
    networks:
      - net
  mysql:
    image: mysql:5.6
    volumes:
      - ./mysql/data:/var/lib/mysql
      - ./mysql/logs:/var/log/mysql
      - ./mysql/conf:/etc/mysql
    restart: always
    # 设置MYSQL_ROOT_PASSWORD环境变量，这里是设置mysql的root密码。这里为root。
    environment:
      MYSQL_ROOT_PASSWORD: root
    ports:
      - "3306:3306"
    expose:
      - 3306
    container_name: mysql
    networks:
      - net
networks:
  net:
配置好后访问是空白页
进入/data/config.php后将$config['setting']['development'] = 1;后还是空白
配置应该没有问题,现在也不知道是什么原因
在程序根目录创建一个info.php看一下效果
可以正常访问,看来是微擎系统的问题,但是现在我也不知道php-fpm发生了什么.要把php运行的日志映射出来看看
看到phpinfo里的error_log显示 no value这样的话需要先映射出php-fpm.conf
      - ./php/php-fpm.conf:/usr/local/etc/php-fpm.conf #php配置文件
这个文件是从哪里来的呢?有点麻烦需要先启动起来,然后
docker exec -it php bash
cp /usr/local/etc/php-fpm.conf /www
这个时候就会在程序目录下看到这个文件,然后把这个文件手动移到~/work/docker/php里面
拷出来后发现只有
include=etc/php-fpm.d/*.conf
这么一句,我操,拷错了.再来一次
cp /usr/local/etc/php-fpm.d/*.* /www
再移过来
      - ./php:/usr/local/etc/php-fpm.d #php配置文件

现在我删除掉nginx的静态目录映射,因为我启用了多站点  所以我不需要再管默认根目录了
然后我又让php-fpm日志映射出来
      - ./php/logs:/usr/local/var/log #php log记录
就这样我把www.conf里面的error_log这个注释掉
重启一下
只有php-fpm的日志,php运行错误的日志没有显示 ,看来还是没有配对,再查
docker exec -it php bash
cd /usr/local/etc/php
里面有php.ini-development  php.ini-production
cp php.ini-development /www
然后再移到php里面
再改一下映射
      - ./php/php.ini:/usr/local/etc/php/php.ini #php配置文件
然后再开启php.ini的日志记录
error_log = /usr/local/var/log/php_errors.log
重启一下
还是空白,没办法了,只能靠猜了
改一下mysql的host连接字符串吧,把mysql改成域名试一下

还是不行
但是在这个过程中发现了另一个问题,数据库地址突然变成了不是映射出来的地址了.好诡异
查了日志后发现有一句
Warning: World-writable config file ‘/etc/mysql/my.cnf’ is ignored
我去掉了my.cnf的映射后数据库正常映射 ,加上后就不正常,百度一下
chmod 644 /etc/my.cnf
重启一下,又正常了.研究了一下原来只能映射my.cnf,但是打开后只有两行包含其它配置的内容,我就把每个文件都看了一下,然后把所有配置都放到这里一个文件里
``` mysql
[mysqld]
skip-host-cache
skip-name-resolve

pid-file	= /var/run/mysqld/mysqld.pid
socket		= /var/run/mysqld/mysqld.sock
datadir		= /var/lib/mysql
log-error	= /var/log/mysql/error.log
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

[mysql]

[mysqldump]
quick
quote-names
max_allowed_packet	= 16M
```

解决了这个问题回过头来继续搞php空白页的问题
怀疑是代码问题,调试代码吧.
跟到cache那里,发现new _PDO类的时候即不报错 后面也不再执行了

估计是这个镜象少东西.从网上找了一个build的代码.
``` dockerfile
FROM php:5.6-fpm

#因为官方镜像是用的debain stretch 所以第一步先替换源 如果是Jessie 要替换对应的源
RUN echo "deb http://mirrors.aliyun.com/debian stretch main contrib non-free" > /etc/apt/sources.list && \
    echo "deb-src http://mirrors.aliyun.com/debian stretch main contrib non-free" >> /etc/apt/sources.list  && \
    echo "deb http://mirrors.aliyun.com/debian stretch-updates main contrib non-free" >> /etc/apt/sources.list && \
    echo "deb-src http://mirrors.aliyun.com/debian stretch-updates main contrib non-free" >> /etc/apt/sources.list && \
    echo "deb http://mirrors.aliyun.com/debian-security stretch/updates main contrib non-free" >> /etc/apt/sources.list && \
    echo "deb-src http://mirrors.aliyun.com/debian-security stretch/updates main contrib non-free" >> /etc/apt/sources.lis

RUN apt-get update \
&& apt-get install -y --no-install-recommends \
libfreetype6-dev \
libjpeg62-turbo-dev \
libmcrypt-dev \
libpng-dev \
libmemcached-dev \
zlib1g-dev \
libbz2-dev \
libgmp-dev \
libedit-dev \
libxml2-dev \
libxslt-dev \
openssl \
libssl-dev \
libpq-dev \
&& docker-php-ext-install -j$(nproc) iconv \
&& docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
&& docker-php-ext-install -j$(nproc) gd  \
bcmath \
mysqli \
pdo_mysql \
bz2 \
calendar \
exif \
gettext \
intl \
pcntl \
readline \
pgsql \
pdo_pgsql \
shmop \
sockets \
wddx \
xsl \
zip \
opcache \
mcrypt \
&& pecl install igbinary-1.2.1 && docker-php-ext-enable igbinary \
&& echo "\n" | pecl install redis && docker-php-ext-enable redis \
&& echo "\n" | pecl install memcache && docker-php-ext-enable memcache \
&& echo "\n" | pecl install memcached-2.2.0  && docker-php-ext-enable memcached \
&& pecl install mongodb && docker-php-ext-enable mongodb \
&& echo "\n" | pecl install mongo && docker-php-ext-enable mongo \
&& docker-php-source delete \
&& apt-get clean; rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /usr/share/doc/*
```
改一下docker-compose.yml里php的images为build:php

终于可以访问啦,啦啦啦
但是css js images这些资源都加载不到.这些资料都是要加载nginx静态资源的.但是nginx配置那里又是只能配置本地的
先是想改成静态也记他们走php后来发现还是没戏

终于在把当前的站点映射了静态之后可以正常跑了.
这样想来nginx天生可以跳动静分离喽.如果我把静态文件都拿到外面去,用nginx只要把路径指到静态那里就可以了吧应该
最终的nginx配置文件如下

``` nginx
server {
    listen          80;
    server_name     fangkuaibao.utools.club;
    index           index.php index.html;
    root            /usr/share/nginx/html/fangkuaibao;
    access_log      /var/log/nginx/fangkuaibao_access.log;
    error_log       /var/log/nginx/fangkuaibao_error.log;
    location / {
        root /usr/share/nginx/html/fangkuaibao;
        index index.php index.html;
    }

    location ~ \.php$ {
        fastcgi_pass    php:9000;
        fastcgi_index   index.php;
        fastcgi_param   SCRIPT_FILENAME /www$fastcgi_script_name;
        include         fastcgi_params;
    }
}
```

最终的docker-compose.yml文件如下
``` yml
version: '3'
# 定义三个服务nginx,php,mysql
services:
  nginx:
    ports:
      - "80:80"
      - "443:443"
    image: nginx
    container_name: nginx
    depends_on:
      - php
    volumes:
      - ~/Work/Code/php/fangkuaibao.norait.cn:/usr/share/nginx/html/fangkuaibao
      - ./nginx/conf.d:/etc/nginx/conf.d #每个站点的配置
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf #nginx的配置文件
      - ./nginx/logs:/var/log/nginx #运行时日志
    networks:
      - net
  php:
    build: php
    container_name: php
    expose: 
      - 9000
    volumes:
      - ~/Work/Code/php/fangkuaibao.norait.cn:/www #php站点目录
      - ./php/php-fpm.d:/usr/local/etc/php-fpm.d #php-fpm配置文件
      - ./php/php.ini:/usr/local/etc/php/php.ini #php配置文件
      - ./php/logs:/usr/local/var/log #php log记录
    networks:
      - net
  mysql:
    image: mysql:5.6
    volumes:
      - ./mysql/data:/var/lib/mysql
      - ./mysql/logs:/var/log/mysql
      - ./mysql/conf/my.cnf:/etc/mysql/my.cnf
    restart: always
    # 设置MYSQL_ROOT_PASSWORD环境变量，这里是设置mysql的root密码。这里为root。
    environment:
      MYSQL_ROOT_PASSWORD: root
    ports:
      - "3306:3306"
    expose:
      - 3306
    container_name: mysql
    networks:
      - net
networks:
  net:
```
最终的php文件夹里的dockerfile是
``` dockerfile
FROM php:5.6-fpm

#因为官方镜像是用的debain stretch 所以第一步先替换源 如果是Jessie 要替换对应的源
RUN echo "deb http://mirrors.aliyun.com/debian stretch main contrib non-free" > /etc/apt/sources.list && \
    echo "deb-src http://mirrors.aliyun.com/debian stretch main contrib non-free" >> /etc/apt/sources.list  && \
    echo "deb http://mirrors.aliyun.com/debian stretch-updates main contrib non-free" >> /etc/apt/sources.list && \
    echo "deb-src http://mirrors.aliyun.com/debian stretch-updates main contrib non-free" >> /etc/apt/sources.list && \
    echo "deb http://mirrors.aliyun.com/debian-security stretch/updates main contrib non-free" >> /etc/apt/sources.list && \
    echo "deb-src http://mirrors.aliyun.com/debian-security stretch/updates main contrib non-free" >> /etc/apt/sources.lis

RUN apt-get update \
&& apt-get install -y --no-install-recommends \
libfreetype6-dev \
libjpeg62-turbo-dev \
libmcrypt-dev \
libpng-dev \
libmemcached-dev \
zlib1g-dev \
libbz2-dev \
libgmp-dev \
libedit-dev \
libxml2-dev \
libxslt-dev \
openssl \
libssl-dev \
libpq-dev \
&& docker-php-ext-install -j$(nproc) iconv \
&& docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
&& docker-php-ext-install -j$(nproc) gd  \
bcmath \
mysqli \
pdo_mysql \
bz2 \
calendar \
exif \
gettext \
intl \
pcntl \
readline \
pgsql \
pdo_pgsql \
shmop \
sockets \
wddx \
xsl \
zip \
opcache \
mcrypt \
&& pecl install igbinary-1.2.1 && docker-php-ext-enable igbinary \
&& echo "\n" | pecl install redis && docker-php-ext-enable redis \
&& echo "\n" | pecl install memcache && docker-php-ext-enable memcache \
&& echo "\n" | pecl install memcached-2.2.0  && docker-php-ext-enable memcached \
&& pecl install mongodb && docker-php-ext-enable mongodb \
&& echo "\n" | pecl install mongo && docker-php-ext-enable mongo \
&& docker-php-source delete \
&& apt-get clean; rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /usr/share/doc/*
```