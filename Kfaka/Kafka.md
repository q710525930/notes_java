# Kafka

## 核心架构

<img src="/Users/xx/Downloads/notes_java/Kfaka/pic/image-20220320003339514.png" alt="image-20220320003339514" style="zoom:50%;" />

## 生产者produce原理

<img src="/Users/xx/Downloads/notes_java/Kfaka/pic/image-20220320003630708.png" alt="image-20220320003630708" style="zoom:50%;" />

因为java自带的serializer序列化比较重，所以kafka自己实现了序列化器；分区器会把消息根据分区发送到发送队列，其中**一个分区的数据全在一个队列Dqueue中**，一个队列默认16k，队列空间默认32m，此部分存储与内存中。当队列满了或者达到设置的固定拉取时间间隔（0ms表示实时拉）sender线程就会拉取发送队列中的数据。sender线程中维护了一个map，key为Broker，value为消息队列，使用滑动窗口发送，最大默认允许有5个消息没收到ack的情况下继续发送。

ack分三种情况：

- 0：不需要ack确认
- 1：Leader收到即可
- -1：所有节点都需要收到

收到ack确认后sender线程去清理发送队列中的数据。如果一直ack失败会进行重试，重试次数为int类型最大值

### 生产者提高吞吐量

1. 增加发送队列，32m -> 64m,16k -> 32k,
2. 增大发送队列固定拉取时间，0ms -> 5ms
3. 发送队列数据进行压缩，让sender一次多拉点

### 生产者数据可靠性

<img src="/Users/xx/Downloads/notes_java/Kfaka/pic/image-20220320014518816.png" alt="image-20220320014518816" style="zoom:50%;" />

### 数据重复（只消费一次）

- 至少一次：ACK级别设置为-1**+**分区副本大于等于2**+**ISR里应答的最小副本数量大于等于2
- 最多一次：ACK级别设置为0
- 精准一次：**幂等性**（指Producer不论向Broker发送多少次重复数据，Broker端都只会持久化一条） **+** **至少一次**
  - Broker端判断消息重复：**<PID, Partition, SeqNumber>**
    - PID是Kafka每次重启都会分配一个新的
    - Partition 表示分区号（因此**只能保证在单分区单会话内不重复**）
    - Sequence Number单调自增
    - <img src="/Users/xx/Downloads/notes_java/Kfaka/pic/image-20220320020738445.png" alt="image-20220320020738445" style="zoom: 33%;" />
  - **因为重启Broker后PID会丢失，所以精准一次消费需要引入事务**
    - <img src="/Users/xx/Downloads/notes_java/Kfaka/pic/image-20220320024128756.png" alt="image-20220320024128756" style="zoom:50%;" />

### 数据有序

kafka生产者 -> broker**只能保证单分区内有序**，多分区下等所有消息到了消费者后再进行排序还不如就只要单分区，单分区有序在开启幂等性的情况下，实现方式是**在broker接收数据时也缓存5个数据**（可以结合sender线程最多接受5个消息没收到broker的ack思考），对缓存的5个数据根据Sequence Number进行排序

<img src="/Users/xx/Downloads/notes_java/Kfaka/pic/image-20220320024740325.png" alt="image-20220320024740325" style="zoom:50%;" />



## Broker原理

### Broker选举

<img src="/Users/xx/Downloads/notes_java/Kfaka/pic/image-20220320035532807.png" alt="image-20220320035532807" style="zoom:50%;" />

1. broker启动注册到AR
2. 多个broker的controller抢先注册controller主节点（第一个到的）
3. broker主节点进行leader选举
4. broker主节点监听其他broker存活状态，上传leader和存活信息到ISR
5. broker从节点从ISR中拉取leader信息
6. broker主节点与生产者通信，从节点主动从主节点拉取信息，存储在log文件中，log文件由多个1g大小的segment组成，segment内部.log顺序写，index类似mysql的页内目录存偏移量
7. 如果leader挂了，那么按照ar中的顺序从前往后选，只要在isr中存活即可当新leader

tip：AR = ISR + OSR（ISR：当前存活生效节点，OSR：落后过多的未生效节点）

副本：topic下的节点概念，leader和follower都叫副本

### leader或者follower挂了怎么整

<img src="/Users/xx/Downloads/notes_java/Kfaka/pic/image-20220320042359016.png" alt="image-20220320042359016" style="zoom:50%;" /><img src="/Users/xx/Downloads/notes_java/Kfaka/pic/image-20220320042412534.png" alt="image-20220320042412534" style="zoom:50%;" />

简单的总结，就是follower挂了，从hw处再重新拉log，直到拉到最新分区的hw；leader挂了，也是其他从节点从hw开始再重新从主节点拉log。简单的说**hw代表当前分区下所有节点的一个一致性状态节点**



### 日志文件存储

**Topic是逻辑上的概念，而partition是物理上的概念，每个partition对应于一个log文件**，该log文件中存储的就是Producer生产的数 据。Producer生产的数据会被不断追加到该log文件末端，为防止log文件过大导致数据定位效率低下，Kafka采取了分片和索引机制， 将**每个partition分为多个segment。每个segment包括:“.index”文件、“.log”文件和.timeindex等文件**。这些文件位于一个文件夹下，该 文件夹的命名规则为:topic名称+分区序号，例如:first-0。

<img src="/Users/xx/Downloads/notes_java/Kfaka/pic/image-20220320044504733.png" alt="image-20220320044504733" style="zoom:50%;" />

