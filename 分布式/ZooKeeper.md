---
typora-root-url: pic
typora-copy-images-to: pic
---

# ZooKeeper

## 基础部分

### ZooKeeper数据结构

采用类似linux文件系统的层级结构：

<img src="/ZooKeeper数据树结构.png" alt="image-20200711232237542" style="zoom:50%;" />

其中节点node有4类：

- 持久节点
- 持久有序节点
- 临时节点
- 临时有序节点

持久节点，只能主动调用 delete 来删除，而临时的 znode，在当创建该节点的客户端崩溃或者关闭了与 ZooKeeper 的连接时，这个节点就会被删除。

持久类型的 znode 为应用保存数据，即使 znode 的创建者不再属于应用系统时，数据会保存下来而不丢失。临时 Znode 仅当创建者的会话有效时这些信息必须有效保存，会话超时或者主动关闭时，临时 znode 会自动消失。**有序节点**的意思是节点名称会自动被分配唯一一个单调递增的整数，类似mysql的自增长主键。

API：

> create /path data 创建一个名为/path的znode节点，并包含数据data。
>
> delete /path 删除名为/path的znode。
>
> exists /path 检查是否存在名为/path的节点。
>
> setData /path data
>
> getData /path
>
> getChildren /path

### ZooKeeper架构

<img src="/ZooKeeper服务器和客户端架构.png" alt="image-20200711233056383" style="zoom:50%;" />

集群模式下有多个server服务器，多个服务器可以复制数据，并且采取**ZAB一致性协议**。

客户端发送请求前需要与服务器集群建立会话，当服务器宕机时会话会自动转移到别的服务器。会话里面的多个请求采用先进先出序列执行。

## 内部原理

ZooKeeper也是需要选举出一个leader节点，处理所有变更请求（如post、put等）然后分发到各个从节点（形成**事务**），而查询请求直接由具体接收到请求的服务器节点本地处理了。每个事务有自己的id（zxid），用来保证各个节点按照顺序执行事务，也用于选举新leader时交换此id从而知道哪个节点版本最新进行同步。

zxid为long型整数（64位），分为两部分：前32位为**时间戳**（epoch），后32位为**计数器**（counter）。

### leader节点选举

其中leader节点选举也用到了zxid，服务器启动后进入LOOKING状态，如果已经存在leader则其他服务器会通知这个服务器，如果不存在则进行leader选举，选举过程：

1. 向其他节点发送自己的投票信息，信息包含了自己节点的节点id（sid）与zxid
2. 其他节点收到发送过来的投票信息后，如果对方**zxid比自己大，或者zxid相同的情况下sid比自己大**，就把当前节点的投票信息更改为收到的投票信息。

可以看出zxid最大也就是版本最新的节点将赢得选举，多个zxid相同的话sid最大的将赢得选举。一个服务器连接到仲裁数量的服务器发来的投票都一样时，就表示群首选举成功。

除了主从节点外，还有一种叫做观察者的节点类型，类似mysql的主备中的备，不参与选举，只是作为扩展读请求的节点。缺点就是新增加了节点会让每个事务多发送一条额外消息。

### 节点中处理器的构成

单机情况：

> PrepRequestProcessor 接受并执行客户端的请求，如果是写操作会生成事务；SyncRequestProcessor 负责将事务持久化到磁盘上。实际上就是将事务数据按照顺序追加到事务日志中，并形成快照数据。最后一个处理器为 FinalRequestProcessor，如果 Request 对象包含事务数据，该处理器就会接受对 ZooKeeper 数据树的修改，否则，该处理器会从数据树中读取数据并返回客户端。

集群：

- leader节点

> 第一个处理器同样是 PrepRequestProcessor，而之后的处理器则为 ProposalRequestProcessor，该处理器会准备一个提议，并将该提议发送给跟随者，并且会把所有请求转发给 CommitRequestProcessor，对于写操作请求，还会把请求转发给 SyncRequestProcessor 处理器。
>
> SyncRequestProcessor 和独立服务器的功能一样，是持久化事务到磁盘上，执行完后会触发 AckRequestProcessor 处理器，它仅仅生成确认消息并返回给自己。
>
> CommitRequestProcessor 会将收到足够多的确认消息的提议进行提交。

- 其余节点

> Follower 服务器是先从 FollowerRequestProcessors 处理器开始，该处理器接收并处理客户端请求，转发请求给 CommitRequestProcessor，同时也会**转发写请求到群首服务器**。CommitRequestProcessor 会直接转发读取请求到 FinalRequestProcessor 处理器，而且对于**写请求，在转发前会等待提交事务**。而群首接收到一个新的写请求时会生成一个提议，之后转发到追随者服务器，在收到一个提议，追随服务器会发送这个提议到 SyncRequestProcessor，SendRequestProcessor 会向群首发送确认消息。
>
> 当群首服务器接收到足够多确认消息来提交这个提议是，群首就会发送提交事务消息给追随者，当收到提交的事务消息时，追随者就通过 CommitRequestProcessor 处理器进行处理。为了保证执行的顺序，CommitRequestProcessor 处理器会在收到一个写请求处理器时暂停后续的请求处理。
>
> 对于观察者服务器不需要确认提议消息，因此观察者服务器并不需要发送确认消息给群首服务器，一般情况下，也不用持久化事务到磁盘。对于观察者服务器是否持久化事务到磁盘，以便加速观察者服务器的恢复速度，可以根据具体情况决定。

