# node.js学习笔记
## 入门
- 创建目录
``` javascript
node init
```
- 热调试启动
``` shell
supervisor ./
```



pm2重启后列表空，可以使用下面命令加截之前的列表，然后

pm2 resurrect 

启动全部

pm2 start all

然后保存项目：pm2 save

最后开机启动：pm2 startup

# 使用配置文件启动

根目录下创建pm2.json文件，可以注入环境变量

{
  "apps": [{
    "name": "zuowenbao",
    "script": "./dist/main.js",
    "cwd": "./",
    "watch": false,
    "error_file": "./logs/website-err.log",
    "out_file": "./logs/website-out.log",
    "log_date_format": "YYYY-MM-DD HH:mm Z",
    "env": {
      "NODE_ENV": "production",
      "WX_NOTIFY_URL": "https://zuowenbao.norait.cn/",
      "WX_APPID": "wxe2571da5ed7b7fb1",
      "WX_APPSECRET": "3c7c9c24dad1ba5600222438cae5ca9f",
      "WX_MCHID": "1515570331",
      "WX_MCHSECRET": "XBuH0LhxqpRICJ4hxTCqDrASpAUGjUuT",
      "WX_P12FILE": "apiclient_cert.p12"
    },
    "exec_mode": "fork"
  }]
}

启动命令：pm2   路径  pm2.json