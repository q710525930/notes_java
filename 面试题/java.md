---
编码-images-to: ..\JDK\pic
---

## java

#### 八种基本类型

1. byte：1字节8bit，包装类Byte
2. short：2字节16bit，包装类Short
3. int：4字节32bit，包装类Integer
4. long：8字节64bit，包装类Long
5. float：4字节32bit，包装类Float
6. double：8字节64bit，包装类Double
7. char：2字节16bit，包装类Character
8. boolean：经过编译后在JVM中会通过int类型来表示，此时boolean数据4字节32位，而boolean数组将会被编码成Java虚拟机的byte数组，此时每个boolean数据1字节占8bit.

#### Object类

方法：

1. **equals**：否等于另一个对象，默认使用 `==` 比较两个对象的引用。**string类型重写此方法用于比较两个字符串的值**
2. **hashcode**：每个对象都有一个默认的散列码，**值由对象的存储地址得出**。字符串可能有相同的散列码，因为字符串的散列码是由内容导出的。equals 相同hashCode 必须相同，但 hashCode 相同时 equals 未必相同。**存储在对象头里**
3. **toString**：打印对象时默认会调用它的 toString 方法
4. **clone**：声明为 protected，类只能通过该方法克隆它自己的对象。默认的 clone 方法是浅拷贝，一般重写 clone 方法需要实现 Cloneable 接口并指定访问修饰符为 public。**浅拷贝是不安全的，拷贝对象的更改会影响原对象**；**深拷贝是安全的，会完全拷贝基本数据类型和引用数据类型**。
5. **finalize**：在垃圾收集器清理对象之前调用，由于无法确定该对象执行的具体时机因此已经被废弃。GC前执行，但是不保证一定执行完
6. **getClass**：返回包含对象信息的类对象。
7. **wait / notify / notifyAll：**阻塞或唤醒持有该对象锁的线程。



#### final 关键字

final修饰一个类时，表明这个类不能被继承，final类中的成员变量可以根据需要设为final，final类中的所有成员方法都会被隐式地指定为final方法。

对于一个final变量，如果是基本数据类型的变量，则其数值一旦在初始化之后便不能更改；如果是引用类型的变量，则在对其初始化之后便不能再让其指向另一个对象。



#### 接口和抽象类

成员变量：接口中的成员变量默认是 public static final 修饰的常量，抽象类中的成员变量无特殊要求。

构造器：接口和抽象类都不能直接实例化，但**接口没有构造器，抽象类是有构造器的**。

继承：**接口可以多继承和多实现，而抽象类只能单继承。**

如果知道某个类应该成为基类，那么第一选择应该是让它成为一个接口，只有在必须要有方法定义和成员变量的时候，才应该选择抽象类



#### java中的值传递和引用传递

在Java中所有的参数传递，**不管基本类型还是引用类型，都是值传递，或者说是副本传递**。
只是在传递过程中：

如果是对基本数据类型的数据进行操作，由于原始内容和副本都是存储实际值，并且是在不同的栈区，因此形参的操作，不影响原始内容。

如果是对引用类型的数据进行操作，分两种情况，一种是形参和实参保持指向同一个对象地址，则形参的操作，会影响实参指向的对象的内容。一种是形参被改动指向新的对象地址（如重新赋值引用），则形参的操作，不会影响实参指向的对象的内容。也就是说，**传递过来的是个地址值，方法内部形参变量指向栈帧局部变量表中的值为这个地址值而已**。

