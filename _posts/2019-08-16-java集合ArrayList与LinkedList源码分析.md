---
layout: post
title: "java集合ArrayList与LinkedList源码分析"
date: 2019-08-16
tags: java
---

#Array源码分析
#####首先分析new ArrayList<>()
```
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
transient Object[] elementData;
public ArrayList() {
        // 初始化数组的容量为{}
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
```
new ArrayList()时，会创建一个Object[] elementData = {} 的数组。由此可以知道list底层是通过Object[]数量来存储数量的

#####分析通过无参构造函数初始化ArrayList时第一次调用add()方法
```
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
transient Object[] elementData;  // 存储list数据的数组
private int size;  //当前arrayList的size(集合容量)
private static final int DEFAULT_CAPACITY = 10;

public boolean add(E e) {
        // 判断容器是否够大，如果超出容器大小需要进行扩容
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }

private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }
// 计算容器需要多大的容量
private static int calculateCapacity(Object[] elementData, int minCapacity) {
        // 如果  elementData 数组的容器为{}     
// 如果通过new ArrayList<>()无参构造函数初始化时，第一次调用add()方法时会进行扩容，将容量变从{}为10
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            // DEFAULT_CAPACITY=10  大于 minCapacity = 1，所以会取DEFAULT_CAPACITY
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
        }

private void ensureExplicitCapacity(int minCapacity) {
// 这个字段用于判断是否执行快速失败原则(fail-fast)
//相当于乐观锁的版本号，因为arraylist是线程不安全的，
//当在进行Iterator迭代遍历时，首先获取modCount的值保存到本地变量中，当遍历时会判断本地变量和modCount(全局变量)是否相等，
//如果不相等(有其他线程改变了集合的结构，会修改modCount的值)，就会抛出ConcurrentModificationException异常
        modCount++;  

        // overflow-conscious code
        // 这里传进来的minCapacity是当前add元素时需要的最小容器(size+1)
        // 当需要的最小容器大于当前容器的容量时，就需要进行扩容
        if (minCapacity - elementData.length > 0)
            // 对容器进行扩容
            grow(minCapacity);
    }

private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;  // 当前容量大小
// 需要扩容到多大    >>1表示/2     
//相当于oldCapacity + (oldCapacity / 2)，也就是需要扩展到原来的1.5倍
        int newCapacity = oldCapacity + (oldCapacity >> 1);  0+0/2=0
// 因为上面方法调用时将minCapacity变为10传过来的
  // 0-10<0，所以会进入下面的第一个if
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;    // 10
// int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            // 最大容量可设置为Integer.MAX_VALUE (2的32次方)
            newCapacity = hugeCapacity(minCapacity);
        // 这方法会进行扩容，会将原数据copy到一个容器大小为newCapacity新的list中，然后返回
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```
当通过无参构造函数初始化ArrayList时，在第一次add时会将list的容量扩容到10，之后每次扩容都是原来的1.5倍，比如当add到第11个元素时，会再次进行扩容，将容量扩容到15

#####分析get()方法
```
public E get(int index) {
        // 判断坐标是否越界
        rangeCheck(index);
        // 获取元素并返回
        return elementData(index);
    }
private void rangeCheck(int index) {
      // size为list中元素的数量
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
// 获取Object数组中index坐标的值，强转成E类型并返回
E elementData(int index) {
        return (E) elementData[index];
    }
```
get方法很简单，先判断坐标是否越界，然后获取elementData数组中元素并返回
这里要提一下size与elementData.length的区别
就相当于new arrayList(10) 与Object[] o = new Object[10]的区别
在没进对arrayList进行add添加元素的时候，不管容器大小是多少  size都为0的，所以就算new arrayList(10)，在get(0)时都会报坐标越界。
而new Object[10]在没有添加元素是，o[0]获取到的是null

