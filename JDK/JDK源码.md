---
typora-root-url: pic
typora-copy-images-to: pic
---

## Map

### HashMap

在java中，所有的数据结构都是由两种基础结构构成的：数组和链表。**HashMap是由一个数组以及其中的链表元素构成的**。

<img src="/HashMap结构图.png" alt="image-20200626004134012" style="zoom:50%;" />

```java
/** 
     * The table, resized as necessary. Length MUST Always be a power of two. 
     *  FIXME 这里需要注意这句话，至于原因后面会讲到 
     */  
    transient Entry[] table;
```

```java
static class Entry<K,V> implements Map.Entry<K,V> {  
        final K key;  
        V value;  
        final int hash;  
        Entry<K,V> next;  
..........  
}  
```

HashMap的get方法获取一个元素原理：

1. 计算key的hashcode，从数组中找到
2. 对应的数组元素
3. 通过key的equals方法在对应链表中找到具体链表中的链表元素

可以知道hashcode与equal方法是查找的核心。equals可以是用户根据自身需求复写；hashcode的计算原理如下（也就是hash算法）：

> hash算法希望所有对象尽可能均匀分布在所有数组的元素中，一种方法是直接用hashcode对数组长度取模（类似除然后取余），不过取模操作实现较慢。java中采用hashcode与数组长度-1进行“**与**”运算（&）的办法来计算hash值。
>
> **hashmap中数组的长度默认为16**，为了让对象尽可能平均分配到所有数组元素中**需要设定为2的n次方**，这是因为如果是2的n次方长度在进行与计算的时候2的n次方-1在二进制中表示就是全为1，这样才能充分发挥与计算的能力。如果不是例如长度为15，那么长度-1为14,14的二进制表示最后一位是0，与计算后始终有几个数值无法计算出来，从而导致分配不平均。

**rehash**

> java中设置当hashmap中元素超过数组总长度的0.75后就进行rehash，结合上面讲的数组长度需要设置为2的n次幂，所以每次rehahs至少数组长度*2.