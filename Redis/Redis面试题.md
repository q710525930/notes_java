# Redis面试题

## 数据结构与类型

- Redis有哪几种数据类型和结构？给你一个 key 怎么知道是用的哪种结构？
- Zset底层数据结构？适用场景？
- string的底层优化？list的使用场景？

## 持久化

- AOF与RDB区别
- RDB写入底层逻辑
- 一台机器8G Redis配置6G 采取rdb模式 读写请求比例为2比8 问会有什么问题？

## 集群

- 扩容时  新老旧节点 数据迁移具体是怎么做的？

- 集群为什么是16384，哨兵模式，选举过程，会有脑裂问题么

  - <details>
    	<summary>答案</summary>
      https://www.cnblogs.com/youngdeng/p/12855424.html?ivk_sa=1024320u
      简单总结，就是因为集群之间每s要互发PING/PONG交换消息，消息体重会携带myslot槽数据，格式为bitmap，每一位代表一个槽，如果该位为1，表示这个槽是属于这个节点的。16384÷8÷1024=2kb。在消息体中，会携带一定数量的其他节点信息用于交换。约为集群总节点数量的1/10，至少携带3个节点的信息，所以节点数量越多，消息体内容越大。
      如果槽位为65536，发送心跳信息的消息头达8k，发送的心跳包过于庞大，浪费带宽。redis的集群主节点数量基本不可能超过1000个，所以16438完全够用。槽位越小，节点少的情况下，传输过程中bitmap压缩比高
    </details>

- 集群的不足

  - <details>
      <summary>答案</summary>
      假设我有一个key，对应的value是Hash类型的。如果Hash对象非常大，是不支持映射到不同节点的！只能映射到集群中的一个节点上！还有就是做批量操作比较麻烦！
      批量操作也就是mset、mget等，集群不同的key会划分到不同的slot中，因此直接使用mset或者mget等操作是行不通的(应对方法:如果执行的key数量比较少，就不用mget了，就用串行get操作。如果真的需要执行的key很多，就使用Hashtag保证这些key映射到同一台redis节点上,语法：对于key为{foo}.student1、{foo}.student2，{foo}student3，这类key一定是在同一个redis节点上。因为key中“{}”之间的字符串就是当前key的hash tags， 只有key中{ }中的部分才被用来做hash，因此计算出来的redis节点一定是同一个!)
    </details>

## 键值特性

- Redis的主键争用问题如何解决

- Redis淘汰和过期策略？如果没有数据可以淘汰或者没有配置淘汰策略读请求可以正常执行吗？

  - <details>
      <summary>答案</summary>
      通过配置redis.conf中的maxmemory这个值来开启内存淘汰功能
      键存储在Redis中，有一个哈希表用于存储这批键及其值，如果这批键中有一部分设置了过期时间，那么这批键还会被存储到另外一个哈希表中，这个哈希表中的值对应的是键被设置的过期时间，实际上会消耗更多的内存，因此建议使用allkeys-lru策略有效率的使用内存。
      - 淘汰策略
      	- allkeys lru: 淘汰最长时间没使用的key
      	- allkeys random: 所有key随机删除
      	- allkeys lfu: 淘汰使用频率的key
      	- volatile lru
      	- volatile random
      	- volatile lfu
      	- volatile ttl：从配置了过期时间的键中驱逐马上就要过期的键
      - 过期策略
      	- 定期删除: 定期遍历设置了过期时间的哈希表，删除到期key，默认10/s，遍历采用贪心策略（随机选择20个key，如果20个中过期比例大于1/4，则重复执行，减少遍历大量key带来的cpu负载）
      	- 惰性删除: 访问指定key时进行过期检查，过期则立即删除且不会返回
      	- 定时删除: 创建过期时间时设置定时器，到时间立即删除
      不管是定期删除还是惰性删除都会存在key没有被删除掉的场景，所以就需要内存淘汰策略进行补充。
      其中lru方式也是同定期删除一样，随机选取N个数再对其进行lru算法删除。lfu计数counter也并不是采取线性增加的计算方式，且新key的初始化counter默认为5可以预防被轻易淘汰。
    </details>

- redis 内存优化？

  - <details><summary>答案</summary>
    	- Redis内部会构建一个数字池，默认是10000。Redis中如果存储的是“123”，Redis是能够识别出来这是一个数字并且按照数字来存储，节省存储空间，且直接存储数字池里面对应的索引就行
      - 复杂类型的存储优化，比如Map，List，Set等，这些集合都有一个特点可大可小
    </details>

## 性能

- Redis为什么是单线程

- key大批量失效？（血崩）

- redis读写分离？

  - <details>
      <summary>答案</summary>
      不做读写分离。edis本身在内存上操作，不会涉及IO吞吐，即使读写分离也不会提升太多性能，Redis在生产上的主要问题是考虑容量，单机最多10-20G，key太多降低redis性能.因此采用分片集群结构，已经能保证了我们的性能。其次，用上了读写分离后，还要考虑主从一致性，主从延迟等问题，徒增业务复杂度。
    </details>

## 应用

- 分布式锁 排行榜 怎么实现

- redis事务？

  - <details>
      <summary>答案</summary>
      生产上采用的是Redis Cluster集群架构，不同的key是有可能分配在不同的Redis节点上的，在这种情况下Redis的事务机制是不生效的。其次，Redis事务不支持回滚操作，简直是鸡肋！所以基本不用！
    </details>

- redis多数据库？

  - <details>
      <summary>答案</summary>
      单机下可以支持16个数据库（db0 ~ db15）,并且每个数据库的数据是隔离的不能共享，在Redis Cluster集群架构下只有一个数据库，即db0。因此，我们没有使用Redis的多数据库功能！
    </details>

