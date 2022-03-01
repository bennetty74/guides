# Redis

Redis是单线程的内存K-V数据库。K是字符串类型的key，V包含多种数据结构。

## Redis数据结构

Redis包含如下数据结构：`String` `List` `Set`  `Hash` `Sorted Set` 

- String：**二级制安全**，意思可以存储图片文件等被序列化后的二进制对象。最大可存储512MB
  
  - HyperLogLog：
  
  - Bitmap：通过特殊指令，将string当成bit处理，可以在string中查询、删除修改string里面特定的bit。如`setbit key 10 1` 表示将第10位设置位1

- List：列表，按照插入顺序排序。一个列表最多可以包含2^32-1个元素（4294967295，每个表超过40亿个元素）。

- Set：集合，无序字符串集合。一个列表最多可以包含232-1个元素（4294967295，每个表超过40亿个元素）。

- Hash：同Java里面的Map，是key-value的键值对。一个hash最多可以包含232-1 个key-value键值对（超过40亿）。

- Sorted Set：时间复杂度插入删除O(logN)，按照内部评分(score的浮动值)进行排序。



> 以上数据结构是针对redis的value角度提出的。即redis包含key，而value包含如上的五种数据结构。

## Redis LRU



Redis可以使用 `maxmemory` 指令来限制最大使用内存，如果超过该内存限制则会执行配置的过期策略。



过期策略分为以下几种：

- noeviction：返回错误

- ALLKEYS
  
  - LRU：在所有key中选择使用时间最老的key清除掉。
  
  - Random：在所有key中随机选择key清除。

- VOLATILE
  
  - LRU：在设置了过期时间的key中选择使用时间最老的key清除掉
  
  - Random：在设置了过期时间的key中随机选择key清除
  
  - TTL：在设置了过期时间的key中选择即将过期时间最短的key清除掉



>  Redis中的LRU使用的是近似LRU的算法来执行过期清理的。本质上是选择样本集中的最老的key去清楚掉。



![](https://redis.io/images/redisdoc/lru_comparison.png)

如图，左上角的是理论上的LRU，使用时间很明显的分层(从上到下分别是最老key，未被清除的key和新添加的key)。

而右上角的样本集大小为10的近似LRU算法明显清除过程中存在一个绝对最老的key没被清除掉。

## Redis 分布式锁

Redis Distributed Lock



**分为单机版本和集群版本。**



### 单机版本

单机版本创建分布式锁的方式如下：

表示针对资源(key)设置一个随机value。NX标识不存在则创建，PX 30000标识过期时间为30S.



```
 SET resource_name my_random_value NX PX 30000
```



获取锁：当resource_name存在时表示获取锁失败。如果不存在表示可以获取锁，但是所得持有时间只有30S。如果30S内客户端没有释放锁，会由Redis自动释放。



释放锁：如下Lua代码表示，只有客户端创建时的 `my_random_value`为申请时的value时，才会成功释放锁。为什么这样做的原因是如果丹村通过del删除resource_name(key)，则可能存在如下情况：



假设客户端A申请锁成功，但是因为某些原因阻塞超过30S，被Redis手动释放；客户端B此时正好申请锁，因为已经被释放掉了，所以客户端B申请成功，但是此时客户端A阻塞完成，准备释放自己方式获取的锁，如果单纯通过del则会删除客户端B的锁，所以需要my_random_value用来唯一标识客户端申请的锁。___

```
if redis.call("get",KEYS[1]) == ARGV[1] then
    return redis.call("del",KEYS[1])
else
    return 0
end
```

### 集群版本



集群版本使用的是RedClock

大意是创建奇数个Redis实例，客户端需要向 `(N/2)+1` 个实例获得锁，才表示真正获得锁。

反之是获取失败。



**该算法未被证明可行性。** 



## Redis事务

`MULTI`   `EXEC` `DISCARD`  `WATCH`

Redis实现原理如下：

Multi指令开启事务，事务操作会放在队列里面。

EXEC执行队列里面的指令，如果其中有一条指令出错，2.6.5之前则会无视，继续执行；之后则是拒绝执行这个事务。

DISCARD则用来放弃事务，准确来说就是清空队列里面的指令序列。

WATCH是乐观锁(CAS)的实现指令，如下指令表明watch和exec期间如果mykey的预期值不同（即被其他事务修改过），该事务就会被丢弃。

```redis
WATCH mykey
val = GET mykey
val = val + 1
MULTI
SET mykey $val
EXEC
```



## 持久化

Redis持久化措施分为以下四种：

- RDB(Redis Database)：在指定间隔时间内存储Redis实例的数据快照。适用于数据备份，文件紧凑，当回复数据时候比AOF来得更快。Redis通过fork子进程的方式，由子进程负责创建数据快照。

- AOF(Append only File)：记录每次对Redis由修改或者删除的操作。由于详细记录了修改，默认位每秒fsync(同步过程有子进程处理)。所以在灾备的情况下，AOF方式丢失的数据最少。同时当AOF文件过大时，Redis还会对记录的指令进行重写以减小AOF文件的体积。

- No Persistence：不持久化

- RDB+AOF：两种结合，