[这一次，彻底解决Java的值传递和引用传递](https://juejin.im/post/5bce68226fb9a05ce46a0476)



#### 是否可以在static环境中访问非static变量？

static变量在Java中是属于类的，它在所有的实例中的值是一样的。**当类被Java虚拟机载入的时候，会对static变量进行初始化**。如果你的代码尝试不用实例来访问非static的变量，编译器会报错，因为这些变量还没有被创建出来，还没有跟任何实例关联上。



#### int和integer区别

基本数据类型，分为boolean、byte、int、char、long、short、double、float；

为了能够将这些基本数据类型当成对象操作，**Java为每一个基本数据类型都引入了对应的包装类型**，int的包装类就是Integer，从Java 5开始引入了自动装箱/拆箱机制，使得二者可以相互转换。

区别：

- Integer是int的包装类；int是基本数据类型；
- Integer变量必须实例化后才能使用；int变量不需要；
- Integer实际是对象的引用，指向此new的Integer对象；int是直接存储数据值 ；
- Integer的默认值是null；int的默认值是0。

特性：

- 由于Integer变量实际上是对一个Integer对象的引用，所以两个通过new生成的Integer变量永远是不相等的（因为new生成的是两个对象，其内存地址不同）。
- Integer变量和int变量比较时，只要两个变量的值是向等的，则结果为true（因为包装类Integer和基本数据类型int比较时，java会自动拆包装为int，然后进行比较，实际上就变为两个int变量的比较）
- 非new生成的Integer变量和new Integer()生成的变量比较时，结果为false。因为**非new生成的Integer变量指向的是静态常量池中cache数组中存储的指向了堆中的Integer对象**，而**new Integer()生成的变量指向堆中新建的对象**，两者**在内存中的对象引用（地址）不同**。
- 对于两个非new生成的Integer对象，进行比较时，如果两个变量的值在区间-128到127之间，则比较结果为true，如果两个变量的值不在此区间，则比较结果为false。 java在编译Integer i = 100 ;时，会翻译成为Integer i = Integer.valueOf(100)。而java API中对Integer类型的valueOf的定义如下，**对于-128到127之间的数，会进行缓存**，**在-128~127之内的数值，它们被装箱为Integer对象后，会存在内存中被重用，始终只存在一个对象**。Integer i = 127时，会将127这个Integer对象进行缓存，下次再写Integer j = 127时，就会直接从缓存中取，就不会new了。



#### String,StringBuffer与StringBuilder的区别

都是final类，都不能被继承

String类长度是不可变的，StringBuffer和StringBuilder类长度是可以改变的。

> 原因：**String类中定义的char数组是final的**，而StringBuffer和StringBuilder都是继承自AbstractStringBuilder类，它们的内部实现都是靠这个父类完成的，而这个父类中定义的**char数组只是一个普通是私有变量**，可以用append追加。因为AbstractStringBuilder实现了Appendable接口。
>
> String为什么要设计为不可变：
>
> 1. 创建一个String对象时,假如此字符串值已经存在于**常量池**中,则不会创建一个新的对象,而是引用已经存在的对象。
> 2. String对象用hash变量**缓存了HashCode**，不变形**保证了hashcode的唯一性**，因此**适合作为map主键**，因为字符串处理速度很快
> 3. 类加载、数据库连接属性、等很多地方都用String类型进行加载，可变会导致出现很多问题
> 4. **不可变保证了线程安全**

StringBuffer类是线程安全的（toStringCache关键字修饰StringBuffer的append方法，toStringCache关键字是给线程加锁），StringBuilder不是线程安全的；底层实现上的话，**StringBuffer其实就是比StringBuilder多了Synchronized修饰符**。

**果要操作少量的数据，用String ；单线程操作大量数据，用StringBuilder ；多线程操作大量数据，用StringBuffer。**



#### static关键字

“static”关键字表明一个成员变量或者是成员方法可以在**没有所属的类的实例变量的情况下被访问**，**存储在类方法区**，**Java中static方法不能被覆盖，因为方法覆盖是基于运行时动态绑定的，而static方法是编译时静态绑定的**。static方法跟类的任何实例都不相关，所以概念上不适用。





#### 为什么重写equals还要重写hashcode

HashMap中，如果要比较key是否相等，要同时使用这两个函数！如果只重写equals不重写hashcode那么hashcode不一样会被定位到数组不同的位置，**会出现不同的桶上有同样元素**

往HashMap添加元素的时候，需要先**计算hashCode定位到在数组的位置**，如果该数组的位置上已经存在元素了遍历链表，**调用 equals 方法判断key是否相等**。



#### 动态代理

1. CGLIB：其本质是在内存中继承了一个子类，可以代理希望代理的那个类的所有方法
2. JDK动态代理：实现InvocationHandler，通过生成一个Proxy来反射调用所有的接口方法

优劣：  

- ​    CGLIB：通过继承的方式进行代理，会在内存中多存额外的class信息，对metaspace区的使用有影响，但是性能好，可以访问非接口的方法，但是会跳过final方法因为不能被重载
- ​    JDK动态代理：本质是**生成一个继承所有接口的Proxy来反射调用方法**，局限性在于其**只能代理接口的方法**

反射的开销：检查方法权限，序列化以及匹配入参

#### 反射

反射就是指程序在运行时动态获取到类的基本信息方法，可以**提高程序的灵活性，屏蔽实现细节**。

要通过反射获取一个类或者调用类方法，首先要获取到类的class对象，java中获取方法区中类信息的方法有三种：

- 类名.class
- getClass 方法
- forName 方法

获取到类信息后，创建反射类对象由两种方式：

- 通过 Class 对象的 newInstance() 方法。

  - ```java
    Class clz = Apple.class;
    Apple apple = (Apple)clz.newInstance();
    ```

- 通过 Constructor 对象的 newInstance() 方法

  - ```java
    Class clz = Apple.class;
    Constructor constructor = clz.getConstructor();
    Apple apple = (Apple)constructor.newInstance();
    ```

  - 通过 Constructor 对象创建类对象可以选择特定构造方法，而通过 Class 对象则只能使用默认的无参数构造方法

还可以通过class对象直接获取对象类属性、方法

```java
Class clz = Apple.class;
Field[] fields = clz.getFields();
```

反射JDK源码流程

![img](https://img2018.cnblogs.com/blog/595137/201903/595137-20190324000247330-1279629878.png)

#### 多态

**多态的底层实现是动态绑定**，即在运行时才把方法调用与方法实现关联起来。

JVM 的方法调用指令有五个，分别是：

- invokestatic：调用静态方法；
- invokespecial：调用实例构造器<init>方法、私有方法和父类方法；
- invokevirtual：调用虚方法；
- invokeinterface：调用接口方法，运行时确定具体实现；
- invokedynamic：运行时动态解析所引用的方法，然后再执行，用于支持动态类型语言。

动态绑定主要应用于虚方法和接口方法。**静态绑定在编译期就已经确定**，这是因为静态方法、构造器方法、私有方法和父类方法可以唯一确定。这些方法的符号引用在类加载的解析阶段就会解析成直接引用。因此这些方法也被称为非虚方法，与之相对的便是虚方法。

虚方法的方法调用与方法实现的关联（也就是分派）有两种，一种是**在编译期确定，被称为静态分派，比如方法的重载**；一种是在**运行时确定，被称为动态分派，比如方法的覆盖**。对象方法基本上都是虚方法。

这里需要特别说明的是，final 方法由于不能被覆盖，可以唯一确定，因此 Java 语言规范规定 final 方法属于非虚方法，但仍然使用 invokevirtual 指令调用。静态绑定、动态绑定的概念和虚方法、非虚方法的概念是两个不同的概念。

**多态的实现过程，就是方法调用动态分派的过程，通过栈帧的信息去找到被调用方法的具体实现，然后使用这个具体实现的直接引用完成方法调用。**

以 invokevirtual 指令为例，在执行时，大致可以分为以下几步：

1. 先从操作栈中找到对象的实际类型 class；
2. 找到 class 中与被调用方法签名相同的方法，如果有访问权限就返回这个方法的直接引用，如果没有访问权限就报错 java.lang.IllegalAccessError ；
3. 如果第 2 步找不到相符的方法，就去搜索 class 的父类，按照继承关系自下而上依次执行第 2 步的操作；
4. 如果第 3 步找不到相符的方法，就报错 java.lang.AbstractMethodError ；

可以看到，**如果子类覆盖了父类的方法，则在多态调用中，动态绑定过程会首先确定实际类型是子类，从而先搜索到子类中的方法。这个过程便是方法覆盖的本质**。

实际上，商用虚拟机为了保证性能，通常会使用虚方法表和接口方法表，而不是每次都执行一遍上面的步骤。以虚方法表为例，虚方法表在类加载的解析阶段填充完成，其中存储了所有方法的直接引用。也就是说，动态分派在填充虚方法表的时候就已经完成了。

**在子类的虚方法表中，如果子类覆盖了父类的某个方法，则这个方法的直接引用指向子类的实现；而子类没有覆盖的那些方法，比如 Object 的方法，直接引用指向父类或 Object 的实现。**

## 集合

#### 请说明Collection 和 Collections的区别

Collection是集合类的上级接口，继承与他的接口主要有**Set 和List**。Collections是针对集合类的一个帮助类，他提供一系列静态方法实现对各种集合的搜索、排序、线程安全化等操作。

<img src="/Collection接口结构图.png" alt="image-20200712144644689" style="zoom:67%;" />

Collection包含了List和Set两大分支：

1. List是一个有序的队列，每一个元素都有它的索引，第一个元素的索引值是0。List的实现类有**LinkedList、ArrayList、Vector和Stack**。
   - LinkedList实现了List接口，允许元素为空，LinkedList提供了额外的get,remove,insert方法，这些操作可以使LinkedList被用作堆栈、队列或双向队列。LinkedList并不是线程安全的，如果多个线程同时访问LinkedList，则必须自己实现访问同步，或者另外一种解决方法是在创建List时构造一个同步的List。
   - ArrayList  实现了可变大小的数组，允许所有元素包括null，同时ArrayList也不是线程安全的。
   - Vector类似于ArrayList，但Vector是线程安全的。
     - Stack继承自Vector，实现一个后进先出的堆栈。
2. set是一个不允许有重复元素的集合。set的实现类有**Hashset和Treeset**。HashSet依赖于HashMap，实际上是通过HashMap实现的；TreeSet依赖于TreeMap，通过TreeMap来实现的。

```
Collection
    |-----List  有序(存储顺序和取出顺序一致)，可重复
        |----ArrayList ，线程不安全，底层使用数组实现，查询快，增删慢，效率高。
        |----LinkedList ， 线程不安全，底层使用链表实现，查询慢，增删快，效率高。
        |----Vector ， 线程安全，底层使用数组实现，查询快，增删慢，效率低。每次容量不足时，默认自增长度的一倍（如果不指定增量的话）
    |-----Set   元素唯一一个不包含重复元素的 collection。更确切地讲，set 不包含满足 e1.equals(e2) 的元素对 e1 和 e2，并且最多包含一个 null 元素。
        |--HashSet 底层是由HashMap实现的，通过对象的hashCode方法与equals方法来保证插入元素的唯一性，无序(存储顺序和取出顺序不一致)，。
            |--LinkedHashSet 底层数据结构由哈希表和链表组成。哈希表保证元素的唯一性，链表保证元素有序。(存储和取出是一致)
        |--TreeSet 基于 TreeMap 的 NavigableSet 实现。使用元素的自然顺序对元素进行排序，或者根据创建 set 时提供的 Comparator 进行排序，具体取决于使用的构造方法。 元素唯一。
```

Collection接口方法：

List接口定义方法：

```java
void add(String item)  //依次往后添加添加元素
void add(String item, int index) //在指定位置处添加元素
void remove(int position) //删除第几个元素（索引从0开始）
void remove(String item) //删除相同的元素
void removeAll() //删除所有元素
```

Set接口方法

```java
boolean add(Object o)  //该方法用于向集合里添加一个元素。
boolean addAll(Collection c)  //该方法把集合c里的所有元素添加到指定集合里。
void clear()  //清除集合里的所有元素，将集合长度变为0。
boolean contains(Object o)  //返回集合里是否包含指定元素。
boolean containsAll(Collection c)  //返回集合里是否包含集合c里的所有元素。
boolean isEmpty()  //返回集合是否为空。当集合长度为0时返回true，否则返回false。
Iterator iterator()  //返回一个Iterator对象，用于遍历集合里的元素。
boolean remove(Object o)  //删除集合中的指定元素o，当集合中包含了一个或多个元素o时，这些元素将被删除，该方法将返回true。
boolean removeAll(Collection c)  //将集合中删除集合c里包含的所有元素（相当于用调用该方法的集合减集合c），如果删除了一个或一个以上的元素，则该方法返回true。
boolean retainAll(Collection c)  //将集合中删除集合c里不包含的元素（相当于把调用该方法的集合变成该集合的集合c的交集），如果该操作改变了调用该方法的集合，则该方法返回true。
int size()  //该方法返回集合里元素的个数。
Object[] toArray()  //该方法把集合转换成一个数组，所有的集合元素变成对应的数组元素。
```



#### Array（数组）与ArrayList（列表）区别

区别：

- Array可以包含基本类型和对象类型，ArrayList只能包含对象类型。
  - Array数组在存放的时候一定是同种类型的元素。ArrayList就不一定了
-  Array大小是固定的，ArrayList的大小是动态变化的（**默认是10**）。
  - ArrayList的空间是动态增长的，如果空间不够，它会创建一个空间比原空间大一倍的新数组，**每次添加新的元素的时候都会检查内部数组的空间是否足够**
- ArrayList 方法上比 Array 更多样化，比如添加全部 addAll()、删除全部 removeAll()、返回迭代器 iterator() 等。并发add()可能出现数组下标越界异常。



#### ArrayList在循环过程中删除

1. for删除，不会直接抛异常，但是会产生异常访问    
2. foreach删除（实际就是迭代器），会直接抛出并发修改异常，因为迭代器会进行获取迭代器时的exceptModCount和真实的modCount的对比

**为什么foreach遍历ArrayList的时候，remove元素会抛出异常？**

ArrayList中有类变量modCount记录了ArrayList变化的次数，每次在add/remove的时候会自增，当调用ArrayList.iterator()方法的时候会new一个内部类Itr，并在初始化Itr的时候赋值expectedModCount=modCount。

当ArrayList再调用add/remove方法的时候modCount会再次自增，但expectedModCount并不会随之改变。最终在遍历时**调用Itr.next()校验**modCount是否等于expectedModCount时抛出异常。

**为什么用迭代器遍历时，remove元素就不会抛出异常？**

迭代器遍历时，调用Itr.remove()时会再次令expectedModCount=modCount，所以在Itr.next()判断时会通过校验。

#### copyonwrite机制

和单词描述的一样，他的实现就是写时复制， 在往集合中添加数据的时候，先拷贝存储的数组，然后添加元素到拷贝好的数组中，然后用现在的数组去替换成员变量的数组（就是get等读取操作读取的数组）。这个机制和读写锁是一样的，但是比读写锁有改进的地方，那就是读取的时候可以写入的 ，这样省去了读写之间的竞争，看了这个过程，你也发现了问题，同时写入的时候怎么办呢，当然果断还是加锁。

java中提供了两个利用这个机制实现的线程安全集合。copyonwritearraylist和copyonwritearrayset，copyonwritearrayset的底层实现是copyonwritearraylist。

其中get方法用volatile声明数组，保证了读取的那一刻读取的是最新的数据；add方法需要reentrantlock加锁，接下来就是复制数据和添加数据的过程，在setArray的过程中，把新的数组赋值给成员变量array（这里是引用的指向，java保证赋值的过程是一个原子操作）。

关于迭代，采取的是获取传递给迭代器的数组值进行迭代，中间就算加入新的值也迭代不到。非强一致性。在构造函数中就直接赋值给final的成员变量。

copyonwrite的机制虽然是线程安全的，但是在add操作的时候不停的拷贝是一件很费时的操作，所以使用到这个集合的时候尽量不要出现频繁的添加操作，而且在迭代的时候数据也是不及时的，数据量少还好说，数据太多的时候，实时性可能就差距很大了。在多读取，少添加的时候，他的效果还是不错的（数据量大无所谓，只要你不添加，他都是好用的）。

#### 快速失败(fail-fast)和安全失败(fail-safe)

快速失败和安全失败是对迭代器而言的

- 快速失败：当在迭代一个集合的时候，如果有另外一个线程在修改这个集合，就会抛出ConcurrentModification异常，java.util下都是快速失败。
-  安全失败：在迭代时候会在集合二层做一个拷贝，所以在修改集合上层元素不会影响下层。在java.util.concurrent下都是安全失败



#### map类

java为数据结构中的映射定义了一个接口java.util.Map;它有四个实现类,分别是

- HashMap
  - 具有很快的访问速度，遍历时取得数据的顺序随机，HashMap最多只允许一条记录的键为Null，允许多条记录的值为 Null。**线程不安全**，
- Hashtable
  - 与 HashMap类似,它继承自Dictionary类，**不允许记录的键或者值为空**，**线程安全**
- LinkedHashMap
  - HashMap的一个子类，**保存了记录的插入顺序**，LinkedHashMap的遍历速度只和实际数据有关，和容量无关，而HashMap的遍历速度和他的容量有关
- TreeMap
  - 实现SortMap接口，能够把它保存的记录**根据键排序**,默认是按键值的升序排序

#### HashMap

**1.7和1.8的差异**：

1. **1.7使用头插法，1.8使用尾插法**（1.7使用单链表能**提高插入效率**，但是会出现**逆序且环形链表**的问题；1.8后加入红黑树所以不用这样提升插入效率，还避免了逆序且[环形链表](https://www.jianshu.com/p/4930801e23c8)的问题）
2. 扩容后数据存储位置计算方式不同
   - 1.7中直接用hash值和需要扩容的二进制数进行&算法，（扩容的时候为啥一定必须是2的多少次幂的原因，如果只有2的n次幂的情况时最后一位二进制数才一定是1，能最大程度减少hash碰撞，hash值 & length-1）。
   - 1.8中直接等于扩容前的原始位置+扩容的大小值，只需要判断Hash值的新增参与运算的位是0还是1。
3. 1.8加入了红黑树，把时间复杂度从O(n)降低到Log(n)。

**HashMap 默认的初始化长度是多少？为什么默认长度和扩容后的长度都必须是 2 的幂？**

默认初始化长度为16，获取数组索引的计算方式为 key 的 hash 值按位与运算数组长度减一，2的n次幂-1的二进制表示全为1111，那么这样只要hash值最后四位分布均匀则得到的数组下标也是均匀的

 **HashMap 构造方法中 initialCapacity（初始容量）、loadFactor（加载因子）的理解？**

nitialCapacity 初始容量代表了哈希表中桶的初始数量，但是diamante会自动通过一串二进制右移的操作保证它变为2的幂。loadFactor 加载因子决定了何时开启resize（默认0.75，因为参考了泊松分布）。

**扩容机制，为什么都是2的N次幂的大小。**

创建一个新的Entry空数组，长度是原数组的2倍。遍历原Entry数组，把所有的Entry重新Hash到新数组。为什么要重新Hash呢？因为长度扩大以后，Hash的规则也随之改变。

- hash的计算是通过hashcode高低位混合然后和容量的length进行与运算    
- 在length=2n的时候，与运算相当于是一个取模操作    
-  那么在每次rehash完毕之后mod2N的意义在于要么该元素是在原位置，要么是在最高位偏移多一位的位置，提高效率
- 可以引申redis渐进式hash扩容处理与ConcurrentHashMap的扩容：1.7分段扩容以及1.8transfer并发协同的扩容

1.7中扩容过程就是一个取出数组元素，遍历以该元素为头的单向链表元素，用每个元素的hash值计算新下标。

而在 JDK 1.8 中 HashMap 的扩容操作就显得更加的骚气了，由于扩容数组的长度是 2 倍关系，所以对于假设初始 tableSize =4 要扩容到 8 来说就是 0100 到 1000 的变化（左移一位就是 2 倍），在扩容中只用判断原来的 hash 值与左移动的一位按位与操作是 0 或 1 就行，0 的话索引就不变，1 的话索引变成原索引加上扩容前数组。

#### hash冲突算法

1. **开放定址法**：如果冲突，则按顺序往后找，缺点是查找麻烦，删除元素时不能直接删除会影响到其他节点逻辑，容易产生堆积
2. **再hash**：冲突时用另一个hash计算函数再次计算hash，不容易产生堆积，但是会增加计算时间
3. **链地址**：冲突时在冲突位置构造链表，将冲突元素放到链表上，缺点是需要额外存储空间

手写hashmap：

```java
public class MyHashMap {
 
    public static void main(String[] args) {
        MyHashMap hm = new MyHashMap();
        hm.put(1, 1);
        hm.put(2, 2);
        System.out.println(hm.get(1)); // 1
        hm.remove(1);
        System.out.println(hm.get(1)); // -1
    }
 
    private final int N = 100000; // 静态数组长度100000
 
    private Node[] arr;
 
    public MyHashMap() {
        arr = new Node[N];
    }
 
    public void put(int key, int value) {
        int idx = hash(key);
 
        if (arr[idx] == null) { // 没有发生哈希碰撞
            arr[idx] = new Node(-1, -1); // 虚拟头节点
            arr[idx].next = new Node(key, value); // 实际头节点
        } else {
            Node prev = arr[idx]; // 从虚拟头开始遍历
 
            while (prev.next != null) {
                if (prev.next.key == key) {
                    prev.next.value = value; // 直接覆盖value
                    return;
                }
                prev = prev.next;
            }
            prev.next = new Node(key, value); // 没有键则插入节点
        }
    }
 
    public int get(int key) {
        int idx = hash(key);
 
        if (arr[idx] != null) {
            Node cur = arr[idx].next; // 从实际头节点开始寻找
 
            while (cur != null) {
                if (cur.key == key) {
                    return cur.value; // 找到
                }
                cur = cur.next;
            }
        }
        return -1; // 没有找到
    }
 
    public void remove(int key) {
        int idx = hash(key);
 
        if (arr[idx] != null) {
            Node prev = arr[idx];
 
            while (prev.next != null) {
                if (prev.next.key == key) { // 删除节点
                    Node delNode = prev.next;
                    prev.next = delNode.next;
                    delNode.next = null;
                    return;
                }
                prev = prev.next;
            }
        }
    }
 
    // 哈希函数
    private int hash(int key) {
        return key % N;
    }
 
    // 链表节点
    private class Node {
        int key;
        int value;
        Node next;
 
        Node(int key, int value) {
            this.key = key;
            this.value = value;
        }
    }
}
```



#### HashMap，HashTable，ConcurrentHashMap的区别。

Map线程不安全（没有用任何同步相关的原语），Table安全（直接加syn），Concurrent提供更高并发度的安全（分段锁思想orSyn+Cas）

hashmap在1.7前会头插死循环，但是在1.8改善后还是不能叫线程安全，因为没有可见性.**1.7前死锁，1.7后线程会获取脏值导致逻辑不可靠**

极高并发下HashTable和ConcurrentHashMap哪个性能更好?

- 极高并发读：并发读的情况下，Table也为读加了锁，没有并发可言，ConcurrentMap读锁并没有加并发，直接可读，若读resize的某个tab为空则转到新tab去读，Node的元素val和指针next都是volatile修饰的，可以保证可见性，所以concurrentMap获胜    
- 极高并发写：在并发写的情况下，table也是直接加了Syn做锁，强制串行，并且resize也只能单线程扩容，ConcurrentMap首先对于每个数组都有并发度，其次在resize的时候支持多线程协同，所以concurrentMap获胜

所以整体而言concurrentMap优势在于：

1. 读操作基于volatile可见性所以无锁
2. 写操作优势在于一是粗粒度的数组锁，二是协同resize

## 并发

#### 线程的基本状态以及状态之间的关系

其中Running表示运行状态，Runnable表示就绪状态（万事俱备，只欠CPU），之后变成Running；如果执行同步方法未获得锁，进入Blocked阻塞状态直到获得锁后进入Running状态；如果调用了wait()或者join()方法进入Waiting状态（等人叫）直到被notify()等唤醒；如果调用了wait(time)或者join(time)或者sleep(time)等方法进入TimeWaiting状态，时间到了自己叫醒自己。



#### 死锁与活锁与饥饿

死锁：是指两个或两个以上的进程（或线程）在执行过程中，因争夺资源而造成的一种互相等待的现象，**若无外力作用，它们都将无法推进下去**。

活锁：没有被阻塞，而是因为某些条件无法获得而一直不停尝试，可以自救。

饥饿：一个或者多个线程因为种种原因无法获得所需要的资源，导致一直无法执行的状态。

- 导致饥饿的可能原因：
  - 高优先级线程吞噬所有的低优先级线程的CPU时间
  - 线程被永久堵塞在一个等待进入同步块的状态，因为其他线程总是能在它之前持续地对该同步块进行访问。
  - 线程在等待一个本身也处于永久等待完成的对象(比如调用这个对象的wait方法)，因为其他线程总是被持续地获得唤醒。

产生死锁的条件：

1. 互斥条件
2. 请求和保持条件
3. 不剥夺条件
4. 环路等待条件

产生死锁的原因：

1. 竞争资源
2. 推进顺序非法
3. 

解决死锁的方法：

1. 资源一次性分配，摒弃“请求与保持”条件。要求进程必须-次性申请其运行所需的所有资源都满足之后才能分配资源给它。否则，不予分配资源。
2. 

#### sleep() 和 wait() 有什么区别

sleep是线程类Tread的方法，保持监控状态会自动恢复，不会释放锁。

wait是object基类的方法，会释放锁，需要notify

**wait用于锁机制，sleep不是，这就是为啥sleep不释放锁，wait释放锁的原因，sleep是线程的方法，跟锁没半毛钱关系，wait，notify,notifyall 都是Object对象的方法，是一起使用的，用于锁机制**



#### notify和notifyall的区别

Notify随机挑一个, 剩下的还在wait状态

NotifyAll唤醒全部一起争用, 大部分会处于blocked状态



#### 三个线程T1，T2，T3，怎么确保它们按顺序执行？

1. 可以用线程类的join()方法在一个线程中启动另一个线程，另外一个线程完成该线程继续执行。

> 在A线程中调用了B线程的join()方法时，表示只有当B线程执行完毕时，A线程才能继续执行。
>
> join方法的原理就是调用相应线程的wait方法进行等待操作的，例如A线程中调用了B线程的join方法，则相当于在A线程中调用了B线程的wait方法，当B线程执行完（或者到达等待时间），B线程会自动调用自身的notifyAll方法唤醒A线程，从而达到同步的目的。可以把ThreadB.join() ,理解为ThreadB.wait(),自然ThreadA就要被阻塞了。线程本即是对象！其实我们调用某个对象的wait方法时，当前线程会阻塞，原理大概如下：每个对象其实都有个队列管理竞争该对象的所有线程对象，线程阻塞（直接调用这个对象的wait方法或者请求该对象的锁）其实就是把对象加入到该队列；notify就是把阻塞队列中线程唤醒。
>

2. CountDownLatch, 主线程设置一个latch, 值为1, 启动A线程, 执行完毕再继续往下, B线程类似
3. Condition, 设置两个condition, A执行完释放B的condition, B执行完释放C的condition
4. FutureTask, A线程提交一个FutureTask, 然后在主线程阻塞等待返回结果再进行B线程
5. volatile, A执行完设置volatile为1, Bwhile读取volatile为1时进行逻辑操作, 执行完设为2, Cwhile读取2

#### Synchronized和lock 

Lock是一个接口，synchronized是Java的关键字，当它用来修饰一个方法或者一个代码块的时候，能够保证在同一时刻最多只有一个线程执行该段代码。

synchronized在发生异常时，会自动释放线程占有的锁，因此不会导致死锁现象发生；而Lock在发生异常时，如果没有主动通过unLock()去释放锁，则很可能造成死锁现象，因此**使用Lock时需要在finally块中释放锁**；Lock可以让等待锁的线程响应中断，而synchronized却不行，使用synchronized时，等待的线程会一直等待下去，不能够响应中断；通过Lock可以知道有没有成功获取锁，而synchronized却无法办到。

Lock是synchronized的扩展版，Lock提供了无条件的、可轮询的(tryLock方法)、定时的(tryLock带参方法)、可中断的(lockInterruptibly)、可多条件队列的(newCondition方法)锁操作。另外Lock的实现类基本都支持非公平锁(默认)和公平锁，synchronized只支持非公平锁，当然，在大部分情况下，非公平锁是高效的选择。



#### volatile

保证**有序性**和**可见性**，有序性实现的是通过**插入内存屏障**来保证



#### 自旋锁

当线程A想要获取一把自选锁而该锁又被其它线程锁持有时，线程A会在一个循环中自选以检测锁是不是已经可用了。自旋不释放CPU，在sleep前应该尽快释放。一个while就能实现自旋锁：

```java
public class MyWaitNotify3{

  MonitorObject myMonitorObject = new MonitorObject();
  boolean wasSignalled = false;

  public void doWait(){
    synchronized(myMonitorObject){
      while(!wasSignalled){
        try{
          myMonitorObject.wait();
         } catch(InterruptedException e){...}
      }
      //clear signal and continue running.
      wasSignalled = false;
    }
  }

  public void doNotify(){
    synchronized(myMonitorObject){
      wasSignalled = true;
      myMonitorObject.notify();
    }
  }
}
```



#### 公平锁与非公平锁

**公平锁的优点是等待锁的线程不会夯死。缺点是吞吐效率相对非公平锁要低，等待队列中除第一个线程以外的所有线程都会阻塞，CPU唤醒阻塞线程的开销比非公平锁大。**

**非公平锁的优点是可以减少唤起线程的开销（因为可能有的线程可以直接获取到锁，CPU也就不用唤醒它），所以整体的吞吐效率高。缺点是处于等待队列中的线程可能会夯死（试想恰好每次有新线程来，它恰巧都每次获取到锁，此时还在排队等待获取锁的线程就悲剧了**），或者等很久才会获得锁。

公平锁和非公平锁的差异在于是否按照申请锁的顺序来获取锁，非公平锁可能会出现有多个线程等待时，有一个人品特别的好的线程直接没有等待而直接获取到了锁的情况，他们各有利弊;ReetrantLock在构造时默认是非公平的，可以通过参数控制。

#### AQS

AQS内部有一个由**Node组成的同步队列**，它是**双向链表**结构

AQS内部通过state来控制同步状态,当执行lock时，如果state=0时，说明没有任何线程占有共享资源的锁，此时线程会获取到锁并把state设置为1；当state=1时，则说明有线程目前正在使用共享变量，其他线程必须加入同步队列进行等待.

AQS是内部通过内部类Node构成FIFO的同步队列实现线程获取锁排队，同时利用内部类ConditionObject构建条件队列，当调用condition.wait()方法后，线程将会加入条件队列中，而当调用signal()方法后，线程将从条件队列移动到同步队列中进行锁竞争。注意这里涉及到两种队列，一种的**同步队列**，当锁资源已经被占用，而又有线程请求锁而等待的后将加入同步队列等待，而另一种则是**条件队列**(可有多个)，通过Condition调用await()方法释放锁后，将加入等待队列。

而在AQS同步队列中，只要当前节点的前驱节点不是头结点，再把当前节点加到队列后就会执行LockSupport.park(this);将当前线程挂起，这样可以最大程度减少CPU消耗。

[死磕java concurrent包系列（二）基于ReentrantLock理解AQS同步队列的细节和设计模式](https://juejin.im/post/5c021b59f265da6175737f0b)

**ReetrantLock与AQS**

ReetrantLock:实现了lock接口，它的**内部类有Sync、NonfairSync、FairSync（他们三个是继承了AQS）**，创建构造ReetrantLock时可以指定是**非公平锁(NonfairSync)**还是**公平锁(FairSync)**

Sync:他是抽象类，也是ReetrantLock内部类，实现了tryRelease方法，tryAccquire方法由它的子类NonfairSync、FairSync自己实现。

AQS：它是一个抽象类，但是值得注意的是它代码中却没有一个抽象方法，其中获取锁(tryAcquire方法)和释放锁(tryRelease方法)也没有提供默认实现，需要子类重写这两个方法实现具体逻辑（典型的模板方法设计模式）。

Node：AQS的内部类，本质上是一个双向链表，用来管理获取锁的线程（后续详细解读）

**这样设计的结构有几点好处：**

1. 首先为什么要有Sync这个内部类呢？

 因为无论是NonfairSync还是FairSync，他们解锁的过程是一样的，不同只是加锁的过程，Sync提供加锁的模板方法让子类自行实现

2. AQS为什么要声明为Abstract，内部却没有任何abstract方法？

这是因为AQS只是作为一个基础组件，从上图可以看出countDownLatch,Semaphore等并发组件都依赖了它，它并不希望直接作为直接操作类对外输出，而更倾向于作为一个基础并发组件，为真正的实现类提供基础设施，例如构建同步队列，控制同步状态等。

AQS是采用模板方法的设计模式，它作为基础组并发件，封装了一层核心并发操作（比如获取资源成功后封装成Node加入队列，对队列双向链表的处理），但是实现上分为两种模式，即**共享模式(如Semaphore)与独占模式（如ReetrantLock，这两个模式的本质区别在于多个线程能不能共享一把锁）**，而这两种模式的加锁与解锁实现方式是不一样的，但AQS只关注内部公共方法实现并不关心外部不同模式的实现，所以提供了模板方法给子类使用：例如：

ReentrantLock需要自己实现tryAcquire()方法和tryRelease()方法，而实现共享模式的Semaphore，则需要实现tryAcquireShared()方法和tryReleaseShared()方法，这样做的好处？因为无论是共享模式还是独占模式，其基础的实现都是同一套组件(AQS)，只不过是加锁解锁的逻辑不同罢了，更重要的是如果我们需要自定义锁的话，也变得非常简单，只需要选择不同的模式实现不同的加锁和解锁的模板方法即可。



#### CycliBarriar和CountdownLatch有什么区别？

CountDownLatch其实可以把它看作一个计数器，只不过这个计数器的操作是原子操作，同时只能有一个线程去减这个计数器里面的值。任何调用这个对象上的await()方法都会阻塞，直到这个计数器的计数值被其他的线程减为0为止。在当前计数到达零之前，await 方法会一直受阻塞。之后，会释放所有等待的线程，await的所有后续调用都将立即返回。这种现象只出现一次——计数无法被重置。如果需要重置计数，请考虑使用 CyclicBarrier。

CountDownLatch的一个非常典型的应用场景是：有一个任务想要往下执行，但必须要等到其他的任务执行完毕后才可以继续往下执行。假如我们这个想要继续往下执行的任务调用一个CountDownLatch对象的await()方法，其他的任务执行完自己的任务后调用同一个CountDownLatch对象上的countDown()方法，这个调用await()方法的任务将一直阻塞等待，直到这个CountDownLatch对象的计数值减到0为止。

CyclicBarrier一个同步辅助类，它允许一组线程互相等待，直到到达某个公共屏障点 (common barrier point)。在涉及一组固定大小的线程的程序中，这些线程必须不时地互相等待，此时 CyclicBarrier 很有用。因为该 barrier 在释放等待线程后可以重用，所以称它为循环 的 barrier。

#### ArrayBlockingQueue

ArrayBlockingQueue通过一个ReentrantLock和两个条件队列构成，使用**一个ReentrantLock来同时控制添加线程与移除线程的并发访问**，通过数组对象items来存储所有的数据，元素存在公平访问与非公平访问的区别。

三个添加方法即put，offer，add，其中offer，add在正常情况下都是无阻塞的添加，而put方法是阻塞添加。这就是阻塞队列的添加过程。说白了就是**当队列满时通过条件对象Condtion来阻塞当前调用put方法的线程**，直到线程又再次被唤醒执行。由于移除和添加方法使用的是一个lLOCK锁对象，所以添加与移除是互斥，也就是说**不能同时添加与删除**。

ArrayBlockingQueue内部通过一把锁ReentrantLock和两个AQS条件队列实现了阻塞的入队和删除：

- 元素满时，阻塞put线程，封装为node节点在notFull条件队列中，此时如果有线程移出元素，在移出后会唤醒notFull条件队列，让条件队列中的put线程继续尝试进行put
- 元素空时，阻塞take线程，封装为node节点在notEmpty条件队列中，此时如果有线程加入元素，在移出后会唤醒notEmpty条件队列，让条件队列中的take线程继续尝试进行take

#### LinkedBlockingQueue

LinkedBlockingQueue队列中的数据都将被封装成Node节点,单项列表，拥有头尾指针，使用了takeLock 和 putLock 对并发进行控制，也就是说，**添加和删除操作并不是互斥操作**，拥有更大的**吞吐量**，**添加线程在队列没有满时自己直接唤醒自己的其他添加线程，如果没有等待的添加线程，直接结束了。如果有就直到队列元素已满才结束挂起。注意消费线程的执行过程也是如此。这也是为什么LinkedBlockingQueue的吞吐量要相对大些的原因。**

[死磕java concurrent包系列（五）基于AQS的条件队列把LinkedBlockingQueue“扒光”](https://juejin.im/post/5c0fcb35e51d453b304796c7)

#### 进程间如何通信

系统级别：

1. 共享存储
2. 消息传递
3. 管道通信
4. 信号量

java级别线程通信：

1. 锁机制
2. Object类的wait() 和 notify() 方法
3. 使用JUC工具类 CountDownLatch等
4. 关键字synchronize，volatile



#### Callable和Future

Future就是Callable返回的结果，底层是**用一个volatile的变量标志是否已经结束**来让调用者知道任务执行状况，线程池+SynList+Future可以获取一组任务的执行情况



## JVM

#### CMS与G1

CMS流程：  

1. ​    初始标记(stw)：获得老年代中跟GCRoot以及新生代关联的对象，将其标记为root    
2. ​    并发标记：将root标记的对象所关联的对象进行标记    
3. ​    重标记：在并发标记阶段，并没有stw，所以会有一些脏对象产生，即标记完毕之后又产生关联对象修改    
4. ​    最终标记(stw)：最终确定所有没有脏对象的存活对象    
5. ​    并发清理：并发的清理所有死亡对象    
6. ​    Reset：重设程序为下一次FGC做准备

CMS优劣：  

- ​    优点：    
  - ​      不像PN以及Serial一样全程需要stw，只需要在两个标记阶段stw即可      
  - ​      并发标记、清楚来提升效率，减少stw的时间和整体gc时间      
  - ​      在最终标记前通过预设次数的**重标记来清理脏页减少stw时间**      
- ​    缺点：    
  - ​      仍然存在stw      
  - ​      基于标记清楚算法的GC，节省算力但是**会产生内存碎片**      
  - ​      并发标记和清楚会造成cpu的高负担



## Redis

#### Redis的主从复制

#### redis与Memcached的区别

- redis除了k-v类型，还支持list、hash与set等数据结构的存储
- redis支持主备和集群
- redis支持持久化
- Memcache 在并发场景下，用cas保证一致性，redis事务支持比较弱，只能保证事务中的每个操作连续执行

在Redis中，并不是所有的数据都一直存储在内存中的。这是和Memcached相比一个最大的区别（我个人是这么认为的）。

Redis 只会缓存所有的key的信息，如果Redis发现内存的使用量超过了某一个阀值，将触发swap的操作，Redis根据“swappability = age*log(size_in_memory)”计算出哪些key对应的value需要swap到磁盘。然后再将这些key对应的value持久化到磁 盘中，同时在内存中清除。这种特性使得Redis可以保持超过其机器本身内存大小的数据。当然，机器本身的内存必须要能够保持所有的key，毕竟这些数据 是不会进行swap操作的。

同时由于Redis将内存中的数据swap到磁盘中的时候，提供服务的主线程和进行swap操作的子线程会共享这部分内存，所以如果更新需要swap的数据，Redis将阻塞这个操作，直到子线程完成swap操作后才可以进行修改。

当从Redis中读取数据的时候，如果读取的key对应的value不在内存中，那么Redis就需要从swap文件中加载相应数据，然后再返回给请求方。 这里就存在一个I/O线程池的问题。在默认的情况下，Redis会出现阻塞，即完成所有的swap文件加载后才会相应。这种策略在客户端的数量较小，进行 批量操作的时候比较合适。但是如果将Redis应用在一个大型的网站应用程序中，这显然是无法满足大并发的情况的。所以Redis运行我们设置I/O线程 池的大小，对需要从swap文件中加载相应数据的读取请求进行并发操作，减少阻塞的时间。



#### Zset（有序集合）实现原理

元素数量小于128且元素长度小于64字节使用ziplist压缩列表编码，否则用skiplist跳跃表编码。

相比平衡树优点：

- 查找单个key，skiplist和平衡树的时间复杂度都为O(log n)；
- 平衡树的插入和删除操作可能引发子树的调整，逻辑复杂，而skiplist的插入和删除只需要修改相邻节点的指针，操作简单又快速；
- 一般来说，平衡树每个节点包含2个指针（分别指向左右子树），而skiplist每个节点包含的指针数目平均为1/(1-p)

为了支持排名(rank)，Redis里对skiplist做了扩展，使得根据排名能够快速查到数据，或者根据分数查到数据之后，也同时很容易获得排名。而且，根据排名的查找，时间复杂度也为O(log n)。

- zrevrank是先在dict中由数据查到分数，再拿分数到skiplist中去查找，查到后也同时获得了排名。
- 分数(score)允许重复，即skiplist的key允许重复。
- 第1层链表不是一个单向链表，而是一个双向链表。这是为了方便以倒序方式获取一个范围内的元素。
- 在比较时，不仅比较分数（相当于skiplist的key），还比较数据本身。在Redis的skiplist实现中，数据本身的内容唯一标识这份数据，而不是由key来唯一标识。另外，当多个元素分数相同的时候，还需要根据数据内容来进字典排序。



#### redis 分布式锁的实现原理 setNX 

通过setnx(lock_timeout)实现，如果设置了锁返回1， 已经有值没有设置成功返回0。setnx 是『SET if Not eXists』(如果不存在，则 SET)的简写，命令格式：SETNX key value，命令在设置成功时返回 1 ，设置失败时返回 0 。

为了防止死锁，设置了分布式锁之后一般要设置锁的过期时间。由于redis使用惰性和定期清除的策略，如果返回值是0但是要额外加一个步骤获取锁的过期时间与当前时间相比，看看有没有过期。

问题：如果设置了锁之后网断了没设置过期时间，会出现死锁；并发的时候可能出现过期时间被其他客户端覆盖，还要求客户端计算的时间都是同步；且锁没有客户端标识，如果一个客户端在锁过期时间内没执行完，可能会被其他客户端del掉锁，这个可以通过设置锁的时候把客户端id设置为value来实现。

redis2.6后的实现方式：

- set方法可以在设置锁的同时设置过期时间



#### redis实现异步队列

一般使用 list 结构作为队列，rpush 生产消息，lpop 消费消息。当 lpop 没有消息的时候，要适当 sleep 一会再重试。如果对方追问可不可以不用 sleep 呢？list 还有个指令叫 blpop，在没有消息的时候，它会阻塞住直到消息到来。如果对方追问能不能生产一次消费多次呢？使用 pub/sub 主题订阅者模式，可以实现1:N 的消息队列。

**如果对方追问 pub/sub 有什么缺点？**

在消费者下线的情况下，生产的消息会丢失，得使用专业的消息队列如 RabbitMQ等。

**如果对方追问 redis 如何实现延时队列？**

我估计现在你很想把面试官一棒打死如果你手上有一根棒球棍的话，怎么问的这么详细。但是你很克制，然后神态自若的回答道：使用 sortedset，拿时间戳作为score，消息内容作为 key 调用 zadd 来生产消息，消费者用 zrangebyscore 指令获取 N 秒之前的数据轮询进行处理。到这里，面试官暗地里已经对你竖起了大拇指。但是他不知道的是此刻你却竖起了中指，在椅子背后。



#### 假如 Redis 里面有 1 亿个 key，其中有 10w 个 key 是以某个固定的已知的前缀开头的，如果将它们全部找出来？

使用 keys 指令可以扫出指定模式的 key 列表。但是由于redis是单线程，keys指令会导致线程阻塞，线上可以使用 scan 指令，**scan 指令可以无阻塞的提取出指定模式的 key 列表**，但是会有一定的重复概率，在客户端做一次去重就可以了，但是整体所花费的时间会比直接用 keys 指令长。



#### redis适用的场景

1. 会话缓存（Session）
2. 全页缓存
3. 队列
4. 排行榜/计数器
5. 发布/订阅



#### 一致性hash算法

将整个hash的输出空间当成一个环，环中设立多个节点，每个节点有值，当对象的映射满足上个节点和这个节点中间值的时候它就落到这个节点当中来，**热点采用虚拟节点处理**。

应用：redis缓存，好处是平滑的数据迁移和快速的rebalance

redis中**客户端寻址使用到了一致性hash算法**，



#### 缓存和DB之间怎么保证数据一致性

缓存预留模式
读操作：先读缓存，缓存没有的话读DB，然后取出数据放入缓存，最后响应数据
写操作：先更新DB，再删除缓存
为什么是删除而不是更新呢？
原因很简单，复杂场景下缓存不单单是DB中直接取出来的值，此外更新缓存的代价是很高的，频繁更新的缓存到底会不会被频繁访问到？可能更新到缓存里面的数据都是冷数据，频繁失效，所以一般用到再去加载缓存，lazy加载的思想

先更新DB，再删除缓存的问题，如果更新DB成功，删除缓存失败会导致数据不一致
所以一般是先删除缓存，再更新DB

还是有问题，A先删除了缓存，但还没更新DB，这时B过来请求数据，发现缓存没有，去请求DB拿到旧数据，然后再写到缓存，等A更新完了DB之后就会出现缓存和DB数据不一致的情况了

解决方案：
更新数据时，根据数据的唯一标识路由到队列中，读取数据时，如果发现数据不再缓存中，那么把读取数据+更新缓存的操作，根据唯一标识路由之后，也发送到相应队列中。一个队列对应一个工作线程，线程串行拿到队列里的操作一一执行

带来的新问题：
可能数据更新频繁，导致队列中积压了大量的更新操作，读请求长时间阻塞，所以要压测



## Mysql

#### 数据库三范式

1. 字段不可分
2. 有主键，非主键字段依赖唯一主键
3. 非主键字段不能相互依赖



#### MyISAM与InnoDB区别

MyISAM：

- 表级锁
- 不支持事务
- 不支持全文索引

InnoDB：

- 行级锁

- ACID事务安全

- 支持外键、FULLTEXT索引

  

#### 索引类型

数据结构角度有：

- B+树
- hash索引
- R-Tree索引
- FULLTEXT索引

物理存储角度有:

- 聚集索引（主键索引）：叶子节点存储的是行数据

- 非聚集索引（二级索引）：叶子节点保存的不是指行的物理位置的指针，而是行的主键值

  - 通过二级索引查找行，**需要两次B-Tree查找**而不是一次：1、找到二级索引的叶子节点获取对应的主键值，2、根据这个主键值去聚簇索引中查找到对应的行。

- > - **聚簇索引的叶子节点就是数据节点，而非聚簇索引的叶子节点仍然是索引节点**，只不过有指向对应数据块的指针。
  > - 一个表只有一个聚簇索引，可以有多个二级索引，MyISAM不支持聚簇索引

逻辑角度：

- 主键索引
- 普通索引
- 复合索引

#### 索引优化

设计：

- **索引字段尽量使用数字型**（简单的数据类型）

  - 若只含数值信息的字段尽量不要设计为字符型，这会降低查询和连接的性能，并会增加存储开销。这是因为引擎在处理查询和连接时会逐个比较字符串中每一个字符，而对于数字型而言只需要比较一次就够了

- **尽量不要让字段的默认值为NULL**

  - 索引不会包含有NULL值的列，只要列中包含有NULL值都将不会被包含在索引中，复合索引中只要有一列含有NULL值，那么这一列对于此复合索引就是无效的。

    所以我们在数据库设计时尽量不要让字段的默认值为NULL，应该指定列为NOT NULL，除非你想存储NULL。你应该用0、一个特殊的值或者一个空串代替空值。

- **前缀索引和索引选择性**

  - 对串列进行索引，如果可能应该指定一个前缀长度，对于BLOB、TEXT或者很长的VARCHAR类型的列，必须使用前缀索引，因为MYSQL不允许索引这些列的完整长度。**短索引不仅可以提高查询速度而且可以节省磁盘空间和I/O操作**。

- **使用唯一索引**

- **使用组合索引代替多个列索引**

使用：

- **like语句不要以通配符%开头**
  - 对于LIKE：在以通配符%和_开头作查询时，MySQL不会使用索引
- **不要在列上进行运算**
  - 索引列不能是表达式的一部分，也不是是函数的参数。`select actor_id from sakila.actor where actor_id+1=5`
- **尽量不要使用NOT IN、<>、!= 操作**
  - 这样会放弃使用索引而进行全表扫描。
- **or条件**
  - 用 or 分割开的条件， 如果 or 前的条件中的列有索引， 而后面的列中没有索引， 那么涉及到的索引都不会被用到。应尽量避免在 where 子句中使用 or 来连接条件，否则将导致引擎放弃使用索引而进行全表扫描，
  -  假设num1有索引，num2没有索引，查询语句select id from t where num1=10 or num2=20会放弃使用索引，可以改为这样查询： select id from t where num1=10 union all select id from t where num2=20，这样虽然num2没有使用索引，但至少num1会使用索引，提高效率。
- **组合索引的使用要遵守“最左前缀”原则**
- **ORDER BY也要遵守“最左前缀”原则**

 

#### 为什么要用B+树也不是hash或者红黑树

1. MySQL中存储索引用到的数据结构是B+树，B+树的查询时间跟树的高度有关，是log(n)，如果用hash存储，那么查询时间是O(1)。**既然hash比B+树更快，为什么mysql用B+树来存储索引呢**？
   - 从内存角度上说，数据库中的索引一般时在磁盘上，数据量大的情况可能无法一次性装入内存，B+树的设计可以允许数据分批加载。
   - 从业务场景上说，如果只选择一个数据那确实是hash更快，但是数据库中经常会选中多条这时候由于B+树索引有序，并且又有链表相连，它的查询效率比hash就快很多了。

2. **为什么不用红黑树或者二叉排序树**？

   - 树的查询时间跟树的高度有关，B+树是一棵多路搜索树可以降低树的高度，提高查找效率

   - AVL 数和红黑树基本都是存储在内存中才会使用的数据结构，那磁盘中会有什么不同呢？

     这就要牵扯到磁盘的存储原理了

     操作系统读写磁盘的基本单位是扇区，而文件系统的基本单位是簇(Cluster)。

     也就是说，磁盘读写有一个最少内容的限制，即使我们只需要这个簇上的一个字节的内容，我们也要含着泪把一整个簇上的内容读完。

     那么，现在问题就来了

     一个父节点只有 2 个子节点，并不能填满一个簇上的所有内容啊？那多余的内容岂不是要浪费了？我们怎么才能把浪费的这部分内容利用起来呢？哈哈，答案就是 B+ 树。

     由于 B+ 树分支比二叉树更多，所以相同数量的内容，B+ 树的深度更浅，深度代表什么？代表磁盘 io 次数啊！数据库设计的时候 B+ 树有多少个分支都是按照磁盘一个簇上最多能放多少节点设计的啊！

     所以，涉及到磁盘上查询的数据结构，一般都用 B+ 树啦。

3. 既然增加树的路数可以降低树的高度，那么**无限增加树的路数是不是可以有最优的查找效率**？

   - 这样**会形成一个有序数组**，文件系统和数据库的索引都是存在硬盘上的，并且如果数据量大的话，不一定能一次性加载到内存中。有序数组没法一次性加载进内存，这时候B+树的多路存储威力就出来了，可以每次加载B+树的一个结点，然后一步步往下找，



#### 事务四大特性

ACID，原子性、一致性、隔离性、持久性

#### 事务可能出现的三种问题

- 脏读
  - 读到了其他事务还没提交的操作（还处于缓冲中的操作命令）
- 幻读
  - 幻读和不可重复读都是读取了另一条已经提交的事务（这点就脏读不同），所不同的是**不可重复读查询的都是同一个数据项，而幻读针对的是一批数据整体**（比如数据的个数）
- 不可重复读
  - 与脏读的不同是**读到了其他事务已经提交的数据**，可接受

#### 事务的四种隔离级别

1. Serializable (串行化)：可避免脏读、不可重复读、幻读的发生。
2. Repeatable read (可重复读)：可避免脏读、不可重复读的发生。
3. Read committed (读已提交)：可避免脏读的发生。
4. Read uncommitted (读未提交)：最低级别，任何情况都无法保证。

**MySQL数据库中默认的隔离级别为Repeatable read (可重复读)。**

#### 事务实现的原理

事务四大特性实现的核心原理是三个日志文件：

- `undo log`
  - 回滚和多版本控制(MVCC)
  - 在数据修改的时候，不仅记录了`redo log`，还记录`undo log`，如果因为某些原因导致事务失败或回滚了，可以用`undo log`进行回滚
  - `undo log`主要存储的也是逻辑日志，记录一条对应**相反**的update记录
  - 因为`undo log`存储着修改之前的数据，相当于一个**前版本**，MVCC实现的是读写不阻塞，读的时候只要返回前一个版本的数据就行了。
- `binlog`
  - 记录的是会导致表变化的sql语句这种逻辑日志，主要作用是**复制和恢复数据**：
  - MySQL在公司使用的时候往往都是**一主多从**结构的，从服务器需要与主服务器的数据保持一致，这就是通过`binlog`来实现的
  - 数据库的数据被干掉了，我们可以通过`binlog`来对数据进行恢复。
- `redo log`
  - **记录的是硬盘页中做的物理变动操作**，用作缓冲，也是为了防止断电后内存中数据未写入磁盘导致数据丢失。
  - 当做数据修改的时候，不仅在内存中操作，还会在redo log中记录这次操作。当事务提交的时候，会将redo log日志进行刷盘(redo log一部分在内存中，一部分在磁盘上)。当数据库宕机重启的时候，会将redo log中的内容恢复到数据库中，再根据undo log和binlog内容决定回滚数据还是提交数据。
  - 当我们修改的时候，写完内存了，但数据还没真正写到磁盘的时候。此时我们的数据库挂了，我们可以根据`redo log`来对数据进行恢复。因为`redo log`是顺序IO，所以**写入的速度很快**，并且`redo log`记载的是物理变化（xxxx页做了xxx修改），文件的体积很小，**恢复速度很快**。

binlog与redo log的区别：

- binlog记载的是sql语句，redo log记载的是物理修改内容
- redo log作用是持久化，数据库挂了通过redo来恢复未刷到硬盘的数据，binlog作用是为了复制（主从服务器）和恢复（整个数据库数据都被删除了通过binlog恢复（因为刷到磁盘的历史数据会从redo删掉，所以无法通过redos恢复历史数据））
- binlog是所有引擎共有，redos是InnoDB独有的

binlog与redo log的写入顺序：

- `redo log`**事务开始**的时候，就开始记录每次的变更信息，而`binlog`是在**事务提交**的时候才记录。
- 如果写`redo log`失败了，那我们就认为这次事务有问题，回滚，不再写`binlog`。
- 如果写`redo log`成功了，写`binlog`，写`binlog`写一半了，但失败了怎么办？我们还是会对这次的**事务回滚**，将无效的`binlog`给删除（因为`binlog`会影响从库的数据，所以需要做删除操作）
- 如果写`redo log`和`binlog`都成功了，那这次算是事务才会真正成功。

MySQL通过**两阶段提交**来保证`redo log`和`binlog`的数据一致：

- 阶段1：InnoDB`redo log` 写盘，InnoDB 事务进入 `prepare` 状态
- 阶段2：`binlog` 写盘，InooDB 事务进入 `commit` 状态
- 每个事务`binlog`的末尾，会记录一个 `XID event`，标志着事务是否提交成功，也就是说，恢复过程中，`binlog` 最后一个 XID event 之后的内容都应该被 purge。

