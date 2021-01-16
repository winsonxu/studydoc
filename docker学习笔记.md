# 全新环境运行Docer
1. 安装centos 7
2. 配置hyper-v的nat网络，右健能上网的网络共享给NAT网络
3. 更新centos源到国内源
4. yum markcache 
  yum update
6. yum install net-tools.x86_64
5. 安装docker
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum makecache fast
yum -y install docker-ce
systemctl start docker
7. 配置阿里云的docker镜像
``` shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://jd3kz6bu.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```
8. docker run hello-world
cp
---

## docker容器自动启动
docker 服务器开机自启动：

1.systemctl is-enabled docker.service  检查服务是否开机启动

2.systemctl enable docker.service  将服务配置成开机启动

3.systemctl start docker.service  启动服务

 

systemctl  相关其他命令：

systemctl disable docker.service 禁止开机启动

systemctl stop docker.service  停止
systemctl restart docker.service  重启

 

//------------------------------------------------------

容器开机启动：

创建容器时候指定restart参数：

docker run    -it -p 6379:6379 --restart=always  --name redis -d redis

对已经创建的容器用docker update 更新:

docker update --restart=always  xxx

 

--restart具体参数值详细信息 :

no - 容器退出时，不重启容器
on-failure - 只有在非0状态退出时才从新启动容器
always - 无论退出状态是如何，都重启容器


## docker compose
###安装
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose
查看安装是否成功
docker-compose -v

# docker命令
## 窗口生命令周期管理
- docker run --name nginx8081 -d -p 8081:80 nginx
	+ -d后台运行
	+ -p端口映射；主机：容器
	+ -P全部映射
	+ -it交互启动
	+ -v数据卷目标挂载；主机：容器
- docker stop nginx8081
- docker start nginx8081
- docker restart nginx8081
- docker kill nginx8081
	+ -i向容器发一个信号
- docker rm cdnginx8081
- docker exec
	docker exec -it nginx8082 /bin/sh
	按Ctrl+P+Q退出

---
# docker mysql配置
创建mysql
docker run -p 3306:3306 --name vipmysql -v /norait/mysql/conf:/etc/mysql/conf.d -v /norait/mysql/logs:/logs -v /norait/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=Hjzy2018. -d mysql:5.6
---
# docker nginx配置
- 创建本地文件
	+ www
	+ logs
	+ conf
- cp docker里的文件到本地文件
docker cp nginx8081:/etc/nginx/nginx.conf ~/nginx/conf
- 将docker的文件映射出来

`docker run -d -p 6601:80 --name vipnginx -v /norait/nginx/www:/usr/share/nginx/html -v /norait/nginx/conf/conf.d:/etc/nginx/conf.d -v /norait/nginx/logs:/var/log/nginx --link vipphp-fpm:php nginx
`

# php
创建php56/php   php56/php-fpm.d，在里面创建php.ini文件和conf.d文件夹

`docker run --name vipphp-fpm -v /norait/nginx/www:/www -v /norait/php56/php:/usr/local/etc/php -v /norait/php56/php-fpm.d:/usr/local/etc/php-fpm.d/ --link vipmysql:mysql -d php:5.6-fpm
`

纯净的php镜像没有缺少扩展的问题解决进入容器 
docker exec -it vipphp-fpm /bin/bash
至目录下 cd /usr/local/bin  
安装扩展 ./docker-php-ext-install pdo_mysql  
安装扩展 ./docker-php-ext-install mysql
安装扩展 ./docker-php-ext-install gd
重启容器 docker php restart


### docker compsose file

``` docker
version: '3'
services:
	mysql:
		container_name: "vipmysql"
		image: "mysql:5.6"
		ports:
			- 3306:3306
		restart: always
		environment:
			- MYSQL_ROOT_PASSWORD=Hjzy2018.
		volumes:
			- "/norait/mysql/conf:/etc/mysql/conf.d"
			- "/norait/mysql/logs:/logs"
			- "/norait/mysql/data:/var/lib/mysql"
	php:
		container_name:"vipphp-fpm"
		depends_on:
			- "vipmysql:mysql"
		image:php:5.6-fpm
		restart: always
		volumes:
			- "/norait/nginx/www:/www"
			- "/norait/php56/php:/usr/local/etc/php"
			- "/norait/php56/php-fpm.d:/usr/local/etc/
			- php-fpm.d/"
	nginx:
	   container_name:"vipnginx"
		image:nginx:latest
		restart: always
		ports:
			- 6601:80
		depends_on:
    		-  "vipphp-fpm:php"
		volumes:
		  - "/norait/nginx/www:/usr/share/nginx/html"
		  - "/norait/nginx/conf/conf.d:/etc/nginx/conf.d"
		  - "/norait/nginx/logs:/var/log/nginx"
```




#Docker-compose常用命令
##2.命令

docker-compose --help你会看到如下这么多命令
build               Build or rebuild services
bundle              Generate a Docker bundle from the Compose file
config              Validate and view the Compose file
create              Create services
down                Stop and remove containers, networks, images, and volumes
events              Receive real time events from containers
exec                Execute a command in a running container
help                Get help on a command
images              List images
kill                Kill containers
logs                View output from containers
pause               Pause services
port                Print the public port for a port binding
ps                  List containers
pull                Pull service images
push                Push service images
restart             Restart services
rm                  Remove stopped containers
run                 Run a one-off command
scale               Set number of containers for a service
start               Start services
stop                Stop services
top                 Display the running processes
unpause             Unpause services
up                  Create and start containers
version             Show the Docker-Compose version information
　　

##3.常用命令

docker-compose up -d nginx                     构建建启动nignx容器

docker-compose exec nginx bash            登录到nginx容器中

docker-compose down                              删除所有nginx容器,镜像

docker-compose ps                                   显示所有容器

docker-compose restart nginx                   重新启动nginx容器

docker-compose run --no-deps --rm php-fpm php -v  在php-fpm中不启动关联容器，并容器执行php -v 执行完成后删除容器

docker-compose build nginx                     构建镜像 。        

docker-compose build --no-cache nginx   不带缓存的构建。

docker-compose logs  nginx                     查看nginx的日志 

docker-compose logs -f nginx                   查看nginx的实时日志

 

docker-compose config  -q                        验证（docker-compose.yml）文件配置，当配置正确时，不输出任何内容，当文件配置错误，输出错误信息。 

docker-compose events --json nginx       以json的形式输出nginx的docker日志

docker-compose pause nginx                 暂停nignx容器

docker-compose unpause nginx             恢复ningx容器

docker-compose rm nginx                       删除容器（删除前必须关闭容器）

docker-compose stop nginx                    停止nignx容器

docker-compose start nginx                    启动nignx容器

 

##docker中安装vim
mv /etc/apt/sources.list /etc/apt/sources.list.bak
    echo "deb http://mirrors.163.com/debian/ jessie main non-free contrib" >> /etc/apt/sources.list
    echo "deb http://mirrors.163.com/debian/ jessie-proposed-updates main non-free contrib" >>/etc/apt/sources.list
    echo "deb-src http://mirrors.163.com/debian/ jessie main non-free contrib" >>/etc/apt/sources.list
    echo "deb-src http://mirrors.163.com/debian/ jessie-proposed-updates main non-free contrib" >>/etc/apt/sources.list
    #更新安装源
    apt-get update
    
 apt-get install -y vim
