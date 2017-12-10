---
lyout:      post
title:      "redis实现锁机制"
subtitle:   "redus实现锁机制"
date:       2017-12-10 10:58:00
author:     "galway"
header-img: "img/home-bg-o.jpg"
catalog:  ture
tags:
    - 存储
    - redis
---


## 1.redis 常用锁命令

* SETNX
set if not exists,缓存不存在时设置
* INCR
键值+1,键不存在操作之前会先设置0

## 2.锁实现的方法

### 2.1 SETNX

锁实现的思路：

```
1.客户端A请求设置key,设置成功则加锁成功
2.客户端B请求设置key,返回失败,加锁失败
3.客户端A执行完代码,删除锁
4.客户端B请求,设置锁成功
5.客户端B执行完删除锁
```

```
//加锁 删除锁示例代码
    public function lock($key, $lefTime = 10)
    {
        $rs = $this->redis->SETNX($key, 1);
        if ($rs) {//无锁，加上锁并设置失效时间
            $set_i = 2;
            do {
                $setRs = $this->redis->EXPIRE($key, $lefTime);
                if (! $setRs) {
                    //warning log
                }
            } while (! $setRs && $set_i-- > 0);
        } else {//有锁，没有失效时间的加上失效时间
            $ltime = $this->redis->TTL($key);
            if (intval($ltime) === -1) {   //如果已经过期或者没有设置过期时间 直接删除锁 否则会无法释放
                $this->redis->EXPIRE($key, $lefTime);
            }
        }
        return $rs == 1 ? true : false;
    
    }
        
    public function delock($key)
    {
        $rs = $this->redis->DEL($key);
        if ($key) {//删除成功
            return true;
        } else {//删除失败
            return false;
        }

    }
```

### 2.2 INCR
锁实现的思路：

```
1.客户端A请求key获取值为1,表示加锁成功
2.客户端B请求设置key,返回2,加锁失败
3.客户端A执行完代码,删除锁
4.客户端B请求,设置锁成功
5.客户端B执行完删除锁
```

### 2.3 SET
使用SETNX和INCR也需要设置过期时间,这样就不是原子操作了,就可能存在expire失败导致锁不能释放的情况。解决办法一种是利用SETNX的示例代码中,其他进程读取到锁没有设置过期时间就加上过期时间。另外一种方法就是利用SET命令,直接加上过期时间

## 3.相关文档

[官方提供的redis分布式锁文档](https://redis.io/topics/distlock)
