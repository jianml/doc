# Redis缓存

## Redis 数据类型

Redis支持五种数据类型：string（字符串），hash（哈希），list（列表），set（集合）及zset(sorted set：有序集合)。

https://www.runoob.com/redis/redis-data-types.html

## 部署方案

#### 单机模式

#### 主从模式

#### 哨兵模式

#### 集群模式

## 淘汰策略（6种）

作为一个内存数据库，redis在内存空间不足的时候，为了保证命中率，就会选择一定的数据淘汰策略，这篇文章主要讲解常见的几种内存淘汰策略。和我们操作系统中的页面置换算法类似。

https://baijiahao.baidu.com/s?id=1655596263155881264&wfr=spider&for=pc

## 持久化机制

https://www.jianshu.com/p/d3ba7b8ad964

#### AOF持久化

#### RDB持久化

## 缓存穿透、缓存击穿、缓存雪崩区别和解决方案

#### 缓存穿透



#### 缓存击穿



#### 缓存雪崩