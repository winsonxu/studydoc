# laravel 7学习笔记

## Mac系统环境变量加载顺序

顺便介绍一下mac的环境变量?

Mac系统的环境变量，加载顺序为：
/etc/profile /etc/paths ~/.bash_profile ~/.bash_login ~/.profile ~/.bashrc

当然/etc/profile和/etc/paths是系统级别的，系统启动就会加载，后面几个是当前用户级的环境变量。后面3个按照从前往后的顺序读取，如果~/.bash_profile文件存在，则后面的几个文件就会被忽略不读了，如果~/.bash_profile文件不存在，才会以此类推读取后面的文件。~/.bashrc没有上述规则，它是bash shell打开的时候载入的


## mac安装PHP

安装phpbrew

https://phpbrew.github.io/phpbrew/

https://www.jianshu.com/p/23cbefe6024b

安装php-version



```bash
brew search php  使用此命令搜索可用的PHP版本
brew install php@7.1 使用此命令安装指定版本的php
brew install brew-php-switcher 安装php多版本切换工具
brew-php-switcher 7.1 切换PHP版本到7.1（需要brew安装多个版本）
```

安装扩展

```bash
pecl version 查看版本信息
pecl help 可以查看命令帮助
pecl search xdebug  搜索可以安装的扩展信息
pecl install xdebug 安装扩展
pecl install http://pecl.php.net/get/redis-4.2.0.tgz 安装指定版本扩展
```

## 安装composer

安装composer
全局镜像配置
composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/

mac版
sudo mv composer.phar /usr/local/bin/composer

## 安装laravel，mysql

为什么要安装7.4呢，因为我安装的laravel是最新的，所以php也要安装最新的，不然会装不了，提示zip插件未安装，搞了半天无果，于是只能装回php7.4。

之前是用的mamp套件，也是因为他只有7.2所以我就把它删除掉了，用mysql原生的和brew install php了。

搞了很久终于把这些都装好了。

``` shell
brew install php
```

```shell
brew install laravel 
```

官网下载mysql for Mac 8.0，然后安装

配置环境变量

打开~/.bash_profile

export PATH="/Users/pengxu/.composer/vendor/bin:$PATH"

## valet

他提供了homestate但是我本机已经安装了mysql和php，所以就用了valet方式来启动。

Composer global require laravel/valet

valet install

valet start

我不喜欢.test结尾

的，更喜欢.app结尾的。所以运行valet tld app

###  配置文件

路径是：/Users/pengxu/.composer/vendor/laravel/valet/

/usr/local/etc/php/7.4/conf.d/php-memory-limits.ini 把内存大小改为2048M

### 设置工作目录

进入~/work/code/php/laravel目录，执行valet park目录

如果要引用其它目录的就进入其它目录，运行valet link xxx

### 开启ssl，不开浏览器访问不了，开了postman就不能用了

这个是必须要的，我开始访问的时候怎么也访问不到，后来发现没有开

valet secure blog

valet unsecure blog

不开浏览器访问不了，开了postman就不能用了，草

## 开启调试

```php
Illuminate\Support\Facades\DB::connection()->enableQueryLog();
```

查询方法

dd(Illuminate\Support\Facades\DB::getQueryLog());

## 安装JWT

https://jwt-auth.readthedocs.io/en/develop/laravel-installation/

composer require tymon/jwt-auth

php artisan vendor:publish --provider="Tymon\JWTAuth\Providers\LaravelServiceProvider"

php artisan jwt:secret



## 跨域

怎么开启跨域，我配置了cors但是没有用？

## 路径

怎么写两个方式的路径

## apidoc

/usr/local/etc/php/7.4/conf.d/php-memory-limits.ini 把内存大小改为2048M

```php artisan apidoc:generate
composer require mpociot/laravel-apidoc-generator

php artisan vendor:publish --provider="Mpociot\ApiDoc\ApiDocGeneratorServiceProvider" --tag=apidoc-config
  
php artisan apidoc:generate
```

### 技巧

User::latest()->get(); 等于 User::orderBy('created_at', 'desc')->get(); 降序排列



User::oldest()->get(); 等于User::orderBy('created_at')->get(); 升序排列

### PHP高级语法

use语句：PHP7可以使用一个use从同一个namespace中导入类，函数，常量，有点类似于.net里的using