---
title: react问题集
tags: 日常问题
---

## react中路由跳转后页面不置顶问题

### 问题: 从页面A跳转到页面B，页面A滚动到中间位置，跳转后页面B也会在中间位置

### 解决方法：在顶部组件的生命周期中进行判断，例如：

```js 
componentWillReceiveProps(nextProps){​​​​​​
　　//当路由切换到新页面时置顶
　　if(this.props.location !== nextProps.location){​​​​​​
　　　　window.scrollTo(0,0)
　　}​​​​​​
}​​​​​
```

<br />

## 在async/await的方法函数中使用setState的问题

在正常的函数方法中调用时，使用的setState()是一个异步处理的过程，但是如果在async 的方法中使用setState()时，执行顺序是有变化的，稍不注意就会被带入坑中,

在使用async的方法中，在await之前的setState方法调用都是异步来执行的，但是在await之后，使用的setState方法就变成了同步执行的了，一张图很容易说明问题：

![图片](image/日常问题/异步加载.png)

