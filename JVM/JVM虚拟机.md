---
typora-copy-images-to: pic
typora-root-url: pic
---

typora-copy-images-to: pic

# JVM虚拟机

## JVM结构

![image-20200606220216618](/JVM内存结构.png)

![image-20200613001233422](/JVM内存结构图.png)

1. 程序计数器

   - 存储当前线程下一条需要执行的命令的地址

2. 虚拟机**栈**

   - 表示线程运行需要的内存，栈内存储的是栈帧，每个栈帧对应一个方法，栈帧内部存储了**方法地址**、**参数**与**局部变量**，这些都存在一个**数字数组类型的局部变量表中**。一个线程只能有一个活动栈用来表示当前正在运行的方法。
   - **垃圾回收GC不涉及栈内存**，因为栈内存在方法调用完后会自己释放。
   - 栈内存并不是越大越好，栈内存越大可分配的进程数量越少。比如物理内存4G，每个栈大小1M，那么最多只能存在4000个线程。
   - 栈中变量是内存私有的（也就是方法内部定义的局部变量），是**线程安全**的，因为这些变量在物理内存不同的线程所属栈中都是私有的。但是**外部定义传入的参数或者是方法内定义然后return到外部的变量也都不是线程安全的。**说白了就是在方法内部定义的对象且没有逃离方法栈帧作用域的变量才是线程安全的。
   - 栈是有大小的，如果栈帧数量过多（无限递归）或者栈帧大小过大（内部局部变量表过大如无限长度的列表数组等）都可能造成**内存溢出**。
   - <img src="/栈帧的内部结构.png" alt="image-20200613002856900" style="zoom:80%;" />
     - 操作数栈：用于存储程序运行时临时需要存储的变量，比如i=j+k,先把j和k入栈然后取出来。一个栈单位能存储32bit类型。
       - ![image-20200613121626508](/操作数栈跟踪图一.png)
       - ![image-20200613121754555](/操作数栈跟踪图二.png)
     - 栈顶缓存：将栈顶元素缓存到cpu寄存器中，这样读取会比读内存快

3. 本地方法栈

   - 一些由C语言实现的方法，java中并不实现。比如Object对象的`Clone()`和`HashCode()`

4. 堆

   - 通过new关键字创建的对象都存放在堆内存中，他是线程共享的，有垃圾回收机制。

5. 方法区

   - 用于存放虚拟机加载的**类的信息**、**常量**（StringTable）、**静态变量**和**及时编译器编译后的代码**等，逻辑上属于堆，但是实际内存分配并不是在堆中，被称为老年代也是因为HotSpot虚拟机用老年代来实现方法区。这样的好处是虚拟机可以像管理堆一样来管理方法区，但是这样也**容易出现内存溢出**问题（特别是常量池引起的）。
   - 这里StringTable在1.8中存放在堆中主要原因是因为之前老年代需要FullGC才能回收，效率低。存放在堆中后只需要minorGC即可触发回收。
   - <img src="/堆内存示意图.png" alt="image-20200607100607721" style="zoom:80%;" />

