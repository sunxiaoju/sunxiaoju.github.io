---
title: ymal文件读取
date: 2021-7-18
tags: 工具库
---

## ymal文件读取

### 背景

因为项目中配置文件放在nacos的配置文件中，从nacos配置中获取，后端使用的事yml文件，而前端用的是json文件的，并且每次修改nacos的话，都需要通过手动的修改方式去调整，而且到生产环境和测试环境的话，我们又没有权限修改，都维护到一个excel中统一由配管人员来统一修改，但是excel中的配置文件复制展示操作起来极不方便，并且还容易出问题。
所以现在改为通过脚本自动化修改配置，开发人员写好自动化的配置脚本之后，配管会员直接运行脚本就好了，这里就引起了一个问题，就是前端的配置文件使用的是json格式，而json格式又不支持自动化脚本修改

### 思路
可将json文件改为yml文件，node数据直接解析将yml文件服务为json对象即可实现

<!-- more -->

###  实现 

这里采用js-ymal插件读取yml文件转为json对应
- 安装
```js
npm install js-yaml
```

- 使用方式

```js
const yaml = require('js-yaml');
const fs   = require('fs');

try {
  // 读取项目本地的yml文件，如果是nacos的话，直接从获取到的就是fileYml
  const fileYml = fs.readFileSync('web.yml', 'utf8');
  const config = yaml.load(fileYml);
  console.log(config);
} catch (e) {
  console.log(e);
}
```


### 拓展

### YML是什么

YAML (YAML Ain't a Markup Language)YAML不是一种标记语言，通常以.yml为后缀的文件，是一种直观的能够被电脑识别的数据序列化格式，并且容易被人类阅读，容易和脚本语言交互的，可以被支持YAML库的不同的编程语言程序导入，一种专门用来写配置文件的语言。可用于如： Java，C/C++, Ruby, Python, Perl, C#, PHP等。

### YML的优点
- YAML易于人们阅读。
- YAML数据在编程语言之间是可移植的。
- YAML匹配敏捷语言的本机数据结构。
- YAML具有一致的模型来支持通用工具。
- YAML支持单程处理。
- YAML具有表现力和可扩展性。
- YAML易于实现和使用。

### 写法


阮一峰老师，这里对jison文件已经介绍的很清楚了，这里就不在赘述了，查看使用请移步至<a href="http://www.ruanyifeng.com/blog/2016/07/yaml.html">YAML 语言教程<a>