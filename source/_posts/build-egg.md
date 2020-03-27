---
title: 搭建egg项目
tags: ['egg', 'node']
categories: ['egg']
---
> 我是使用脚手架搭建，内容参照egg官网，只是把我自己搭建的过程记录下来

[egg](https://eggjs.org/zh-cn/)

## 创建脚手架

先创建文件夹、进入文件夹
> mkdir egg-example && cd egg-example

进行脚手架的搭建
> npm init egg --type=simple

安装依赖
> npm i

启动看看
> npm run dev

## 文件结构

创建出来的目录

```
      |-controller\ 用于解析用户的输入，处理后返回相应的结果
      |
app --|-public\     用于放置静态资源
      |
      |-router.js   用于配置 URL 路由规则
```

### controller
这里的文件与router相对应，也是网页输出的地方，这样就有一个简单的项目流程了

大概的流程是
> 写路径 => 在controller里创建对应的js => js输出内容到网页


刚刚创建的项目，内容非常的少，现在我们一点点补充内容

## ejs 模板输出
我们现在的输出手段太简陋了，现在用我们最喜欢的html来输出，这里我选择的是ejs模板，这也是egg支持的

这里参考[初始化 egg+ejs 项目](http://www.bubuko.com/infodetail-2418915.html?__cf_chl_jschl_tk__=0578d2c37537b3d4c84bc9b1511e56acb77a1ee0-1585030048-0-AX04Pg0386Mmn33Y8Qy6cueW9-uBFb9YunQZ6uUfVqoiL-03jK5smQrOZwMt7yAonk2EArL-m_3uo6JC_bqNelx3rFT9X9v-WqU48dkaOXmxxdXbyMsjV09TmTj9ajkyptfS5R9TsXCum05LVILinEntDZyfsrPn4XbzaVl2XD3GtqZEj6ingxvlDnv-Vg1bUL8SdDCEC3LrliMJUeTmanZXSlrxnK1ld6MunNwieWzCn8knA4JCXu2o9KeIuaw6DCbw7L_2ejxrTr1pzhe118_qurIOIKLxjUVzyLwVrP4-dhRJiEg_VCnfAS4dKVG_vQ)

### 安装 egg-view-ejs
> npm install egg-view --save

> npm install egg-view-ejs --save

### 配置 egg-view-ejs

进入 config/plugin.js，将文件的内容清空，修改成下面这样

```
// 支持ejs
exports.ejs = {
  enable: true,
  package: 'egg-view-ejs',
};
```

这里我发现，有严格模式，于是我想把严格模式关掉，egg脚手架自带了一些规则，在.eslintrc文件里，我习惯用我这套规则，如果要添加的话，可以参考[.eslintrc.js相关配置](https://www.cnblogs.com/QQPrincekin/p/10882309.html)

```
{
  "parserOptions": {
      "ecmaVersion": 2017
  },
  "env": {
      "es6": true,
  },
}

```

现在继续ejs的修改，在 config/config.default.js下添加配置

```
  // 支持ejs
  config.view = {
    defaultViewEngine: 'ejs',
    mapping: {
      '.ejs': 'ejs',
    },
  };
```

### 写ejs

开始写之前，先在app文件夹下创建view文件夹

新建home.ejs

ejs的书写方式，参考[ejs官网](https://ejs.bootcss.com/)

渲染的方法，在我们的输出入口controller里设置

```
async index() {
  const ctx = this.ctx;
  await ctx.render('home.ejs', { data:要传的参数 });
}
```
render的第二个参数，是对象，可以将一些数据放到ejs里用

这里默认是在view文件夹下，如果想链接到view\pc\index.ejs，就是'pc\index.ejs'

现在的流程变成

> 设置url => 在controller里设置出口口 => 写ejs => 在出口将ejs输出

## ejs扩展（helper）

helper，是创建在app\extend\helper.js 里的js，在这里定义的函数，可以直接在ejs里引用

比如我在helper.js里定义一个函数
```
module.exports = {
  stringAdd(str) {
    return str + " + helper";
  }
}
```

这时，ejs里可以这样写
```
<h2><%= helper.stringAdd('测试') %></h2>
```
页面展示为
```
测试 + helper
```

## 知识扩展:规范
controller只是设置数据和输出口的地方，egg支持我们将代码规范起来

### context（逻辑）

egg支持我们将逻辑区分出来，我们可以在app\extend里创建context.js，这里的函数，会被合并到egg内置的ctx对象里

### service（服务）

egg支持我们将网络服务区分出来，我们可以在app\service里创建js，js里的函数，会被合并到egg内置的ctx对象的service对象里

### 例子

我们在context.js里写函数
```
## app\extend\context.js
module.exports = {
  cs() {
    return '测试';
  }
};
```
我们在service\里写函数
```
## app\service\home.js

const Server = require('egg').Service;

class HomeServer extends Server {
  /**
   * 测试用
   * @param {String}} str
   */

  stingAdd(str) {
    return str + '后手添加';
  }

}

module.exports = HomeServer;
```

然后，我们在controller里使用这些逻辑
```
## controller\home.js
const Controller = require('egg').Controller;

class HomeController extends Controller {
  async index() {
    const { ctx } = this;
    const css = ctx.cs();
    const assignData = {
      title: 'egg',
      cs: css,
    };
    assignData.csTitle = this.service.home.stingAdd('?')
    await ctx.render('home.ejs', assignData);
  }
}

module.exports = HomeController;
```

ejs上的代码
```
  <h2><%= helper.stringAdd(cs) %></h2>
  <h2><%= csTitle %></h2>
```
最后，页面输出的是

> 测试 + helper\
> ?后手添加

## js的使用
差不多到了用js的时候了，egg有专门的静态文件存放地，我们把js等静态文件建在public文件夹下

egg内置了自动继承静态资源配置

### 使用方法

如下所示静态资源，我们在html里可以通过"/public/index.js"进行访问
```
app/public
    |--index.js
```
egg的静态资源管理还有一些默认配置。[参考资料](https://www.jianshu.com/p/5eb0d5a10267)

### 打包
只是简单的引用js肯定不行，我们要对js进行打包处理。[打包js](https://muguang146.github.io/2020/03/25/build-js/)、[多环境打包js](https://muguang146.github.io/2020/03/25/build-js-environment/)

## 获取hash后缀的js
> 打包好的hash后缀js如何使用？
### 创建引用路径存放地
为了能正确的引用到对应的js，我们要在打包js的时候，把js的引用路径保存下来

我们在public下，建个scripts文件夹，放置js的引用路径
```
app/bublic
      |-- script
```
### 编写插件

插件能让我们在webpack运行的过程中插入一些程序，这次我们要在打包完js的时候生成对应的引用路径。[插件是什么](https://www.webpackjs.com/concepts/plugins/)、[编写插件](https://segmentfault.com/a/1190000012840742)

首先，先创建插件js，这次我在webpack文件夹下创建一个插件plugins文件夹，放置一些插件js
```
app/public
      |-- webpack
            |-- plugins
                  |-- jsScriptsPlugins.js
```
jsScriptsPlugins.js就是我们创建的插件js，内容如下
```
const fs = require('fs');
const path = require('path');

var JsScriptsPlugins = function(){

}

JsScriptsPlugins.prototype.apply = function(compiler){
  compiler.plugin("emit", function(compilation, done){ // emit 事件
    for(let fileName in compilation.assets){ // 获取输出的静态资源并遍历
      var name = fileName.split('.')[0];  // 获取js前缀
      var template = `module.exports = { // 设置内容模板
        'path': ['public/release/${fileName}']}` 
      var filePath = path.join(__dirname, "../../scripts/") + name + ".js";// 引入路径的存放js
      fs.writeFile(filePath, template, 'utf8', function(error){ // 写入文件
        if(error){
          console.log(error)
        }
        console.log(`\x1B[36m写入${filePath}成功\x1B[0m`);
        console.log(`\x1B[36m内容为${template}\x1B[0m`);
        console.log('\n');
      });
    }
    done();
  });
}


module.exports = JsScriptsPlugins;
```
Webpack 会调用 BasicPlugin 实例的 apply 方法给插件实例传入 compiler 对象。[compiler 钩子](https://www.webpackjs.com/api/compiler-hooks/)

在 Webpack 运行的生命周期中会广播出许多事件，Plugin 可以监听这些事件，这次我们用的是emit事件
* emit 事件

> 生成资源到 output 目录之前。

* compilation.assets

> 存放当前所有即将输出的资源

* fs.writeFile()

> 文件写入：fs.writeFile('文件路径'，'要写入的内容'，['编码']，'回调函数');

我们在emit事件中，获取到了即将打包的js资源，因为是对象类型，我们用for...in进行遍历操作。

遍历的操作流程大概为
> 获取js名 => 获取js前缀 => 设置内容模板 => 设置对应的存放路径js => 将设置好的内容写入文件里

模板内容是根据打包后，输出的地址来设置的

【注】这里有个需要注意的，emit事件的第二个参数done，这是个通知函数，处理完毕后执行done，以通知 Webpack，如果不执行done，运行流程将会一直卡在这不往下执行 

### 引用插件
插件设置好了，现在开始引人插件，回到webpack文件里来

```
// 引入js
const jsScriptsPlugin = require('./plugins/jsScriptsPlugins');

module.exports = {
  entry: entry,
  output: {
    filename: "[name].[chunkhash].js",
    path: outputPath,
    publicPath: path.resolve(__dirname, "../release")
  },
  plugins: [
    new jsScriptsPlugin(),  // 加入插件
  ]
}

```
这样插件就生效了

### 不同环境引用js

我们现在打包了两种环境的js，如何区分引用才好呢？

我是简单的在context.js里添加函数来获取
```
module.exports = {
  getProdScript(name){
    return require('../public/scripts/' + name)['path'];
  },
  getDevScript(name){
    return 'public/dist/' + name + '.js';
  }
}
```
分别返回不同环境，相同js的路径，然后我们在controller里，将路径作为数据传入到ejs，ejs再进行引入

```
// app/public/controller/home.js
const Controller = require('egg').Controller;

class HomeController extends Controller {
  async index() {
    const { ctx } = this;
    await ctx.render('home.ejs',{script: ctx.getDevScript('a')});
  }
}

module.exports = HomeController;

// app/public/view/home.ejs

<script src="<%= script %>"></script>
```

## 可配置打包文件
我们在之前的打包入口配置中，都是已写死的方法来做的，现在我们要把打包入口配置，进行优化

### 配置文件
先建一个文件夹config，用来放配置文件,文件配置按下面的来

> app/public/webpack/config/a.js

```
const path = require('path');

module.exports = {
  on: 1,  // 是否需要打包
  scriptRules: [
    {
      enable: true, // 为了支持配置多个文件，这里再细化处理了，判断这个js是否需要打包
      name: 'a',  // 打包入口配置键
      entry: path.resolve(__dirname, '../../a.js')  // 打包入口配置值
    },
  ]
}
```

### 生成入口配置

再在webpack下创建webpack.settings.js文件

> app/public/webpack/webpack.settings.js

老样子，先上代码
```
const fs = require('fs');
const path = require('path');
const confPath = path.join(__dirname, "./config");

function getEnrty (confPath) {
  if(fs.existsSync(confPath)){  // 判断路劲是否存在
    var files = fs.readdirSync(confPath); // 获取路径下的所有文件，返回数组
    var fileArray = [];
    files.forEach(file =>{
      var filePath = path.join(confPath, file); // 配置文件的路径
      var stat = fs.statSync(filePath); // 获取文件信息
      var isFile = stat.isFile(); // 判断是否是文件
      if(isFile){ //  如果是
        fileArray.push(path.join(confPath, file));  // 存入数组
      }
    })
    // 遍历文件数组，提取文件的内容，判断on是否为1，将需要打包的内容重新放入一个数组
    var configModules = fileArray.map(require).filter(item => item.on === 1);
    var entry = {};

    // 遍历需要打包的文件配置，再细化，判断scriptRules是否需要打包，最后将需要打包的文件配置按入口要求，设置到对象里
    configModules.forEach(config => {
      config['scriptRules'].forEach(scriptRules => {
        if(scriptRules.enable){
          entry[scriptRules['name']] = scriptRules['entry']
        }
      })
    });
    return entry; // 返回入口配置
  }else{
    return {};
  }
}

entry = getEnrty(confPath);

module.exports = entry;
```
先讲下这个文件作用，这个js会把config下的配置文件内容全部读取，然后根据配置，分辨哪些js需要打包，哪些不需要，最后返回一个对象，用于打包入口

### 引用入口配置

引用这个配置就很简单了

在webpack打包js里，直接引用
```
const entry = require('./webpack.settings'); // 引入配置对象


entry: entry, // 主文件入口，加上即可用
```

这个方法，开发环境和生成环境打包都可以用，十分方便，统一管理，统一配置

## app.js，服务启动时执行

在项目根目录下创建app.js文件，暴露一个函数，项目启动时会执行这个函数

```
// app.js
module.exports = function () {
  console.log("服务启动")
}
```
这样，每次我们项目启动的时候，都会看到"服务启动"