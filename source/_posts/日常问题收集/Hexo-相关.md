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

## Hexo的文档中添加图片

hexo 文档中添加图片的话，需要安装一下 hexo-asset-image 插件
```js
npm install https://github.com/CodeFalling/hexo-asset-image --save
```
使用下边这种方式即可引入，加载图片是以source为根路径的，如果图片放的位置是这样的：/source/image/a.png, 使用时引入方法：image/a.png 即可！
```js
![图片默认文字](路径)
```

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