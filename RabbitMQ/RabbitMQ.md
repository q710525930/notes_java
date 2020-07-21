---
typora-copy-images-to: pic
typora-root-url: pic
---

# RabiitMQ

## RabbitMQ基础

MQ即消息队列，适用于以下场景：

1. 信息的发送者和接收者如何维持这个连接，如果一方的连接中断，这期间的数据如何方式丢失？

2. 如何降低发送者和接收者的耦合度？

3. 如何让Priority高的接收者先接到数据？

4. 如何做到load balance？有效均衡接收者的负载？

5. 如何有效的将数据发送到相关的接收者？也就是说将接收者subscribe 不同的数据，如何做有效的filter。

6. 如何做到可扩展，甚至将这个通信模块发到cluster上？

7. 如何保证接收者接收到了完整，正确的数据？

   **AMDQ**协议解决了以上的问题，而RabbitMQ实现了**AMQP**。

RabbitMQ结构：

1. Broker：消息队列服务进程
2. Exchange：消息队列交换机
3. Queue：消息队列

![image-20200629013000532](/RabbitMQ结构图.png)

所有模式都遵循的消息发布接收基础流程：

- 发布
  - product 与 Broker**建立TCP连接**
  - product 与 Broker**建立通道Channel**
  - 通过通道**发送消息给Broker**
  - Exchange将消息转发到对应接收队列
- 接收
  - consumer 与 Broker**建立TCP连接**
  - consumer 与 Broker**建立通道Channel**
  - consumer **监听接收队列Queue**
  - 接收队列消息

基础Demo：

```java
public class Producer01 {     
    //队列名称     
    private static final String QUEUE = "helloworld";     public static void main(String[] args) throws IOException, TimeoutException {         Connection connection = null;         
        Channel channel = null;
        try{             
            ConnectionFactory factory = new ConnectionFactory();             
            factory.setHost("localhost");             
            factory.setPort(5672);             
            factory.setUsername("guest");             
            factory.setPassword("guest");             
            factory.setVirtualHost("/");
            //rabbitmq默认虚拟机名称为“/”，虚拟机相当于一个独立的mq服务器            
            //创建与RabbitMQ服务的TCP连接             
            connection  = factory.newConnection();             
            //创建与Exchange的通道，每个连接可以创建多个通道，每个通道代表一个会话任
            channel = connection.createChannel();             
            /**              
            * 声明队列，如果Rabbit中没有此队列将自动创建              
            * param1:队列名称             
            * param2:是否持久化              
            * param3:队列是否独占此连接              
            * param4:队列不再使用时是否自动删除此队列              
            * param5:队列参数              
            */             
            channel.queueDeclare(QUEUE, true, false, false, null);
            String message = "helloworld小明"+System.currentTimeMillis();             
            /**              
            * 消息发布方法              
            * param1：Exchange的名称，如果没有指定，则使用Default Exchange              
            * param2:routingKey,消息的路由Key，是用于Exchange（交换机）将消息转发到指定的消息队列              
            * param3:消息包含的属性              
            * param4：消息体
             */
            /**              
            * 这里没有指定交换机，消息将发送给默认交换机，每个队列也会绑定那个默认的交换机，但是不能显 示绑定或解除绑定             *  默认的交换机，routingKey等于队列名称              
            */             
            channel.basicPublish("", QUEUE, null, message.getBytes());             System.out.println("Send Message is:'" + message + "'");         
        }
        catch(Exception ex){             
            ex.printStackTrace();         
        }finally{             
            if(channel != null){                 
                channel.close();             
            }             
            if(connection != null){                 
                connection.close();             
            }         
        }     
    } 
}

public class Consumer01 { 
    private static final String QUEUE = "helloworld";          public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory factory = new ConnectionFactory();         
        //设置MabbitMQ所在服务器的ip和端口         
        factory.setHost("127.0.0.1");         
        factory.setPort(5672);         
        Connection connection = factory.newConnection();         
        Channel channel = connection.createChannel();         
        //声明队列         
        channel.queueDeclare(QUEUE, true, false, false, null);         
        //定义消费方法         
        DefaultConsumer consumer = new DefaultConsumer(channel) {             
            /**              
            * 消费者接收消息调用此方法              
            * @param consumerTag 消费者的标签，在channel.basicConsume()去指定              
            * @param envelope 消息包的内容，可从中获取消息id，消息routingkey，交换机，消息和重传标志 (收到消息失败后是否需要重新发送)              
            * @param properties              
            * @param body              
            * @throws IOException
            */             
            @Override             
            public void handleDelivery(String consumerTag,                                        Envelope envelope,                                        AMQP.BasicProperties properties,                                        byte[] body)
                throws IOException {
                //交换机                 
                String exchange = envelope.getExchange();                 
                //路由key                 
                String routingKey = envelope.getRoutingKey();                 
                //消息id                 
                long deliveryTag = envelope.getDeliveryTag();                 
                //消息内容                 
                String msg = new String(body,"utf‐8");                 System.out.println("receive message.." + msg);             
            }         
        };         
        /**          
        * 监听队列String queue, boolean autoAck,Consumer callback          
        * 参数明细          
        * 1、队列名称          
        * 2、是否自动回复，设置为true为表示消息接收到自动向mq回复接收到了，mq接收到回复会删除消息，设置 为false则需要手动回复          
        * 3、消费消息的方法，消费者接收到消息后调用此方法          
        */         
        channel.basicConsume(QUEUE, true, consumer);       
    } 
}

```



