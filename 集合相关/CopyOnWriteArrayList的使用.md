# 什么是CopyOnWrite容器  
CopyOnWrite容器即**写时复制**的容器。通俗的理解是当我们往一个容器添加元素的时候，不直接往当前容器添加，而是先`将当前容器进行Copy，复制出一个新的容器，然后新的容器里添加元素，添加完元素之后，再将原容器的引用指向新的容器`。这样做的好处是我们可以对CopyOnWrite容器进行并发的读，而不需要加锁，因为当前容器不会添加任何元素。所以CopyOnWrite容器也是一种读写分离的思想，读和写不同的容器。
## CopyOnWriteArrayList的实现原理
```java
public boolean add(T e) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {

        Object[] elements = getArray();

        int len = elements.length;
        // 复制出新数组

        Object[] newElements = Arrays.copyOf(elements, len + 1);
        // 把新元素添加到新数组里

        newElements[len] = e;
        // 把原数组引用指向新数组

        setArray(newElements);

        return true;

    } finally {

        lock.unlock();

    }

}

final void setArray(Object[] a) {
    array = a;
}
```
这里提一下`Arrays.copyOf(T t,int length)`这个方法,该方法只会构造一个新的数组,然后拷贝原数组中每一个元素的引用,换句话说`原数组和新数组中的引用指向的是同一个对象`,请看以下测试:
```java
public class Test {
	String name;
	public Test(String name) {
		super();
		this.name = name;
	}
	public static void main(String[] args) {
		Object[] objects={new Test("张三")};
		Object[] newObjects=Arrays.copyOf(objects,objects.length+1);
		System.out.println(objects[0]==newObjects[0]);//true,说明内存中指向同一个对象
	}
}


```
读的时候不需要加锁，如果读的时候有多个线程正在向ArrayList添加数据，读还是会读到旧的数据，因为写的时候不会锁住旧的ArrayList
```java
public E get(int index) {
    return get(getArray(), index);
}
```
其实这是一种读写分离的设计思想,线程并发读与并发写的使用使用的不是同一个数组,提升了并发访问的效率.
## CopyOnWrite的缺点
CopyOnWrite容器有很多优点，但是同时也存在两个问题，即内存占用问题和数据一致性问题。所以在开发的时候需要注意一下。
 - **内存占用问题**:虽然容器中的对象不会被拷贝两份,但在修改容器的时候内存中还是会有两个容器,如果数据比较多的时候内存占用就会比较大了.
 - **数据一致性问题**:CopyOnWrite容器只能保证数据的最终一致性，不能保证数据的实时一致性。所以如果你希望写入的的数据，马上能读到，请不要使用CopyOnWrite容器

## 使用场景总结  
CopyOnWriteArrayList适合在多线程访问时读取操作远大于写入操作的场景中,否则都应该使用其它方案代替它.
