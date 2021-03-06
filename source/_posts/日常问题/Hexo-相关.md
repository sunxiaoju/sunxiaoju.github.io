---
layout: post draft
title: Hexo 相关
date: 2021-06-26 21:39:40
tags: Hexo 
---


##  Hexo 生成的博客点击文章不会跳转界面，变成了直接下载
调查发现是因为修改了_config.xml（主文件夹下的设置文件，不是theme里面的）中的permalink参数，在修改中把最后一个斜杠给去掉了，重新加上之后可以点击下载了

```js
// 文章的 永久链接 格式 
permalink: :title/
```

<br/>

---

<!-- more -->

## Hexo的文档中添加图片

hexo 文档中添加图片的话，需要安装一下 hexo-asset-image 插件
将_config.yml文件中的配置文件改为true
```js
post_asset_folder: true
```
```js
npm install https://github.com/CodeFalling/hexo-asset-image --save
```
以在_posts文件中使用，在与创建的md文件的同一个文件夹下，创建一个与md同名的文件夹，然后再引入的时候，可以直接使用
```js
![图片默认文字](文件夹/图片)
```
例如：
在myFAQ文件夹下创建一个myInfo.md文件,需要引入图片时,则：
- 首先在myFAQ文件加载创建一个myInfo的文件夹，image.png放入文件夹中
- 在myInfo.md中引入：``` ![image](myInfo/image.png)  ```

## Hexo使用小技巧

- `<!-- more -->`： 在首页列表界面，为了能让多个文档信息展示看起来没那么乱，可以使用此标签使文档阶段，展示read more按钮

## 创建新的md文件
```js
  // 在__posts下创建一个 'test1.md'
  hexo new test1
  // 在__posts/工具库文件夹下创建一个 'test2'
  hexo new page --path 工具库/test2
```

```js
  hexo new [layout] <title>
```
- 新建一篇文章。如果没有设置 layout 的话，默认使用 _config.yml 中的 default_layout 参数代替。如果标题包含空格的话，请使用引号括起来.




## Hexo 遇到问题

###  hexo g报错  ---   line.mathALL is not funciton问题解决
```js
// 错误类似
FATAL { err:
TypeError: line.matchAll is not a function
at res.value.res.value.split.map.line (/home/ubuntu/hexoblog/node_modules/hexo-util/lib/highlight.js:128:26)
at Array.map ()
at closeTags (/home/ubuntu/hexoblog/node_modules/hexo-util/lib/highlight.js:126:37)
at highlight (/home/ubuntu/hexoblog/node_modules/hexo-util/lib/highlight.js:119:10)
at highlightUtil (/home/ubuntu/hexoblog/node_modules/hexo-util/lib/highlight.js:23:16)
at data.content.dataContent.replace (/home/ubuntu/hexoblog/node_modules/hexo/lib/plugins/filter/before_post_render/backtick_code_block.js:92:17)
...
```


### 原因
  是node版本问题,针对于低版本node问题, node12.0.0以上就没有这个问题
  node版本12.0.0的Node.js中支持String.matchAll()

### 解决办法 
- 如果是本地启动报错，升级nodejs版本到12.0.0以上版本
- 如果使用了travis CI 打包是出现的，在.travis.yml中，将 node_js对应的参数改为12或12以上版本
  ```js
  node_js:
    - 12 # use nodejs v12 LTS
  ```
- config.xml中的 highlight->enable的值从true更改为false，也可以避免异常。

两种解决方法

① 请将nodejs升级到高于12.0.0的版本。

② config.xml中的 highlight->enable的值从true更改为false，这样可以避免异常。
