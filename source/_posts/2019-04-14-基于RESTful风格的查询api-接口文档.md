---
title: 基于RESTful风格的查询api设计-接口文档
date: 2019-04-14 13:59:31
categories:
tags:
---
在REST中，资源的返回结构与返回数量是由服务端决定；在GraphQL，服务端只负责定义哪些资源是可用的，由客户端自己决定需要得到什么资源,所谓的api查询。我们的想法是在REST中服务端决定的资源，其实也是通过客户端所给予的信息来进行反馈（资源定位和http的动作），如果客户端和服务端之间有一套约定，这个约定框架下客户端给的信息足够多，那么也可以让服务端来满足客户端的查询需求，即`按需请求`和`按需返回`，这个和GraphQL的思想是一致的。
至于为什么不统一使用GraphQL，每个公司的大背景下会有每种不同的实际情况，满足多端的需求又尽快的让服务端和客户端每个人都熟悉GraphQL，成本比较大；当然在有条件的额情况下统一使用GraphQL是最好的。所以我们就想用中间层和传统的REST来改造现有数据接口，又符合GraphQL的思想，在客户端的查询条件下，满足合并模块数据、过滤筛选冗余数据，选取特定数据的需求。



| 查询符号 | 定义  |
| ------ | ------ |
| $& | 与，默认 |
| $| | 或 |
| $^ | 非，剔除此条件下的数据 |
| $< | 小于，适用于数值  |
| $> | 大于，适用于数值  |
| $<= | 小于等于，适用于数值 |
| $>= | 大于等于，适用于数值  |
| $<> | 区间，适用于数值  |


```json
/**
$& 与 
$| 或
$^ 非
$< 小于
$<= 小于等于
$> 大于
$>= 大于等于
$= 等于
$<> 之间
*/ 
{
  "options": {
    "api": "https://api.xx.com/client.action",
    "params":{
      "client": "wh5",
      "functionId": "list",
      "clientVersion": "10.0.0",
      "jsonp": "_jsonpv17m5jchomm"
    },
    "filterWithoutMerge": true //过滤分类中是否要排除整合过的数据
  },
   // == 整合类，整合期望数据到一个分组中 ==
  "merge": {
    // titles分组  需要整合
    // 1. id为'665,666'且content不为'img'
    // 2. id为'667,668'且content为'skuid'或者'img'
    "titles": [{
      "id": "665,666",
      "$^content": "img"
    }, {
      "id": "667,668",
      "content": ["skuid", "img"]
    }],
    "skucards": {
      "type": "5"
    }
  },
  // == 过滤类，正向与反向过滤数据，可在options中配置过滤选项 ==
  "filter": {
    "$^mayLikeProducts":{  //剔除mayLikeProducts中以下满足条件的子项  mayLikeProducts为数组写法
      // 剔除 groupId小于8414500且shopId在024100和024112之间的子项目
      // 或者剔除groupName为'今日推荐'且shopName'戴尔商用商红专卖店'的子项目
      "$<groupId": "8414500",
      "$<<shopId": ["024100","024112"],
      "$|": {
        "groupName": "今日推荐",
        "shopName": "戴尔商用商红专卖店"
      }
    },
    "$^recommendProducts":{  //recommendProducts为对象,查找自身中以下满足条件 选择剔除或者保留自身
      "$>=leftStocks": "1000" //保留库存大于等于1000的recommendProducts
    },
    "$|":{
      // 或者保留headInfo(子项的hotProducts不为0的)
      // 保留mayLikeProducts(子项timeBegin大于1550163689000的)
      // 过滤对象本身为数组则适用于子项过滤，过滤对象为数值、字符串、布尔值、对象等针对自身做过滤
      "headInfo": {
        "$^hotProducts": "0"  //数值等于0或者数组长度为0
      },
      "mayLikeProducts": {
        "$>timeBegin": "1550163689000"  
      }
    }
  }
}

```

>原创内容，欢迎交流转载请注明出处