---
lyout:      post
title:      "MongoDB调研"
subtitle:   "MongoDB基本应用"
date:       2017-01-10 16:47:00
author:     "galway"
header-img: "img/home-bg-o.jpg"
catalog:  ture
tags:
    - mongoDB
    - 数据库
    - 存储
---


## 简介

mongo是C++开发的基于文件存储的一种文档式开源数据库系统。
其支持的数据结构非常松散，是一种二进制的json结构bson，因此可以存储复杂的数据类型。
并且其强大的查询语句，支持对数据进行索引，几乎可以实现关系型数据库中单表的所有查询功能。
因此，它是一个面向集合的，模式自由的文档型数据库。
 
### 特点

* 面向集合的存储，适合存储对象(包括大型对象，视频)和json形式的数据
* 模式自由，存储不需要定义任何结构
* 支持丰富的动态查询，查询指令使用json标记，可轻易查询内嵌对象和数组
* 支持索引
* 监视工具，可用于分析数据库性能
* 复制和故障恢复，支持服务器间的复制，进行数据冗余备份和故障处理
* 自动分片和云级别的动态扩展，水平的数据库集群，可动态添加机器
 

### 应用场景

#### 1.适用场景

* 网站数据，适合实时的插入，更新与查询，并具备网站实时数据存储所需的复制及高度伸缩性
* 信息基础设施的持久化缓存，性能高，可避免下层的数据源过载
* 对象和json数据的存储，文档化的查询和存储结构
* 大尺寸，低价值的数据，关系型数据库存储一些文件比较昂贵，可选择文件存储的方式
* 高伸缩性的存储场景，内置了对 MapReduce 引擎的支持，支持动态添加集群机器
 
#### 2.不适用的场景

* 高度事务性的系统
* 传统的商业智能应用
* 复杂的级联查询
 
## 操作

### 结构

#### 1.存储逻辑结构，与关系型数据库结构的对比

|MongoDB|关系型数据库|
|:--------:|:---------:|
|文档(document)|行(row)|
|集合(collection)|表(table)|
|数据库(database)|数据库(database)|
 
#### 2.数据结构

默认数据目录是/data/db，存储了所有mongoDB的数据文件。
一个mongo实例支持多个数据库，每个数据库都是由一个.ns文件和一些数据文件组成。.ns文件存储了数据库的元文件，包括其命名空间和索引等
mongo采用预分配空间的机制，始终保持额外的空间和空余的数据文件。数据文件每重新分配，大小为上个的2倍，并且最大为2G。
 
### CRUD基本操作

#### 1.插入

> ```
> > db.request_entity.insert({"code":123, "url":"www.baidu.com", "update_time":new Date()});
> > db.request_entity.find();
> { "_id" : ObjectId("58747e8d0bbf2f8da758aa6e"), "code" : 123, "url" : "www.baidu.com", "update_time" : ISODate("2017-01-10T06:26:21.039Z") }
> //request_entity 为collection, 在第一次插入时会默认创建, 默认存储的数据库为test
> ```  

_注意_：插入数据会自动生成一个主键_id, 类型为ObjectId。 MongoDB不支持原生的自增组件
 
#### 2.查找

* 普通查询

> ```
> > var data=db.request_entity.find().toArray();
> > data[0];  //数组形式
> > db.request_entity.find().forEach(printjson); //循环输出
> ```

* 条件查询

> ```
> > db.request_entity.find({code:123});              //select * from request_entity where code = 123
> > db.request_entity.find({code:123}, {url:true});  //select url from request_entity where code = 123
> > db.request_entity.findOne();                     //返回第一条数据
> > db.request_entity.find().limit(2);               //限制结果集长度
> ```
 
#### 3.修改

> ```
> > db.request_entity.update({code:123},{$set:{id:"jin.baidu.com"}});
> //code 为123的纪录中，url改为jin.baidu.com。 也可以添加新的字段
> ```
 
#### 4.删除

> ```
> > db.request_entity.remove({code:123});    //删除一行纪录
> ```

 
### 性能优化

#### 与mysql的性能对比

[链接](http://blog.csdn.NET/e421083458/article/details/8849247)给出了mysql和mongoDB的性能对比，mongo在大规模数据的情况下插入和查询上性能上表现优异。

mongoDB在删除数据方面性能不如musql，更新数据上当数据量过百万时mysql性能更好。
在实际项目中为了减小开销传统关系数据库在web开发中  会采取连接池的方式 提高效率，而在mongodb中mongo实例化其实就是一个连接，默认貌似是10个，高并发会采取队列的方式等待 线程安全，当然去生产环境肯定是需要配置的
 
#### 索引

* 基础索引

> ```
> > db.request_entity.ensureIndex({code:-1});    //code降序索引，1为升序，-1为降序
> ```

* 文档索引

> ```
> > db.request_entity.insert({code:123,addr:{"city":"beijing","tel":010}});
> > db.request_entity.ensureIndex({addr:1});      
> > db.request_entity.find({addr:{"city":"beijing","tel":010}});    //查询顺序也应该和索引一致
> ```

* 组合索引和唯一索引

> ```
> > db.request_entity.ensureIndex({code:1,addr:-1},{unique:true});
> ```

* 删除索引

> ```
> > db.request_entity.dropIndexes();
> > db.request_entity.dropIndex({code:1});
> ```

 
#### explain命令
explain()可以获知系统如何处理查询请求，观察系统如何利用索引来加快检索，从而针对索引进行优化

 
#### profile优化器
类似于Mysql中的慢查询日志，mongoDB中需要开启profile纪录慢查询日志或者全部查询日志
 
#### 常用优化方案

* 根据需求设计数据字段，包括范式化、非范式化、折衷设计
* 索引
由于mongoDB是内存数据库，所以当数据量大于内存时查询就会非常慢，因此要为常用的项建立索引。
另外，当数据索引大于内存时，也会造成查询速度变慢。因此比较重要的是，保证热数据的量小于内存。
因此，要设计合理的索引并且尽量少

* 限制返回结果条数
* 只查询使用到的字段，而不是全部字段
* 采用capped collection
* 采用Server Side Code Execution
* 使用Hint，强制使用索引
* 采用Profiling
* [引擎优化](http://diyitui.com/content-1459560904.39084552.html)
 
## 安装

#### 开发机

在开发机上可以直接使用jumbo安装mongo，安装流程如下：

1. jumbo search mongo
2. jumbo install mongodb-bin
3. cd /home/work/.jumbo/opt/mongodb/
4. mkdir data   mkdir log
5. export LC_ALL=C
6. 启动服务  ./bin/mongod --dbpath=/home/work/.jumbo/opt/mongodb/data --logpath=/home/work/.jumbo/opt/mongo/log/mongodb.log
7. shell交互页面  ./bin/mongo   默认端口号27017

 
#### 本地

1. 到[官方地址](https://www.mongodb.com/download-center#community)下载相应的安装包
2. 解压到相应的文件夹
3. 在相应文件夹下重复上述4-7

        


