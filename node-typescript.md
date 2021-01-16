一、把npm换成淘宝源，下载速度快

npm config set registry https://registry.npm.taobao.org --global
 
npm config set disturl https://npm.taobao.org/dist --global
 

二、安装依赖库

npm i @types/node --save-dev --registory=https://registry.npm.taobao.org
npm i ts-node --save-dev --registory=https://registry.npm.taobao.org
npm i typescript --save-dev --registory=https://registry.npm.taobao.org
npm i nodemon --save-dev --registory=https://registry.npm.taobao.org
 

三、创建 tsconfig.json 文件

复制代码
{
    "compilerOptions": {
        "target": "es2018",
        "module": "commonjs",
        "types": ["node"],
        "outDir": "./dist/",
        "rootDir": "./src/",
        "strict": true,
        "moduleResolution": "node",
        "esModuleInterop": true,
        "experimentalDecorators": true
    },
    "exclude": ["./test"]
}
复制代码
 

四、创建 nodedom.json 文件

{
    "ignore": ["**/*.test.ts", "**/*.spec.ts", ".git", "node_modules"],
    "watch": ["src"],
    "exec": "npm start",
    "ext": "ts"
}
 

五、给 package.json 添加脚本命令（应该知道 package.json 是 npm init 命令生成出来的吧。。。基础哦）

"scripts": {
    "start": "ts-node src/picture.ts",
    "dev": "nodemon src/picture.ts",
  },
 

六、使用命令

npm run dev
 

另外：

如果需要设置默认安装路径的话，可以用下面的命令，路径可以自己改

npm config set prefix "D:\ProgramFile\nodejs\node_modules\node_global"
 
npm config set cache "D:\ProgramFile\nodejs\node_modules\node_cache"
 