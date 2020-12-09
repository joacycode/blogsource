---
title: dotjs+webpack渲染研究
date: 2019-02-15 11:35:01
categories: dotjs
tags:
  - webpack
  - dotjs
---
dotjs只有120行，比较轻量的模板渲染工具，也有其他的类似`nunjucks` `art-template`等等，三者都可以考虑使用感觉都不错，`dotjs`和`nunjucks`可以在浏览器和node环境里做模板引擎。当然现在的主流前端框架把这类`jquery`时代的东西慢慢边缘化，node环境里也可以采用同构的方式，也不是必须需要这类工具了。如果在移动端开发中都不用考虑兼容性，ie8及以上都可以使用，ie8以下拒绝提供服务=。=
但是我想说，不是所有的项目都是要vue、react这样的框架去上手的，虽然用脚手架启动一个框架很容易，但是觉得没有必要，如果你能想到最快的方式就是用原生的办法搞定，比如临时需要一个页面支持，无任何拓展维护，那么这个项目就不值得用框架大费周折。主流框架一定得用，但是不能站在其他能实现方案技术的对立面，玩呗~


> 北交委提醒您：实现方案千万条，合适第一条

#### 划重点：
  -- 使用`dotjs-loader`进行分模块开发
  -- 熟悉`dotjs`语法
  -- `.tpl` `.def`后缀可以随便起，建议使用`.tpl` 
  -- 模块填充数据后进行拼接
### template
模板这部分如果不放在`webpack`里使用`loader`的话，就需要使用`script`标签，通过`id`方式获取里面的内容，再塞进去数据，就是浏览器即时渲染，查看源码部分，`{` `}`这些插值符号都会存在。

```html
<script id="header" type="text/x-dot-template">
</script>
<script>
const headTpl = doT.template(document.querySelect('#header').innerHTML)
const html = headTpl({
  //...
})
</script>
```
<mark>以下使用webpack: </mark>
不熟悉`dotjs`语法下面有文档传送门
```html
<!-- head.tpl -->
{{##def.head:
  {{? it.styleNight}}
  <div class="head-b">黑色主题：{{=it.headBlack}}<div>
  {{??}}
  <div class="head-w">白色主题：{{=it.headWhite}}<div>
  {{?}}
#}}
{{#def.head}}
```
```html
<!-- body.tpl -->
<img src="{{=it.entryimg || 'http://dummyimage.com/320x180/2d8cf0/fff&text=joacycode'}}" width="100%"/>
<div class="title">标题：{{=it.title}}</div>
<div class="subtitle">副标题：{{=it.subtitle}}</div>
<div class="city">城市：{{=it.city}}</div>
<p>商品序号</p>
<ul class="skulist">
{{~it.list :value:index}}
<li>
{{=value}}
</li>
{{~}}
</ul>
```
```html
<!-- footer.tpl -->
{{##def.footer:
  <div class="footer">尾部信息：{{=it.footer}}<div>
#}}
{{#def.footer}}
```

### controller
```javascript
// 引入模板 dotjs-loader处理后为模板函数 入参数据
import bodyTplFn from './template/body.tpl'
import headTplFn from './template/head.tpl'
import footerTplFn from './template/footer.tpl'
```
```javascript
// 使用模板函数
  const bodyHtml = bodyTplFn({
    title,
    subtitle,
    list,
    city,
    entryimg
  }) // string
  const headHtml = headTplFn({
    styleNight,
    headBlack,
    headWhite
  }) // string
  const footerHtml = footerTplFn({
    needFooter,
    footer
  }) // string

  // 字符串模板拼接
  const html = headHtml + bodyHtml + footerHtml
```
```javascript
// 引入axios请求工具
import axios from 'axios'
```

easy-mock设置的随机数据如下：
<img width="100%" src='easymock.png'>
```javascript
// 从easy-mock获取数据
axios.get('https://www.easy-mock.com/mock/5c9dbda0bca325336a73bffc/mock/activity').then((res) => {
  const { data = {} } = res
  const {
    title = '',
    subtitle = '',
    list = '',
    styleNight = '',
    headBlack = '',
    headWhite = '',
    needFooter = '',
    footer = '',
    city = '',
    entryimg = ''
  } = data
  // ...
})
```
### webpack
默认情况，模板中的数据必须用 `it` 作为引用。修改设置中的 `varname`，可以改变默认的变量名。
还有其他可以更改的参数比如模板语法写法，控制空白字符等等
```javascript
//  dotjs-loader
 {
    test: /\.tpl$/,
    loader: 'dotjs-loader',
    options: {
      varname: 'it'
    }
  }
```

### style
增加`css-loader` `scss-loader` `mini-css-extract-plugin` `node-sass`.
`mini-css-extract-plugin`用于提取压缩css到同一个文件中
```javascript
{
  test: /\.scss|\.css$/,
  use: [
    MiniCssExtractPlugin.loader,
    'css-loader',
    'sass-loader'
  ]
}
```
```javascript
// 引入样式
import './assets/main.scss'
```
最终显示结果：
<img width="50%" src='done.png'>

#### 使用链接：
[Dotjs](http://olado.github.io/doT/index.html) 120行轻量化模板引擎
[Nunjucks](http://mozilla.github.io/nunjucks/templating.html) 本文拓展模板引擎
[EasyMock](https://easy-mock.com/) 在线mock数据，基于mockjs,使用占位符可随机生成数据
[Webpack Config](https://www.webpackjs.com/configuration/) webpack 官方比较全的配置表
[dotjs-loader](https://www.npmjs.com/package/dotjs-loader) dotjs的webpack模块化`loader`

#### 项目地址
[https://github.com/joacycode/dot-webpack](https://github.com/joacycode/dot-webpack)

>原创内容，欢迎交流转载请注明出处