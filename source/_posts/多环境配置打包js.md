---
title: 多环境配置打包js、打包多个js
tags: ['js', 'webpack']
---

# 区分不同环境打包js
首先，简易的打包js，可以去看[简易打包js](https://muguang146.github.io/2020/03/25/build-js/)

## 文件目录
先看下目录，我是在[egg项目](https://muguang146.github.io/2020/03/24/build-egg/)的public文件夹下进行操作的，目录结构如下
```
app/public
    |-- dist      开发环境的打包文件夹
    |-- release   生产环境的打包文件夹
    |-- webpack   webpack配置文件存放地
          |-- webpack.config.js     webpack基础配置文件
          |-- webpack.dev.js        开发环境配置文件
          |-- webpack.prod.js       生产环境配置文件
    |-- a.js      需要打包的js
    |-- b.js      需要打包的js
    |--package.json 脚本文件
```

## 脚本文件配置
我们在脚本文件，配置我们的打包脚本
```
  "scripts": {
    "build": "cd webpack && webpack --mode production",
    "dev": "cd webpack && webpack --mode development --watch"
  }
```
这里之所以又讲一遍脚本文件，是因为这里的mode配置，我们等下会用到

【注】build是生产环境，dev是开发环境

## webpack配置
### webpack基础文件配置
先配置webpack.config.js文件，我们脚本运行的就是这个文件
```
module.exports = (env, argv) => {
  return argv.mode === "production" ? require('./webpack.prod') : require('./webpack.dev');
}
```
webpack支持配置文件webpack.config.js中export出一个函数，该函数接受两个参数

* env：环境
* argv：参数

这里我们用argv获取到了之前在scripts脚本里设置的mode参数，用于区分环境，然后根据不同的环境，引入不同的webpack配置，完成了webpack配置上的环境区分

### webpack开发环境配置
现在开发环境的webpack打包配置
```
const path = require('path'); // 获取当前绝对路径

module.exports = {
  entry: {
    a: "../a.js",
    b: "../b.js"
  }, // 主文件入口
  output: {
    filename: "[name].js", // 打包后的文件
    path: path.resolve(__dirname, "../dist"),
    publicPath: path.resolve(__dirname, "../dist")
  }
}
```
这里大部分属性我在[简易打包js](https://muguang146.github.io/2020/03/25/build-js/)已经说过了，补充一些没说过的

* publicPath
> publicPath 并不会对生成文件的路径造成影响，主要是对你的页面里面引入的资源的路径做对应的补全，常见的就是css文件里面引入的图片

这个属性我也讲不太清，建议加上，值可以和path一样。

参考一些资料，[path与publicPath的区别](https://blog.csdn.net/HeliumLau/article/details/70666433)，[关于webpack的path和publicPath。
](https://www.cnblogs.com/bydzhangxiaowei/p/8972662.html)

* filename: "[name].js"
注意这里打包的文件名，我用了"[name]"，因为这次打包的，是多个js一起打包，我在入口文件的地方entry用了对象的写法，这是webpack支持的一种多个入口的写法，而我们在filename里用的name就是我们entry设置的对象键，比如{a:"../a.js"}，那么它的name就是a，打包出来的js名，就是a.js

### webpack生产环境配置
#### 基础配置
首先，先进行打包，过程和开发环境差不多
```
const path = require('path'); // 获取当前绝对路径
const entry = {   // 入口配置
  a: "../a.js",
  b: "../b.js"
};
module.exports = {
  entry: entry, // 主文件入口
  output: {
    filename: "[name].[chunkhash].js", // 打包后的文件
    chunkFilename:"[name].js",
    path: outputPath,
    publicPath: path.resolve(__dirname, "../release")
  }
}
```
为了防止缓存，这里我们打包的js名添加了hash值，会根据js内容生成

#### 防堆积，打包js
因为有hash为后缀，我们的js打包，不会替换了，这样会出现堆积js的现象，于是我们要在打包js前，将目标文件夹里重复的js文件处理掉

先上代码，这里同样是在webpack.prod.js修改
```
const fs = require('fs');
const outputPath = path.resolve(__dirname, '../release');   // 获取打包的目标文件夹的绝对路径
const entry = {   // 入口配置
  a: "../a.js",
  b: "../b.js"
};
function removeFile (entry, outputPath){
  var files = [];
  if(fs.existsSync(outputPath)){    // 查询路径是否存在，返回Boolean值
    files = fs.readdirSync(outputPath); // 方法将返回一个包含“指定目录下所有文件名称”的数组对象。
    files.forEach((file) => {
      var fileName = file.split('.')[0];    // 获取文件名
      var curPath = outputPath + "/" + file;  // 文件的绝对路径
      if(entry[fileName]){
        fs.unlinkSync(curPath);   // 删除文件操作
      }
    });

  }
}

removeFile(entry, outputPath);
```
这里我们用到了fs的一些函数。

* fs.existsSync

> 查询路径是否存在，返回Boolean值

* fs.readdirSync

> 方法将返回一个包含“指定目录下所有文件名称”的数组对象。

* fs.unlinkSync

> 删除文件操作

主要是根据我们的打包入口文件，获取js名，然后将打包文件里，同名的js先删掉，这样，我们打包出来后，就不会有重复的js了

