# Electron学习笔记

## 与vue整合



创建background.js文件，里面写electron主进程，创建窗口，事件这些

Package.json

"main": "background.js",

npm i -D electron

npm i -D electron-devtools-installer

npm i -D  vue-cli-plugin-electron-builder



## 图标

### windows的就是icon图标

网上有很多可以直接生成的，生成了favicon.icon后放到public里。大写需要256*256的不然会报错

### 生成mac用的icns图标

1 准备一个 1024 * 1024 的png图片，假设名字为 pic.png
2 命令行 $ mkdir tmp.iconset，创建一个临时目录存放不同大小的图片
3 把原图片转为不同大小的图片，并放入上面的临时目录

全部拷贝到命令行回车执行，执行结束之后去tmp.iconset查看十张图片是否生成好

sips -z 16 16     pic.png --out tmp.iconset/icon_16x16.png
sips -z 32 32     pic.png --out tmp.iconset/icon_16x16@2x.png
sips -z 32 32     pic.png --out tmp.iconset/icon_32x32.png
sips -z 64 64     pic.png --out tmp.iconset/icon_32x32@2x.png
sips -z 128 128   pic.png --out tmp.iconset/icon_128x128.png
sips -z 256 256   pic.png --out tmp.iconset/icon_128x128@2x.png
sips -z 256 256   pic.png --out tmp.iconset/icon_256x256.png
sips -z 512 512   pic.png --out tmp.iconset/icon_256x256@2x.png
sips -z 512 512   pic.png --out tmp.iconset/icon_512x512.png
sips -z 1024 1024   pic.png --out tmp.iconset/icon_512x512@2x.png
4 通过iconutil生成icns文件 $ iconutil -c icns tmp.iconset -o Icon.icns，此时你的目录应该有了你想要的 O(∩_∩)O

生在后复制到public里的favicon.icns

在vue.config.js里增加

pluginOptions: {

​        electronBuilder: {

​            builderOptions: {

​                'appId': 'com.norait.office',

​                'win': {// win相关配置

​                    'icon': './dist_electron/bundled/favicon.ico',

​                },

​                'mac': {

​                    'icon': './dist_electron/bundled/favicon.icns',

​                }

​            }

​        }

​    }

## 打包

增加github的缓存提高打包速度

Mac 上为：`~/Library/Caches/electron/`

Package.json

name， productName, version,description,author都要写

scripts里加

"electron:build": "vue-cli-service electron:build",



## 电子签名





## 自动更新





## 获取机器码（主进程与页面通讯）





## 日志

npm install electron-log

const log = require('electron-log');

log.info('Hello, log');

位置

on Linux: ~/.config/{app name}/logs/{process type}.log
on macOS: ~/Library/Logs/{app name}/{process type}.log
on Windows: %USERPROFILE%\AppData\Roaming{app name}\logs{process type}.log



## 导出EXCEL

主进程进加MSG_SAVE_DIALOG事件



渲染进程增加事件



## 楼帮手发布流程

后台发布
1.客户环境svn update
2.php artisan migrate

windows客户端发布
1.修改package.json版本号
2.npm run electron:build
3.将dist_electron里的最新生成的【latest.yml】【楼帮手_setup_版本号.exe.blockmap】【楼帮手_setup_版本号.exe】三个文件放到ftp上
4.更新www.norait.net网站的最新版本

mac客户端发布
1.1.修改package.json版本号
2.npm run electron:build
3.将dist_electron里的最新生成的【latest-mac.yml】【楼帮手_setup_版本号.dmg.blockmap】【楼帮手_setup_版本号.dmg】三个文件放到ftp上
4.更新www.norait.net网站的最新版本