## Rabbit基础工作模式

### Work Queues （工作队列）

![image-20200629014606224](/Work Queues.png)

基础结构上多个消费者，发送步骤不变，消费者轮询接收消息，一个消息只能被一个消费者接收一次，消费者处理完手上的消息后才能接收下一个消息。**避免立刻执行资源密集型任务**，好处是能够很容易的并行工作，仅仅**通过增加更多的消费者就可以解决问题**

### Publish/subscribe（发布-订阅）

![image-20200629015057986](/Publishsubscribe.png)

生产者发送到Broker的所有信息交换机都会发送到所有队列，所有**消费者都能收到所有信息**

### Routing

<img src="/Routing.png" alt="image-20200629020604098" style="zoom: 67%;" />

路由模式，顾名思义交换机内维持了各个队列的路由，具体消息将有交换机识别路由信息**发送到指定队列**，

### Topics

基础也是路由模式，但是与Routing不同在于**Topics可以一条消息路由到多条队列**，如果路由到所有队列则退化为发布-订阅模式

### Header

也是类似Routing，但是不同在于Routing使用类似ip的key来识别进行路由，而Header使用k-v作为识别。

### RPC

消费者主动申请调用生产者，消费者调用时**向RPC请求队列发送请求**调用，服务端监听RPC队列收到请求执行方法后**将结果加入RPC响应队列**，消费者监听RPC响应队列获取结果。

## RabbitMQ特性

### confirm 消息

消息发送流程为：生产者-Broker-消费者，上面是从整个大体保证生产者的消息成功发送到消费者，要想实现上面的消息可靠性传递，得保证消息能成功发送到Broker才行，confirm消息就是**Broker收到生产者者的消息后返回一个ack消息**

### Return 消息

Broker收到消息会返回一个ack回执，但是收到消息后如果这个消息找不到对应路由队列怎么办呢？生产者可以内部监听Return Listener，这样的话**生产者就会收到路由不可达的消息**

### 消费者限流

如果消费者进程down了一段时间，然后一回连上来发现队列里面有超过自己处理能力数量的消息，全接下来消费者就挂了。

RabbitMQ提供了一个方式，开启**非自动签收**，一定数量消息被消费者确认前，不会发送新消息。

### TTL队列/消息

也就是支持配置**消息生存时间**，超过会被自动清除

### 死信队列

当消息**被拒绝**、**TTL超时**、**队列长度已满**时会变成死信，会被发送到一个专门处理死信的Exchange(交换机)中，可以对这个交换机进行监听做相应处理。

## RabbitMQ应用

### 保障消息100%投递成功

以下四点是**生产端可靠性投递**的要求

- 保障消息的成功发出
- 保障MQ节点的成功接收
- 发送端收到MQ节点(Borker)确认应答
- 完善的消息进行补偿机制

方案一：

- 发送消息后将**消息存入数据库**，消费者收到消息后**修改数据库中消息状态为已收到**，定时任务去**重发超时未收到的消息**
- <img src="/消息落库.png" alt="image-20200629232015376" style="zoom: 50%;" />

方案二：

- 用一个回调任务监听Broker，将消费者处理后的confirm消息存入数据库，然后从消费者发送的二次消息队列接收查询数据库是否有此消息，没有则表明消费者没有发送comfirm消息，此消息需要重新发送。
- <img src="/消息二次发送.png" alt="image-20200629233954878" style="zoom:50%;" />

### 幂等性

概念：**一个操作多次执行产生的结果与一次执行产生的结果一致**

在RabbitMQ中通俗的讲，就是保证一个消息不会被执行多次（重复消费）。主流实现方法：

1. 唯一ID+指纹码，数据库主键去重

   - 根据消息生成一个全局唯一ID，然后还需要加上一个指纹码，目的就是为了保障这次操作是绝对唯一的。
   - 将ID + 指纹码拼接好的值作为数据库主键，就可以进行去重了。在消费消息前呢，先去数据库查询这条消息的指纹码标识是否存在，没有就执行insert操作，如果有就代表已经被消费了，就不需要管了
   - 坏处就是在高并发时，如果**数据库会有写入性能瓶颈**。
     - 解决方案：根据 ID 进行分库分表，对 id 进行算法路由，落到一个具体的数据库，然后当这个 id 第二次来又会落到这个数据库，这时候就像我单库时的查重一样了。利用算法路由把单库的幂等变成多库的幂等，分摊数据流量压力，提高性能。

2. 利用Redis原子性加锁

   - 利用redis的API是原子性特性，使用**redis加锁**的方式，进入请求之后首选尝试获取锁对象，锁对象的键就是用户的id，如果获取成功，判断用户已经同步数据，可以直接返回，提示用户已经同步；如果没有直接执行同步数据的业务逻辑，最后将锁释放。如果在进入方法之后获取锁失败，有可能就是在第一次请求还没有结束的时候，接着又发起了请求，那么这个时候是获取不到锁的，也就不会发生数据同步出现同步好几次的情况。

   - 问题：

     - 是否要进行数据落库,如果落库,那么数据库和缓存如何做到原子性?如果你想用事务,放弃吧,Redis缓存事务和MySQL事务根本不是同一个事务

     - 如果不落库,那么都存储到缓存中,定时同步的策略如何设置为好?
     - 假设redis写入读取都出问题，可以在Redis命令执行失败时，将消息落库

   - > [Redis如何保证接口的幂等性？](https://juejin.im/post/5d2c9292e51d4550723b14a2)