### 会话与监视

在独立模式下，单个服务器会跟踪所有的会话；而在**仲裁模式下会话则由群首服务器来跟踪和维护。而追随者服务器仅仅是简单地把客户端连接的会话信息转发到群首服务器**。

为了保证会话的存活，服务器需要接收会话的心跳信息。群首服务器发送一个 PING 信息给它的追随者们，追随者们返回自从最新一次 PING 消息之后的一个 session 列表。群首服务器每半个 tick 就会发送一个 ping 信息给追随者们。

监视点是由读取操作所设置的**一次性触发器**，每个监视点有一个特定操作来触发，即通过监视点，客户端可以对指定的 znode 节点注册一个通知请求，在发生时就会收到一个单次的通知。监视点只会存在内存，而不会持久化到硬盘，当客户端与服务端的连接断开时，它的所有的监视点会从内存中清除。因为客户端也会维护一份监视点的数据，在重连之后，监视点数据会再次同步到服务端。

## ZooKeeper源码

### 服务端

ZooKeeper 服务的启动方式分为三种，即单机模式、伪分布式模式、分布式模式。本章节主要研究分布式模式的启动模型，其主要要经过 **Leader 选举**，**集群数据同步**，**启动服务器**。

分布式模式下的启动过程包括如下阶段，

- 解析 config 文件；
- 数据恢复；
- 监听 client 连接(但还不能处理请求)；
- bind 选举端口监听 server 连接；
- 选举；
- 初始化 ZooKeeperServer；
- 数据同步；
- 同步结束，启动 client 请求处理能力。

#### 服务端启动流程

启动类是`QuorumPeerMain`

1. 参数化加载config文件后，启动`DatadirCleanupManager` 线程进行transaction log和snapshot的定时清理。

2. 根据参数判断单机还是集群模式，单机的话直接进入`ZooKeeperServerMain.main()`方法，集群进入runFromConfig函数；

3. 进入`QuorumPeer.start()`方法

   <img src="/QuorumPeer.start().png" alt="image-20200712121508043" style="zoom:67%;" />

   - 执行`loadDataBase`方法，建出内存数据结构 `DataTree`，完成**数据恢复**

   - 创建`ServerCnxnFactory` 工厂类实例，创建的`ServerCnxn`实例代表一个客户端与服务端的连接。

   - `startLeaderElection()`进行**leader选举**初始化工作，

     - Leader 选举涉及到节点间的网络 IO，`QuorumCnxManager` 就是负责集群中各节点的网络 IO，`QuorumCnxManager` 包含一个内部类 Listener，Listener 是一个线程，这里启动 Listener 线程，主要启动选举监听端口并处理连接进来的 Socket；`FastLeaderElection` 就是封装了具体选举算法的实现。

   - `super.start()`，QuorumPeer 本身也是一个线程，其继承了 Thread 类，这里就是启动 QuorumPeer 线程，就是执行 `QuorumPeer.run` 方法。

     - 一直循环通过 `getPeerState`方法获取当前节点状态，如果是LOOKING选举状态进入 ` FastLeaderElection.lookForLeader`方法，直到选举出leader跳出。
     - 然后 QuorumPeer.run 进行下一轮次循环，通过 `getPeerState` 获取当前 `serverState` 状态，如果是 LEADING，则表示当前节点当选为 LEADER，则进入 Leader 角色分支流程，执行作为一个 Leader 该干的任务；如果是 FOLLOWING 或 OBSERVING，则进入 Follower 或 Observer 角色，并执行其相应的任务。注意：进入分支路程会一直阻塞在其分支中，直到角色转变才会重新进行下一轮次循环，比如 Follower 监控到无法与 Leader 保持通信了，会将 `serverState` 赋值为 LOOKING，跳出分支并进行下一轮次循环，这时就会进入 LOOKING 分支中重新进行 Leader 选举。

     ![image-20200712121741548](/QuorumPeer.run.png)

[ZooKeeper 源码和实践揭秘](https://mp.weixin.qq.com/s?__biz=MjM5ODYwMjI2MA==&mid=2649745966&idx=1&sn=50a6b9892783b9509c02ac0db0f4167e&chksm=bed3795589a4f0439dd060cc6c4df9272a99ed6c55e4d1b96d5001d9249a5d53aee35c495a28&mpshare=1&scene=1&srcid=0629ovmWsWLD4kvqa7mxGUjb&sharer_sharetime=1593437387297&sharer_shareid=3a51fb775af6e5d8bb83940b3ddca7a7&key=9b24a98f9ab80cf5bdc800b98506279841066fdb9eec48131a21e73f670018ad7e6972bc3d6142c99989e501bae8ece2f375a01ec5303edd35a9bd12136a3e55d95957b895e98aeccc39b3acd003f5b2&ascene=1&uin=MjAzMzgzMTg4MQ%3D%3D&devicetype=Windows+10+x64&version=62090529&lang=zh_CN&exportkey=A5leeXANnXSkcMcnCYq%2FA7U%3D&pass_ticket=0jYrLeLPNd%2BCT3XAwCZDEoYbitBYQk2PGtjbV8rhIsBjqB2B6JkScb3SYtHwuUA5)