6. 常量池

   - 常量池就是一张表，用于存放代码运行需要的一些常量，**虚拟机指令根据这张表查找到要执行的类名、方法名、参数类型和字面量等信息**。用一个例子来说明

     ```java
     public class HelloWorld{
     	public static void main(){
             System.out.println("hello world");
         }
     }
     ```

     - 上述代码在运行时需要编译成二进制字节码（包含**类基本信息**、**常量池**、**类方法定义**（包含了**虚拟机指令**））

     - 使用javac工具查看字节码反编译后的详细信息：

       ```java
       javac -v HelloWorld.class
       ```
   
       ```java
    // 类基本信息区域
       Classfile /D:/java/xczx_server/xc-framework-parent/test-rabbitmq-producter/src/main/java/com/xuecheng/test/rabbitmq/HelloWorld.class
      Last modified 2020-6-7; size 453 bytes
         MD5 checksum 0d9276e177a9044c9da40452da91d7d5
      Compiled from "HelloWorld.java"
       public class com.xuecheng.test.rabbitmq.HelloWorld
         minor version: 0
         major version: 52
      flags: ACC_PUBLIC, ACC_SUPER
         
        // 常量池区域
           Constant pool:
           #1 = Methodref          #6.#15         // java/lang/Object."<init>":()V
              #2 = Fieldref           #16.#17        // java/lang/System.out:Ljava/io/PrintStream;
           #3 = String             #18            // Hellow wordl
              #4 = Methodref          #19.#20        // java/io/PrintStream.println:(Ljava/lang/String;)V
           #5 = Class              #21            // com/xuecheng/test/rabbitmq/HelloWorld
              #6 = Class              #22            // java/lang/Object
           #7 = Utf8               <init>
              #8 = Utf8               ()V
              #9 = Utf8               Code
             #10 = Utf8               LineNumberTable
             #11 = Utf8               main
             #12 = Utf8               ([Ljava/lang/String;)V
             #13 = Utf8               SourceFile
             #14 = Utf8               HelloWorld.java
             #15 = NameAndType        #7:#8          // "<init>":()V
             #16 = Class              #23            // java/lang/System
             #17 = NameAndType        #24:#25        // out:Ljava/io/PrintStream;
             #18 = Utf8               Hellow wordl
             #19 = Class              #26            // java/io/PrintStream
             #20 = NameAndType        #27:#28        // println:(Ljava/lang/String;)V
             #21 = Utf8               com/xuecheng/test/rabbitmq/HelloWorld
             #22 = Utf8               java/lang/Object
             #23 = Utf8               java/lang/System
             #24 = Utf8               out
             #25 = Utf8               Ljava/io/PrintStream;
             #26 = Utf8               java/io/PrintStream
             #27 = Utf8               println
             #28 = Utf8               (Ljava/lang/String;)V
             
       // 类方法定义
       {
         public com.xuecheng.test.rabbitmq.HelloWorld();
           descriptor: ()V
           flags: ACC_PUBLIC
           Code:
             stack=1, locals=1, args_size=1
                0: aload_0
                1: invokespecial #1                  // Method java/lang/Object."<init>":()V
                4: return
             LineNumberTable:
               line 3: 0
       
         public static void main(java.lang.String[]);
           descriptor: ([Ljava/lang/String;)V
           flags: ACC_PUBLIC, ACC_STATIC
           Code:
             stack=2, locals=1, args_size=1
                // 具体操作语句，#2表明从常量池中读取#2变量
                0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
                3: ldc           #3                  // String Hellow wordl
                5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
                8: return
             LineNumberTable:
               line 18: 0
               line 19: 8
       }
       SourceFile: "HelloWorld.java"
       
       ​```
       ```
   
   - **运行时常量池**
   
     - 上述常量池是单个类的类常量池，在类被加载到内存中后会被放入运行时常量池，运行时常量池有点不同就是他的常量池里面存储的将不再是常量编号123，而是真实的内存地址。
   
   - StringTable是一个hashtable结构的表，不能扩容。与常量池不一样
   
     - 常量池中的字符串只是符号，在第一次用到的时候才变为对象
     - StringTable另一个作用是利用其哈希结构特性来**避免对象的重复创建**
     - 在java1.8中**字符串变量的拼接底层是利用Stringbuilder实现，会调用new关键字新创建一个对象，位置在堆中。而字符串常量的拼接则是编译期优化的结果，然后直接在StringTable里面查找，没有就在StringTable中加入这个常量，并不会在堆中新创建一个对象**
   
     - **懒加载**：代码里面的新的字符串并不会在编译时立即加载到StringTable，而是在代码运行到那一行发现StringTable里面没有找到对应字符串然后再加进去
   
     - java1.8中s.intern()方法可以将堆中的对象放入StringTable，并返回StringTable中这个对象，如果StringTable中已经存在相等对象则不放入直接返回StringTable中的对象。而在java1.6中则是先复制一份对象然后再放入StringTable中
   
     - 面试题：
   
       ![image-20200607120517729](/StringTable面试题.png)
   
     - StringTable调优
   
       - **优化StringTable的hash大小**。因为StringTable是哈希结构，所以调优的一个点就是**优化变量存入StringTable的时间**，因为哈希结构随机读取时间复杂度固定是O(1)，但是hash结构存在“桶”的特性，如果hash表大小设置的比较小，那么每个桶内链表存储的元素就会变多，这样的话特别是存入的时候时间影响非常明显。
       
       - 另外一个就是如果代码里面存在list等支持重复值的数据结构存储了大量元素，这样可能会造成内存占用过大。可以在添加元素到列表时使用iternal()方法，重复元素重用常量池对象，这样可以大大降低内存占用。
   
7. 直接内存

   - 用于NIO操作时的数据缓冲区
   - 不参与垃圾回收
   - 直接内存由来：
     - <img src="/划分直接内存前.png" alt="image-20200607193042981" style="zoom:80%;" />
     - 可以看到上述是在划分直接内存前，如果java要读取磁盘文件，内容需要先存到系统缓存区，然后复制到java堆缓冲区之后java代码才能读取。
     - <img src="/划分直接内存后.png" alt="image-20200607193250092" style="zoom:80%;" />
     - 而在内存中划分一个直接内存后，可以减少一次从系统缓冲区到java堆缓冲区的复制步骤。
   - 直接内存回收：
     - 直接内存并不能被垃圾回收GC算法处理，但是如果GC清除掉了bytBuffer这个对象后，那么会自动调用usafe里面的清除方法来释放内存。

### 方法在内存中执行详细过程

所要执行的代码：

```java
public class test1{
    public static void main(String[] args){
        int a=10;
        int b=Short.MAX_VALUE +1;
        int c=a+b;
        System.out.println(c);
    }
}
```

方法执行流程图示：

首先执行bipush命令表示将字节码push入栈帧中的操作数栈中，istore1表示将操作数栈顶数据弹出，存入局部变量表中的槽1中：

![image-20200613183813975](/方法执行流程图一.png)

接下来给b赋值，并不像上面a赋值10一样，因为b赋值的值超过了short类型长度，所以先存入运行时常量池，ldc表示取出运行时常量池中的数据

![image-20200613184336995](/方法执行流程图二.png)

![image-20200613184450852](/方法执行流程图三.png)

接下来则进入c=a+b流程，iload表示从局部变量表中取出对应整形数据，然后执行iadd命令相加此时弹出栈中已有的两个数并压入相加后的值，再将值存入局部变量表

![image-20200613184658367](/方法执行流程图四.png)

![image-20200613184854605](/方法执行流程图五.png)

完成计算后，接下来执行`System.out.println(c)`命令，可以看到getstatic意思是从运行时常量池中获取方法，然后将这个对象存入堆中，再把这个对象的引用地址压入操作数栈中

![image-20200613190004430](/方法执行流程图六.png)

![image-20200613190112605](/方法执行流程图七.png)

所谓a++和++a在内存执行上的区别就是a++是**先**执行iload命令，将a从局部变量槽中取出**压入操作数栈**，**再**在槽内部实现**自增**加1也就是iinc命令。

## 类加载

类加载整个流程整体宏观作用就是**ClassLoader类加载器从文件系统中加载class文件到内存中的方法区**

![image-20200612003419084](/类加载过程.png)

类加载过程：

1. 加载
   - 给他一个全限定类名，自动去硬盘找到**类文件转化为二进制字节流**
   - 把字节流代表的静态存储结构转化为方法区（永久代或元空间）运行时数据结构（也就是**把类的信息存入方法区**）
   - **在内存中生成这个类的java.lang.class对象**，用于作为访问方法区中这个类信息的入口
2. 链接
   - 验证
     - 验证字节流格式符不符合类规范，比如字节流开头必须是caffebbaby之类
   - 准备
     - **为类变量分配内存且赋零值**（static修饰等，但是final 修饰的static会在编译期就已经分配，这个阶段则会显示初始化）
     - 但是**不会为实例变量分配初始化**，因为实例变量属于每个具体对象，会与对象一起保存在堆中。
   - 解析
     - 常量池符号引用转变为具体指针引用，主要针对于类或者接口、字段、类方法等。
3. 初始化
   - 执行类构造器方法`<clinit>`(并不是类的构造器，此方法由javac编译期自动生成，按照类中代码顺序**收集类变量与静态代码块组成**，如果类里面没得以上两种则不会生成)，此方法加载类会被自动上锁

### 类加载器

类加载器分为两种：

1. 引导类加载器（Bootstrap ClassLoader）
   - 使用C语言实现，嵌套在JVM内部，用来加载JAVA核心类库与JVM自身需要的类
   - 并不继承自java.lang.ClassLoader，没有父加载器
2. 自定义加载器（User-Defined ClassLoader）
   - 所有继承自抽象类ClassLoader的类加载器都属于自定义加载器，java语言编写，父加载器为引导类加载器
   - 比如扩展类加载器与应用程序加载器：
     - 扩展类加载器：从JDK安装目录的jre/lib/ext子目录下加载类库
     - 应用程序加载器（App ClassLoader）：负责加载环境变量classpath，是程序的默认加载器
3. 自定义类加载器
   - 用来隔离加载类、修改类加载方式或者防止源码泄露等
   - <img src="/用户自定义类加载器步骤.png" alt="image-20200612020853051" style="zoom:80%;" />

获取类加载器的几种方法：

<img src="/获取类加载器的几种方法.png" alt="image-20200612015946327" style="zoom:80%;" />

### 双亲委派机制

类在需要用的时候才会被加载，程序一般当前加载器默认为应用程序加载器，双亲委派机制就是当前加载器如果有父加载器就一层层往上先问父加载器能不能加载这个类，一直到最大祖先引导类加载器，之后的逻辑就是如果爸爸不行了儿子才上。

![image-20200612235545771](/双亲委派机制.png)

优点：

- 避免类重复加载
- 避免核心类被篡改：因为都是先拿着类名从根到下问，如果是类名如String的核心类引导类加载器会默认加载java自带的JDK类，不会加载用户自定义的String类.也叫沙箱安全机制。

在java中判断两个类相同必须满足**被同一个类加载器加载而且类名也相同**

## 垃圾回收

### 对象可回收判定

#### 引用计数器法

如果一个对象地址被别的对象引用那么这个对象的引用计数器就+1，为0则表示可回收。

优点：实时性高；回收过程无需挂起进程，内存不足申请时立刻报outofmember错误；更新计数器只会影响到当前对象。

缺点：每次对象被引用都要更新计数器；一直维护计数器浪费CPU；无法解决**循环引用**问题（A->B,B->A）

####  可达性分析

为了解决引用计数器无法解决循环问题，标记清除法使用**可达性分析**方法来判断一个对象是否可以被回收。

可达性分析通过一些列“GC ROOT”节点出发，如果这个root节点能够到达的对象就表示存活，到不了的就表示判定可回收 ，有点像有向图的遍历。

<img src="/可达性分析流程图.png" alt="image-20200606103710139" style="zoom:50%;" />

可作为GC ROOT的对象：

- 虚拟机栈中引用的对象
- 方法区中类静态属性引用的对象
- 方法区中常量引用的对象
- 本地方法栈中JNI（native方法）引用的对象

不管是引用计数器还是可达性，判定方法都是依靠的“引用”，在java1.2版本前引用的定义是如果一个数据存储的值是指向另一个内存的起始地址那么判定被引用。1.2版本后java将引用分为4种：

- 强引用：类似`Object obj = new Object()`这样的引用，与老版本引用一样，引用存在对象就不会被回收。
- 软引用：描述有用但是非必须的对象，此类对象在系统将要发生内存溢出异常之前会被放进回收范围进行二次回收，之后内存还不够才会报出异常。可以配合引用队列释放软引用自身
- 弱引用：比软引用更弱，只能生存到下一次垃圾回收（FullGC，普通GC不一定会回收弱引用）。可以配合引用队列释放弱引用自身
- 虚引用：最弱，作用就是这个对象被删除的时候接收到一个通知而已。必须配合引用队列，比如byteBuffer，被引用对象回收时，虚引用入队列，由Refrence Handler线程调用虚引用方法直接释放内存
- 终结器引用：没有被强引用的对象，终结器引用被放入引用队列，然后会有一个优先级很低的线程Finalize Handiler线程去查看终结器引用，查找到之后就会去**调用对象的finalize方法**，第二次GC后对象才会被释放，如果finalize方法迟迟不执行完，那么对象会一直存在于内存中。

真正标记一个对象可以被回收需要经历**两次标记**过程，上述的可达性分析过程中的不可达标记只是第一次标记，如果被标记不可达后，还需要判断是否需要执行`finalize()`方A法，如果此方法没有复写`finalize()`方法或者`finalize()`方法已经被执行过这两种情况下都是没必要执行`finalize()`方法。否则将需要执行的对象放入F-Queue队列中，会被java创建的一个线程以非阻塞方式去运行。对象如果在`finalize()`方法中将自己赋值给某个变量或者对象可以完成自我拯救。

### 垃圾回收算法

jvm中年轻代和老年代因为对象平均生存时间不同，所以采用不同的算法，老年代对象存活时间平均比较长，所以多使用标记-清除或者标记-整理算法。

#### 标记清除法

算法分为**标记**和**清除**两个阶段，标记上面我们已经讲过，java中使用的是可达性分析方法。此方式是最基础方法，后续方法的改进都是基于此方法做出的。

缺点：

- **效率低**：因为每次清除都要遍历所有对象，且GC时需要**停止所有进程**，不利于交互性强的应用
- **内存碎片化**：此方法清理出来的内存不连续

#### 复制算法

为了解决上述问题中的**效率**问题，提出复制算法，将内存分为两块，from和to。from存放对象，to平时为空，当垃圾回收时对from区域进行回收，存活的对象放入to区中，清空from区，最后from和to区名字互换。

缺点是这样会降低一半的内存使用，不过经过分析可知大部分内存其实都是“死得快”，所以from区和to区的比例大概9:1就够了。java中每次都是使用一块Eden区和一块Survivor区做为from区，另一块Survivor区作为to区。比例也都是8:1:1。如果Survivor区空间不够则需要依赖老年代进行担保，也就是直接存入老年代。

<img src="/java复制算法.png" alt="image-20200606113110378" style="zoom:80%;" />

优点：

- 垃圾对象较多时效率不错
- 解决了内存碎片化问题

缺点：

- 如果垃圾对象较少情况下，就会进行大量复制，不适用与老年代
- 会降低内存使用率，且需要额外空间进行担保

#### 标记-整理算法

在标记-清除算法的基础上，为了解决另外一个**内存碎片化**问题，标记后不立即清除，而是先把保留的对象移动到一段，然后对边界后面的所有内存进行清除。

<img src="/标记-整理算法.png" alt="image-20200606180159640" style="zoom:80%;" />

缺点是移动内存需要一定的开销。

#### 分代回收

<img src="/分代回收.png" alt="image-20200607211646863" style="zoom:80%;" />

### 垃圾回收器

上面介绍了几种算法，实际java中有以下几种垃圾回收器：

![image-20200610003058314](/垃圾回收器总结.png)

<img src="/各种垃圾回收器运行位置结构图.png" alt="image-20200609003406056" style="zoom:80%;" />

#### serial（串行回收器）

为什么叫串行回收器呢，因为他是一个单线程收集器，这个串行收集器除了表示是单线程意外，还代表在进行垃圾回收时其他工作必须暂停知道回收结束（Stop the world）。

测试参数：

```java
// 设置堆初始大小和最大内存都是16M
-XX:UseSerialGC -XX:+PrintGCDetails -Xmsm6m -Xmx16m
```

#### ParNew

唯一能与CMS（Concurrent Mark Sweep）收集器一起工作的收集器。

#### CMS收集器

也叫**响应优先**收集器，这里的响应优先指的是一次GC总时长越短越好。基于标记-清除算法，总共有4个阶段：

- 初始标记
  - 标记GC Roots所关联的对象
  - STW
- 并发标记
  - 根据roots对象做可达性分析
- 重新标记
  - 修正并发标记阶段用户程序引起的标记变动的对象
  - 会扫描所有堆内存，引发STW时间较长
- 并发清除

- ![image-20200607222819126](/CMS工作流程.png)

整个过程中耗时最长的两个阶段**并发标记**和**并发清除**都不需要STW也就是说可以与用户线程并发，所以宏观上可以看做与**用户线程并发**，**低停顿**。

缺点：

- **对CPU资源敏感**；并发阶段虽然不会导致停顿但是占用了线程（或者说CPU资源）导致影响程序运行速度，降低总的吞吐量。当CPU在4个以上时默认占用1/4的CPU，但是当CPU数量少于4个时就影响很大了。
- **无法处理浮动垃圾**；可以看到并发清理阶段用户程序也在运行，那么这时候可能产生“**浮动垃圾**”，只有等到下一次GC才能进行收集。且垃圾收集时需要预留部分内存给用户程序运行使用，不能等到老年代到达100%再进行处理，JDK1.5中老年代触发收集的阈值是68%，1.6中是92%。如果内存还是不能满足需要那么就会退化为Serial Old收集器处理老年代，会导致停顿时间过长。
- **碎片化内存**；因为是标记-清楚算法，所以不可避免老年代内存会碎片化导致如果对象足够大但是找不到连续内存存储从而触发Full GC，参数-XX:+UseCMSCompactAtFullCollection开关参数可以开启在Full GC之前整理内存，但是整理过程无法并发会导致停顿时间边长。

#### G1收集器

推出G1收集器的目的：**在平衡的情况下进一步追求吞吐量和响应时间**。目前已有吞吐量优先的ParNew和响应时间优先的CMS。

优点：

- 并发性：同一时间可以有多个GC线程同时工作，此时STW
- 并行性：可以与用户线程同时工作
- 也属于分代收集器；从堆结构看不再要求eden区、年轻代和老年代空间是连续的，且大小不固定；将堆空间分为若干个区域（region）；同时兼顾年轻代和老年代。

- 内存整理：因为是将堆内存分为一个个region，回收以region为单位，在region之间采用复制算法，可以保证大对象有内存分配而不会因为找不到连续内存而引起提前触发下一次GC。
- 可预测停顿时间（软实时）：相比于CMS优势之一
  - G1可以选择性选取部分region进行回收，相比全局停顿**更能控制停顿时间**，也是G1更适合大堆内存情况的原因之一，当堆大小6G时停顿时间大概0.5s。
  - G1会跟踪每个region里面垃圾的大小，从而维护一个回收优先队列，优先回收性价比最高的region，从而结合上点可以在单位时间得到更高的垃圾回收率。
  - 停顿表现上，最好情况下不一定好过CMS的最好，但是最差情况下能好过CMS的最差

缺点：

- **垃圾回收时的内存占用**和**程序运行负载**会高过CMS
- **小内存CMS表现更优，大内存G1更优**，平衡点在6-8G

当**堆内存超过50%是活跃数据**、**对象分配和提升年代频率较大**、**GC停顿长于0.5s**时可以用G1替换CMS。在HotSpot实现下，除了G1其他收集器都是用内置JVM线程执行GC，而G1的GC在JVM线程处理下较慢时可以调用应用程序线程处理。

region区域：

- <img src="/region.png" alt="image-20200609012625152" style="zoom:80%;" />

- 其中如果一个region先是eden区，在垃圾回收后变为空白区域（空闲region），放入系统存储空闲region的列表，下一次可能这个region会被分配为old区或者其他区。
- 为什么要引入大对象区呢？因为大对象会被默认直接分配老年代，然而有些大对象是短命的，可能造成老年代内存不够引发Full GC。如果一个region放不下一个大对象那么就会寻找连续H区，都找不到就会触发Full GC。G1中大对象虽然有独立的区但是逻辑上还是被当成老年代。

G1收集器工作流程：

<img src="/G1收集器工作流程.png" alt="image-20200609004708547" style="zoom:80%;" />

- 年轻代GC：与其他收集器一样，eden区满了开始回收，放入survivor区或者old区。此过程并行（启动多线程执行年轻代回收）且STW
  - <img src="/年轻代GC.png" alt="image-20200610003800114" style="zoom:80%;" />
- 年轻代GC+并发标记过程：当堆内存使用超过45%后开始并发标记
  - <img src="/年轻代GC+并发标记过程.png" alt="image-20200610002931689" style="zoom:80%;" />
- 混合回收：标记完后马上开始混合回收。老年代的回收并不会回收全部老年代，会从维护的**高性价比**队列中选择回收小部分老年代，这些老年代可以和年轻代一起被回收。
  - <img src="/混合回收-1.png" alt="image-20200610003924852" style="zoom:80%;" />
  - <img src="/混合回收-2.png" alt="image-20200610004038968" style="zoom:80%;" />

Rmemebered Set（记忆集）

- 因为一个region中的对象可能会被很多其他region区域引用，包括年轻代和老年代，这样的话每次Minor GC都还需要扫描一下老年代的GC Roots，这样对于大堆来说很蠢。为了解决这个问题引入记忆集，每个region都有一个记忆集用来存储**这个region里被其他region里的对象引用**的对象信息。 

## 定位分析

### GC分析

参数：

|         含义          |                             参数                             |
| :-------------------: | :----------------------------------------------------------: |
|      堆初始大小       |                             -Xms                             |
|      堆最大大小       |                 -Xmx 或 -XX:MaxHeapSize=size                 |
|      新生代大小       |      -Xmn 或 (-XX:NewSize=size + -XX:MaxNewSize=size )       |
|  幸存区比例（动态）   | -XX:InitialSurvivorRatio=ratio 和 -XX:+UseAdaptiveSizePolicy |
|      幸存区比例       |                   -XX:SurvivorRatio=ratio                    |
|       晋升阈值        |              -XX:MaxTenuringThreshold=threshold              |
|       晋级详情        |                -XX:+PrintTenuringDistribution                |
|        GC详情         |               -XX:+PrintGCDetails -verbose:gc                |
| FullGC 前 MinorGC开关 |                  -XX:+ScavengeBeforeFullGC                   |

1. 首先打印出GC详细信息

   - VM参数：

   ```java
   -Xms20M -Xmx20M -Xmn10M -XX:+UseSerialGC -XX:+PrintGCDetails -verbose:gc
   ```

   - 代码：

   ```java
   public class Helloworld {
       public static void main(String[] args) {
       }
   }
   ```

   - 打印出的GC信息：

   ```
   Heap
    def new generation   total 9216K, used 2163K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
     eden space 8192K,  26% used [0x00000000fec00000, 0x00000000fee1cda0, 0x00000000ff400000)   // 年轻代Eden区
     from space 1024K,   0% used [0x00000000ff400000, 0x00000000ff400000, 0x00000000ff500000)   // 年轻代from区
     to   space 1024K,   0% used [0x00000000ff500000, 0x00000000ff500000, 0x00000000ff600000)   // 年轻代to区
     
    tenured generation   total 10240K, used 0K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
      the space 10240K,   0% used [0x00000000ff600000, 0x00000000ff600000, 0x00000000ff600200, 0x0000000100000000)
      
    Metaspace       used 3276K, capacity 4496K, committed 4864K, reserved 1056768K
     class space    used 357K, capacity 388K, committed 512K, reserved 1048576K
   ```

2. 接着存入7M大小对象

   - ```java
     public class Helloworld {
         private static final  int __512KB= 512 * 1024;
         private static final  int __1MB = 1024 * 1024;
         private static final  int __6MB = 6 * 1024 * 1024;
         private static final  int __7MB = 7 * 1024 * 1024;
     
         public static void main(String[] args) {
             ArrayList<byte[]> list = new ArrayList<>();
             // 存入7M大小对象
             list.add(new byte[__7MB]);
         }
     }
     ```

     ```java
     [GC (Allocation Failure) [DefNew: 1999K->613K(9216K), 0.0035453 secs] 1999K->613K(19456K), 0.0049288 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
     Heap
      def new generation   total 9216K, used 8191K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
       eden space 8192K,  92% used [0x00000000fec00000, 0x00000000ff366830, 0x00000000ff400000)
       from space 1024K,  59% used [0x00000000ff500000, 0x00000000ff599578, 0x00000000ff600000)
       to   space 1024K,   0% used [0x00000000ff400000, 0x00000000ff400000, 0x00000000ff500000)
      tenured generation   total 10240K, used 0K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
        the space 10240K,   0% used [0x00000000ff600000, 0x00000000ff600000, 0x00000000ff600200, 0x0000000100000000)
      Metaspace       used 3342K, capacity 4496K, committed 4864K, reserved 1056768K
       class space    used 365K, capacity 388K, committed 512K, reserved 1048576K
     ```

     可以看到触发了一次minor GC，eden区直接占到了92%，from区也就是这次gc前的to区域也占据了59%

3. 接着再存入512K对象

   - ```java
     public class Helloworld {
         ......
         public static void main(String[] args) {
             ArrayList<byte[]> list = new ArrayList<>();
             list.add(new byte[__7MB]);
             list.add(new byte[__512KB]);
         }
     }
     ```

     ```java
     [GC (Allocation Failure) [DefNew: 1999K->613K(9216K), 0.0021522 secs] 1999K->613K(19456K), 0.0022249 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
     Heap
      def new generation   total 9216K, used 8703K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
       eden space 8192K,  98% used [0x00000000fec00000, 0x00000000ff3e6840, 0x00000000ff400000)
       from space 1024K,  59% used [0x00000000ff500000, 0x00000000ff599658, 0x00000000ff600000)
       to   space 1024K,   0% used [0x00000000ff400000, 0x00000000ff400000, 0x00000000ff500000)
      tenured generation   total 10240K, used 0K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
        the space 10240K,   0% used [0x00000000ff600000, 0x00000000ff600000, 0x00000000ff600200, 0x0000000100000000)
      Metaspace       used 3348K, capacity 4496K, committed 4864K, reserved 1056768K
       class space    used 365K, capacity 388K, committed 512K, reserved 1048576K
     
     Process finished with exit code 0
     ```

4. 再存入512K

   - ```java
     public class Helloworld {
        	......
         public static void main(String[] args) {
             ArrayList<byte[]> list = new ArrayList<>();
             list.add(new byte[__7MB]);
             list.add(new byte[__512KB]);
             list.add(new byte[__512KB]);
         }
     }
     ```

     ```java
     [GC (Allocation Failure) [DefNew: 1999K->606K(9216K), 0.0028449 secs] 1999K->606K(19456K), 0.0029364 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
     [GC (Allocation Failure) [DefNew: 8614K->513K(9216K), 0.0058665 secs] 8614K->8283K(19456K), 0.0059244 secs] [Times: user=0.01 sys=0.00, real=0.01 secs] 
     Heap
      def new generation   total 9216K, used 1189K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
       eden space 8192K,   8% used [0x00000000fec00000, 0x00000000feca92f0, 0x00000000ff400000)
       from space 1024K,  50% used [0x00000000ff400000, 0x00000000ff4804b8, 0x00000000ff500000)
       to   space 1024K,   0% used [0x00000000ff500000, 0x00000000ff500000, 0x00000000ff600000)
      tenured generation   total 10240K, used 7770K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
        the space 10240K,  75% used [0x00000000ff600000, 0x00000000ffd96920, 0x00000000ffd96a00, 0x0000000100000000)
      Metaspace       used 3185K, capacity 4496K, committed 4864K, reserved 1056768K
       class space    used 348K, capacity 388K, committed 512K, reserved 1048576K
     ```

     可以看到刚才eden区占用到了92%，再放入512k后eden区满了，触发了一次minor GC。而这次由于年轻代已经村放不下那么多了，所以对象放入老年代。

5. 直接存入大于新生代大小的**大对象**

   ```java
   public class Helloworld {
       ......
       public static void main(String[] args) {
           ArrayList<byte[]> list = new ArrayList<>();
           list.add(new byte[__8MB]);
       }
   }
   ```

   ```java
   Heap
    def new generation   total 9216K, used 2163K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
     eden space 8192K,  26% used [0x00000000fec00000, 0x00000000fee1cda0, 0x00000000ff400000)
     from space 1024K,   0% used [0x00000000ff400000, 0x00000000ff400000, 0x00000000ff500000)
     to   space 1024K,   0% used [0x00000000ff500000, 0x00000000ff500000, 0x00000000ff600000)
    tenured generation   total 10240K, used 8192K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
      the space 10240K,  80% used [0x00000000ff600000, 0x00000000ffe00010, 0x00000000ffe00200, 0x0000000100000000)
    Metaspace       used 3337K, capacity 4496K, committed 4864K, reserved 1056768K
     class space    used 364K, capacity 388K, committed 512K, reserved 1048576K
   ```

   可以发现大对象直接存入老年代。如果再次放入8M内存会触发OOM（OutOfMemery）内存溢出，此时会清空线程占用的堆内存。

### GC调优

#### 新生代

特点：

- new方法对象分配廉价，每个线程都有自己的内存分配缓冲区TLAB，防止多线程并发情况下分配内存冲突
- 死亡对象内存回收0代价，大部分对象死得快
- Minor GC时间很短

调优方向：

1. 新生代大小
   - 参数：-Xmn
   - 新生代大点肯定比小好，因为绝大多数对象都是在这个阶段，大点可以减少minor GC次数。但是**并不是越大越好**，因为新生代大了，minor GC后晋升的对象过多，老年代装不下就要引发Full GC。**一般设置为堆内存的25%-50%之间**。
2. 幸存区
   - 大小最好是能覆盖过活跃对象和晋升对象
3. 晋升阈值
   - 让长时间存活对象尽早晋升

#### 老年代

老年代内存尽量大最好，避免浮动垃圾引起的并发清除失败导致Full GC。如果正常情况没Full GC那么就别动老年代，发生了再调大1/4左右。

#### 案例分析

1. Minor GC和Full GC都很频繁：

   > 有可能是因为新生代空间较小首先引起频繁Minor GC，这种情况下survivor区不够了晋升条件会放宽，更多短命对象进入老年代从而引发频繁Full GC

2. CMS在请求高峰期发生Full GC，单词暂停时间较长

   > 之前说到过单词暂停时间较长可以去打印GC日志查看每个阶段时长信息，如果是重新标记阶段时间过长，那么之前说过重新标记阶段会扫描所有堆内存的对象，可以在重新标记之前执行以下minor GC清除新生代垃圾从而降低这个堆内存扫描时间。

3. 

### CPU占用过高定位

1. 先用top命令定位占用CPU过高进程（**定位进程**）
2. 再用 `ps H -eo pid,tid,%cpu |grep <进程id>`定位进程的哪个具体线程（**定位线程**）
3. `jstack <进程id>`
   - 可以根据线程id转换为16进制从而定位到具体代码行数

### 死锁定位

同上直接用`jstack <进程id>`，看最后

### 堆内存容量判断

1. 使用jsp查看系统中的java进程
2. jmap工具：
   - `jmap -head <进程id>`

或者直接使用图形化工具`jconsole`或`jvisualvm`