<img src="/Users/xx/Downloads/notes_java/Kfaka/pic/image-20220320044552314.png" alt="image-20220320044552314" style="zoom:50%;" />

日志默认保存 7 天，对于只用保存一个key的最新消息情况，可以开启日志压缩（对于相同key的不同value值，只保留最后一个版本），缺点是对于同样的key早期记录会丢失，导致offset不连续

### kafka高效读写原理

1. 分区并行度高
2. 读采用index稀疏索引（类似mysql页内目录）
3. 写log顺序写磁盘（同样的磁盘，顺序写能到 600M/s，而随机写只有 100K/s，其**省去了大量磁头寻址的时间**）
4. 页缓存+零拷贝
   - <img src="/Users/xx/Downloads/notes_java/Kfaka/pic/image-20220320045119381.png" alt="image-20220320045119381" style="zoom:50%;" />
   - Kafka重度依赖底层操作系统提供的PageCache（页缓存）功能。当上层有写操作时，操作系统只是将数据写入 PageCache。当读操作发生时，先从PageCache中查找，如果找不到，再去磁盘中读取。实际上PageCache是把尽可能多的空闲内存 都当做了磁盘缓存来使用。



## 消费者原理

因为推送消息难以适应多个消费者消费速率不一致的情况，所以kafka消费者主动从broker拉数据，缺点是没消息消费者会陷入空转。

<img src="/Users/xx/Downloads/notes_java/Kfaka/pic/image-20220320050134556.png" alt="image-20220320050134556" style="zoom:50%;" />

kafka消费以消费者组为单位，每个消费者逻辑上属于一个消费者组。其中一个分区的数据可以被多个消费者组消费，但是只能被一个消费者组内的一个消费者消费

### 消费者组初始化

<img src="/Users/xx/Downloads/notes_java/Kfaka/pic/image-20220320050332556.png" alt="image-20220320050332556" style="zoom:50%;" />

1. 根据消费者组的groupId进行hash，决定该消费者组协调器在哪个broker
2. 消费者都加入消费者组
3. 协调器发送topic信息给消费者组leader，leader指定内部消费规则（哪个具体消费者消费哪个分区）后告诉协调器
4. 协调器拿到分区-消费者方案后进行指令下发
5. 消费者与协调器保持心跳检测（3s一次，45s默认下线），消费者消费时间过长（5min）也会再平衡



### 消费者消费过程

<img src="/Users/xx/Downloads/notes_java/Kfaka/pic/image-20220320051541622.png" alt="image-20220320051541622" style="zoom:50%;" />

client每次拉取通过最小拉取、最大拉取和拉取间隔（未达到最小值也拉）进行拉取，回调确认放入缓冲队列completedFetches，消费者再从缓冲队列中拉

### 消费者分区和再平衡

一个topic有多个分区，消费者组中的哪些comsumer去消费哪个分区，是几个字说不清的

主流消费分区策略：

1. Range
   - **以单个topic为粒度**
   - 对分区进行123排序，然后对消费者组内的消费者也进行123排序，分区由哪个消费者消费 = 分区id / 消费者id，这样的话消费者id靠前的可能会比靠后的多消费一个分区
   - 缺点是在有多个topic情况，消费者id靠前的会比靠后的多n（topic数量）个消费分区，压力可能略大
   - 挂掉后再平衡重新计算一遍即可
2. RoundRobin
   - **以所有topic为粒度**
   - 把所有的 partition 和所有的 consumer 都列出来，然后按照 hashcode 进行排序，最后 通过轮询算法来分配 partition 给到各个消费者。
3. Sticky（粘性分区）
   - 在执行一次新的分配之前， 考虑上一次分配的结果，尽量少的调整分配的变动，可以节省大量的开销。
   - 首先会尽量均衡的放置分区 到消费者上面，在出现同一消费者组内消费者出现问题的时候，会**尽量保持原有分配的分区不变化。**
4. CooperativeSticky

**默认策略是Range+ CooperativeSticky**。Kafka可以同时使用多个分区分配策略



### 消费者offset

<img src="/Users/xx/Downloads/notes_java/Kfaka/pic/image-20220320053001857.png" alt="image-20220320053001857" style="zoom:50%;" />

__consumer_offsets中存储以key-value格式，key为group.id+topic+ 分区号，value 就是当前 offset 的值。每隔一段时间，kafka 内部会对这个 topic 进行 compact，也就是每个 group.id+topic+分区号就保留最新数据。

kafka提供自动提交offset和消费者手动提交offset的功能，也能指定offset消费

### 重复消费和漏消费

- 重复消费：已经消费了数据，但是 offset 没提交。
- 漏消费：先提交 offset 后消费，有可能会造成数据的漏消费

<img src="/Users/xx/Downloads/notes_java/Kfaka/pic/image-20220320053844195.png" alt="image-20220320053844195" style="zoom:50%;" />

### 消费者事务（只消费一次）

需要Kafka消费端将消费过程和提交offset 过程做原子绑定，需要消费者下游也支持事务

<img src="/Users/xx/Downloads/notes_java/Kfaka/pic/image-20220320054127353.png" alt="image-20220320054127353" style="zoom:50%;" />

### 消费者提高吞吐量

1. 如果是消费者单机消费速度慢，可以增加分区数量然后增加消费者数量
2. 如果是消费者处理速度不及时，提高每批次拉取的数量。
