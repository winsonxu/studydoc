# macOS下配置nginx和php

## 先停掉Apache

sudo launchctl unload -w /System/Library/LaunchDaemons/org.apache.httpd.plist

看一下80端口是否还有占用的

sudo lsof -i:80

没有了就可以操作后面的了。

## 安装Home Brew

##  安装Nginx

1. 手动删除旧版本

   /usr/local/Cellar/nginx目录、/usr/local/etc/nginx/、/usr/local/var/www目录

2. 终端执行安装nginx

``` shell
brew install nginx
```

3. 安装完成后操作如下命令

启动nginx

```  shell 
sudo nginx
```

访问nginx网站

``` code 
http://localhost:8080/
```

查看nginx日志

``` shell
/usr/local/var/log/nginx/error.log
/usr/local/var/log/nginx/access.log
```

停止nginx服务

``` shell
sudo nginx -s stop
```

重启nginx服务

``` shell
sudo nginx -s reload
```

4. 修改nginx配置

   macos需要使用root owner，否则改了路径后会提示 is forbidden (13: Permission denied)

   user root owner;

   ``` code
   /usr/local/etc/nginx/nginx.conf
   ```

   更改默认根目录，增加index.php的支持

   ``` code
   location / {
   	root 自己的本地路径;
   	index index.html index.htm index.php
   }
   ```

   取消localtion ~ \.php$ { }的注释

   将root更改为自己本地的路径

   将fastcgi_param  SCRIPT_FILENAME 的值改为 $document_root$fastcgi_script_name;

   配置上传文件大小，在http节中增加

   ``` code
   client_max_body_size 100m;
   ```

   

5. 运行php修改php-fpm文件

   复制默认的默认配置为当前配置

   ``` shell
   sudo cp /private/etc/php-fpm.conf.default /private/etc/php-fpm.conf
   sudo cp /private/etc/php-fpm.d/www.conf.default /private/etc/php-fpm.d/www.conf
   ```

   设置php-fpm的日志文件，否则启动php-fpm时会报错，打开/private/etc/php-fpm.conf

   ``` shell
   error_log = 你的日志文件目录/php-fpm.log
   ```

   macos设置php-fpm运行权限，否则会报Primary script unknown

   打开/private/etc/php-fpm.d/www.conf文件

   ``` config
   user = 当前用户名
   group = staff
   ```

   php上传文件大小设置

   在/usr/local/etc/php/7.4/php.ini中找到upload_max_filesize改成200M
   
   upload_max_filesize = 200M

   配置完成后执行

   ``` shell
   sudo php-fpm
   ```

   重启php-fpm

   ```shell
   sudo  killall  php-fpm  
   ```
   
   在根目录下创建index.php

   ``` php
<?php
     phpinfo();
   ?>
   ```
   
   访问http://localhost:8080/index.php
   
   

## 日常操作

### 启动nginx

sudo nginx

sudo php-fpm

### 停止nginx

sudo nginx -s stop

sudo  killall  php-fpm  

## 创建多站点

创建多站点的配置文件夹

修改nginx配置

``` code
/usr/local/etc/nginx/nginx.conf
```

增加一个新的server

``` /usr/local/etc/nginx/nginx.conf
server {
        listen       80;
        server_name  ocr.local.com;
        root         /Users/pengxu/Work/Code/php/ocr.local.com;
        location / {
            root   /Users/pengxu/Work/Code/php/ocr.local.com;
            index  index.html index.htm index.php;
        }
        location ~ \.php$ {
           root           /Users/pengxu/Work/Code/php/ocr.local.com;
           fastcgi_pass   127.0.0.1:9000;
           fastcgi_index  index.php;
           fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
           include        fastcgi_params;
        }
    }
```







