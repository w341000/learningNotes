## HashMap概述  
Java中的数据存储方式有两种结构，一种是数组，另一种就是链表，前者的特点是连续空间，寻址迅速，但是在增删元素的时候会有较大幅度的移动，所以数组的特点是**查询速度快，增删较慢**。  
而链表由于空间不连续，寻址困难，增删元素只需修改指针，所以链表的特点是**查询速度慢、增删快**。  
那么有没有一种数据结构来综合一下数组和链表以便发挥他们各自的优势？答案就是哈希表。哈希表的存储结构如下图所示：  
![hash表](/images/hash表.jpg)   
从上图中，我们可以发现哈希表是由**数组+链表**组成的，一个长度为16的数组中，每个元素存储的是一个链表的头结点，通过类似`h & (length-1)`的操作,得到要添加的元素所要存放的的**数组位置**。
每一个用来存放链表的数组都称之为`bucket`或者叫`桶`.  
键值对所存放的数据结构其实是HashMap中定义的一个Entity内部类，数组来实现的，属性有key、value和指向下一个Entity的next。   
**构造方法:**  
```java
public HashMap(int initialCapacity, float loadFactor) {    
        if (initialCapacity < 0)    
            throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);    
        if (initialCapacity > MAXIMUM_CAPACITY)    
            initialCapacity = MAXIMUM_CAPACITY;    
        if (loadFactor <= 0 || Float.isNaN(loadFactor))    
            throw new IllegalArgumentException("Illegal load factor: " + loadFactor);    
        // Find a power of 2 >= initialCapacity    
        int capacity = 1;    //可以看到初始容量最小为1
        while (capacity < initialCapacity)    
            capacity <<= 1;    

        this.loadFactor = loadFactor;    
        threshold = (int)(capacity * loadFactor);    
        table = new Entry[capacity];    
        init();    
}
```
提一下HashMap中的两个参数`initialCapacity`初始容量及`loadFactor`加载因子.
hash表的数组实际大小是2的倍数但会>=initialCapacity, 加载因子是表示Hsah表中元素的填满的程度.若:加载因子越大,填满的元素越多,好处是,空间利用率高了,但:冲突的机会加大了.反之,加载因子越小,填满的元素越少,好处是:冲突的机会减小了,但:空间浪费多了.  
**放入元素的操作:**
```java
public V put(K key, V value) {
       if (key == null)
           return putForNullKey(value);//可以放null的
       int hash = hash(key);
       int i = indexFor(hash, table.length);//查找key在数组中的索引位置
       for (Entry<K,V> e = table[i]; e != null; e = e.next) {//遍历该位置所有链表
           Object k;
           if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {//如果发现key相同则替换value
               V oldValue = e.value;
               e.value = value;
               e.recordAccess(this);
               return oldValue;
           }
       }

       modCount++;//记录修改次数,以便在遍历的时候发现修改抛出异常
       addEntry(hash, key, value, i);
       return null;
   }
```  
`putForNullKey(value)`说明HashMap可以存放空的key
再看看`addEntry(hash, key, value, i)`这个方法:
```java
void addEntry(int hash, K key, V value, int bucketIndex) {
       if ((size >= threshold) && (null != table[bucketIndex])) {//threshold为加载因子对应的临界值,hash表中存放的元素超过临界值则扩容为原来数组的2倍大
           resize(2 * table.length);
           hash = (null != key) ? hash(key) : 0;
           bucketIndex = indexFor(hash, table.length);
       }

       createEntry(hash, key, value, bucketIndex);//进行构建entry的操作
   }
```  
再看看`createEntry(hash, key, value, bucketIndex)`这个方法:  
```java
void createEntry(int hash, K key, V value, int bucketIndex) {
       Entry<K,V> e = table[bucketIndex];
       table[bucketIndex] = new Entry<>(hash, key, value, e);//构建一个entry并且将原来指向旧链表的引用改为指向新链表
       size++;
   }
```  
由此可以看到HashMap的添加元素的操作过程.  
**接下来看看获取元素:**  
```java
public V get(Object key) {
        if (key == null)
            return getForNullKey();//可以直接返回null对应的value
        Entry<K,V> entry = getEntry(key);

        return null == entry ? null : entry.getValue();
    }
```
关键的`getEntry(key)`代码如下:
```java
final Entry<K,V> getEntry(Object key) {
        int hash = (key == null) ? 0 : hash(key);//获得key对应的hash
        for (Entry<K,V> e = table[indexFor(hash, table.length)];//根据hash获取在数组中的索引,并且遍历对应链表看看有没有包含该key
             e != null;
             e = e.next) {
            Object k;
            if (e.hash == hash &&
                ((k = e.key) == key || (key != null && key.equals(k))))//有则返回对应的entry
                return e;
        }
        return null;//否则返回null
    }
```  
通过上面的源代码分析,可以看到HashMap对元素的存放非常依赖key的hashcode和equals方法.  
## HashTable的实现
HashMap与HashTable的区别非常小,HashTable在存放key或者value为null的元素时直接抛出异常:
```
if (value == null) {
            throw new NullPointerException();
        }
```  
并且HashTable在每个方法上都加上Synchronize关键字,所以性能比较差:
```java
 public synchronized V put(K key, V value) {...}
 public synchronized V get(Object key) {...}

```  
### HashMap与HashTable的主要区别
综上所述:  
1. HashMap是非线程安全的，HashTable是线程安全的，内部的方法基本都经过synchronized修饰。
2. 因为同步、哈希性能等原因，性能肯定是HashMap更佳，因此HashTable已被淘汰。
3. HashMap允许有null值的存在，而在HashTable中put进的键值只要有一个null，直接抛出NullPointerException。
