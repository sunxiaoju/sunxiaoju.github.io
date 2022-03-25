---
layout: post draft
title: csrf 项目实际使用
date: 2022-03-20 16:59:54
tags: web 安全
---

## eggjs 开启CSRF防护
eggjs官方文档： https://www.eggjs.org/zh-CN/core/security#%E5%AE%89%E5%85%A8%E5%A8%81%E8%83%81-csrf-%E7%9A%84%E9%98%B2%E8%8C%83

### 开启CSRF配置
```js
// config/config.default.js
module.exports = {
  security: {
    csrf: {
      enable: true
    },
  },
};
```
开启之后，在网页访问的时候，会默认网cookie中set一个csrfToken

### eggjs 提供一个查询csrfToken接口

- 为什么要在eggjs中提供一个接口呢？
官方提供的文档中有这么一段描述：
```js
Double Cookie Defense：将 token 设置在 Cookie 中，在提交（POST、PUT、PATCH、DELETE 等）请求时提交 Cookie，并通过 header 或者 body 带上 Cookie 中的 token，服务端进行对比校验。
```
客户端我们需要的header中和body中也需要传输一下这个csrfToken的信息，但是为了安全起见，项目中所有的cookie都需要是httponly，所以的客户端就读取不到这个cookie，就需要在egg中加一个接口去返回这个token。
需要注意的是，这个查询csrfToken需要放置在界面渲染之后，到再开始渲染界面，防止异步操作引起失败。

- 开启一个新的接口

```js
/**
 * 首先我们采用的方案，在一个会话固定采用一个token，用户退出或者切换的时候，更换薪token
 * 1. 首先从请求的cookie中获取，如果浏览器的cookie中如果有CSRFToken的话，不会从新生成
 * 2. 如果请求的cookie中不包含csrfToken的话，此时egg会在response中返回一个setCookie，其中就包含了csrfToken，可以从这里边获取
 * 3. 如果都没有的胡啊，表示没有开启，置空即可
*/
getToken() {
  const { ctx } = this;
  const cookie = ctx.header?.cookie || '';
  let csrfToken = '';
  if (cookie) {
    csrfToken = getCookie(cookie, 'csrfToken');
  }
  if (!csrfToken) {
    const setCookie = ctx.response?.header?.['set-cookie'] as [] || [''];
    const cookiesInfo = setCookie.join(';');
    csrfToken = getCookie(cookiesInfo, 'csrfToken');
  }
  ctx.body = {
    csrfToken
  }
}

// 解析cookie信息
function getCookie(cookies, objName) {// 获取指定名称的cookie的值
  const arrStr = cookies.split("; ");
  for (let i = 0; i < arrStr.length; i++) {
    const temp = arrStr[i].split("=");
    if (temp[0] === objName) return unescape(temp[1]);  // 解码
  }
  return "";
}

```

- client 部分

增加请求，将返回的token，存储到sessionStorge或者直接挂在window上即可

在请求拦截中header中统一增加请求头： x-csrf-token
```js
axios.interceptors.request.use(
  config => {
    config.headers.Accept = 'application/json';
    config.headers['x-csrf-token'] = window.csrfToken || '';

    return config;
  },
  error => {
    error && message.error(error.msg || error.code);
    return Promise.reject(error);
  },
);
```
- 最后注意在退出登录或者切换用户的时候，需要手动刷新csrfToken，

```js
// egg中刷新csrfToken的方式
login() {
  // 调用 rotateCsrfSecret 刷新用户的 CSRF token
  ctx.rotateCsrfSecret();
  。。。
}
```
以上就可以完成eggjs最基本的一个CSRF防御，如果还不能满足需求的话，可以参考正文之前提供的，eggjs文档。

## koa中开启CSRF防护

### 插件引入
在koa中没有想egg中直接提供的一套方案，koa中csrf防护需要借助中间件来完成，我们这里采用的是:koa-csrf

```js
yarn add koa-scrf
```

### 中间件使用

```js
const CSRF = require('koa-csrf'); 

// 引入CSRF信息,默认会自动网cookie中set一个csrfToken
this._app.use(new CSRF({
  invalidTokenMessage: 'Invalid CSRF token',
  invalidTokenStatusCode: 403,
  excludedMethods: ['GET', 'HEAD', 'OPTIONS'],
  disableQuery: false
}));
```
- 开启一个新的接口， 在进入系统之前获取一下接口获取一下token

```js
/**
 * 获取token，
*/
router.get('/getToken', async(ctx, next) => {
  ctx.body = {
    csrfToken: ctx.csrf
  };
})
```
- client 部分 跟eggjs以往csrf赛一个即可

增加请求，将返回的token，存储到sessionStorge或者直接挂在window上即可

在请求拦截中header中统一增加请求头： x-csrf-token
```js
axios.interceptors.request.use(
  config => {
    config.headers.Accept = 'application/json';
    config.headers['x-csrf-token'] = window.csrfToken || '';

    return config;
  },
  error => {
    error && message.error(error.msg || error.code);
    return Promise.reject(error);
  },
);
```

- 更新token
```js
// 在入口处将 将值设置为空
ctx.session.secret = null
```