---
title: 搭建egg项目
tags: ['egg', 'node']
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
只是简单的引用js肯定不行，我们要对js进行打包处理。[打包js](https://muguang146.github.io/2020/03/25/build-js/)