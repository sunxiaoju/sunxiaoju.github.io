---
layout: post draft
title: redis 分布式锁
date: 2022-04-2 21:37:37
tags:
---

## 背景
在项目开发使用eggjs开发项目，在多个实例上运行的时候，同一个定时任务，可能是会多次调用，造成跑出来的数据错乱。

## 解决思路
我们采用比较简单的方式通过redis锁实现分布式锁：

1. 在执行job之前，先创建一个redis锁
2. job执行完成之后，再将redis解锁


## 通过 Redlock 插件实现
Redlock 在 Redis 单实例或多实例中提供了强有力的保障，本身具备容错能力，它会从 N 个实例使用相同的 key、随机值尝试 set key value [EX seconds] [PX milliseconds] [NX|XX] 命令去获取锁，在有效时间内至少 N/2+1 个 Redis 实例取到锁，此时就认为取锁成功，否则取锁失败，失败情况下客户端应该在所有的 Redis 实例上进行解锁。

#### 优势
- redislock 支持多实例
- 为防止死锁在创建锁的时，同时一个超时时间
- 操作起来比较方便

#### 插件

```js
yarn add redlock@4.2.0
yarn add @types/redlock
yarn add ioredis
```

#### 使用
我们可以封装一个redislock工具，然后再项目启动的时候，初始化一次，然后挂在到全局的上，方便直接调用，以 eggjs 为例介绍，


```js
// Redislock.ts
const RedLock = require('redlock');
const Redis = require('ioredis');
// 导入ts类型
import Redlock = require('redlock');

type LockType = (key: string) => Promise<Redlock.Lock | null>;

interface RedisType {
  host: string;
  password: string;
  port: number;
  db: number;
};

// redis集群实例信息
const redisList = [{
  host: '10.1.1.1',
  password: 'xxxx',
  port: 6379,
  db: 0,
},
{
  host: '10.1.1.1',
  password: 'xxxx',
  port: 6379,
  db: 0,
}]


// redis 可以值前缀
const REDIS_KEY = 'JOB_LOCKER';
// 防止死锁，设置超时时间
const REDIS_TIMEOUT = 10 * 60 * 1000;

class Redislock {
  redlock: Redlock;
  constructor() {
    const clientList = redisList.map((item: RedisType) => (new Redis(item)));
    // 实例化redlock锁，支持redis集群
    this.redlock = new RedLock(clientList, {
      // 重试延迟（ms）
      retryDelay: 200,
      // 重试次数
      retryCount: 5,
    });
  }

  /**
   * @sub 创建redis锁方法
   * @params key 区分不同的job，采用不同的锁
   * @return job锁
   */
  lock: LockType = async key => {
    try {
      const redislock = await this.redlock.lock(`${REDIS_KEY}_${key}`, REDIS_TIMEOUT);
      return redislock;
    } catch (err) {
      return null;
    }
  };
}

export default Redislock;
```

```js
// app.ts
import RedisLock from '.redisLock';
// 项目启动支出，初始化一个redis锁
async didReady() {
  //实例化一个redis锁，挂载到app上
  this.app.redlock = new RedisLock(this.app);
}

```

```js
  // 定时处理上报的数据 redis同步到base数据
  async task(ctx) {
    const lock = await app.redlock.lock('report');
    if (lock) {
      
      // job任务信息

      // 释放锁
      lock.unlock();
    }
  }
```

