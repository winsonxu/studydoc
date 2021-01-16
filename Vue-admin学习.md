Vue-admin学习

npm install -g --registry=http://registry.npm.taobao.org
vue项目安装
npm i node-sass --sass_binary_site=https://npm.taobao.org/mirrors/node-sass/

注册全局
npm config set registry https://registry.npm.taobao.org --global
npm config set disturl https://npm.taobao.org/dist --global

electron自定义缓存

下载最新包

https://npm.taobao.org/mirrors/electron/

- Linux: `$XDG_CACHE_HOME` or `~/.cache/electron/`
- macOS: `~/Library/Caches/electron/`
- Windows: `$LOCALAPPDATA/electron/Cache` or `~/AppData/Local/electron/Cache/`

├── httpsgithub.comelectronelectronreleasesdownloadv1.7.9electron-v1.7.9-darwin-x64.zip 

│   └── electron-v1.7.9-darwin-x64.zip