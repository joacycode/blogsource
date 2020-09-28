---
title: 限频节流函数debounce&throttle
date: 2018-12-01 10:54:21
categories: js
tags: 
 - debounce
 - throttle
toc: true
---

`lodash`的function模块里有两个方法，`debounce` 和 `throttle`，可以做节流限频使用，为在某些特定的环境下做必要的优化。
这两个方法了解原理后也可以自己去实现。

### 场景一 曝光埋点
在一个多屏长度的页面中，A模块的A元素在出现在屏幕可视区域时候（即曝光）触发埋点事件，随即上报相应参数。
思路：A模块A元素距离文档顶部的高度是固定值设为`bodyOffsetTop`，添加onscroll事件到window上，通过监听滚动事件，不断计算此时被卷入上去的文档高度`docScrollTop`,假设屏幕可视区域的高度为`viewHeight`。
当以下条件成立的时候，我们就认为该元素是出于可曝光的区域的。
{% codeblock lang:js%}
bodyOffsetTop <= docScrollTop + viewHeight
{% endcodeblock %}
具体实现代码：
<!--more-->
```javascript
//递归方式获得元素距离文档顶部真实高度，不受父元素position定位影响
const bodyOffsetTop = (el)=>{
    if(el.offsetParent.nodeName.toLowerCase() === 'body'){
      return el.offsetTop
    }else{
      return el.offsetTop + bodyTopOffsetTop(el.offsetParent)
    }
}
const verticalExpoEvent = (el,callback)=>{
  if( el && el.className.indexOf('hasExpoed') > -1 || !el) return
  const srcollyTop = document.body.scrollTop || document.documentElement.scrollTop
  const viewHeight = document.body.clientHeight
  const elBodyOffsetTop = bodyOffsetTop(el)
  const expoHeight =  srcollyTop + viewHeight
  if(expoHeight >= elBodyOffsetTop){
    el.className = el.className + ' hasExpoed' //增加曝光后不再曝光标识
    callback && callback()
  }
}
window.addEventListener('scroll',verticalExpoEvent)
```
以上代码可以实现功能，拿出去用看似也没有什么问题，毕竟现在的机器的都很高级嘛，处理器内存什么的依然可以有丝滑般体验。
但是，作为一个有追求的严谨的coding工程师,不仅要code的出来功能还要在乎功能的高性能高可用高维护，职业素养必修课之一啊。
上面的代码可以实现我们想要的功能，但是window的`onscroll`事件的监听是需要非常谨慎使用的，因为以像素为单位，以你滚动速度维度，以你一秒滚动一百个像素的来计算，机器需要一秒执行你的onscroll回调函数一百次，其中的计算量大多数都是无效的，非`pay-load`，以至于不怎么高端的机器，吃着火锅滑着屏幕的时候手机就死机了。
这时候就需要在有效时候做`节流`
*解决思路：* 无论以什么做触发单位，滑动速度有多快，都以固定的频率去执行监听事件。

### 场景二 表单合法性即时检查
用户在填写表单输入时候，需要即时的对用户的输入合法性进行检测,如敏感词等
思路： 
- 使用`onchange`或者`onkeyup`做监听
- 限制监听`keyCode`范围
- 剪切和黏贴事件会change表单值
- 中文输入法处理

```javascript
// between func
(function(){
let enLock = false
const between = (val, x, y)=>{
  if(typeof val !== 'number' || typeof x !== 'number' || typeof y !== 'number') throw new Error('Each agru should be a number')
  return (val >= x && val <= y)
}
$el.addEventListener('keyup',(e)=>{
  if(enLock) return
  if( between(e.keyCode, 48, 57) || between(e.keyCode, 65, 90)){
    console.log(e.target.value)
    // 合法性检查 ...
  }
})
$el.addEventListener('cut',(e)=>{
  setTimeout(()=>{
		console.log(e.target.value)
    // 合法性检查 ...
	},0)
})
$el.addEventListener('paste',(e)=>{
  setTimeout(()=>{
		console.log(e.target.value)
    // 合法性检查 ...
	},0)
})
$el.addEventListener('compositionstart', function(e){
    enLock = true
})
$el.addEventListener('compositionend', function(e){
    enLock = false
    console.log(e.target.value)
    // 合法性检查 ...
})

})()
```
以上代码也可以实现功能，但还是有问题，用户在进行输入的时候，每个字符汉字都需要进行整篇的合法性检查，违背了高效性，浪费了一定的资源，我们其实只需要在用户停止输入的短时间内或者离开表单时候进行合法性检查即可。
*解决思路：* 用户输入间隔期间或者离开之前进行检测。


### throttle & debounce
[debounce](https://www.lodashjs.com/docs/4.17.5.html#debounce)  限频
简单理解：在限定的时间内如果有新触发，那么上一次的触发无效，限时重新计算。限定的时间无触发或者大于限定的时间触发，都会按照正常触发执行。

[throttle](https://www.lodashjs.com/docs/4.17.5.html#throttle)  节流
简单理解：在限定的时间内如果有新触发，那么函数会按照限定时间间隔执行，直到限定的时间无触发或者大于限定的时间触发。

*举例子* 
有一个button $btn, 绑定click事件callback函数
```javascript
$btn.addEventListener('click', callback)
$btn.addEventListener('click', _.debounce(callback,1000,{leading:false,trailing:true}))
$btn.addEventListener('click', _.throttle(callback,1000,{leading:true,trailing:true}))

```
*啥都没做* 那么你点击的手速多快，这个callback函数就会被执行多少遍。
*debounce* 做了1秒的限频，无限快的手速点击$btn，10秒后callback被执行 *0* 次，11秒后callback被执行1次
*throttle* 做了1秒的限频，无限快的手速点击$btn，10秒后callback被执行 *10* 次


但是如果你是...
<img src='flash.jpg' width=200 style="border-radius:50%">
[闪电](https://baike.baidu.com/item/%E9%97%AA%E7%94%B5/19463688?fr=aladdin)点击的间隔超过1秒，那么这两个限制函数对你无效。

以上场景一和场景二的代码可优化为如下：
```javascript

// 曝光埋点
window.addEventListener('scroll',_.throttle(verticalExpoEvent,100))

// 表单合法性即时检查
$el.addEventListener('keyup',_.debounce((e)=>{
  if(enLock) return
  if( between(e.keyCode, 48, 57) || between(e.keyCode, 65, 90)){
    console.log(e.target.value)
    // 合法性检查 ...
  }
},500))
```
>原创内容，欢迎交流转载请注明出处