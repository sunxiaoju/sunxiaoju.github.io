---
layout: post draft
title: 下载字节流文件
date: 2022-03-13 17:45:57
tags: 日常问题
---

### 背景
上篇整理了附件下载，所以就像整理一下字节流格式的文件下载都整哪些方式


参考： https://juejin.cn/post/6989413354628448264#heading-6
### 为什么要使用字节流的方式下载文件
- 加密
  直接通过地址下载是没有办法进行加密的，但是通过字节流的话，可以对每一个byte或者是一些其他手段进行编码，客户端封装一个解码的类就好。
- 记录
  服务器难免会有一些需求，哪个用户都下载了哪些资源，哪些玩家都做了哪些事情等等。。。如果没有get或者post请求去下载或者访问的话，服务器很难做到对数据进行记录。
- 批量获取
  在做应用或者是游戏的时候，难免会批量的获取一些资源，将筛选条件发送给服务器，服务器整合资源，用一个流将所有文件和到一起发送，客户端在以相应的规则拆分字节流，存储资源。既方便记录，也方便客户端分辨资源。

### 下载处理方式
字节流下载，就跟我们的正常接口请求一样，只不过是通过接口请求将子接口下载到浏览器，然后浏览器再将其转化为文件。

根据项目实例来战来解析字节流下载的一下相关处理方式
```js

export function downloadFile(url, data = {}, method = 'get', callback) {
  const options = {
    // 将返回类型设置为 arraybuffer类型
    responseType: 'arraybuffer',
  };
  let requestMethod;
  if (method.toLowerCase() === 'get') {
    requestMethod = axios.get(url, options);
  } else {
    requestMethod = axios.post(url, data, options);
  }
  requestMethod.then(response => {
    // 返回码 status不等于200的时候返回报错
    if (response.status != 200) {
      // 将arraybuffer数据转为 json
      const errData = JSON.parse(String.fromCharCode.apply(null, new Uint8Array(response.data)));
      if (typeof callback === 'function') {
        callback({
          type: 'error',
          data: errData.message
        });
      }
      return false;
    }
    // 获取请求头中返回的文件名字
    let fileName = window.decodeURI(
      response.headers['content-disposition'] && response.headers['content-disposition'].split('=')[1],
    );
    if (fileName === 'undefined' || fileName === null || fileName === '') {
      if (typeof callback === 'function') {
        callback({
          type: 'error',
          data: response.headers.msg || '未获取到文件名'
        });
      }
      return false;
    }
    if (fileName.startsWith('"') && fileName.endsWith('"')) {
      fileName = fileName.slice(1, -1);
    }

   
  });
}

saveFile = (response, fileName) => {
  // Blob转化的文件类型，如果格式不匹配的话，转文件的话会乱码，取文件名字的后缀判断即可
  const mimeTypeMapping = {
    xlsx: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet;charset=UTF-8',
    xls: 'application/vnd.ms-excel;charset=UTF-8',
    docx: 'application/vnd.openxmlformats-officedocument.wordprocessingml.document;charset=UTF-8',
    doc: 'application/msword;charset=UTF-8',
    pdf: 'application/pdf;charset=UTF-8',
  };
  
  const fileType = fileName.split('.')[1];
  const link = document.createElement('a');

  // 将文件转化为链接
  link.href = window.URL.createObjectURL(
    new Blob([response.data], {
      type: mimeTypeMapping[fileType],
    }),
  );
  link.download = fileName;
  document.body.appendChild(link);
  link.click();
  if (typeof callback === 'function') {
    callback({
      type: 'success'
    });
  }
  document.body.removeChild(link);
}
```

### 批量下载
- 多次循环调用接口下载即可
- 可以让server端将文件字节流放在一起返回，然后client根据约定的分割格式，去拆分并形成不同的文件。（未尝试过）