#####分析remove方法
```
public E remove(int index) {
        // 判断坐标是否越界
        rangeCheck(index);

        modCount++;
        // 获取需要删除的元素，删除成功后会方法返回
        E oldValue = elementData(index);
        
        int numMoved = size - index - 1;
        if (numMoved > 0)
            // 对数组进行元素并对坐标进行重新排序，这个方法是jvm底层的native方法
            // 需要5个参数,
            1. src:源数组； 
            2.srcPos:源数组要复制的起始位置；
            3.dest:目标数组； 
            4.destPos:目标数组放置的起始位置； 
            5.length:复制的长度。

            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        // 数组元素的坐标重新排序后，需要对最后一个坐标设置为null，因为删除了一个元素，它后面的元素就会向前移动一位
        elementData[--size] = null; // clear to let GC do its work
        return oldValue;
    }
```
>#####ArrayList总结
>1.list集合底层是通过Object[]数组来存储的
>2.new ArrayList()时，初始化的容器是没有容量大小(相当于0)的，在第一次add时会将容器扩容到10，之后每次扩容数为原来的1.5倍。因为new ArrayList()初始化的容器是没有容量的，在第一次add时会进行扩容，而扩容会降低效率，所以**阿里巴巴**规范也有提到，一般在使用arrayList时，都会估算需要容量，初始化容器的大小，比如new ArrayList(10)，这样在add时会减少扩容的次数
>3.在删除元素时，会将元素删除，并将这个元素后面的元素向前移动，然后置null最后一个位置
>比如第一个图为原数组，删除 C后，会将D和E向前移动，然后将最后一个位置置为null
>![image.png](https://upload-images.jianshu.io/upload_images/14890912-4f2c380f2a853c28.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
>4.arrayList是线程不安全的，在Iterator迭代遍历时，如果modCount被修改了，会执行快速失败原则(fail-fast)，抛出ConcurrentModificationException异常
>5.如果想解决4问题，可以使用Vector集合或者Collections.synchronizedList(new ArrayList<>());或者CopyOnWriteArrayList集合类
>6.查询速度快，查询时间复杂度小(O(1)，直接通过index到数组上找)，空间复杂度大



#LinkedList源码分析
```
    transient int size = 0;   // size表示当前集合元素的大小，add会size++  remove会size--
    transient Node<E> first;  // 当前集合的第一个节点
    transient Node<E> last;  // 当前集合的最后一个节点
    public LinkedList() {
    }
```
new LinkedList只是初始化一个空的list，LinkedList是通过链表结构的node(节点)来存储的，首先看看Node<E>的结构
```
private static class Node<E> {
        E item;    // 当前节点的元素
        Node<E> next;  // 当前节点的下一个节点
        Node<E> prev;  // 当前节点的上一个节点

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```
在LinkedList中增加A   B   C三个元素，它的存储结构如下图，每个元素对会封装成Node节点对象来存储
![image.png](https://upload-images.jianshu.io/upload_images/14890912-6859ff3bf33db7ec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#####LinkedList的add方法
```
transient Node<E> first;  // 当前集合的第一个节点
transient Node<E> last;  // 当前集合的最后一个节点
public boolean add(E e) {
        // 在链表的最后增加元素
        linkLast(e);
        return true;
    }

void linkLast(E e) {
        final Node<E> l = last;   // 获取当前集合的最后一个节点
        final Node<E> newNode = new Node<>(l, e, null);  // 封装当前需要添加的节点
        last = newNode; // 将需要添加的节点设置为当前链表的最后一个节点
        if (l == null)  // 如果last为null(表示当前元素没有node对象)
            first = newNode;  第一个node设置为当前增加的节点对象
        else
            l.next = newNode;  // 否则在最后一个节点的下一个位置加上当前需要增加的节点对象
        size++;
        modCount++;
    }
```
调用add时会在链表的最后增加一个节点来存储这个元素,这里提一点linkedList.add(null);在增加是会添加一个Node对象来存储，这个null值是赋值在node.item=null，不要理解成node=null
#####LinkedList的get方法
```
public E get(int index) {
        // 检查坐标是否越界
        checkElementIndex(index);
        // 获取这个坐标的元素并返回
        return node(index).item;
    }

Node<E> node(int index) {
         // 通过折半查找(二分查找)法来查询node节点所在的位置
        // 如果index小于size的一半，就从第一个元素开始找，不断向下一个元素找
        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {  否则从最后一个元素开始往前找
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
```
LinkedList不像数组那样每个元素都有一个索引标识，只能从第一个node对象或者最后一个node对象遍历index次(first.next或last.next执行index次)找到这个元素，比如链表长度为100  需要查询index 60的元素  如果从first节点开始找，就需要循环遍历60次。linkedList底层通过折半查找(二分查找)法，60大于50(一半)，从100开始循环遍历40次就能定位到，提高查询效率

#####LinkedList的remove方法
```
public E remove(int index) {
         // 检查坐标是否越界
        checkElementIndex(index);
        // 先查询出node节点，然后删除这个节点
        return unlink(node(index));
    }

E unlink(Node<E> x) {
        // assert x != null;
        final E element = x.item;
        final Node<E> next = x.next;
        final Node<E> prev = x.prev;

        if (prev == null) {
            first = next;
        } else {
            prev.next = next;
            x.prev = null;
        }

        if (next == null) {
            last = prev;
        } else {
            next.prev = prev;
            x.next = null;
        }

        x.item = null;
        size--;
        modCount++;
        return element;
    }
```
LinkedList删除元素，其实就是将这个元素的node节点对象的prev(上一下)node节点对象和next(下一个)节点对象的所关联的prev和node的指向修改，然后将当前node节点的属性都置空(便于GC回收)
>#####LinkedList总结
>1.LinkedList底层是通过双向链表结构node节点对象来存储的，node对象有三个属性item(当前元素值)，prev(上一个node对象)，next(下一个node对象)。
>2.linkedList.add(null)，存储的是node.item=null，而不是node=null。
>3.增加和删除的速度快，空间复杂度小(不需要记录节点的位置)，查询时间复杂度大(查询相对慢 O(n)，比如有100个元素，找第30个。就需要把链表的node从第一个开始node.next循环30次才能找出来)


#####set集合
```
public HashSet() {
        map = new HashMap<>();
    }
public TreeSet() {
        this(new TreeMap<E,Object>());
    }

private static final Object PRESENT = new Object();
public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }
```
set集合底层用的是map，HashSet用的是HashMap，TreeSet用的是TreeMap
set.add时会把元素值存到map的key中，value为new Object()对象
因为map中的key是唯一的，所以set集合可以保存元素的唯一性