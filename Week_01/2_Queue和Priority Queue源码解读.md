## Queue
> `Queue` docs: https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/Queue.html
>
> `LinkedList` docs: https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/LinkedList.html
> 
> version: java12

在Java中,`Queue`是一个抽象的接口，定义了队列（特性：FIFO）最基本的操作，在其之下，又分别了定义其它的接口继承了`Queue`：

* BlockingQueue<E>：阻塞队列
* BlockingDeque<E>：继承了`BlockingQueue`,是阻塞的双端队列
* Deque<E>：双端队列
* TransferQueue<E>：继承了`BlockingDeque`,在其基础上又增加了保证生产者阻塞直到元素被某个线程消费的行为，具体实现查看`LinkedTransferQueue`

以上的`BlockingQueue`、`BlockingDeque`、`TransferQueue`用于多线程，这里不过多介绍。我们主要看`Deque`的实现`LinkedList`,并以实现`Queue`的视角描述。

### Queue 
`Queue`定义了几个基本操作如下：

| |会抛出异常|返回特殊值(一般为null)|
|:----:|----|----|
|插入| add(e)|offer(e)|
|删除|remove()|poll()|
|取出元素|element()|peek()|

当然，Java内常用的链表数据结构`LinkedList`是实现了`Deque`接口。`Deque`接口是在`Queue`的基础上又扩展了一些函数。

| |会抛出异常|返回特殊值(一般为null)|
|:----:|----|----|
|插入头| addFirst(e)|offerFirst(e)|
|插入尾| addLast(e)|offerLast(e)|
|删除头|removeFirst()|pollFirst()|
|删除尾|removeLast()|pollLast()|
|取出头|elementFirst()|peekFirst()|
|取出尾|elementLast()|peekLast()|

`Deque`的操作定义除了有`FIFO`的特性之外，它也具备栈的特征——`LIFO`。

|栈方法(`Stack`) |	等效的`Deque`方法|
|:----:|:----:|
|push(e)	|addFirst(e)|
|pop()	|removeFirst()|
|peek()|	getFirst()|


### LinkedList 
对`LinkedList`这样的实现来说,首先要定义出大致的结构：
```Java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{
    transient int size = 0;

    /**
     * Pointer to first node.
     * Invariant: (first == null && last == null) ||
     *            (first.prev == null && first.item != null)
     */
    transient Node<E> first;

    /**
     * Pointer to last node.
     * Invariant: (first == null && last == null) ||
     *            (last.next == null && last.item != null)
     */
    transient Node<E> last;

    private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
}
```
对`LinkedList`的数据结构来说，它定义了`first`和`last`两个首尾指针，指向它首尾位置的元素。其中`Node`是每个元素在这个数据结构中的定义，除了本身的内容之外，还定义了`prev`、`next`指向前后两个节点。

##### add() 、offer()
不论是`add()`、`addLast()`方法，最终都是使用的`linkLast()`方法。
其实现就是取出`last`节点，实例化一个新的`Node`,设置新节点的`prev`，并将新的节点设置为当前新的`last`节点。
```java
public class LinkedList<E>{
    /**
     * Links e as last element.
     */
    void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }
}
```

##### remove()、poll()
`remove()`和`poll()`真实实现都在`unlinkFirst()`方法中。
本质上就是取出第一个元素，置空`next`，再取出第二个元素，将第二个元素的`prev`置空,这样就完成了头元素的删除操作。
```java
public class LinkedList<E>{

    public E removeFirst() {
        final Node<E> f = first;
        if (f == null)
            throw new NoSuchElementException();
        return unlinkFirst(f);
    }

    /**
     * Unlinks non-null first node f.
     */
    private E unlinkFirst(Node<E> f) {
        // assert f == first && f != null;
        final E element = f.item;
        final Node<E> next = f.next;
        f.item = null;
        f.next = null; // help GC
        first = next;
        if (next == null)
            last = null;
        else
            next.prev = null;
        size--;
        modCount++;
        return element;
    }

}
```

### element()、peek()
`element()`和`peek()`都是直接返回`LinkedList`定义的`first`元素，无非判定抛异常或者返回null而已。
这里不多赘述。


## PriorityQueue
> docs: https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/PriorityQueue.html
>
> version: java12

`PriorityQueue`也是`Queue`的一个具体实现，是基于优先级堆的无界优先队列，其功能主要是实现了元素的有序性。如果想要使用线程安全的优先队列，则应该使用`PriorityBlockingQueue`。
`PriorityQueue`的排序是自然顺序排序或者是实现`Comparator`，这取决于构造的时候用哪个构造函数，插入的元素不能为空，不能不可比较。

### PriorityQueue Define
```java
public class PriorityQueue<E> extends AbstractQueue<E>
    implements java.io.Serializable {
    
    private static final int DEFAULT_INITIAL_CAPACITY = 11;
    
    /**
     * Priority queue represented as a balanced binary heap: the two
     * children of queue[n] are queue[2*n+1] and queue[2*(n+1)].  The
     * priority queue is ordered by comparator, or by the elements'
     * natural ordering, if comparator is null: For each node n in the
     * heap and each descendant d of n, n <= d.  The element with the
     * lowest value is in queue[0], assuming the queue is nonempty.
     */
    transient Object[] queue; // non-private to simplify nested class access

    /**
     * The number of elements in the priority queue.
     */
    private int size = 0;

    /**
     * The comparator, or null if priority queue uses elements'
     * natural ordering.
     */
    private final Comparator<? super E> comparator;

    /**
     * The number of times this priority queue has been
     * <i>structurally modified</i>.  See AbstractList for gory details.
     */
    transient int modCount = 0; 
}
```
`PriorityQueue`底层是一个数组作为容器，初始容量为11。

那为什么是数组呢？
这源自于`PriorityQueue`数据结构的设计是二叉小顶堆，而它正好可以通过数组表示。

假如定义的数组数据如下：
```
Character[] queue = new Character[]{'a','b','c','d','e','f','g'};
```
那么数组下标为零的元素就在堆顶，此时堆顶元素为'a',那么'a'左边的元素就是'b','a'右边的元素就是'c'，'b'的左边元素为'd'，右边元素为'e'，以此类推，最终构成了小顶堆的结构。
如下图所示:

<img src="https://raw.githubusercontent.com/Zack-Zz/algorithm024/main/Week_01/imgs/min_head_heap.png" width= "500" alt="structureV1.0" align=center />
