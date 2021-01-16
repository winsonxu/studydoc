# docker安装
面向百度安装,跳过
# docker-compose安装
跳过
# 创建docker工作区
~/work/docker
├── docker-compose.yml
├── mysql
│   ├── conf
│   │   ├── my.cnf
│   │   ├── my.cnf.fallback
│   │   ├── mysql
│   │   │   ├── conf.d
│   │   │   │   ├── docker.cnf
│   │   │   │   ├── mysql.cnf
│   │   │   │   └── mysqldump.cnf
│   │   │   ├── my.cnf -> /etc/alternatives/my.cnf
│   │   │   ├── my.cnf.fallback
│   │   │   ├── mysql.cnf
│   │   │   └── mysql.conf.d
│   │   │       └── mysqld.cnf
│   │   └── mysql.cnf
│   ├── data
│   ├── dockerfile
│   └── logs
├── nginx
│   ├── conf.d
│   │   └── default.conf
│   ├── dockerfile
│   ├── logs
│   │   ├── access.log(自动创建)
│   │   └── error.log(自动创建)
│   └── static(nginx静态根目录)
├── php
│   ├── dockerfile
│   ├── php-fpm.conf
│   └── php.ini
# 编写docker-compose.yml
version: '3'
# 定义三个服务nginx,php,mysql
``` yml
services:
  nginx:
    ports:
      - "80:80"
      - "443:443"
    image: nginx
    container_name: nginx
	# nginx依赖php,有了依赖后会根据先启动依赖
    depends_on:
      - php
    volumes:
      - ./code/php:/usr/share/nginx/html
      - ./nginx/conf.d:/etc/nginx/conf.d
      - ./nginx/logs:/var/log/nginx
	  # 要关联上php否则返回not file input
    links:
      - "php:php"
  php:
    image: php:5.6-fpm
    container_name: php
    # 向Docker内部暴露端口
    expose: 
      - 9000
    volumes:
      - ./code/php:/www
	  # 要连上mysql这样程序可以用mysql进行连接
    links:
      - "mysql:mysql"
  mysql:
    image: mysql:5.6
    volumes:
      - /Users/pengxu/Work/Docker/mysql/data:/var/lib/mysql
      - /Users/pengxu/Work/Docker/mysql/logs:/var/log/mysql
	  - /Users/pengxu/Work/Docker/mysql/conf:/etc/mysql
    restart: always
    # 设置MYSQL_ROOT_PASSWORD环境变量，这里是设置mysql的root密码。这里为root。
    environment:
      MYSQL_ROOT_PASSWORD: root
    ports:
      - "3306:3306"
	  # 向Docker内部暴露端口
    expose:
      - 3306
    container_name: mysql
networks:
  net1:
    driver: bridge
```

# 编写nginx配置文件
~/Work/Docker/nginx/conf.d/default.conf
``` nginx
server {
    listen          80;
    server_name     localhost;
    index           index.php index.html;
    root            /usr/share/nginx/html;
    access_log      /var/log/nginx/access.log;
    error_log       /var/log/nginx/error.log;
    location / {
        root /usr/share/nginx/html;
        index index.php index.html;
    }

    location ~ \.php$ {
        root /usr/share/nginx/html;
        fastcgi_pass    php:9000;
        fastcgi_index   index.php;
		# 这里一定要写成/www不然会找不到路径
        fastcgi_param SCRIPT_FILENAME /www$fastcgi_script_name;
        include         fastcgi_params;
    }
}
```

# 修改配置mysql配置
~/Work/Docker/mysql/conf/mysql/mysql.conf.d/mysqld.cnf
``` cnf
[mysqld]
pid-file	= /var/run/mysqld/mysqld.pid
socket		= /var/run/mysqld/mysqld.sock
datadir		= /var/lib/mysql
log-error	= /var/log/mysql/error.log
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
```

# nginx启用多站点
~/work/docker/nginx/conf.d
将之前的default.conf去掉location ~\.php$这段 

创建一个abc.conf的
在这里写上location ~\.php$这段 
写上server_name为域名
用utools做个内网穿透