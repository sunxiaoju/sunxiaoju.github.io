---
layout: post draft
title: node接入nacos
date: 2021-07-18 19:56:45
tags: 工具库
---

## 读取nacos配置文件

### 优点
- 为了更好、更灵活的管理项目文件，所以将项目的配置文件抽取到nacos中
- 为方便通过脚本控制更新，减少手动出错的概率，我们采用yml文件格式管理

### 所需插件
```js
yarn add nacos
yarn add js-yaml
```
<!-- more -->
### 使用说明
```js
const path = require('path');
const ymal = require('js-yaml');

/**
 * nacos配置信息可以都从node配置中获取  ---- process.env.x
 * 可以通过docker启动的服务，可以直接在配置的时候，注入到环境中
*/
const config = {
    // 服务器地址： nacos文件的服务器地址
    serverAddr: process.env.CONFIG_SERVER_URL || '',
    // 命名空间：，一般是根据根据不同的环境区分
    namespace: process.env.DEPLOY_ENV || 'dev',
    // 组：配置文件所属的组（项目）
    group: process.env.NACOS_GROUP || 'project',
    // 配置文件名
    dataId: process.env.NACOS_DATA_ID || 'web.yml',
};
...
// 根据namespace和 serverAddr 获取nacos对应服务和环境
const configClient = new NacosConfigClient({serverAddr: config.serverAddr, namespace: config.namespace});
/**
 * 根据 dataId 和 group 获取对应的配置文件, 获取配置文件许下
 * configClient.getConfig 是一个promise的异步方法
*/ 
nacosRes = await configClient.getConfig(config.dataId, config.group);
// 获取nacos文件，返回的是一个字符串，将字符串转为json格式
const nacosConfig = ymal.load(nacosRes);
```

### 实际项目使用时
一般本地开发不需要去连接nacos，只有线上的时候，本地可以调去项目内容的文件，方便修改调整

```js
// getNacos.js
const path = require('path');
const ymal = require('js-yaml');

/**
 * nacos配置信息可以都从node配置中获取  ---- process.env.x
*/
const config = {
    // 服务器地址： nacos文件的服务器地址
    serverAddr: process.env.CONFIG_SERVER_URL || '',
    // 命名空间：，一般是根据根据不同的环境区分
    namespace: process.env.DEPLOY_ENV || 'dev',
    // 组：配置文件所属的组（项目）
    group: process.env.NACOS_GROUP || 'project',
    // 配置文件名
    dataId: process.env.NACOS_DATA_ID || 'web.yml',
};
const getConfig = () => {
    return new Promise(async (resolve) => {
        let  nacosRes = '';
        // 通过启动的环境变量控制 NODE_ENV=development时，获取本地配置文件
        if (process.env.NODE_ENV === 'development') {
            // 读取本地配置文件内容 process.env.ENV 控制当前环境，用于区分使用哪个环境启动
            nacosRes = fs.readFileSync(path.join(__dirname, `./config.nacos.${ || 'local'}.yml`), 'utf-8')
        } else {
            // 根据namespace和 serverAddr 获取nacos对应服务和环境
            const configClient = new NacosConfigClient({serverAddr: config.serverAddr, namespace: config.namespace});
            /**
             * 根据 dataId 和 group 获取对应的配置文件, 获取配置文件许下
             * configClient.getConfig 是一个promise的异步方法
            */ 
            nacosRes = await configClient.getConfig(config.dataId, config.group);
        }
        // 获取nacos文件，返回的是一个字符串，将字符串转为json格式
        const nacosConfig = ymal.load(nacosRes);
        // 成功之后回调
        resolve(nacosConfig || {});
    })
}

module.exports = getConfig;

// server.js
/**
 * 先获取nacos信息，在启动服务
*/
getConfig().then(config => {
    // 获取nacos信息
})
```
### 参考
json转yml在线地址： https://oktools.net/json2yaml
本地配置文件的命名参考：config.nacos.dev1.yml

```js
// 启动命令
"scripts": {
    "start:dev1": "cross-env NODE_ENV=development ENV=dev1 node server",
  },
```