## java

### java并发

1. Java线程池核心参数与工作流程，拒绝策略

   > 工作流程：
   >
   > ![img](https://uploadfiles.nowcoder.com/files/20211017/530285728_1634401631650/thread-pool.png)
   >
   > 核心参数：
   >
   > 1. corePoolSize：核心线程池数量
   >
   > 2. maximumPoolSize：最大线程池数量
   >
   > 3. BlockingQueue：阻塞队列
   >
   > 4. keepAliveTime：非核心线程空闲后存活时间
   >
   > 5. TimeUnit：时间单位
   >
   > 6. ThreadFactory：创建线程的线程工厂
   >
   > 7. RejectedExecutionHandler：拒绝策略
   >
   >    ```
   >    AbortPolicy：默认的策略，直接抛出RejectedExecutionException
   >    DiscardPolicy：不处理，直接丢弃
   >    DiscardOldestPolicy：将等待队列队首的任务丢弃，并执行当前任务
   >    CallerRunsPolicy：由调用线程处理该任务
   >    ```

2. volatile关键字的原理与作用

   > 作用：**保证内存可见性与防止指令重排序**
   >
   > 1. **保证内存可见性原理**：
   >
   >    - 因为cpu与物理内存之间通信速度远远慢过cpu处理速度，普通情况下每个线程都会有自己的缓存，读取一个数据k=v后把这个值存入线程内部缓存，如果其他线程更新了内存中的值，但是线程因为缓存里面有会默认直接读取缓存里面的数据，并不知道内存中值实际已经发生了变化。变量加了volatile，之后线程就不会在缓存里面存储，每次读取这个值都会去内存里面读，这样值发生了变化就会随时知道。
   >    - 操作系统提供了总线锁定机制来保证缓存一致性：cpu通过cpu总线与外部通信，当cpu做修改操作时，会发出lock指令去锁住共享变量对应内存（独享的moment），其他cpu要操作该共享内存就阻塞。
   >    - 因为锁总线没有并发性，所以有了缓存一致性协议MESI：MESI每个单词为一个状态，cpu在对缓存数据进行读写的时候，需要根据其状态来决定是要执行或者阻塞。对于一些不能被缓存的MESI会失效。**但是MESI有致命问题，可能一个cpu修改了缓存数据后没有立马写会内存，此时其他cpu读取到的还是旧数据**
   >    - 加入volatile关键字会插入lock指令，相当于一个内存屏障，有三个功能：
   >      - 保证内存屏障前的代码都执行完毕后再执行内存屏障后的代码（防止指令重排序）
   >      - 将当前**处理器缓存内的数据立即写会系统内存**，先行发送原则保证（A在B之前执行，那么A的影响能被B知道）
   >      - 上条写会内存中的操作会让其他缓存该数据的cpu内缓存地址无效（其他处理器从总线传播器中嗅探到，发现自己缓存数据地址被修改后将其设置为无效状态，等到**下次修改该缓存时再从系统内存中读取到cpu缓存**）
   >
   > 2. **防止指令重排序原理**：
   >
   >    - 对象创建过程有三个步骤：
   >
   >      1. new开辟空间
   >      2. 执行init方法
   >      3. 赋值
   >
   >      如果没有禁止指令重排序可能会导致23步骤互换，假设程序执行了13但是还未执行2，其他流程判断对象已经赋值可用直接拿来使用导致了使用半初始化状态对象。java虚拟机volatile内部**使用内存屏障来禁止指令重排序**
   >
   > cpu读取数据流程：先从处理器L1和L2里面依次找，找不到就找所有线程共享的L3缓存，如果还找不到则在主存中去找；如果主存中找到则把数据所存在的行数据（64字节）一起放入所有缓存层级。inter cpu使用**MESI缓存一致性协议**保证缓存可见性，如果MESI不行则直接锁总线

3. Synchronized和Lock的实现原理与区别

   > synchronized：交给jvm托管执行，悲观锁，其他线程只能通过阻塞等待线程释放，cpu在线程切换时会有上下文切换，当多竞争的时候会有频繁切换上下文的开销。异常会自动释放锁；只能等待锁的释放，不能响应中断
   >
   > lock ： Java 写的控制锁的类，异常不会自动释放锁，必须在finally中释放锁。等待锁过程中可以用 interrupt 来中断等待
   >
   > 
   >
   > synchronized 使用 Object 对象本身的 wait 、notify、notifyAll 调度机制，而 Lock 可以使用 Condition 进行线程之间的调度。

4. synchronized原理

   > Synchronized通过对象内部的监视器锁（monitor）来实现，监视器锁依赖于底层操作系统的 Mutex Lock（互斥锁）实现。而操作系统实现线程之间的切换需要从用户态转换到核心态，这个成本非常高，状态之间的转换需要相对比较长的时间，这就是为什么 Synchronized 效率低的原因。
   >
   > monitor可以理解为一种同步机制，可以理解为一个对象，每一个 Java 对象就有一把看不见的锁，称为内部锁或者 Monitor 锁。Monitor 是线程私有的数据结构，每一个线程都有一个可用 monitor record 列表，同时还有一个全局的可用列表。每一个被锁住的对象都会和一个 monitor 关联，同时 monitor 中有一个 Owner 字段存放拥有该锁的线程的唯一标识，表示该锁被这个线程占用。

5. 

6. Java中创建线程的几种方式

   > 1. **继承Tread类，重写run方法**：创建Thread子类的实例，即创建了线程对象。调用线程对象的start()方法来启动该线程
   > 2. **实现Runable接口**：创建Runnable实现类的实例，并以此实例作为Thread的target来创建Thread对象，该Thread对象才是真正的线程对象。调用线程对象的start()方法来启动该线程。
   > 3. **创建Callable接口的实现类，并实现call()方法**：该call()方法将作为线程执行体，并且有返回值，可以抛出异常，可以获取一个Future对象了解执行结果
   >
   > 其中，实现Runable接口和实现Callable接口还可以继承其他类，可扩展性强，且多个线程可以共享一个target对象，非常适合多个相同线程来处理同一份资源的情况，从而可以将CPU、代码和数据分开，形成清晰的模型，较好地体现了面向对象的思想，但是要访问当前线程，则必须使用Thread.currentThread()方法。
   >
   > 使用继承Tread类创建，直接使用this即可访问当前线程。
   >
   > 在本质上，创建线程只有一种方式，就是构造一个 Thread 类（其子类其实也可以认为是一个 Thread 类）。
   >
   > 而构造 Thread 类又有两种方式，一种是继承 Thread 类，一种是实现 Runnable接口。其最终都会创建 Thread 类（或其子类）的对象。
   >
   > https://segmentfault.com/a/1190000037589073#:~:text=%E5%9C%A8%E6%9C%AC%E8%B4%A8%E4%B8%8A%EF%BC%8C%E5%88%9B%E5%BB%BA%E7%BA%BF%E7%A8%8B,%E5%85%B6%E5%AD%90%E7%B1%BB%EF%BC%89%E7%9A%84%E5%AF%B9%E8%B1%A1%E3%80%82
   >
   > 

7. 

8. AQS实现机制

   > AQS是多线程访问共享资源的同步器框架，提供了一个FIFO队列（双向链表），是一个抽象类，定义了一些同步锁状态获取方法（互斥和共享），通过**模板方法设计模式**，继承类中实现下述方法
   >
   > ```java
   > // 互斥模式
   > protected boolean tryAcquire(int arg);
   > protected boolean tryRelease(int arg);
   > 
   > // 共享模式
   > protected int tryAcquireShared(int arg);
   > protected boolean tryReleaseShared(int arg);
   > ```
   >
   > AQS原理：
   >
   > - AQS使用一个volatile的int类型的成员变量state来表示同步状态，通过CAS修改同步状态的值。当线程调用 lock 方法时 ，如果 state=0，说明没有任何线程占有共享资源的锁，可以获得锁并将 state加1。如果 state不为0，则说明有线程目前正在使用共享变量，其他线程必须加入同步队列进行等待。
   > - 当前线程获取同步状态失败时，同步器会将当前线程以及等待状态（独占或共享 ）构造成为一个节点（Node）并将其加入同步队列并进行自旋，当同步状态释放时，会把首节点中的后继节点对应的线程唤醒，使其再次尝试获取同步状态。
   >
   > node结构：
   >
   > | 方法和属性值 | 含义                              |
   > | :----------- | :-------------------------------- |
   > | waitStatus   | 当前节点在队列中的状态            |
   > | thread       | 表示处于该节点的线程              |
   > | prev         | 前驱指针                          |
   > | predecessor  | 返回前驱节点，没有的话抛出npe     |
   > | nextWaiter   | 指向下一个处于CONDITION状态的节点 |
   > | next         | 后继指针                          |
   >
   > waitStatus枚举值：
   >
   > | 枚举      | 含义                                           |
   > | :-------- | :--------------------------------------------- |
   > | 0         | 当一个Node被初始化的时候的默认值               |
   > | CANCELLED | 为1，表示线程获取锁的请求已经取消了            |
   > | CONDITION | 为-2，表示节点在等待队列中，节点线程等待唤醒   |
   > | PROPAGATE | 为-3，当前线程处在SHARED情况下，该字段才会使用 |
   > | SIGNAL    | 为-1，表示线程已经准备好了，就等资源释放了     |

9. Threadlocal原理、使用场景、内存泄漏问题

   > ![img](https://pic2.zhimg.com/80/v2-e2d4b8eac152596232d3e32313927d59_1440w.jpg)
   >
   > TreadLocalMap的key是指向TreadLocal实例的弱引用指针，当TreadLocal不被外部强引用时会被回收，导致key为null，但是value还被强引用无法回收造成内存泄漏，只有等到Thread线程结束时才能被回收。
   >
   > 为什么不是强引用而是弱引用呢？因为如果是强引用的话TreadLocalMap会一直持有ThreadLocal的强引用导致TreadLocal无法被回收必须手动删除。弱引用当key为null，在下一次ThreadLocalMap调用set(),get()，remove()方法的时候会被清除value值。
   >
   > 
   >
   > 每个Thread维护着一个ThreadLocalMap的引用，ThreadLocalMap是ThreadLocal的内部类，用Entry来进行存储调用ThreadLocal的set()方法时，实际上就是往ThreadLocalMap设置值，key是ThreadLocal对象，值是传递进来的对象。调用ThreadLocal的get()方法时，实际上就是往ThreadLocalMap获取值，key是ThreadLocal对象
   >
   > **ThreadLocal本身并不存储值**，它只是**作为一个key来让线程从ThreadLocalMap获取value**
   >
   > 
   >
   > 使用场景：
   >
   > 1. 保存线程上下文信息，避免参数显示传递，比如saas中的租户用户信息
   > 2. 线程间数据隔离
   > 3. spring事务启动时会给当前线程绑定jdbc connection对象，存放在TreadLocal中，这样在整个事务中都使用该绑定的connection执行操作，实现了隔离性。
   > 4. session会话等线程级别的操作（Session 的特性很适合 ThreadLocal ，因为 Session 之前当前会话周期内有效，会话结束便销毁）

2. 

11. 线程的状态及转移

    > 见多线程文档

12. Java如何保证线程安全

    > 1. 无状态
    > 2. 资源不可变
    > 3. 安全的发布
    > 4. volatile
    > 5. synchronized
    > 6. lock
    > 7. cas
    > 8. threadlocal

13. 悲观锁与乐观锁及其区别 

    > 悲观锁默认数据会被修改，所以获取数据就会加锁；乐观锁获取不加锁，只有在更新的时候判断是否被其他人修改过，如果被修改则不更新。读取频繁使用乐观锁，写入频繁使用悲观锁。

14. Synchronized锁升级的策略

    > ![preview](https://segmentfault.com/img/remote/1460000022904668/view)
    >
    > 1. 无锁
    > 2. 偏向锁：执行到synchronized代码块时锁对象变为偏向锁（通过cas修改对象头锁标志位），执行完同步代码块之后并不会释放锁，方便后面再次执行时判断偏向锁中对象头里面的线程id是否是自己，是的话就不用再次加锁。只有当其他锁竞争偏向锁时才会释放偏向锁。偏向锁的撤销，需要等待全局安全点，即在某个时间点上没有字节码正在执行时，它会先暂停拥有偏向锁的线程，然后判断锁对象是否处于被锁定状态。如果线程不处于活动状态，则将对象头设置成无锁状态，并撤销偏向锁，恢复到无锁（标志位为01）或轻量级锁（标志位为00）的状态。
    > 3. 轻量级锁：偏向锁被别的线程竞争时偏向锁升级到轻量级锁，通过自旋的方式尝试获取锁
    > 4. 重量级锁：轻量级锁自旋超过10次后默认升级，处于重量级锁时其他尝试竞争的线程会阻塞

15. 

7. 如何设置线程池参数

   > 线程池大小：
   >
   > - 线程池线程数量太小，大量请求响应比较慢，甚至会出现任务队列大量堆积任务导致OOM。线程池线程数量过大，会同时争取 CPU 资源导致大量的上下文切换（cpu给线程分配时间片，当线程的cpu时间片用完后保存状态，以便下次继续运行），从 而增加线程的执行时间，影响了整体执行效率。
   > - **CPU 密集型任务(N+1)**： 设置为 N（CPU 核心数）+1，多出来的一个线程是为了防止某些原因导致的任务暂停（线程阻塞，如io操作，等待锁，线程sleep）而带来的影响。一旦某个线程被阻塞，释放了cpu资源，而在这种情况下多出来的一个线程就可以充分利用 CPU 的空闲时间
   > - **I/O 密集型任务(2N)**： 线程等待 I/O 操作会被阻塞，释放 cpu资源，这时就可以将 CPU 交出给其它线程使用。因此可以多配置一些线程，具体的计算方法：最佳线程数 = CPU核心数 * (1/CPU利用率) = CPU核心数 * (1 + (I/O耗时/CPU耗时))，一般可设置为2N

8. 

9. 

10. 

11. 

12. 

13. 

14. 

23. ReentrantLock原理

    > ReentrantLock是Lock接口的**唯一实现类**，是一个**可重入锁**，分为**公平锁**和**非公平锁**，二者的区别就在获取锁机会是否和排队顺序相关。
    >
    > ```java
    > public void lock() {
    >         sync.lock(); // 调用静态内部类sync的lock方法，这里具体有公平锁和非公平锁两种实现
    > }
    > 
    > // 非公平锁静态内部类sync的lock实现
    > final void lock() {
    >     if (compareAndSetState(0, 1)) // 通过CAS设置状态，设置成功那么直接获取锁，反之调用acquire(1)进入同步队列中
    >         setExclusiveOwnerThread(Thread.currentThread()); // 该方法作用是为锁设置独占线程，其实也就是一个赋值操作
    >     else
    >         acquire(1);
    > }
    > 
    > // 公平锁静态内部类sync的lock实现
    > final void lock() {
    >     //调用了AQS的acquire函数,这是关键函数之一
    >     acquire(1);
    > }
    > ```
    >
    > 剩下代码略，看多线程.md
    >
    > 

24. 

25. 

27. 

28. 

29. 

30. 

31. 

32. 

33. 

34. 

35. 

36. java自带的4种线程池 

    > 1. newFixedThreadPool（固定线程数量线程池）
    >
    >    ```cpp
    >    public static ExecutorService newFixedThreadPool(int nThreads) {
    >            return new ThreadPoolExecutor(nThreads, nThreads,
    >                                          0L, TimeUnit.MILLISECONDS,
    >                                          new LinkedBlockingQueue<Runnable>());
    >        }
    >    ```
    >
    >    顾名思义，就是创建线程数量固定的线程池，线程池的`corePoolSize`和`maximumPoolSize`大小一样，并且`keepAliveTime`为0，传入的队列`LinkedBlockingQueue`为无界队列。在说`ThreadPoolExecutor`的时候也说过，传入一个无界队列，`maximumPoolSize`参数是不起作用的。
    >
    >    节省创建线程开销，但是当没有任务时也会消耗资源
    >
    > 2. newSingleThreadExecutor（单线程池）
    >
    >    ```cpp
    >    public static ExecutorService newSingleThreadExecutor() {
    >            return new FinalizableDelegatedExecutorService
    >                (new ThreadPoolExecutor(1, 1,
    >                                        0L, TimeUnit.MILLISECONDS,
    >                                        new LinkedBlockingQueue<Runnable>()));
    >        }
    >    ```
    >
    >    从代码中也能看得出来，`corePoolSize`和`maximumPoolSize`都是1，`keepAliveTime`是0L, 传入的队列是无界队列。线程池中永远只要一个线程在工作。
    >
    >    可保证顺序地执行各个任务
    >
    > 3. **newCachedThreadPool**
    >
    >    ```cpp
    >    public static ExecutorService newCachedThreadPool() {
    >            return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
    >                                          60L, TimeUnit.SECONDS,
    >                                          new SynchronousQueue<Runnable>());
    >        }
    >    ```
    >
    >    可缓存线程池，说道缓存一般离不开过期时间，该线程池也是，`corePoolSize`设置为0，`maximumPoolSize`设置为int最大值，不同的是，线程池传入的队列是`SynchronousQueue`，一个同步队列，该队列没有任何容量，每次插入新数据，必须等待消费完成。当有新任务到达时，线程池没有线程则创建线程处理，处理完成后该线程缓存60秒，过期后回收，线程过期前有新任务到达时，则使用缓存的线程来处理。
    >
    >    可以无限制增加线程数量，无用线程缓存60s后关闭。注意可能OOM
    >
    > 4. newScheduledThreadPool
    >
    >    
    >
    >    ```cpp
    >    public static ScheduledExecutorService newScheduledThreadPool(
    >                int corePoolSize, ThreadFactory threadFactory) {
    >            return new ScheduledThreadPoolExecutor(corePoolSize, threadFactory);
    >        }
    >    ```
    >
    >    这个线程池使用了`ScheduledThreadPoolExecutor`，该线程池继承自`ThreadPoolExecutor`, 执行任务的时候可以指定延迟多少时间执行，或者周期性执行。

37. 

38. 

39. 线程池优点，线程池里线程报的异常如何抓取，在线程池外捕获

    > Runnable接口的run方法所界定的边界就可以看作是线程代码的边界，所有的具体线程都实现这个方法，所以线程代码不能抛出任何checked异常。所有的线程中的checked异常都只能被线程本身消化掉。也符合线程本身的定义。
    >
    > 但是，线程代码中是可以抛出错误(Error)和运行级别异常(RuntimeException)的。Error俺们可以忽略，因为通常Error是应该留给vm的，而RuntimeException确是比较正常的，当线程代码抛出运行级别异常之后，线程会中断。主线程不受这个影响，不会处理这个RuntimeException，而且根本不能catch到这个异常。会继续执行自己的代码 :)
    > 所以得到结论：**线程方法的异常只能自己来处理**。但是，万事儿总有办法：
    >
    > 方法一：为线程设置未捕获异常处理器UncaughtExceptionHandler，
    >
    > 方法二：如果调用方只是想知道被调用方发生了哪些异常，可以返回List<Exception>（此时子线程已经终结）

40. 

80. callable和runable的区别

    > callable：核心是call()方法，能返回泛型对象，可以作为线程池的`submit()`方法入口参数
    >
    > runable：核心是run()方法，无返回值，Tread类只支持runable， 作为线程池方法`execute()`的入口参数来执行任务
    >
    > 都是接口，都需要执行Thread.start()

81. 





### java基础

1. 类加载机制

   > 　　java是使用**双亲委派模型**来进行类的加载的，所以在描述类加载过程前，我们先看一下它的工作过程：
   >
   > > 双亲委托模型的工作过程是：如果一个类加载器（ClassLoader）收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把这个请求委托给父类加载器去完成，每一个层次的类加载器都是如此，因此所有的加载请求最终都应该传送到顶层的启动类加载器中，只有当父类加载器反馈自己无法完成这个加载请求（它的搜索范围中没有找到所需要加载的类）时，子加载器才会尝试自己去加载。
   > > 使用双亲委托机制的好处是：能够有效确保一个类的全局唯一性，当程序中出现多个限定名相同的类时，类加载器在执行加载时，始终只会加载其中的某一个类。
   >
   > **1、加载**
   >
   > ​	由类加载器负责根据一个类的全限定名来读取此类的二进制字节流到JVM内部，并存储在运行时内存区的方法区，然后将其转换为一个与目标类型对应的java.lang.Class对象实例
   >
   > #### 2、验证
   >
   > ​	格式与语义验证
   >
   > #### 3、准备
   >
   > ​	为类中的所有**静态变量分配内存空间，并设置初始值**（由于还没有产生对象，实例变量不在此操作范围内）；**被final修饰的static变量（常量）直接赋值**；
   >
   > #### 4、解析
   >
   > ​	将**常量池中的符号引用转为直接引用**（得到类或者字段、方法在内存中的指针或者偏移量，以便直接调用该方法），这个可以在初始化之后再执行。解析需要静态绑定的内容。 // 所有不会被重写的方法和域都会被静态绑定
   >
   > 　　**以上2、3、4三个阶段又合称为链接阶段**，链接阶段要做的是将加载到JVM中的二进制字节流的类数据信息合并到JVM的运行时状态中。
   >
   > #### 5、初始化（先父后子）
   >
   > 4.1 为静态变量赋值
   >
   > 4.2 执行static代码块
   >
   > > 注意：static代码块只有jvm能够调用
   > > 　　　如果是多线程需要同时初始化一个类，仅仅只能允许其中一个线程对其执行初始化操作，其余线程必须等待，只有在活动线程执行完对类的初始化操作之后，才会通知正在等待的其他线程。
   >
   >  
   >
   > 因为子类存在对父类的依赖，所以**类的加载顺序是先加载父类后加载子类，初始化也一样。**不过，父类初始化时，子类静态变量的值也有有的，是默认值。
   >
   > 最终，方法区会存储当前类类信息，包括类的**静态变量**、**类初始化代码**（**定义静态变量时的赋值语句** 和 **静态初始化代码块**）、**实例变量定义**、**实例初始化代码**（**定义实例变量时的赋值语句实例代码块**和**构造方法**）和**实例方法**，还有**父类的类信息引用。**

2. ‘==’与equals区别

   > “==”是判断对象地址是否相等，equals是Object的方法，可以重写
   >
   > 重写equal():
   >
   > ```java
   > public boolean equals(Object otherObject){       //测试两个对象是否是同一个对象，是的话返回true
   >            if(this == otherObject) {  //测试检测的对象是否为空，是就返回false
   >                return true;   
   >            } 
   >            if(getClass() != otherObject.getClass() || otherObhect instanceof this) {  //对otherObject进行类型转换以便和类A的对象进行比较
   >                return false; 
   >            }
   >            A other=(A)otherObject; 
   >            return Object.equals(类A对象的属性A，other的属性A）&&类A对象的属性B==other的属性B……;
   >     }  
   > ```
   >
   > 重写hashcode():
   >
   > ```java
   >    public int hashCode() {
   >         int result = 17;
   >         result = 31 * result + mInt;
   >         result = 31 * result + (mBoolean ? 1 : 0);
   >         result = 31 * result + Float.floatToIntBits(mFloat);
   >         result = 31 * result + (int)(mLong ^ (mLong >>> 32));
   >         long mDoubleTemp = Double.doubleToLongBits(mDouble);
   >         result =31 * result + (int)(mDoubleTemp ^ (mDoubleTemp >>> 32));
   >         result = 31 * result + (mString == null ? 0 : mMsgContain.hashCode());
   >         result = 31 * result + (mObj == null ? 0 : mObj.hashCode());
   >         return result;
   >     }
   > ```
   >
   > https://www.cnblogs.com/myseries/p/10977868.html

3. 接口与抽象类的区别

   > 接口（Interface）：抽象方法的集合，抽象方法不能有默认实现，不能被实例化，可以被实现，且必须实现所有抽象方法，只能有public修饰符，接口中除了static、final变量，不能有其他变量，支持多继承
   >
   > 抽象类（abstract）：抽象类里的抽象方法可以有默认实现，可以有构造器（因为毕竟属于类），可以有public等所有修饰符，一个对象只能有一个父级抽象类

4. 重写和重载的区别

   > 

5. String与StringBuffer，StringBuilder区别

   > String是不可变对象，每次改变都会生成一个新的String对象；StringBuffer内部通过一个数组维护，线程安全，StringBuilder线程不安全，但是单线程下效率比StringBuffer高。

6. 

7. Java反射机制

   > 运行时获取指定类对象和方法的机制，编译器正常流程是.java经过.javac编译后变成.class文件，然后JVM装载运行.class文件。
   >
   > 1. 获取class对象：
   >
   >    ```java
   >    // 使用 Class.forName 静态方法
   >    Class clz = Class.forName("java.lang.String");
   >
   >    // 使用.class方法
   >    Class clz = String.class;
   >
   >    // 使用类对象的 getClass() 方法
   >    String str = new String("Hello");
   >    Class clz = str.getClass();
   >    ```
   >
   > 2. 通过反射类获取对象
   >
   >    ```vbnet
   >    // 通过 Class 对象的 newInstance() 方法。
   >    Class clz = Apple.class;
   >    Apple apple = (Apple)clz.newInstance();
   >
   >    // 通过 Constructor 对象的 newInstance() 方法
   >    Class clz = Apple.class;
   >    Constructor constructor = clz.getConstructor();
   >    Apple apple = (Apple)constructor.newInstance();
   >    ```
   >
   > 3. 通过反射获取方法、属性
   >
   >    ```vbnet
   >    // 反射获取属性
   >    Class clz = Apple.class;
   >    Field[] fields = clz.getFields();
   >    for (Field field : fields) {
   >        System.out.println(field.getName());
   >    }
   >                   
   >    //反射执行方法
   >    Method method = clz.getMethod("setPrice", int.class); 
   >    method.invoke(object, 4);   //就是这里的invoke方法
   >    ```
   >
   > ![img](https://img2018.cnblogs.com/blog/595137/201903/595137-20190324000247330-1279629878.png)
   >
   > JDK 源码使用了代理的设计模式去实现最大化性能，先是检查初始化，如果开启动态代理就会使用动态实现（直接生成字节码），否则就会生成一个委派实现，使用本地实现来完成。
   >
   > 动态实现和本地实现相比，执行速度要快上20倍，这是因为**动态实现直接执行字节码，不用从java到c++ 再到java 的转换**，但是因为生成字节码的操作比较耗费时间，所以如果仅一次调用的话反而是本地时间快3到4倍。**当一个反射调用次数达到15次时，委派实现的委派对象由本地实现转换为动态实现**，这个过程称之为Inflation。

8. final,finally,finalize区别？finalize作用

   > final是关键字，修饰类表示类不能被继承，修饰方法表示方法不能被重载，修饰变量表示变量必须在声明时赋值（JAVA虚拟机为变量设定的默认值不记作一次赋值）且不能更改。
   >
   > finally是try catch的最后执行关键字
   >
   > finalize是object类的一个方法，GC在清理ft对象时调用了它的finalize()方法，java中允许使用finalize()方法在垃圾收集器将对象从内存中清除出去之前做必要的清理工作，也就是在垃圾收集器之前被调用

9. 

10. 

11. 

12. 

13. 

14. Object类中的基本方法

    > - getclass()
    > - equals()
    > - hashcode()
    > - Wait()
    > - Wait(time)
    > - notify()
    > - notifyAll()
    > - Finalize()
    > - toString()
    > - Native clone()

15. 

16. 

17. 

18. 

19. 

20. 

21. 

22. 

23. 

24. 

25. 

26. 

27. 

28. 

29. 

30. new一个对象的过程

    > 1. **加载并初始化类**：先在常量池中能否定位到一个类的符号引用，检查类是否加载到内存，如果没有则需要通过双亲委派去进行加载
    >
    > 2. **创建对象**：在堆中分配内存（**因为jit的逃逸分析优化后，对象可能分配在栈中**），堆地址工整可能会有指针碰撞（移动已用堆空间与未使用堆空间的分隔指针），不工整的话会有一个空闲列表；
    >
    >    堆内存分配多线程情况下，采用cas分配失败重试方法或者为每个线程在Java堆中预先分配一小块内存解决线程安全问题
    >
    >    实例变量赋默认值；执行实例初始化方法；虚方法表记录了方法的动态绑定地址，子类重写方法后会指向子类的方法地址；
    >
    > 3. **执行init()方法与栈中新建引用对象**
    >
    > https://www.jianshu.com/p/16349fc5cf3d

31. 

32. 

33. 

34. 

35. 

36. 

37. 

38. 

39. 

40. Cglib代理 

    > - JDK实现动态代理需要实现类通过接口定义业务方法。
    > - CGLib采用了非常底层的字节码技术，其原理是通过目标类的字节码为一个类创建子类，并在子类中采用方法拦截的技术拦截所有父类方法的调用，顺势织入横切逻辑。
    > - 底层使用字节码处理框架ASM，来转换字节码并生成新的类。
    > - 更详细一点说，代理类将目标类作为自己的父类并为其中的每个非final委托方法创建两个方法：
    >   一个是与目标方法签名相同的方法，它在方法中会通过super调用目标方法；
    >   另一个是代理类独有的方法，称之为Callback回调方法，它会判断这个方法是否绑定了拦截器（实现了MethodInterceptor接口的对象），若存在则将调用intercept方法对目标方法进行代理，也就是在前后加上一些增强逻辑。intercept中就会调用上面介绍的签名相同的方法。

41. java注解

    > 注解是一个实现了Annotation的接口，里面每一个属性，其实就是接口的一个抽象方法。
    >
    > 调用getDeclaredAnnotations()方法的时候，返回一个代理$Proxy对象，这个是使用jdk动态代理创建，使用Proxy的newProxyInstance方法，传入接口 和InvocationHandler的一个实例(也就是 AnotationInvocationHandler ) ，最后返回一个实例。
    >
    > 在创建代理对象之前，解析注解时候 从该注解类的常量池中取出注解的信息，包括之前写到注解中的参数，然后将这些信息在创建 AnnotationInvocationHandler时候 ，传入进去 作为构造函数的参数。
    >
    > 注解是一个特殊的标记，通过修改RetentionPolicy的值：source、class、runtime决定运行时间，source编译后则废弃，class加载字节码到jvm时候废弃，runtime一直有效。作用范围可以定义为方法或者属性。

9. 

### Java容器

1. hashmap底层原理

   > 

2. 

3. 

4. 

5. HashMap与HashTable区别

   > hashTable线程安全（通过syncorized），都实现了map接口，hashmap允许null键（设计角度，单线程null键的含义自己加的自己知道，多线程可能多个线程对null键的定义不同）



### java虚拟机

1. jit编译优化：

   > javac将java源代码编译转化成字节码，然后解释器解释java字节码为机器语言；**Java编译器经过解释执行，其执行速度必然会比直接执行可执行的二进制字节码慢很多。为了解决这种效率问题，引入了 JIT（Just In Time ，即时编译） 技术。**JIT技术之后，Java程序还是通过解释器进行解释执行，当JVM发现某个方法或代码块运行特别频繁的时候，就会认为这是“热点代码”（Hot Spot Code)。然后**JIT会把部分“热点代码”翻译成本地机器相关的机器码，并进行优化，然后再把翻译后的机器码缓存起来**，以备下次使用。
   >
   > 主要的热点代码识别方式是热点探测（Hot Spot Detection），HotSpot虚拟机中采用的主要是基于计数器的热点探测（为每个方法和代码块建立计数器，执行次数超过阀值就认为是热点方法）
   >
   > **除了缓存，还有：逃逸分析、 锁消除、 锁膨胀、 方法内联、 空值检查消除、 类型检测消除、 公共子表达式消除等**
   >
   > 1. 逃逸分析：主要由**标量替换、栈上分配**（分析出一个新的对象的引用的使用范围作用域从而决定是否要将这个对象分配到堆上还是栈上，当一个对象在方法中被定义后，它可能被外部方法所引用，例如作为调用参数传递到其他地方中，称为方法逃逸。）
   > 2. 锁膨胀、锁消除、锁粗话：锁消除指的是如果检测不到某段代码被共享和竞争的可能性，就会将这段代码所属的同步锁消除掉。锁粗话指的是将多个连续的加锁、解锁操作连接在一起，扩展成一个范围更大的锁。
   >
   > 

18. 对象的生命周期

    > 1. 创建阶段(Created)
    > 2. 应用阶段(In Use)：至少被一个强引用指向
    > 3. 不可见阶段(In
    > 4. visible)：不再被程序内强引用指向，可以理解为超出了该对象的作用域
    > 5. 不可达阶段(Unreachable)：不再被任何GCRoot强引用指向
    > 6. 收集阶段(Collected)：垃圾回收期发现对象不可达，如果重写了finalize()方法会进行执行
    > 7. 终结阶段(Finalized)：执行完finalize()后仍不可达，进入此阶段，等待垃圾回收器进行回收
    > 8. 对象空间重分配阶段(De-allocated)

26. 垃圾对象是否立即回收

    > 不会，比如被设置为null的对象，只是断开了当前栈针的引用，垃圾回收器是运行在后台的线程，只有当程序运行到安全点（safePoint）或者安全区域（safeRegion）时才会扫描对象引用关系，可回收的会进行标记，但是此时也不会立即进行回收，重写了finalize()方法会放入F-queue队列等待java的低优先级线程不保证完全执行，在下个回收周期进行回收。

27. 

### java框架

1. SpringAOP的底层原理

   > **动态代理**，基于Jdk实现InvocationHandler 底层使用反射技术，基于CGLIB实现 字节码技术。**如果有接口则执行jdk动态代理，否则执行cglib动态代理**

2. SpringBean的生命周期

   > 1. 准备元数据：spring启动时候扫描xml等配置文件装配BeanDefinition，然后放到beanDefinitionMap中（key是beanName，value是BeanDefinition对象），遍历beanDefinitionMap执行beanFactoryPostProcesser后置处理器修改占位符。
   > 2. 实例化对象：通过反射选择合适的构造器创建对象，然后进行属性注入
   > 3. 初始化：首先判断该Bean是否实现了Aware相关的接口，如果存在则填充相关的资源；然后执行后置处理器的before、after方法（这期间可以自定义行为，比如在初始化后从kafka拉取数据等）和init方法。
   > 4. 销毁：执行destroy()等方法
   >
   > 
   >
   > - BeanPostProcessor
   > - InstantiationAwareBeanPostProcessor
   >
   > 这两兄弟可能是Spring扩展中**最重要**的两个接口！InstantiationAwareBeanPostProcessor作用于**实例化**阶段的前后，BeanPostProcessor作用于**初始化**阶段的前后。正好和第一、第三个生命周期阶段对应。
   >
   > 参考：https://www.jianshu.com/p/1dec08d290c1

3. Spring中IOC的底层原理

   > 控制反转，将原先需要new出来的对象,我先把它实例化,同时把它放到一个容器里,这样后面需要这个对象的时候,直接通过注入的方式拿到。在spring中维护了两个map，一个是BeanDefiinition配置map，一个是bean实例map，

4. SpringMVC工作流程

   > ![SpringMVC工作流程](https://segmentfault.com/img/remote/1460000024416086)
   >
   > 1. 用户向服务端发送一次请求，这个请求会先到前端控制器DispatcherServlet(也叫中央控制器)。
   > 2. DispatcherServlet接收到请求后会调用HandlerMapping处理器映射器。由此得知，该请求该由哪个Controller来处理（并未调用Controller，只是得知）
   > 3. DispatcherServlet调用HandlerAdapter处理器适配器，告诉处理器适配器应该要去执行哪个Controller
   > 4. HandlerAdapter处理器适配器去执行Controller并得到ModelAndView(数据和视图)，并层层返回给DispatcherServlet
   > 5. DispatcherServlet将ModelAndView交给ViewReslover视图解析器解析，然后返回真正的视图。
   > 6. DispatcherServlet将模型数据填充到视图中
   > 7. DispatcherServlet将结果响应给用户

5. SpringBoot的常用注解

   > ![preview](https://segmentfault.com/img/remote/1460000024416085/view)

6. Spring如何解决循环依赖

   > **三级缓存**，bean实例化时候会被放入第三级缓存
   >
   > A依赖B的话，A在实例化之后放入第三级缓存，属性注入时候发现依赖B去实例化B，B实例化之后发现依赖A，便在第三级缓存中找到A的实例进行注入。
   >
   > 三级缓存的Value是ObjectFactory，从第三级中拿到的是代理对象（因为A可能存在AOP），代理工厂一旦生成一次后就放入二级缓存（没必要生成多次），等属性注入完毕后放入一级缓存，正常代码中getBean就是从一级缓存中获取的。
   >
   > ![preview](https://pic1.zhimg.com/v2-04ff2751d227f5580fcd3341c8747489_r.jpg?source=1940ef5c)

8. Spring、SpringMVC 、Springboot的区别

   > Spring MVC是Spring的一个模块，是一个web框架。通过Dispatcher Servlet, ModelAndView 和 View Resolver，开发web应用变得很容易。
   >
   > Spring Boot实现了自动配置，降低了项目搭建的复杂度。
   >
   > **Spring MVC和Spring Boot都属于Spring，Spring MVC 是基于Spring的一个 MVC 框架，而Spring Boot 是基于Spring的一套快速开发整合包**
   >
   > 

9. 

10. SpringBoot 自动装配机制

    > ```java
    > // springBoot核心注解
    > @SpringBootApplication
    > public class Application {
    >  public static void main(String[] args) throws Exception {
    >      SpringApplication.run(Application.class, args);
    >  }
    > }
    > 
    > // @SpringBootApplication 源码
    > @Target(ElementType.TYPE)
    > @Retention(RetentionPolicy.RUNTIME)
    > @Documented
    > @Inherited
    > @SpringBootConfiguration // 重要
    > @EnableAutoConfiguration // 重要
    > @ComponentScan(excludeFilters = { // 重要
    >      @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
    >      @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
    > public @interface SpringBootApplication {
    > }
    > 
    > ```
    >
    > 1. @SpringBootConfiguration
    >
    >    继承自spring的@Configuration，相当于一个spring的xml文件，配合@Bean注解，可以在里面配置需要Spring容器管理的bean。**通过这个注解来加载IOC容器配置**，在启动类 里面标注了@Configuration，意味着它其实也是一个 IoC 容器的配置类
    >
    >    ```java
    >    @Configuration
    >    public class MockConfiguration{
    >        @Bean
    >        public MockService mockService(){
    >            return new MockServiceImpl();
    >        }
    >    }
    >    ```
    >
    > 2. @ComponentScan
    >
    >    **告诉Spring需要扫描哪些包或类**：通常与@Configuration一起配合使用，相当于xml里面的`<context:component-scan>`。如果不设值的话默认扫描@ComponentScan注解所在类的同级类和同级目录下的所有类，所以对于一个Spring Boot项目，一般会把入口类放在顶层目录中，这样就能够保证源码目录下的所有类都能够被扫描到。
    >
    > 
    >
    > 3. @EnableAutoConfiguration
    >
    >    ```java
    >    @SuppressWarnings("deprecation")
    >    @Target(ElementType.TYPE)
    >    @Retention(RetentionPolicy.RUNTIME)
    >    @Documented
    >    @Inherited
    >    @AutoConfigurationPackage
    >    @Import(EnableAutoConfigurationImportSelector.class) // 核心
    >    public @interface EnableAutoConfiguration {
    >    }
    >    ```
    >
    >    核心是`@Import(EnableAutoConfigurationImportSelector.class)`l里面导入的`EnableAutoConfigurationImportSelector.class`。从classpath中搜寻所有的 META-INF/spring.factories 配置文件，并将其中org.springframework.boot.autoconfigure.EnableutoConfiguration 对应的配置项通过反射实例化为对应的标注了@Configuration的JavaConfig形式的IoC容器配置类，然后汇总为一个并加载到IoC容器。通过 spring.factories 的配置，并结合 @Condition 条件，完成bean的注册；
    >    
    >    1. 核心EnableAutoConfigurationImportSelector：基于ImportSelector实现动态加载bean，

11. 

12. 

13. SpringBoot启动过程

    > 

14. 

15. 

16. 

17. 

18. 

19. 

20. 

21. 

22. 

23. 

24. 

25. 

26. 

27. 

28. 

29. 

30. 

31. 

32. 

33. 

34. 

35. 

36. 

37. 

38. 

39. 

40. 

41. 

42. springboot的autowaire注解原理

    > 调用getBean方法，找到直接赋值，找到多个相同的，根据变量名Id进行匹配，可以使用qualify指定id



### **设计模式**

1. 

2. 单例模式的几种实现方式

   > 1. 懒汉式
   >
   >    ```java
   >    public final class Singleton {
   >    
   >        private static Singleton instance;
   >    
   >        private Singleton() {
   >            //
   >        }
   >    
   >        public static Singleton getInstance() {
   >            synchronized (Singleton.class) {
   >                if(instance == null) {
   >                    instance = new Singleton();
   >                }
   >            }
   >            return instance;
   >        }
   >    }
   >    ```
   >
   > 2. 饿汉式
   >
   >    ```java
   >    public final class Singleton {
   >    
   >        //类加载时就初始化
   >        private static final Singleton instance = new Singleton();
   >    
   >        private Singleton() {
   >            //
   >        }
   >    
   >        public static Singleton getInstance() {
   >            return instance;
   >        }
   >    }
   >    ```
   >
   > 3. 双重校验锁
   >
   >    ```java
   >    public final class Singleton {
   >    
   >        //用volatile修饰对象，禁止指令重排序优化
   >        private volatile static Singleton instance; 
   >    
   >        private Singleton() {
   >            //
   >        }
   >    
   >        public static Singleton getInstance() {
   >            if(instance == null) {
   >                synchronized (Singleton.class) {
   >                    if(instance == null) {
   >                        instance = new Singleton();
   >                    }
   >                }
   >            }
   >            return instance;
   >        }
   >    }
   >    ```
   >
   > 4. 静态内部类
   >
   >    ```java
   >    public final class Singleton {
   >    
   >        private Singleton() {
   >        }
   >      
   >        private static class SingletonHolder {
   >            private static final Singleton INSTANCE = new Singleton();
   >        }
   >    
   >        public static Singleton getInstance() {
   >            return SingletonHolder.INSTANCE;
   >        }
   >    }
   >    ```
   >
   > 5. 

3. Spring使用了哪些设计模式？

   > - 1.简单工厂(非23种设计模式中的一种)
   >   - BeanFactory：传入一个唯一的标识来获得Bean对象，实现松耦合，额外处理（增强功能）
   > - 2.工厂方法
   >   - FactoryBean：实现了FactoryBean接口的bean是一类叫做factory的bean。其特点是，spring会在使用getBean()调用获得该bean时，会自动调用该bean的getObject()方法，所以返回的不是factory这个bean，而是这个bean.getOjbect()方法的返回值。
   > - 3.单例模式
   >   - 依赖注入：Spring的依赖注入（包括lazy-init方式）都是发生在AbstractBeanFactory的getBean里。getBean的doGetBean方法调用getSingleton进行bean的创建。**spring依赖注入时，使用了 双重判断加锁 的单例模式**
   > - 4.适配器模式
   >   - SpringMVC中的适配器HandlerAdatper：根据Handler规则执行不同的Handler。
   > - 5.装饰器模式
   > - 6.代理模式
   >   - AOP：底层动态代理模式实现
   > - 7.观察者模式
   >   - spring的事件驱动模型使用的是 观察者模式
   > - 8.策略模式
   >   - Spring框架的资源访问Resource接口：针对不同的底层资源，Spring 将会提供不同的 Resource 实现类，不同的实现类负责不同的资源访问逻辑。
   > - 9.模版方法模式
   >   - JDBC:父类定义了骨架（调用哪些方法及顺序），某些特定方法由子类实现，好处是代码复用，减少重复代码。除了子类要实现的特定方法，其他方法及方法调用顺序都在父类中预先写好了。JdbcTemplate是抽象类，不能够独立使用，我们每次进行数据访问的时候都要给出一个相应的子类实现,这样肯定不方便，所以就引入了回调。
   >
   > https://zhuanlan.zhihu.com/p/114244039

4. 




##  **Network** 

###  传输层 

1. Tcp与Udp区别 

   > TCP与UDP都属于传输层
   >
   > 1. TCP面向链接，通信前要建立三次握手机制的连接，UDP（无连接）
   > 2. TCP具有可靠性，UDP具有不可靠性
   > 3. TCP面向字节流，UDP面向报文；UDP没有拥塞控制，因此网络出现拥塞不会使源主机的发送速率降低（对实时应用很有用，如IP电话，实时视频会议等）
   > 4. TCP只支持点对点，UDP支持多对多
   > 5. TCP首部开销20字节;UDP的首部开销小，只有8个字节
   > 6. TCP逻辑通道是可靠全双工，UDP不可靠

2. Tcp三次握手四次挥手及对应的状态

   > 
   >
   > ![img](https://uploadfiles.nowcoder.com/images/20211214/3639882_1639448445600/FDB15782B1300540E6E5005E99607E98) 
   >
   > - 开始，客户端和服务端都处于 CLOSED 状态。先是服务端主动监听某个端口，处于 LISTEN 状态 
   > - 客户端会随机初始化序号（ client_isn ），将此序号置于 TCP 首部的「序号」字段中，同时把 SYN 标志位置为 1 ，表示 SYN 报文。接着把第一个 SYN 报文发送给服务端，表示向服务端发起连接，该报文不包含应用层数据，之后客户端处于 SYN-SENT 状态。 
   > - 服务端收到客户端的 SYN 报文后，首先服务端也随机初始化自己的序号（ server_isn ），将此序号填入TCP 首部的「序号」字段中，其次把 TCP 首部的「确认应答号」字段填入 client_isn + 1 , 接着把 SYN和 ACK 标志位置为 1 。最后把该报文发给客户端，该报文也不包含应用层数据，之后服务端处于 SYN-RCVD 状态。 
   > - 客户端收到服务端报文后，还要向服务端回应最后一个应答报文，首先该应答报文 TCP 首部 ACK 标志位置为 1 ，其次「确认应答号」字段填入 server_isn + 1 ，最后把报文发送给服务端，这次报文可以携带客户到服务器的数据，之后客户端处于 ESTABLISHED 状态。 
   > - 服务器收到客户端的应答报文后，也进入 ESTABLISHED 状态。
   >
   > 
   >
   > ![img](https://uploadfiles.nowcoder.com/images/20211214/3639882_1639448543867/607EC0069710A2D1656EAAD1D19A2E1E) 
   >
   > - 客户端打算关闭连接，此时会发送一个 TCP 首部 FIN 标志位被置为 1 的报文，也即 FIN 报文，之后客户端进 FIN_WAIT_1 状态。 
   > - 服务端收到该报文后，就向客户端发送 ACK 应答报文，接着服务端进入 CLOSED_WAIT 状态。 
   > - 客户端收到服务端的 ACK 应答报文后，之后进入 FIN_WAIT_2 状态。 
   > - 等待服务端处理完数据后，也向客户端发送 FIN 报文，之后服务端进入 LAST_ACK 状态。 
   > - 客户端收到服务端的 FIN 报文后，回一个 ACK 应答报文，之后进入TIME_WAIT 状态 
   > - 服务器收到了 ACK 应答报文后，就进入了 CLOSED 状态，至此服务端已经完成连接的关闭。 
   > - 客户端在经过 2MSL 一段时间后，自动进入 CLOSED 状态，至此客户端也完成连接的关闭。

3. Tcp流量控制与拥塞控制

   > 1. 流量控制
   >
   >    根据接收缓冲区的接收能力（剩余可接收数据的空间）调节发送端发送数据的滑动窗口大小，使得发送方的发送速率与接收方的接收速率相匹配（**TCP 在收到数据包回复的 ACK 报文里会带上自己接收窗口的大小**，接收端需要根据这个值调整自己的发送策略）
   >
   >    如果`发送方的滑动窗口变为0了`，使用**零窗口探测机制用来向接收端探测接收窗口的大小**（发送端会发送一个`零窗口探测的包`，其实就是一个ACK报文，以此来知悉接收方是否具备接收能力）
   >
   > 2. 拥塞控制
   >
   >    超时或连续接收到3个冗余的ACK报文来判断拥塞，发送速率由拥塞窗口来控制，拥塞控制涉及`慢启动`、`拥塞避免`、`快重传`、`快恢复`四种[算法](https://www.nowcoder.com/jump/super-jump/word?word=算法)，它的本质是控制拥塞窗口的变化
   >
   >    拥塞窗口：在收到对端 ACK 之前自己还能传输的[最大数](https://www.nowcoder.com/jump/super-jump/word?word=最大数)据量大小，这里要与`滑动窗口`区分开，滑动窗口的大小是根据接收到的对端ACK报文中接收窗口的大小来指定的。真正的发送窗口大小取决于滑动窗口与拥塞窗口中的最小值。
   >
   >    发送方维护一个叫做拥塞窗口的状态变量Cwnd，其值取决于网络拥塞程度，动态变化，慢启动[算法]()阈值ssthresh，发送窗口大小Swnd
   >
   >    - `慢启动算法`：发送窗口大小 = 拥塞窗口大小（一个较小的值MSS），随着双方通信，收到确认应答报文，拥塞窗口指数级增长，超过慢开始阈值后，使用拥塞避免[算法]()
   >    - `拥塞避免算法`：拥塞窗口随着传输轮次增大，当cwnd > ssthresh，拥塞窗口的大小呈线性增长
   >    - `快重传算法`：在传输过程中有报文丢失，发送方累计连续3次收到重复确认报文，就将相应的报文段立即重传，而不是在该报文段的超时重传计时器超时重传
   >    - `快恢复算法`：当收到三次重复确认报文后，进入快恢复阶段。拥塞窗口大小 cwnd 设置为 ssthresh，将拥塞阈值 ssthresh 降低为 cwnd 的一半，再进行线性增长
   >
   >    

4. Tcp如何保证可靠传输

   > 1. TCP自动分隔数据块
   > 2. TCP会对分隔后的包进行编号，接收方进行排序后传输给应用层
   > 3. TCP接收端会丢弃重复数据
   > 4. **校验和：** 计算首部检验和；
   > 5. **流量控制：** 使用滑动窗口控制发送方发送速率方式丢失
   > 6. **拥塞控制：** 当网络拥塞时，减少数据的发送。
   > 7. **ARQ协议：** 也是为了实现可靠传输的，它的基本原理就是每发完一个分组就停止发送，等待对方确认。在收到确认后再发下一个分组。
   >    - 停止等待ARQ：发送一组等收到这组的ack后才能发下一组，结合超时重传的超时定时器实现
   >    - 连续ARQ：连续发多个数组，通常结合滑动窗口实现，采用累积确认
   > 8. **超时重传：** 当 TCP 发出一个段后，它启动一个定时器，等待目的端确认收到这个报文段。如果不能及时收到一个确认，将重发这个报文段。
   >
   > 

5. TCP为什么要三次握手

   > 1. **双向连接，至少要三次握手**。
   >
   >    - 这个问题的本质是, 信道不可靠, 但是通信双发需要就某个问题达成一致. 而要解决这个问题, 无论你在消息中包含什么信息, 三次通信是理论上的最小值. **所以三次握手不是 TCP 本身的要求, 而是为了满足"在不可靠信道上可靠地传输信息"这一需求所导致的**. 请注意这里的本质需求,信道不可靠, 数据传输要可靠. 
   >
   >    - ```
   >      前提 1: TCP 协议要保证双方可以通信，
   >      即：发送端接收端要确认自己发送的信息对方能接收到，对方发送的信息自己能接收到
   >      前提 2: 在前提一的情况下发送的越少越好
   >                                    
   >      假设 1: 一次握手 即发送端向接受端发送一个包
   >      结论: 发送端无法确认自己发送的信息对方是否收到
   >                                    
   >      假设 2: 两次握手 发送端 发一个包 接收端回一个包
   >      结论：发送端可以确定自己发送的信息能对方能收到 也能确定对方发的包自己能收到 
   >      但接收端只能确定 对方发的包自己能收到 无法确定 自己发的包对方能收到
   >                                    
   >      假设3: 三次握手 发送端 发一个 接收端回一个 发送端接收到之后再发一个
   >      结论 这样发送端和接收端就能确定双方可以通信了
   >                                    
   >      所以三次是满足要求的最小值
   >                                    
   >      三次握手这个说法不好，其实是双方各一次握手，各一次确认，其中一次握手和确认合并在一起
   >                                    
   >      之所以存在 3-way hanshake 的说法，是因为 TCP 是双向通讯协议，
   >      作为响应一方(Responder) 要想初始化发送通道，必须也进行一轮 SYN + ACK。
   >      由于 SYN ACK 在 TCP 分组头部是两个标识位，因此处于优化目的被合并了。
   >      所以达到双方都能进行收发的状态只需要 3 个分组。
   >                                    
   >      所以实际上理解成两次（单向通讯）和四次（不考虑合并）也未尝不可。
   >                                    
   >      原则上任何数据传输都无法确保绝对可靠，三次握手只是确保可靠的基本需要。
   >      双方都需要确认自己的发信和收信功能正常，收信功能通过接收对方信息得到确认，发信功能需要发出信息—>对方回复信息得到确认。
   >      需要第三次握手的原因在于 Server 端在第二次握手（发出信息）后并不知道对方是否能够接收、己方的发送功能是否正常。
   >      但此时数据的单向通道已经建立，对于 Client 来说，已经确认了 Server 端可以接收信号，因此可以单向给 Server 发送数据了。
   >      ```
   >
   > 2. **为了防止已失效的连接请求报文段突然又传送到了服务端，因而产生错误。**
   >
   >    - Client发送的第一个请求建立链接的报文到晚了，服务端一直处于listen状态，接受到这样一个请求就会误认为是Client想建立一个新链接，**假设不采用“三次握手”，那么只要 Server 发出确认，新的连接就建立了。**白白浪费服务器资源
   >
   > 

6. 四次挥手timewait的作用

   > 1. **保证 TCP 协议的全双工连接能够可靠关闭**保证最后的 ACK 能让被动关闭⽅接收
   >
   >    ```
   >    如果 Client 直接 CLOSED 了，那么由于 IP 协议的不可靠性或者是其它网络原因，导致 Server 没有收到 Client 最后回复的 ACK。
   >    那么 Server 就会在超时之后继续发送 FIN，此时由于 Client 已经 CLOSED 了，就找不到与重发的 FIN 对应的连接，
   >    最后 Server 就会收到 RST 而不是 ACK，Server 就会以为是连接错误把问题报告给高层。
   >    这样的情况虽然不会造成数据丢失，但是却导致 TCP 协议不符合可靠连接的要求。
   >    所以，Client 不是直接进入 CLOSED，而是要保持 TIME_WAIT，当再次收到 FIN 的时候，能够保证对方收到 ACK，最后正确的关闭连接。
   >    ```
   >
   > 2. **保证这次连接的重复数据段从网络中消失**
   >
   >    ```
   >    防止client建立的新连接与刚关闭的连接的端口号可能相同，前一次连接的某些延迟数据在建立新连接之后才到达 Server，被TCP 判定那个延迟的数据是属于新连接的。TCP 协议判断不同连接的依据是 socket pair。 TIME_WAIT 状态等待 2 倍 MSL，这样可以保证本次连接的所有数据都从网络中消失。
   >    ```

7. TCP为什么粘包？如何处理

   > **tcp作为面向流的协议，不存在“粘包问题”**，这个问题意思应该指的是
   >
   > 1. tcp面向流，数据不会保持send边界，导致接收侧有可能一下子收到多个应用层报文，需要应用开发者自己分开
   > 2. 多个小尺寸数据因为nagle算法被封装在一个报文内，需要应用层自己拆
   >
   > 粘包原因：
   >
   > -  要发送的数据小于TCP发送缓冲区的大小，TCP将多次写入缓冲区的数据一次发送出去，将会发生粘包。
   > -  接收数据端的应用层没有及时读取接收缓冲区中的数据，将发生粘包。
   > -  要发送的数据大于TCP发送缓冲区剩余空间大小，将会发生拆包。
   > -  待发送数据大于MSS（最大报文长度），TCP在传输前将进行拆包。
   >
   > 处理方式：
   >
   > - 发送端：关闭Nagle算法
   > - 接收端：接收端无法处理，只能丢给应用层处理
   > - 接收方应用层：
   >   - 用分隔符表示一个包
   >   - 用带长度的报文格式
   >   - 用固定包大小，不足的补（包头定长，以特定标志开头，里带着负载长度，这样接收侧只要以定长尝试读取包头，再按照包头里的负载长度读取负载就行了，多出来的数据都留在缓冲区里即可）

8. 

9. TCP为什么要四次挥手

   > **保证TCP建立的全双工通道能够可靠关闭**
   >
   > 和 TCP 建立连接时的原因相同，因为 TCP 是全双工通信，只不过三次握手时的 SYN + ACK 是放在一个报文中了，而四次挥手时，己方 ACK 和 FIN 报文是分开发送的。
   >
   > ```
   > 服务端在 LISTEN 状态下，收到建立连接请求的 SYN 报文后，把 ACK 和 SYN 放在一个报文里发送给客户端。
   > 而关闭连接时，当收到对方的 FIN 报文时，仅仅表示对方不再发送数据了但是还能接收数据，
   > 己方也未必全部数据都发送给对方了，所以己方可以立即 close，也可以发送一些数据给对方后，
   > 再发送 FIN 报文给对方来表示同意现在关闭连接，因此，己方 ACK 和 FIN 一般都会分开发送。
   > ```

10. 如何设计可靠的UDP

    > 

11. 

12. 

13. 

14. 

15. TCP半链接队列和全连接队列

    > 服务端收到客户端发起的 SYN 请求后，**内核会把该连接存储到半连接队列**，并向客户端响应 SYN+ACK，接着客户端会返回 ACK，服务端收到第三次握手的 ACK 后，内核会把连接从半连接队列移除，然后创建新的完全的连接，并将其添加到 accept队列，等待进程调用accept 函数时把连接取出来。半链接队列和全连接队列超长后都会直接丢弃，其中两个队列大小由back_log参数指定

16. 

17. 

18. 

19. 

20. 

21. 

22. TCP超时重传时间与次数

    > 必须大于RTT，但是不能太长。TCP采样RTT，进行加权平均计算，动态调整超时重传计时器的时间。
    >
    > 每当遇到一次超时重传的时候，都会将下一次超时时间间隔设为先前值的两倍。两次超时，就说明网络环境差，不宜频繁反复发送。

23. 

24. 

25. 

26. 

27. 

28. 

29. 

30. 

31. 

32. 

33. 

34. 

35. 

36. 

37. 

38. 

39. 

40. 

41. 

42. 四次挥手为什么要等待2MSL

    > 1 个 MSL 确保四次挥手中主动关闭方最后的 ACK 报文最终能达到对端
    > 1 个 MSL 确保对端没有收到 ACK 重传的 FIN 报文可以到达；
    > 2MSL = 去向 ACK 消息最大存活时间（MSL) + 来向 FIN 消息的最大存活时间（MSL）

43. 

### Http相关 

1. Http、Https、两者区别

   > - 端口：http 80，https 443
   > - http明文，https使用ssl加密
   > - https需要申请ca证书
   > - http无状态，HTTPS协议是由SSL+HTTP协议构建的可进行加密传输、身份认证的网络协议，比http协议安全。

2. Https的加密流程 

   > 1. 客户端和服务器端通过TCP建立连接
   > 2. 客户端给出支持SSL协议版本号，一个客户端随机数(Client random，请注意这是第一个随机数)，客户端支持的加密方法等；
   > 3. 服务端返回带公钥的证书，一个服务器生成的随机数(Server random，注意这是第二个随机数)等
   > 4. 客户端认数字证书的有效性，然后生成一个新的随机数(Premaster secret)，然后使用数字证书中的公钥，加密这个随机数，
   > 5. 服务端使用自己的私钥，获取爱丽丝发来的随机数(即Premaster secret)；(非对称加密的过程了)
   > 5. 双方通过约定的加密方法(通常是[AES算法](https://link.zhihu.com/?target=https%3A//zh.wikipedia.org/wiki/%E9%AB%98%E7%BA%A7%E5%8A%A0%E5%AF%86%E6%A0%87%E5%87%86))，使用前面三个随机数，生成**对话密钥**，用来加密接下来的通信内容；
   > 6. 建立通信
   >
   > ![preview](https://segmentfault.com/img/bVcTsOL/view)

3. Get与Post的区别

   > 都是基于**超文本传输协议**（**HTTP**）的应用层协议。
   >
   > 区别：
   >
   > - 请求参数：get放url，Post放requestBody（GET请求url长度限制主要根据不同的服务器来定，具体报文里面并未限制GET的url长度为1024kb；Get也可以带requestBody，但是不同的服务器不保证能接收到；Post的url也可以放参数）
   > - 请求缓存：Get会被缓存，Post不会
   > - 安全性：Post更安全，GET提交的数据都将显示到URL上，页面会被浏览器缓存，其他人查看历史记录会看到提交的数据，而POST不会。另外GET提交数据还可能会造成CSRF攻击
   > - 历史记录：GET请求参数会被完整保留在浏览历史记录里，而POST中的参数不会被保留
   > - 编码方式：GET只支持url编码，POST支持多种
   > - 对参数的数据类型：GET只接受ASCII字符，而POST没有限制。
   > - **GET产生一个TCP数据包，POST产生两个TCP数据包**：对于GET方式的请求，浏览器会把http header和data一并发送出去，服务器响应200 OK(返回数据); 而对于POST，浏览器先发送header，服务器响应100 continue，浏览器再发送data，服务器响应200 OK(返回数据)。注意，`尽管POST请求会分两次，但body 是紧随在 header 后面发送的，根本不存在『等待服务器响应』一说`。
   >
   > 
   >
   > 

4. Http响应状态码

   > - 1xx：代表请求已被接受，需要继续处理
   >   - **100（继续）**：客户端应该继续请求，服务器已经接收到请求头，并且客户端应继续发送请求主体，例如POST
   >   - 101 （切换协议）
   >   - 102 Processing（WebDAV；RFC 2518）
   > - 2xx：成功
   >   - 200（成功）
   >   - 201（已创建）：常见于POST与PUT
   >   - 202（已接受）
   >   - 203（非授权信息）（自HTTP / 1.1起）
   >   - 204（无内容）
   >   - 205（重置内容）
   > - 3xx：重定向
   >   - 300（多种选择）
   >   - 301 （永久移除）：请求的网页已永久移动到新位置，会自动将请求者转到新位置，新URI在`Location`域中返回。
   >   - 302（临时移动）: 新的临时性的URI应当在响应的`Location`域中返回。
   >   - 303（查看其他位置）：请求的url在另一个url中，且需要使用GET方式请求
   >   - 304（缓存）
   >   - 307（临时重定向）
   >   - 308（永久重定向）
   > - 4xx：客户端错误
   >   - 400（错误请求）：客户端请求错误，如参数、格式、数据量太大、Cookies已损坏等
   >   - 401（未授权）
   >   - 402 （要求付费）
   >   - 403（禁止）
   >   - 405（方法禁用）：GET、POST等
   >   - 407（需要代理授权）
   >   - 408（请求超时）
   >   - 413（请求实体过大）
   >   - 414（请求的 URI 过长）
   > - 5xx：服务器错误
   >   - 500（内部服务器错误）
   >   - 502（错误网关）
   >   - 503（服务不可用）
   >   - 504（网关超时）
   >   - 505 （HTTP 版本不受支持）

5. Http请求、响应的报文格式

   > HTTP协议客户端请求request消息包括以下格式：`请求行（request line）、请求头部（header）、空行、请求数据`；
   > ![在这里插入图片描述](https://segmentfault.com/img/remote/1460000023940347)
   >
   > 服务端响应response也由四个部分组成，分别是：`响应行、响应头、空行、响应体`。
   > ![在这里插入图片描述](https://segmentfault.com/img/remote/1460000023940348)

6. Http2.0、Http1.1、Http1.0有哪些特性

   > HTTP1.0:
   >
   > - **无状态、无连接**：浏览器的每次请求都需要与服务器建立一个`TCP`连接，服务器处理完成后立即断开`TCP`连接（无连接），服务器不跟踪每个客户端也不记录过去的请求（无状态）。
   > - 缺点：无状态性可以借助`cookie/session`机制来做身份认证和状态记录，无连接的特性导致最大的性能缺陷就是**无法复用连接**。每次发送请求的时候，都需要进行一次`TCP`的连接，而`TCP`的连接释放过程又是比较费事的。这种无连接的特性会使得网络的利用率非常低。其次就是**队头阻塞**（`head of line blocking`）。由于`HTTP1.0`规定下一个请求必须在前一个请求响应到达之前才能发送。假设前一个请求响应一直不到达，那么下一个请求就不发送，同样的后面的请求也给阻塞了。
   >
   > HTTP1.1：
   >
   > - **长连接**：`HTTP1.1`增加了一个`Connection`字段，通过设置**`Keep-Alive`可以保持`HTTP`连接不断开**，避免了每次客户端与服务器请求都要重复建立释放建立`TCP`连接，提高了网络的利用率。如果客户端想关闭`HTTP`连接，可以在请求头中携带`Connection: false`来告知服务器关闭请求。
   > - 请求**管道化**（`pipelining`）：基于`HTTP1.1`的长连接，使得请求管线化成为可能。管线化使得请求能够“并行”传输。举个例子来说，假如响应的主体是一个`html`页面，页面中包含了很多`img`，这个时候`keep-alive`就起了很大的作用，能够进行“并行”发送多个请求。（注意这里的“并行”并不是真正意义上的并行传输）但是**服务器必须按照客户端请求的先后顺序依次回送相应的结果，以保证客户端能够区分出每次请求的响应内容**。所以并没有解决HTTP1.0中的队头阻塞问题，HTTP管道化仅仅把先进先出队列从客户端（请求队列）迁移到服务端（响应队列）。**目前浏览器中的并行基本上都是通过建立多个TCP链接实现**
   > - 加入了**缓存处理（强缓存和协商缓存）**新的字段如`cache-control`
   > - 支持**断点传输**，增加了**Host字段**（使得一个服务器能够用来创建多个Web站点）
   >
   > HTTP2.0：
   >
   > - **二进制分帧**：在应用层和传输层之间增加一个二进制分帧层，把原来`HTTP1.x`的`header`和`body`部分用`frame`重新封装了一层而已
   > - **多路复用（连接共享）**：**所有的HTTP2.0通信都在一个TCP链接中进行**，链接中可以有多个双向流，流以消息为单位传输，一个消息包含多个帧，这些帧可以乱序，接收方根据每个帧头部的`stream id`来进行分组识别，并且数据流支持设置优先级解决多路复用带来的关键请求被阻塞问题。**`HTTP2.0`基于“二进制分帧”实现了真正的并行传输**，它能够在一个`TCP`上进行任意数量`HTTP`请求。
   > - **头部压缩**：`HTTP2.0`使用`encoder`来减少需要传输的`header`大小，通讯双方各自`cache`一份`header fields`表，既避免了重复`header`的传输，又减小了需要传输的大小。
   > - **服务器推送**
   >
   > HTTP3.0：
   >
   > - 基于google的QUIC协议，而quic协议是使用udp实现的，QUIC 严格要求加密后才能建立连接，适用于所有数据
   > - 减少了tcp三次握手时间，以及tls握手时间；
   > - 解决了http 2.0中前一个stream丢包导致后一个stream被阻塞的问题；
   > - 优化了重传策略，重传包和原包的编号不同，降低后续重传计算的消耗；
   > - 连接迁移，不再用tcp四元组确定一个连接，而是用一个64位随机数来确定这个连接；
   > - 更合适的流量控制。

7. http协议的几种方法（get、post、delete等）

   > 1. GET
   > 2. POST
   > 3. PUT
   > 4. Delete
   > 5. HEAD
   > 6. Options：预检，用于描述目标资源的通信选项。通过该请求来知道服务端是否允许跨域请求。
   > 6. CONNECT：建立一个到由目标资源标识的服务器的隧道，多用于 HTTPS 和 WebSocket 。
   > 6. TRACE：沿着到目标资源的路径执行一个消息环回测试，多数线上服务都不支持
   > 6. PATCH：对资源应用部分修改。

8. https如何保证可靠性

   > SSL+CA证书
   >
   > 由于非对称加密也可能被中间人攻击，例如：当服务端发送给客户端的公钥A被拦截，中间人生成了假的公钥B发给客户端，客户端以为公钥B是服务端的便用公钥B加密后发给中间人，中间人用公钥B解密后再用公钥A加密再发送给服务端。
   >
   > CA证书可以理解为找了个公信第三方，服务器在给客户端下发公钥的过程中，会把公钥以及服务器的信息通过hash生成一个摘要信息，**用CA提供的私钥对摘要信息进行加密**，从而形成数字签名。最后连同最初未进行hash的信息生成一份数字证书。当客户端收到数字证书时，会利用CA提供的公钥进行解密，得到摘要信息，通过对比解密的摘要信息以及未hash的信息，如果相同就证明是服务器下发。就是**利用一个第三方权威机构，去对公钥和私钥进行下发**，从而保障了数据传递的安全性。虽然可以利用浏览器对服务器证书检查的缺陷，通过伪造CA证书的方式，进行SSL中间人攻击。但实际上在这个过程中，现在主流浏览器都会弹出警告，SSL攻击就相当于失败了，因此从这个意义上讲，SSL协议依然是无法攻破的。

9. 

10. 

11. 

12. 

13. 

14. 

15. 

16. 

17. 

18. 

19. Http与Tcp的关系与区别

    > 1. HTTP 对应于应用层，定义的是传输数据的内容的规范；TCP 协议对应于传输层，定义的是数据传输和连接方式的规范
    > 2. HTTP 协议是在 TCP 协议之上建立的，HTTP 在发起请求时通过 TCP 协议建立起连接服务器的通道，请求结束后，立即断开 TCP 连接
    > 3. HTTP 是无状态的短连接，而 TCP 是有状态的长连接
    
19. 

19. 

19. 

19. 

19. 

19. 

19. 

19. 

19. 

19. 

19. 

19. 简述CA证书

    > 浏览器判断证书是否规范校验：
    >
    > 1. 验证域名、有效期是否正确。
    > 2. 判断证书来源是否合法。（每份签发证书可以根据验证链查找到对应的根证书，操作系统、浏览器会在本地存储权威机构的根证书，利用本地根证书可以对对应机构签发证书完成源验证）
    > 3. 与CA服务器进行校验，判断证书是否被篡改。
    > 4. 通过CRL（Certificate Revocation List 证书注销列表）和OCSP（Online Certificate Status Protocol 在线证书状态协议）判断证书是否已吊销。
    
32. 



### 应用层

1. Cookie与Session原理与区别 

   > cookie是在客户端保持状态，而session是在服务器端保持状态。
   >
   > **cookie：**
   >
   > - 主要包括：name=value即键值对，expires 和 Max-Age，Domain和Path，secure和HttpOnly。
   >
   > - **交互流程**
   >
   >   - Step1.客户端发起http请求到Server
   >
   >   - Step2. 服务器返回http response,其中可以包含Cookie设置
   >
   >     - HTTP/1.1 200 OK
   >
   >       Content-type: text/html
   >
   >       Set-Cookie: name=value
   >
   >       Set-Cookie: name2=value2; Expires=Wed, 09 Jun 2021 10:18:14 GMT
   >
   >   - Step3. 后续访问
   >
   >     - GET /spec.html HTTP/1.1
   >
   >       Cookie: name=value; name2=value2
   >
   > - 作用：
   >
   >   - 会话管理，几乎用户登录状态
   >   - 个性化信息，比如记录上一次登录的账号默认填写
   >   - 记录用户行为等
   >
   > - 种类：
   >
   >   - Session Cookie：只在会话期间内有效，即当关闭浏览器的时候，它会被浏览器删除。设置session cookie的办法是：在创建cookie不设置Expires即可。
   >   - Persistent Cookie：设置cookie的属性Max-Age为1个月等
   >   - HttpOnly Cookie：仅在http状态下，对脚本无效，从而避免了跨站攻击时JS偷取cookie的情况。当你使用javascript在设置同样名字的cookie时，只有原来的httponly值会传送到服务器。
   >   - 3rd-party cookie：第三方cookie，常用语阿里妈妈之类的广告服务商。广告商就可以采集用户的一些习惯和访问历史。
   >
   > **Session**：
   >
   > - 代表服务器与浏览器的一次会话过程，存放在服务器端的内存中，**浏览器得到的只是SessionID**；请求中带了SesionId服务器会匹配到对应Seesion使用，如果没带则会创建一个SessionId。session因为请求（request对象）而产生，同一个会话中多个request共享session对象，可以直接从请求中获取到session对象。
   > - 一个常见的误解是以为session在有客户端访问时就被创建，然而事实是直到某server端程序调用 HttpServletRequest.getSession(true)这样的语句时才被创建。
   > - SessionId携带方式：
   >   - cookie：cookie只是最优雅的实现session的方式，因为cookie对用户来说不可见，同时会自动在HTTP报文中传输
   >   - url重写：当cookie被禁用时，可能会附带在url路径或者查询参数中
   >   - 隐藏在表单中

2. DNS工作原理

   > Client -->hosts文件 --> 本地DNS解析器缓存 -->本地DNS服务器 -->本地DNS服务器缓存 --> DNS Server (recursion
   > 递归) --> DNS Server Cache -->DNS iteration(迭代) --> 根--> 顶级域名DNS-->二级域名
   > https://developer.aliyun.com/article/446543

3. 

4. 浏览器禁用了Cookie以后还能用Session吗

   > 参考第一问

### 网络层

3. 子网掩码的作用

   > 屏蔽部分ip地址区分网络与主机
   >
   > 将大的子网络区分为小的子网

4. 

### 网络体系结构

1. 浏览器上输入地址后的整个请求过程

   > 1. URL解析：通过DNS解析将URL解析为IP地址。浏览器缓存 - > 本地DNS缓存 -> DNS服务器
   > 2. 建立TCP连接：封装数据，从应用层到链路层，http数据+tcp首部+ip首部+以太网首部
   >
   > 应用层：请求头（方法，地址，协议），请求体
   >
   > 传输层：发起TCP连接，以报文段分割数据。建立连接是三次握手过程
   >
   > 网络层：数据段打包，加入源ip与目的ip。判断是否同一网段，是则根据Mac地址发送，否则查找路由表下一跳地址，以及使用arp协议查询mac地址。

2. 

## mysql 

### Mysql索引

1. 索引的数据结构对比（hash、B树与B+树），为什么不用[红黑树]()

   > 1. 二叉查找树(BST)：解决了排序的基本问题，但是由于无法保证平衡，可能退化为链表；
   > 2. 平衡二叉树(AVL)：通过旋转解决了平衡的问题，但是旋转操作效率太低；
   > 3. 红黑树：通过舍弃严格的平衡和引入红黑节点，解决了AVL旋转效率过低的问题，但是在磁盘等场景下，**树仍然太高，IO次数太多**；且红黑树一个节点只能存一个值
   > 4. B树：通过将二叉树改为**多路平衡查找树**（也就是有多个子节点），解决了树过高的问题；
   > 5. B+树：在B树的基础上，将非叶节点改造为不存储数据的纯索引节点，进一步降低了树的高度；此外将叶节点使用指针连接成链表，范围查询更加高效。

2. Mysql索引数据结构 

   > **InnoDB 的数据是按「数据页」为单位来读写，数据页的大小为16kb**，数据页之间通过双向链表的形式组织起来，物理上不连续，但是逻辑上连续。
   >
   > 数据页内的行记录通过单链表按照主键顺序链接，页目录里面将记录根据4-8个分槽维护每个槽中最后一个记录地址，**使用二分法快速定位要查询的记录在哪个槽（哪个记录分组），定位到槽后，再遍历槽内的所有记录，找到对应的记录**
   >
   > ![img](https://img-blog.csdnimg.cn/img_convert/f78da573e6dc4d00f8023cac699345ba.png)
   >
   > ![图片](https://img-blog.csdnimg.cn/img_convert/7cb783d4da641c7b5bf719112f8d2e94.png)
   >
   > ![图片](https://img-blog.csdnimg.cn/img_convert/944d760ea3437eb0624e0952f89b22a7.png)
   >
   > https://bbs.huaweicloud.com/blogs/317532
   >
   > 
   
3. Mysql的聚簇索引和非聚簇索引作用与区别

   > 聚簇索引：叶节点包含了完整的数据记录， 一个表仅有一个聚簇索引
   >
   > 非聚簇索引（辅助索引）：叶节点存放的是主键值，需要二次回表根据主键查询具体数据记录。如果非聚簇索引叶节点中包含了要查询的数据（查主键id，或者a,b联合索引查询select b where a=1这种）便可以不用回表，也称作**索引覆盖**。

4. 索引失效的几种场景 

   > 1. 没有遵循**最左匹配原则**
   > 2. 索引列做计算
   > 3. 使用`!=`等
   > 4. 使用`Like`
   > 5. 字符串不加单引号（可能发送隐式转换）
   > 6. 使用or

5. Mysql索引优化与设计规则

   > 设计原则：
   >
   > 1. 根据where后的字段建立
   > 2. 尽量在唯一字段建立（区分度的公式是count(distinct col)/count(*)）
   > 3. 短索引原则：长字符串字段列设置索引，最好指定前缀长度，节省大量索引空间。
   > 4. 索引列别太多：每个索引需要额外的磁盘空间，并降低写操作的性能。并且在修改表内容的时候，索引会进行更新，更有甚至需要重构，索引列越多，所花费的时间就会越长。
   > 5. 聚合函数
   > 6. 排序
   > 7. 避免回表
   >
   > 优化原则：
   >
   > 1. 最左匹配原则：最左匹配原则并不是指查询条件的顺序，而是指查询条件中是否包含索引最左列字段；
   > 2. 

6. 

7. 简述索引作用与优缺点

   > 优点：
   >
   > 1. 创建唯一索引，保证每一行数据的唯一性
   > 2. 加快检索速度
   > 3. 加快排序分组速度
   > 4. 可以在查询过程中使用优化隐藏器
   >
   > 缺点：
   >
   > 1. 索引的创建和更新维护需要时间
   > 2. 索引需要占据额外空间

8. 

9. 

10. 简述什么是联合索引

    > 联合索引也是一颗B+树，只不过有多个索引列而已，使用时候需要遵循最左匹配原则

11. 唯一索引与主键索引的区别

    > 1. 主键索引只能有一个，唯一索引可以有多个
    >
    > 2. 主键索引not null

12. 什么时候需要加索引

    > 1. 当数据具有唯一性时，指定唯一索引，提高查询速度
    > 2. 频繁进行排序或者分组时（group by或者order by）

12. 

14. 简述什么是索引下推

    > Mysql.5.6之后引入，**组合索引满足最左匹配，但是遇到非等值判断时匹配停止**。
    >
    > 目的：**减少了回表的操作次数。减少了上传到 MySQL SERVER 层的数据**。
    >
    > 1. MySQL 服务层：解析 SQL 的语法、语义、生成查询计划、接管从 MySQL 存储引擎层上推的数据进行二次过滤等等。
    >
    > 2. MySQL 存储引擎层：按照 MySQL 服务层下发的请求，通过索引或者全表扫描等方式把数据上传到 MySQL 服务层。
    > 3. MySQL 索引扫描：根据指定索引过滤条件（比如 where id = 1) ，遍历索引找到索引键对应的主键值后回表过滤剩余过滤条件。
    > 4. MySQL 索引过滤：通过索引扫描并且基于索引进行二次条件过滤后再回表。
    >
    > 在不使用ICP的情况下，在使用非主键索引（又叫普通索引或者二级索引）进行查询时，存储引擎通过索引检索到数据，然后返回给MySQL服务器，服务器然后判断数据是否符合条件 。
    >
    > 在使用ICP的情况下，如果存在某些被索引的列的判断条件时，MySQL服务器将这一部分判断条件传递给存储引擎，然后由存储引擎通过判断索引是否符合MySQL服务器传递的条件，规避掉不满足的索引记录，**只有当索引符合条件时才会将数据检索出来返回给MySQL服务器 。**
    >
    > 
    >
    > 例：
    >
    > ```sql
    > SELECT * from user where  name like '陈%' and age=20
    > ```
    >
    >  name like '陈%' 不是等值匹配，所以 age = 20 这里就用不上 (name,age) 组合索引了。如果没有索引下推，组合索引只能用到 name，age 的判定就需要回表才能做了。5.6之后有了索引下推，age = 20 可以直接在组合索引里判定。
    >
    > 

    

13. 

### Mysql事务 

1. Mysql的默认隔离级别、不同等级隔离级别解决的问题与实现原理

   > - READ UNCOMMITEED：读不提交，在另一个事务没有提交的情况下，也能读取，可能会出现脏读情况
   > - READ COMMITTED：读已提交，不可重复读，会出现幻读现象。
   > - REPEATABLE READ：可重复读，可避免脏读以及幻读(使用Next-Key Lock算法)现象，在本事务未提交的情况下不会读到不同的数据。（**默认隔离级别**）
   > - SERIALIZATION：序列化，没有并发，虽能避免脏读、幻读等情况，但性能较低，select语句会加一个共享锁，所以不支持一致性非锁定读
   >
   > 默认隔离级别在5.6之前是RC，5.7之后是RR。因为**在主从复制下，5.0版本之前的binlog仅支持statement模式（记录逻辑sql语句），5.0之后才引入row模式，而在RC模式下binlog会有bug导致主从数据不一致**。
   >
   > 但是在互联网项目中一般设置为RC，因为
   >
   > 1. **在RR隔离级别下，更容易出现死锁**
   > 2. **在RR隔离级别下，where条件列未命中索引会锁表(Gap锁)，而RC只会锁行(在RC隔离级别下，其先走聚簇索引，进行全部扫描,调用unlock_row方法，把不满足条件的记录放锁)**
   > 3. **在RC隔离级别下，半一致性读(semi-consistent)特性增加了update操作的并发性**
   >
   > https://www.cnblogs.com/longe-blog/p/15637016.html

2. Mysql事务及特性

   > 1. 原子性
   >
   >    > 原子性指整个数据库事务是不可分割的工作单位。事务中的所有数据库操作要么全部成功，要么全部撤销。
   >
   > 2. 一致性
   >
   >    > 一致性指事务将数据库从一种一致的状态转变为下一种一致的状态。在事务开始之前和结束之后，数据库的完整性约束没有破坏。
   >
   > 3. 隔离性
   >
   >    > 事务的隔离性要求，事务提交前对其他事务不可见。通常这使用锁来实现。
   >
   > 4. 持久性
   >
   >    > 事务一旦提交，其结果就是永久性的。即使发生宕机等事故，数据库也能将数据恢复。

3. Mvcc实现机制(RC和RR隔离级别下的区别)

   > 通过 **`read view`** 和**版本链**实现；版本链保存有历史版本记录，通过`read view` 判断当前版本的数据是否可见，如果不可见，再从版本链中找到上一个版本，继续进行判断，直到找到一个可见的版本。
   >
   > 
   >
   > 版本链通过表的三个隐藏字段实现：
   >
   > - `DB_TRX_ID`：当前事务id，通过事务id的大小判断事务的时间顺序。 
   > - `DB_ROLL_PRT`：回滚指针，指向当前行记录的上一个版本，通过这个指针将数据的多个版本连接在一起构成`undo log`版本链。 
   > - `DB_ROLL_ID`：主键，如果数据表没有主键，InnoDB会自动生成主键。
   >
   > 使用事务更新行记录的时候，就会生成版本链，执行过程如下：
   >
   > 1. 用排他锁锁住该行； 
   > 2. 将该行原本的值拷贝到`undo log`，作为旧版本用于回滚； 
   > 3. 修改当前行的值，生成一个新版本，更新事务id，使回滚指针指向旧版本的记录，这样就形成一条版本链。
   >
   > ![img](https://uploadfiles.nowcoder.com/files/20211126/753969457_1637935460674/mvcc11.png)
   >
   > 在`read view`内部维护一个活跃事务[链表]()，表示生成`read view`的时候还在活跃的事务。这个[链表]()包含在创建`read view`之前还未提交的事务，不包含创建`read view`之后提交的事务。
   >
   > 不同隔离级别创建read view的时机不同。
   >
   > - read committed：每次执行select都会创建新的read_view，保证能读取到其他事务已经提交的修改。
   > - repeatable read：在一个事务范围内，第一次select时更新这个read_view，以后不会再更新，后续所有的select都是复用之前的read_view。这样可以保证事务范围内每次读取的内容都一样，即可重复读。
   >
   > ![img](https://uploadfiles.nowcoder.com/files/20211126/753969457_1637935460681/read_view10.png)
   >
   > `DATA_TRX_ID` 表示每个数据行的最新的事务ID；`up_limit_id`表示当前快照中的最先开始的事务；`low_limit_id`表示当前快照中的最慢开始的事务，即最后一个事务。
   >
   > 如果`DATA_TRX_ID` < `up_limit_id`：说明在创建`read view`时，修改该数据行的事务已提交，该版本的记录可被当前事务读取到。 
   >
   > 如果`DATA_TRX_ID` >= `low_limit_id`：说明当前版本的记录的事务是在创建`read view`之后生成的，该版本的数据行不可以被当前事务访问。此时需要通过版本链找到上一个版本，然后重新判断该版本的记录对当前事务的可见性。 
   >
   > 如果`up_limit_id` <= `DATA_TRX_ID` < `low_limit_i`：  
   >
   > 1. 需要在活跃事务[链表]()中查找是否存在ID为`DATA_TRX_ID`的值的事务。 
   > 2. 如果存在，因为在活跃事务[链表]()中的事务是未提交的，所以该记录是不可见的。此时需要通过版本链找到上一个版本，然后重新判断该版本的可见性。 
   > 3. 如果不存在，说明事务trx_id 已经提交了，这行记录是可见的。
   >
   
4. Mysql的bin log、redo log、undo log日志文件及其作用

   > redo log：
   >
   > 当执行更新操作时候，InnoDB引擎会把记录先写到redo_log里面，并更新内存，然后在适当时候，更新磁盘上的记录。redo_log使用**循环写**的方式，从头写到尾，**写到末尾从头开始循环写**。writePos记录当前写到的位置，checkPoint记录已经更新到数据文件的位置。
   >
   > redo_log 由两部分组成，redo log buffer和redo log file。日志回先写到redo log buffer中，然后更新到操作系统的OS buffer中转，最后更新到redo log file磁盘文件中。
   >
   > 
   >
   > bin_log:
   >
   > 属于MySQL Server层的日志。可以实现**主从复制**和**数据恢复**两个作用。
   >
   > bin_log的记录格式有三种：
   >
   > - Statement: 记录sql语句
   > - Row: 记录行数据修改的细节
   > - Mixed: 前两种的混合
   >
   > 
   >
   > undo_log:
   >
   > 逻辑日志，提供回滚和多个行版本控制(MVCC)，存在于表空间
   >
   > - 提供回滚
   >   - 在数据修改的时候，不仅记录了redo，还记录了相对应的undo，存储sql恢复语句，可以借助该undo进行回滚。
   > - 多个行版本控制(MVCC)
   >   - 应用到行版本控制的时候，也是通过undo log来实现的：当读取的某一行被其他事务锁定时，它可以从undo log中分析出该行记录以前的数据是什么，从而提供该行版本信息，让用户实现非锁定一致性读取。

5. 什么是两阶段提交?

   > 当记录更新到内存之后，会进行以下操作：
   >
   > 1. 将更新写入redo_log，进入prepare阶段
   > 2. 写bin_log
   > 3. 提交事务，处于commit阶段

6. 如果两阶段提交异常mysql重启会发生什么

   > 第一种情况，如果redo_log有commit记录，直接提交事务。
   > 第二种情况，redo_log只有prepare记录，判断bin_log是否完整（bin_log的commit标记），如果完整提交事务，否则回滚。

7. 

8. MVCC 是否能彻底解决幻读

   > **NO**
   >
   > mysql的读分为**快照读**和**当前读**，快照读通过mvcc实现，当前读（增删改查或者显式使用共享锁或排它锁）通过next-lock实现
   >
   > 
   >
   > 对于可重复读默认使用的就是next key lock，但是对于“唯一索引”，比如主键的索引，next key lock会降级成行锁，而不会锁住一个区间。因此，如果上面的事务1的update使用的是主键，事务2也使用主键进行插入，那么实际上事务2根本不会被阻塞，可以立即插入并返回。而对于非唯一索引，next key lock则不会降级。
   >
   > 
   >
   > mvcc出现幻读的case：
   >
   > 1. a事务先select，b事务insert确实会加一个gap锁，但是如果b事务commit，这个gap锁就会释放（释放后a事务可以随意操作），
   >
   > 2. a事务再select出来的结果在MVCC下还和第一次select一样，
   >
   > 3. 接着a事务不加条件地update，这个update会作用在所有行上（包括b事务新加的），
   >
   > 4. a事务再次select就会出现b事务中的新行，并且这个新行已经被update修改了.
   >
   > 上面这样，事务2提交之后，事务1再次执行update，因为这个是当前读，他会读取最新的数据，包括别的事务已经提交的，所以就会导致此时前后读取的数据不一致，出现幻读。

   

9. 

10. 不可重复读与幻读的区别

    > 不可重复读指的是一个事务内读到其他事务的提交导致记录本身变化，幻读是记录数量变化

11. 

12. 

13. 

14. ACID怎么保证原子性

15. 

16. 

17. mysql事务回滚及提交的原理

    > 讨论MySQL的事务回滚机制，也就是在说MySQL的事务原子性是如何实现的。**redo log用于保证事务持久性；undo log则是事务原子性和隔离性实现的基础。**
    >
    > **当发生回滚时，InnoDB会根据undo log的内容做与之前相反的工作**：对于每个insert，回滚时会执行delete；对于每个delete，回滚时会执行insert；对于每个update，回滚时会执行一个相反的update，把数据改回去。

18. 能不能只要bin log或者redo log?

    > 如果只要bin_log，没法崩溃恢复。如果只要redo_log，理论上是可以的，但是bin_log有redo_log没有的优点，比如redo_log是循环写，历史记录无法保留。
    >
    > 

19. 数据最终落盘是从redo_log更新来的，还是从内存更新来的？

    > 数据页被修改后，称为脏页，最终数据落盘，是把内存中的数据页写入磁盘。当崩溃发生后，如果某个数据页可能丢失了更新，会将其读入内存，通过redo_log对内存中的数据页进行修改。

20. 

### 查询性能优化

1. Mysql sql优化，慢Sql如何排查

   > 1. 是否走索引：explain查看
   > 2. 是否select数据过大导致网络传输耗时慢
   > 3. 是否在高峰期
   > 4. 是否同时有定时任务操作数据库导致表锁等
   > 5. 不同的机房是否都会
   > 6. A->B->C->数据库，是否中间链路耗时
   
1. Inner join与left join区别

   > https://segmentfault.com/a/1190000017369618
   
1. Mysql查询优化器机制

   > 在一条单表查询语句真正执行之前，MySQL的查询优化器会找出执行该语句所有可能使用的方案，对比之后找出成本最低的方案，这个成本最低的方案就是所谓的执行计划，之后才会调用存储引擎提供的接口真正的执行查询。
   >
   > QL优化通常包括两项工作:**一是逻辑优化、二是物理优化**。这两项工作都要对语法分析树的形态进行修改，把语法分析树变为查询树。其中，**逻辑查询优化将生成逻辑查询执行计划**。在生成逻辑查询执行计划过程中，根据关系代数的原理，把语法分析树变为关系代数语法树的样式，原先SQL语义中的一些谓词变化为逻辑代数的操作符等样式，这些样式是一个临时的中间状态，经过进一步的逻辑查询优化，如执行常量传递、选择下推等(如一些节点下移，一些节点上移)，从而生成逻辑查询执行计划。在生成逻辑查询计划后，查询优化器会进一步对查询树进行物理查询优化。**物理优化会对逻辑查询进行改造，改造的内容主要是对连接的顺序进行调整**。SQL语句确定的连接顺序经过多表连接算法的处理，可能导致表之间的连接顺序发生变化，所以树的形态有可能调整。物理查询优化除了进行表的连接顺序调整外，还会使用代价估算模型对单个表的扫描方式、两表连接的连接算法进行评估，选择每一项操作中代价最小的操作为下一步优化的基础。物理查询优化的最终结果是生成最终物理查询执行计划。
   >
   > 
   >
   > 逻辑优化分为：
   >
   > 1. 条件优化
   > 2. 计算全表扫描成本
   > 3. 找出所有能用到的索引
   > 4. 针对每个索引计算不同的访问方式的成本
   > 5. 选出成本最小的索引以及访问方式
   >
   > 
   >
   > 物理优化阶段，主要解决的问题如下:
   >
   > - 从可选的单表扫描方式中，挑选什么样的单表扫描方式是最优的？
   > - 对于两个表连接时，如何选择是最优的？
   > - 对多个表连接，连接顺序有多种组合，是否要对每种组合都探索？如果不全部探索，怎么找到最优的一种组合？
   >
   > https://zhuanlan.zhihu.com/p/56790651
   
1. 大表怎么优化

   > 1. 限定数据的范围。比如：用户在查询历史信息的时候，可以控制在一个月的时间范围内；
   > 2. 读写分离： 经典的数据库拆分方案，主库负责写，从库负责读；
   > 3. 通过分库分表的方式进行优化，主要有垂直拆分和水平拆分。
   
1. sql语句执行流程

   > https://cloud.tencent.com/developer/article/1103154
   
1. 

### mysql基础

1. weher和having区别

   > - 二者作用的对象不同，`where`子句作用于表和视图，`having`作用于组。
   > - `where`在数据分组前进行过滤，`having`在数据分组后进行过滤。

### 分库分表

1. 分库分表

   > 当单表的**数据量达到1000W或100G**以后，优化索引、添加从库等可能对数据库性能提升效果不明显，此时就要考虑对其进行切分了。切分的目的就在于减少数据库的负担，缩短查询的时间。
   >
   > 1. 垂直
   >
   >    - 垂直分表：”大表拆小表“，基于列字段进行的；将不常用的， 数据较大，长度较长（比如text类型字段）的拆分到“扩展表“。
   >    - 垂直分库：一个系统中的不同业务进行拆分，比如用户User一个库，商品Producet一个库，订单Order一个库， 切分后，要放在多个服务器上，而不是一个服务器上和服务的“治理”，“降级”机制类似，也能对不同业务的数据分别的进行管理，维护，监控，扩展等。
   >
   >    **行记录变小，数据页可以存放更多记录，在查询时减少I/O次数**。但是主键出现冗余，需要管理冗余列，会引起表连接JOIN操作，可以通过在业务服务器上进行join来减少数据库压力，对于一些核心业务（如订单）依然存在单表数据量过大的问题。
   >
   > 2. 水平拆分
   >
   >    - 水平拆表：针对数据量巨大的单张表（比如订单表），按照某种规则（RANGE,HASH取模等），切分到多张表里面去。 但是这些表还是在同一个库中，所以库级别的数据库操作还是有IO瓶颈。不建议采用。
   >    - 水平拆库：将单张表的数据切分到多个服务器上去，每个服务器具有相应的库与表，只是表中数据集合不同。 水平分库分表能够有效的缓解单机和单库的性能瓶颈和压力，突破IO、连接数、硬件资源等的瓶颈。
   >
   >    水平拆分规则：
   >
   >    - 引入id与库节点的映射表，缺点是引入了新的单点
   >    - range：从0到10000一个表，10001到20000一个表；好处天然支持水平扩展，单表大小可控，
   >    - hash取模：比如根据订单后四位进行取模，
   >    - 地理区域：华东、华北等
   >    - 时间
   >    
   >    在扩容时需要进行数据迁移
   >
   > 当表数据总量比较大（超过200G）时进行垂直拆分，当表记录数比较大（超过1000w）时进行水平拆分；当数据库链接数比较大时进行分库。
   >
   > 分库分表问题：
   >
   > 1. 分布式事务支持，单库本地事务能满足，垮库的只有使用分布式事务支持
   > 2. 多库结果合并（group by、order by）
   > 3. 多库join，因为可能要join的数据不在一个库中
   > 3. 分布式id，比如雪花，uuid，数据库维护一个id表进行自增，redis，美团leaf等。
   >
   > https://tech.meituan.com/2016/11/18/dianping-order-db-sharding.html
   >
   > https://www.jianshu.com/p/fddf278e30cb

## OS

1. 用户态和内核态的区别

   > 操作系统将内存分为**内核空间**与**用户空间**，用户空间只能访问局部内存，内核空间可以访问所有内存；用户程序执行在用户态，如果用户程序需要执行系统指令调用，需要中断并切换到内核态；

2. 用户态线程与内核态线程

   > 程序启动需要创建进程，然后进程都有一个主线程负责执行。**进程可以分成用户态进程和内核态进程两类**，进程可以通过api创建用户态线程，也能通过系统调用创建内核态线程
   >
   > 用户态线程：
   >
   > 1. **同时只能有一个用户态线程并发**
   > 2. **调度完全由进程负责**：对操作系统透明，操作系统调度以所属进程为单位，阻塞了操作系统也无法感知
   > 3. I/O需要进行系统调度
   >
   > 内核态线程：
   >
   > 1. 可以并发
   > 2. I/O不需要系统调度切换
   > 3. 切换线程时也需要内核态的切换
   >
   > 总结：用户态线程工作在用户空间，内核态线程工作在内核空间。用户态线程调度完全由进程负责，通常就是由进程的主线程负责。相当于进程主线程的延展，使用的是操作系统分配给进程主线程的时间片段。内核线程由内核维护，由操作系统调度。
   >
   > 用户态线程无法跨核心，一个进程的多个用户态线程不能并发，阻塞一个用户态线程会导致进程的主线程阻塞，直接交出执行权限。这些都是用户态线程的劣势。内核线程可以独立执行，操作系统会分配时间片段。因此内核态线程更完整，也称作轻量级进程。内核态线程创建成本高，切换成本高，创建太多还会给调度算法增加压力，因此不会太多。

3. 

## Redis

### Redis问题与解决方案

1. 缓存穿透、缓存雪崩原因及解决方案

   > 穿透：查询redis中没有的key，导致大量请求到Db；
   >   - 缓存为空的数据；布隆过滤器；
   >
   > 血崩：大量key同一时间过期，导致大量请求到Db；
   >
   > - 分散key失效时间；设置二级缓存；高可用方案
   >
   > 击穿：单个key过期，对该热点key的大量请求到Db；
   >   - 分布式锁控制访问，如redis的setnx互斥锁先进行判断，保证不会有大量访问到Db；

### Redis使用场景 

1. Redis实现分布式锁

   > 加锁：
   >
   > 1. 使用setnx：`SETNX key value expire *`，缺点加锁和设置过期时间不是原子操作
   > 2. 使用setex：`SETEX key seconds value`，加锁和设置过期时间是原子操作，如果key存在将覆盖旧值
   > 3. 使用psetex：同setex，只是时间单位变为毫秒
   >
   > 解锁：直接删除key，但是加锁解锁并非原子操作，可以使用lua脚本拼装
   >
   > 
   >
   > 问题：
   >
   > 1. **redis时钟漂移**（服务器时间变早）导致提前失效
   > 2. **重复加锁**：主从模式下master挂掉slave晋升导致的重复加锁
   >    - 使用redlock算法：
   >      - 按顺序向5个master节点请求加锁，如果key的TTL为5s那么获取锁的超时时间为50ms
   >      -  根据设置的超时时间来判断，是不是要跳过该master节点。
   >      -  如果大于等于三个节点加锁成功，并且使用的时间小于锁的有效期，即可认定加锁成功啦。
   >      -  如果获取锁失败，解锁！
   >    - RedLock并没有完全解决Redis单点故障存在的隐患，也没有解决时钟漂移以及客户端长时间阻塞而导致的锁超时失效存在的问题，锁的安全性隐患依然存在。
   >      - 假设一共有5个Redis节点：A、B、C、D、E，客户端1和2分别加锁
   >        1. 客户端1成功锁住了A，B，C，获取锁成功（但D和E没有锁住）。
   >        2. 节点C的master挂了，然后锁还没同步到slave，slave升级为master后丢失了客户端1加的锁。
   >        3. 客户端2这个时候获取锁，锁住了C，D，E，获取锁成功。
   > 3. 业务执行过长导致锁自动释放：（加入看门狗机制解决）
   > 4. 业务代码忘解锁，看门狗机制生效中，锁无法释放（虚引用监听，保底）Redis 实现分布式锁实际上是通过setnx 命令, 如果有该key值, 则设置失败, 没有该key, 设置成功.

2. 

3. 简述Redis使用场景

   > 

4. Redis集群如何设计

5. Redis实现消息队列 

   > 1. **使用list**： LPUSH，RPOP这样的方式，风险点在业务代码里面需要使用`while(true)`不停轮训，**可使用 BLPOP、BRPOP 这种阻塞式读取的命令优化**；但是如果list队列删除但是由于网络原因消费者没收到，则无法还原，**缺少消息确认ack机制**，可通过**RPOPLPUSH、BRPOPLPUSH (阻塞)**从一个 list 中获取消息的同时把这条消息复制到另一个 list 里(可以当做备份)，而且这个过程是原子的。这样我们就可以在业务流程安全结束后，再删除队列元素，实现消息确认机制。
   > 2. **使用zet**：实现延时消息队列，原理就是将消息加到 zset 结构后，将要被消费的时间戳设置为对应的 score 即可，只要业务数据不会是重复数据就 OK。
   > 3. **订阅发布**：Redis 通过 PUBLISH 、 SUBSCRIBE 等命令实现了订阅与发布模式， 这个功能提供两种信息机制， 分别是订阅/发布到频道和订阅/发布到模式，缺点是**消息无法持久化**，如果出现网络断开、Redis 宕机等，消息就会被丢弃。而且也没有 Ack 机制来保证数据的可靠性，假设一个消费者都没有，那消息就直接被丢弃了。**redis 5.0后加入了stream，提供了消息的持久化和主备复制功能**，可以让任何客户端访问任何时刻的数据，并且能记住每一个客户端的访问位置，还能保证消息不丢失。它就像是个仅追加内容的消息链表，把所有加入的消息都串起来，每个消息都有一个唯一的 ID 和对应的内容。而且消息是持久化的。

6. Redis实现限流

   > 1. 基于setnx:：比如我们需要在10秒内限定20个请求，那么我们在setnx的时候可以设置过期时间10，当请求的setnx数量达到20时候即达到了限流效果。弊端是比如当统计1-10秒的时候，无法统计2-11秒之内，如果需要统计N秒内的M个请求，那么我们的Redis中需要保持N个key等等问题
   > 2. 基于zset：可以将请求打造成一个zset数组，当每一次请求进来的时候，value保持唯一，可以用UUID生成，而score可以用当前时间戳表示，因为score我们可以用来计算当前时间戳之内有多少的请求数量。而zset数据结构也提供了range方法让我们可以很轻易的获取到2个时间戳内有多少请求。可以做到滑动窗口的效果，并且能保证每N秒内至多M个请求，缺点就是zset的数据结构会越来越大。
   > 3. 基于redis令牌桶：结合Redis的List数据结构leftPop来获取令牌，令牌桶算法提及到输入速率和输出速率，当输出速率大于输入速率，那么就是超出流量限制了。每访问一次请求的时候，可以从Redis中获取一个令牌，如果拿到令牌了，那就说明没超出限制。再依靠Java的定时任务，定时往List中rightPush令牌，当然令牌也需要唯一性，所以我这里还是用UUID进行了生成
   >
   > 这些限流方式我们可以在AOP或者filter中加入以上代码，用来做到接口的限流

## MQ 

### Kafka

1. **Kafka 与传统消息系统之间有三个关键区别**

   > 1. 持久化日志
   > 2. 分布式
   > 3. 实时流处理

2. kafka中zookeeper干嘛的

   > 在kafka1.1之前zookeeper中存储 主controller、AR、ISR

3. 说说kafka中的offset

   > 关于位移（Offset），其实在kafka的世界里有两种位移：
   >
   > - 分区位移：生产者向分区写入消息，每条消息在分区中的位置信息由一个叫offset的数据来表征。假设一个生产者向一个空分区写入了 10 条消息，那么这 10 条消息的位移依次是 0、1、…、9；
   > - 消费位移：消费者需要记录消费进度，即消费到了哪个分区的哪个位置上，这是消费者位移（Consumer Offset）。
   >
   > 分区位移一旦写入，不可改变，因为代表的是改消息在分区内的顺序，没必要改变。而消费者位移则不同，它可能是随时变化的，毕竟它是消费者消费进度的指示器。
   >
   > 
   >
   > 消费位移，记录的是 Consumer 要消费的下一条消息的位移，**切记，是下一条消息的位移！** 而不是目前最新消费消息的位移，在__consumer_offsets中存储以key-value格式，key为group.id+topic+ 分区号，value 就是当前 offset 的值

4. 说说leader副本和follower副本

   > 引入副本逻辑：对于大吞吐量系统，**副本可以不刷盘，节省大量性能**；使用简单的ISR可以保证使用N+1个节点即可保证n个节点的容错
   >
   > 
   >
   > leader副本对外提供读写（2.4版本后follower也能提供读），follower主动从leader拉数据，副本间用HW（高水位线保证数据一致性），但高水位值无法保证Leader连续变更场景下的数据一致性，因此，社区引入了Leader Epoch机制，来修复高水位值的弊端。

5. 什么是Leader Epoch机制

   > https://t1mek1ller.github.io/2020/02/15/kafka-leader-epoch/
   >
   > **Kafka通过Leader Epoch来解决follower日志恢复过程中潜在的数据不一致问题。**
   >
   > 原因：**因为follower的HW是不可靠的**
   >
   > - follower的HW更新有延时，重启时可能会错误的截断了已经提交了的日志
   > - 因为异步刷盘的策略，全崩溃的情况下选出的leader并不一定包含所有已提交的日志，而follower还是以HW为准，错误的判断了自身日志的合法性

6. Kafka的哪些场景中使用了零拷贝（Zero Copy）？

   > 在Kafka中，体现Zero Copy使用场景的地方有两处：**基于mmap的索引**和**日志文件读写所用的TransportLayer**。
   >
   > 
   >
   > 索引都是基于MappedByteBuffer的，也就是让用户态和内核态共享内核态的数据缓冲区，此时，数据不需要复制到用户态空间。不过，mmap虽然避免了不必要的拷贝，但不一定就能保证很高的性能。在不同的操作系统下，mmap的创建和销毁成本可能是不一样的。很高的创建和销毁开销会抵消Zero Copy带来的性能优势。由于这种不确定性，在Kafka中，只有索引应用了mmap，最核心的日志并未使用mmap机制。
   >
   > 再说第二个。TransportLayer是Kafka传输层的接口。它的某个实现类使用了FileChannel的transferTo方法。该方法底层使用sendfile实现了Zero Copy。对Kafka而言，如果I/O通道使用普通的PLAINTEXT，那么，Kafka就可以利用Zero Copy特性，直接将页缓存中的数据发送到网卡的Buffer中，避免中间的多次拷贝。相反，如果I/O通道启用了SSL，那么，Kafka便无法利用Zero Copy特性了。
   >
   > 
   >
   > 

   

7. 什么是mmap

   > mmap：程序运行中所需要的内存可能大于实际内存，不可能一次加载所有数据到内存中，将一部分放在硬盘，待需要时再放入内存，所以需要虚拟内存。在计算机上有一个页表（page table），就是映射虚拟内存页到物理内存页的，更确切的说是页号到页帧号的映射，而且是一对一的映射。虚拟内存页的个数 > 物理内存页帧的个数，岂不是有些虚拟内存页的地址永远没有对应的物理内存地址空间？不是的，操作系统是这样处理的。操作系统有个页面失效（page fault）功能。操作系统找到一个最少使用的页帧，使之失效，并把它写入磁盘，随后把需要访问的页放到页帧中，并修改页表中的映射，保证了所有的页都会被调度。
   >
   > 从硬盘上将文件读入内存，都要经过文件系统进行数据拷贝，并且数据拷贝操作是由文件系统和硬件驱动实现的，理论上来说，拷贝数据的效率是一样的。
   > 但是通过内存映射的方法访问硬盘上的文件，效率要比read和write系统调用高，这是为什么？
   >
   > - read()是系统调用，首先将文件从硬盘拷贝到内核空间的一个缓冲区，再将这些数据拷贝到用户空间，实际上进行了两次数据拷贝；
   > - map()也是系统调用，但没有进行数据拷贝，当缺页中断发生时，直接将文件从硬盘拷贝到用户空间，只进行了一次数据拷贝。
   >
   > 所以，采用内存映射的读写效率要比传统的read/write性能高。

1. 



## 微服务

### 限流

1. 如果要对大流量接口进行限流，在哪一层做，思路是什么

   > **大流量接口限流，必须在单节点做，不能做成集群限流，因为每个服务能承载的最大容量肯定不一样**，依赖于上下游服务。被其他服务依赖很多的核心服务接口，配合压测工具知道极限流量是多少，然后配置最大流量，熔断降级策略。
   >
   > 

2. 高并发情况下做接口权限校验，在哪一层，思路是什么

   > 高并发问题除了qps，还有rt、tps。大部分高并发redis都能抗住，大不了不丢弃慢点。百万级qps做个客户端多级缓存预热，静态cdn方案够了。
   >
   > 接口权限必定要在gateway层去实现，方便统一管理和减少代码冗余。

### 幂等性

1. 怎样保证接口幂等性

   > 幂等性指的是一个接口重复请求返回结果一致，可能原因有重复提交、网络问题导致超时重试、消息重复消费，前端可以通过一些表单限制之类的进行一定控制，后端可以通过以下方式：
   >
   > 1. **Token**：分为申请token和提交请求两个阶段，申请token时顺便保存在redis中，第一次提交请求后将redis中的token删除，后续如果重复提交redis中未找到对应token则判定重复提交
   > 2. **数据库防重表**：将订单号等唯一信息（也可以是多个字段信息合并的md5）设置为数据库唯一索引，第一次请求向数据库中插入数据，更新数据支付结果并不删除数据。缺点是系统容错性比较低，数据库单点问题会导致系统不可用
   > 3. **分布式锁，防重表的分布式方案**：解决数据库的单点问题，但是需要考虑过期时间
   > 4. **状态机**：订单的待提交，待支付，已支付，取消。必须按顺序执行。
   > 5. **mvcc**：博客点赞次数自动+1的接口，乐观锁的一种实现，在数据更新时需要去比较持有数据的版本号，版本号不一致的操作无法成功。

### 分布式事务

1. 分布式事务几种实现方式

   > 1. **2pc**：两阶段提交，第一阶段向所有数据库发送准备命令，所有数据库都返回ok后第二阶段发送提交事务命令。缺点是同步阻塞，如果协调者在第二阶段发送提交请求之后挂掉，而唯一接受到这条消息的参与者执行之后也挂掉了，即使协调者通过选举协议产生了新的协调者并通知其他参与者进行提交或回滚操作的话，都可能会与这个已经执行的参与者执行的操作不一样，当这个挂掉的参与者恢复之后，就会产生数据不一致的问题。
   > 2. **3pc**：三阶段提交，**准备阶段、预提交阶段和提交阶段**，准备阶段只是询问状态，预提交约等于2pc的准备阶段。引入了参与者超时机制，并且增加了预提交阶段使得故障恢复之后协调者的决策复杂度降低，但整体的交互过程更长了，性能有所下降，并且还是会存在数据不一致问题。无论是2PC还是3PC都不能保证分布式系统中的数据100%一致。
   > 3. **TCC补偿事务**：最终一致性分布式事务方案（柔性事务），发送提交请求后，根据请求执行结果，来判断是要继续执行下一步或者进行回滚补偿
   > 4. **最大努力通知**
   > 5. **本地消息表**：最终一致性分布式事务方案（柔性事务），将分布式事务拆分成本地事务，新建一个表用于记录事务状态，使用定时任务轮训事务消息然后通过mq发送执行
   > 6. MQ**消息事务**：最终一致性分布式事务方案（柔性事务），相比于本地消息表省去了数据库层面的本地事务
   >    1. 一般用支持事务消息的mq比如RocketMq（内部实现类似两阶段提交，先发送半消息给broker（此半消息consumer不可见），再执行本地事务，本地事务执行完之后再发送提交or回滚消息）
   >    2. kafka需要配合其幂等机制实现Exactly one，类似消费者参与实现两阶段提交，首先消费者发送请求，通过broker中的事务协调器记录到日志-事务日志中，然后生产者再发送具体消息，由消费者去过滤消息，消费完成后消费者再发送提交or回滚请求，由事务协调器来进行两阶段提交。
   >    3. https://juejin.cn/post/6867040340797292558
   >
   > https://zhuanlan.zhihu.com/p/183753774
   >
   > https://www.dockone.io/article/9804



## 问题排查

1. 线上cpu 100%怎么排查

   > 不管什么问题，既然是CPU飙升，肯定是**查一下耗CPU的线程，然后看看GC**。
   >
   > 1. `top`：查看cpu占用率高的进程pid
   > 2. `top -hp <pid>` : 查看进程内线程cpu占用率排名
   > 3. `jstack <线程pid>| vim +<进程pid>`：查看线程堆栈信息，或者使用 `jstack <线程pid> > ./18571.stack`导出到文件,注意此处需要将线程pid转16进制，因为jvm内线程pid是16进制，如果有“VM Thread”字样说明是gc线程
   > 4. `jstat -gcutil 进程号 统计间隔毫秒 统计次数`：查看GC变化情况，如果发现返回中FGC很大且一直增大-》确认Full GC! 也可以使用“jmap -heap 进程ID”查看一下进程的堆内从是不是要溢出了，特别是老年代内从使用情况一般是达到阈值(具体看垃圾回收器和启动时配置的阈值)就会进程Full GC。
   > 5. `jmap -dump:format=b,file=filename 进程ID`:导出某进程下内存heap输出到文件中
   >
   > 原因分析
   >
   > ###### 1.内存消耗过大，导致Full GC次数过多
   >
   > 执行步骤1-4：
   >
   > - 多个线程的CPU都超过了100%，通过jstack命令可以看到这些线程主要是垃圾回收线程-》上一节步骤2
   > - 通过jstat命令监控GC情况，可以看到Full GC次数非常多，并且次数在不断增加。--》上一节步骤4
   >
   > 确定是Full GC,接下来找到具体原因：
   >
   > - 生成大量的对象，导致内存溢出-》执行步骤6，查看具体内存对象占用情况。
   > - 内存占用不高，但是Full GC次数还是比较多，此时可能是代码中手动调用 System.gc()导致GC次数过多，这可以通过添加 -XX:+DisableExplicitGC来禁用JVM对显示GC的响应。
   >
   > ##### 2.代码中有大量消耗CPU的操作，导致CPU过高，系统运行缓慢；
   >
   > 执行步骤1-3：在步骤3jstack，可直接定位到代码行。例如某些复杂算法，甚至算法BUG，无限循环递归等等
   >
   > ##### 3.由于锁使用不当，导致死锁。
   >
   > 执行步骤1-3： 如果有死锁，会直接提示。关键字：deadlock.步骤四，会打印出业务死锁的位置。
   >
   > 造成死锁的原因：最典型的就是2个线程互相等待对方持有的锁。
   >
   > ##### 4.随机出现大量线程访问接口缓慢。
   >
   > 代码某个位置有阻塞性的操作，导致该功能调用整体比较耗时，但出现是比较随机的；平时消耗的CPU不多，而且占用的内存也不高。
   >
   > 思路：
   >
   > 首先找到该接口，通过压测工具不断加大访问力度，大量线程将阻塞于该阻塞点。

   > https://www.cnblogs.com/dennyzhangdd/p/11585971.html
   >
   > https://cloud.tencent.com/developer/article/1824235

2.  