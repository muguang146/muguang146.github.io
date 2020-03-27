---
title: 打包js
tags: ['webpack', 'js']
categories: ['webpack']
---
## 简易的打包js
我在网上找了些资料，自己动手试了试打包js，记录下过程

### 安装依赖
安装插件
> npm install --save-dev webpack

> npm install --save-dev webpack-cli

### 编写webpack
我们先建一个webpack.config.js的文件，然后进行配置
```
const path = require('path'); // 获取当前绝对路径

module.exports = {
  entry: "./index.js", // 主文件入口
  output: {
    filename: "bundle.js", // 打包后的文件
    path: path.resolve(__dirname, "./dist"),
  }
}
```
这是一个很简单的配置，配置了entry需要打包的文件，再配置filename打包后文件的名字和path打包文件放置的位置

path.resolve是拼接路径用的，__dirname是获取绝对路径，再加上"./dist"我们的目标文件夹，这样可以得到我们目标文件夹的绝对路径

### 编写启动程序(package.json)
npm 允许在package.json文件里面，使用scripts字段定义脚本命令。详情可以看[npm scripts 使用指南](http://www.ruanyifeng.com/blog/2016/10/npm_scripts.html)

我们现在设置打包的脚本
```
"scripts": {
    "build": "webpack --mode production",
    "dev": "webpack --mode development --watch"
  }
```

脚本里，我用了一些参数
* mode

mode有三个选项，
1. none
2. production(生产环境)
3. development(开发环境)

* watch

监听js自动打包

### 启动脚本，打包js

> npm run dev （生产环境）能监听js自动打包

> npm run build （生产环境）生产环境打包

js已经打包到dist文件夹里了