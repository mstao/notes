# 基础

## *AQS

**AQS核⼼思想是，如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的⼯作线程，并且将共享资源设置为锁定状态。如果被请求的共享资源被占⽤，那么就需要⼀套线程阻塞等待以及被唤醒 时锁分配的机制，这个机制AQS是⽤CLH队列锁实现的，即<font color=green>将暂时获取不到锁的线程加⼊到队列中</font>。**



基于 AQS 实现的类：  

- ReentrantLock、CountDownLatch

- Semaphore

- FutureTask

- ReentrantReadWriteLock



说明：

- 提供通用机制来原子性管理同步状态、阻塞和唤醒线程，以及维护被阻塞线程的队列。

- 至少有一个 acquire 操作，会阻塞调用线程，除非 / 直到 AQS 的状态允许这个线程继续执行

- 至少有一个 release 操作，改变 AQS 的状态，之后可允许一个或多个阻塞线程被解除阻塞

- 基于 "复合优先于继承" 原则，内部私有的 Sync，公有方法委托给这个内部子类。

- 使用模板方法设计模式

- 通知模式：当 product 往满的队列中添加元素时阻塞住生产者，当消费者消费了一个队列中的元素后，会通知生产者当前队列可用。



**底层结构**

队列同步器

内部维护着一个 FIFO 的双向链表，且该链表支持 CAS 的设置尾部，以及 CAS 更新值和下一个指向。

只要基于 AQS 机制便能够具备两种特性

<img src="http://img.janhen.com/202101301726391552634940837.png" alt="1552634940837" style="zoom:67%;" />

- 使用 Node 实现 FIFO 队列，可以用于构建锁或者其他同步装置的基础框架
- 利用了一个 int 类型表示状态
- 使用方法是继承
- 子类通过继承井通过实现它的方法管理其状态 `acquire` 和 `release` 的方法操纵状态
- 可以同时实现排它锁和共享锁模式（独占、共享）



### CLH 队列

基于 AQS 的两大特性：

- 提供共享式或独占式的获取方式

![1552636705615](http://img.janhen.com/202101301726421552636705615.png)
**两大核心：**

- state: volatile 的 int 变量，代表共享资源
- queue: FIEO 线程等待队列，多线程争用资源阻塞时进入该队列

**两种资源共享方式：**
- 独占式： 只有一个线程能执行，如 ReentrantLock
- 共享式：多个线程可同时执行， 如 Semaphore/CountDownLatch

**开放的API**
- isHeldExclusively： 线程是否独占资源
- tryAcquire / tryRelease: 尝试独占获取 / 释放资源
- tryAcquireShared / tryReleaseShared: 尝试共享获取 / 释放资源



**底层结构**

（1） FIFO 的双端队列

```java
abstract class AbstractQueuedSynchronizer
    extends AbstractOwnableSynchronizer
    implements java.io.Serializable {
transient volatile Node head;   // FIFO
transient volatile Node tail;
volatile int state;   // one core mean shared resources
}
```

（） Node

内部的 FIFO 是双端队列

```java
static final class Node {
    static final Node SHARED = new Node();
    static final Node EXCLUSIVE = null;

    static final int CANCELLED =  1;
    static final int SIGNAL    = -1;
    static final int CONDITION = -2;
    static final int PROPAGATE = -3;
    
    volatile int waitStatus;
    volatile Node prev;
    volatile Node next;
    volatile Thread thread;
    Node nextWaiter;
}
```

（3） 核心 state

```java
protected final int getState() {
    return state;
}
protected final void setState(int newState) {
    state = newState;
}
protected final boolean compareAndSetState(int expect, int update) {
    // See below for intrinsics setup to support this
    return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```

（4） 核心 FIFO 队列

多线程争用资源被阻塞时会进入此队列，

```java
final boolean compareAndSetHead(Node update) {
    return unsafe.compareAndSwapObject(this, headOffset, null, update);
}
final boolean compareAndSetTail(Node expect, Node update) {
    return unsafe.compareAndSwapObject(this, tailOffset, expect, update);
}
static final boolean compareAndSetWaitStatus(Node node,
                                             int expect,
                                             int update) {
    return unsafe.compareAndSwapInt(node, waitStatusOffset,
                                    expect, update);
}
static final boolean compareAndSetNext(Node node,
                                       Node expect,
                                       Node update) {
    return unsafe.compareAndSwapObject(node, nextOffset, expect, update);
}
```

（5） ConditionObject

见下部分中 Condition







**性质**

同步器是实现锁（也可以是任意同步组件）的关键，在锁的实现中聚合同步器，利用
同步器实现锁的语义。可以这样理解二者之间的关系：
（1）**锁是面向使用者的**，它定义了使用者与锁交互的接口（比如可以允许两个线程并
行访问），隐藏了实现细节；
（2）**同步器面向的是锁的实现者**，它简化了锁的实现方式，屏蔽了同步状态管理、线
程的排队、等待与唤醒等底层操作。或者可以把AQS认为是锁的实现者的一个父类。
锁和同步器很**好地隔离了使用者和实现者所需关注的领域**。



**方法实现**

没有任何抽象方法，

（） protected

提供了五个方法可以在子类中重载，都是空实现直接抛出异常，必须要自己在子类中进行实现，这也是“模板方法模式”的一种体现和使用。

```java
tryAcquire(int);
tryRelease(int);
tryAcquireShared(int);
tryReleaseShared(int);
isHeldExclusively();
```

（2） public final 类别

除了上述protected 类别的方法，还有一个关键的类别就是public final 类别，这是因为，这是我们可以直接使用的方法，称之为“模板方法”，当我们实现自定义的同步组件的时候，我们可以调用这些模板方法获取我们需要的东西。

同步器提供的上述模板方法基本上分为3类：**独占式获取与释放同步状态、共享式获取与释放同步状态、查询同步队列中的等待线程情况**。

自定义同步组件将使用同步器提供的模板方法来实现自己的同步语义。只有掌握了同步器的工作原理才能更加深入地理解并发包中其他的并发组件。



（3） protected final 级别

我们至少应该知道了我们要对int类型的同步状态进行修改，下边的三个方法
提供了可以修改：
另外还有三个：hasWaiters、getWaitQueueLength、getWaitingThreads三个方法。



## CAS

底层是通过 CPU 的指令实现的

（） 原理

compareAndSet利用 JNI 来完成 CPU 指令的操作。



缺点：

只能锁定一个变量；

存在 CAS 的循环开销大；

存在 ABA 问题(可通过 AtomicStampedReference 解决)；



（） ABA 问题



（） 解决

通过版本号的方式来解决，乐观锁每次在执行数据的修改操作时，都会带上一个版本号，一旦版本号和数据的版本号一致就可以执行修改操作并对版本号执行+1操作，否则就执行失败。



# 原子类

> Atomic 是指⼀个操作是不可中断的。即使是在多个线程⼀起执⾏的时 候，⼀个操作⼀旦开始，就不会被其他线程⼲扰。

AtomicInteger、AtomicBoolean、AtomicLong ...

## AtomicBoolean 

**底层结构 | 底层原理**

内部通过将其转换成 Integer 来调用 Unsafe 类的方法实现的。

三者底层都是基于 Unsafe 类：

```java
static final Unsafe unsafe = Unsafe.getUnsafe();
static final long valueOffset;
volatile int value;
AtomicBoolean(boolean initialValue) {
    value = initialValue ? 1 : 0;
}
```

JDK7 中 AtomicInteger

```java
public final int getAndIncrement() {
    for (;;) {
        int current = get();
        int next = current + 1;
        if (compareAndSet(current, next))
            return current;
    }
}
```



**核心方法**

（1） compareAndSet

```java
final boolean compareAndSet(boolean expect, boolean update) {
    int e = expect ? 1 : 0;
    int u = update ? 1 : 0;
    return unsafe.compareAndSwapInt(this, valueOffset, e, u);
}
```

（2） get

借助 volatile 无锁访问

```java
final boolean get() {
    return value != 0;
}
```



**适用场景**
适合用来处理 *只执行一次* 的逻辑，只有一个订单的抢夺；

作为关闭线程的标识；



## AtomicInteger

AtomicInteger 类主要利⽤ CAS (compare and swap) + volatile 和 native ⽅法来保证原⼦操作， 从⽽避免 synchronized 的⾼开销，执⾏效率⼤为提升。

CAS的原理是拿期望的值和原本的⼀个值作⽐较，如果相同则更新成新的值。UnSafe 类的 objectFieldOffset() ⽅法是⼀个本地⽅法，这个⽅法是⽤来拿到“原来的值”的内存地址，返回值是 valueOffset。另外 value 是⼀个volatile变量，在内存中可⻅，因此 JVM 可以保证任何时刻任何线 程总能拿到该变量的最新值。





## LongAdder

JDK8 引入，通过 Cell 分割单元的方式，通过对其求和来实现，

在高并发下性能比 AtomicLong 更高，避免了 AtomicLong 不断循环重试的开销；

在高并发下可能数据不精确；

**底层结构**

（1） 总览

```java
class LongAdder extends Striped64 implements Serializable
```

（2） 父类

```java
transient volatile Cell[] cells;
transient volatile long base;
transient volatile int cellsBusy;
```





**Cell 机制**

内部分割成各个小单元

在高并发的情况下性能相较于 AtomicLong 高

高并发下通过 分散来提供性能


缺陷： 可能导致误差

优先使用 LongAdder



（1） add

操作分片

其中 increment() 和 decrement() 都是通过该方法实现的

```java
void add(long x) {
    Cell[] as; long b, v; int m; Cell a;
    if ((as = cells) != null || !casBase(b = base, b + x)) {
        boolean uncontended = true;
        if (as == null || (m = as.length - 1) < 0 ||
            (a = as[getProbe() & m]) == null ||
            !(uncontended = a.cas(v = a.value, v + x)))
            longAccumulate(x, null, uncontended);
    }
}
```



（2） 获取当前值

高并发下，效率高，不能保证绝对的精确。

```java
long sum() {
    Cell[] as = cells; Cell a;
    long sum = base;
    if (as != null) {
        for (int i = 0; i < as.length; ++i) {
            if ((a = as[i]) != null)
                sum += a.value;
        }
    }
    return sum;
}
```



**与 AtomicLong 的区别**

① 底层实现上： AtomicLong 底层 CAS 在循环中不断，开销大；

LongAdder 将单点分散到各个 cell 中，但可能存在一定的偏差，适合用于大并发。

② 适用场景： LongAdder 适合用于大并发，AtomicLong 适合对准确性高的场合





## 其他

**原子更新引用类型**

（） AtomicReference



**（）AtomicStampedReference**

用来解决 CAS 中的 ABA 问题而引入的

用于解决 ABA 问题而出现的，不仅比较 期望的值还要比较 stamp.
类似于 db 中的乐观锁。



更新值的时候还必须要更新时间戳，只有当值满足预
期且时间戳满足预期的时候，写才会成功！

```java
compareAndSet(V expectedReference, V newReference, int expectedStamp, int newStamp)
```



AtomicStampedReference带有时间戳的对象引用类型
为了表述一个有状态迁移的AtomicReference 而升级为带有时间戳的对象引
用AtomicStampedReference ， AtomicStampedReference 解决了上述对象在修
改过程中，丢失状态信息的问题，使得对象的值不仅与预期的值相比较，还通过时间
戳进行比较，这就可以很好的解决对象被反复修改导致线程无法正确判断对象状态的
问题。





**原子更新数组类型**

- AtomicReferenceArray: 引⽤类型数组原⼦类

- AtomicIntegerArray:

- AtomicLongArray:

增加了操作对应索引的 API, 由原来的对象作为组合对象传入





**原子更新属性类型**

AtomicIntegerFieldUpdater
AtomicLongFieldUpdater
AtomicReferenceFieldUpdater



# 并发容器

## BlockingQueue

阻塞队列的四种方法： 

![1552635139942](http://img.janhen.com/202101301727441552635139942.png)

![1553499862597](http://img.janhen.com/202101301726491553499862597.png)

### ArrayBlockingQueue

**底层结构**

底层采用循环数组实现，每个索引位置都存放元素；

```java
final Object[] items;
int takeIndex;
int putIndex;
int count;
```



**阻塞和并发控制：** 

全局唯一锁配合 notEmpty, notFull 的 Condition 实现；

对于 size() 需要全局锁定才能获取；

锁的粒度相较于 LinkedBlockingQueue 粗，且 LinkedBlockingQueue 类似一种锁分离的思想，take 和 put 两个操作使用不同的锁进行处理。

```java
final ReentrantLock lock;
final Condition notEmpty;
final Condition notFull;
```

```java
ArrayBlockingQueue fairQueue = new ArrayBlockingQueue<>(10, true);
```



**与 LinkedBlockingQueue 的区别**

① 队列的大小： ArrayBlockingQueue 有界的必须初始化，LinkedBlockingQueue 无界，可能出现 OOM；

② 数据存储容器： 基于数组作为数据存储容器，LinkedBlockingQueue 采用 Node 的链表；

③ 实现并发方式： ArrayBlockingQueue 实现锁未分离，共用同一把锁，而 LinkedBlockingQueue 实现 take, put 分离，提高队列的并发量，效率更高；

④ 由于ArrayBlockingQueue 采用的是数组的存储容器，因此在插入或删除元素时不会产生或销毁任何额外的对象实例，而LinkedBlockingQueue 则会生成一个额外的Node 对象。这可能在长时间内需要高效并发地处理大批量数据的时，对于GC 可能存在较大影响。





### LinkedBlockingQueue

**底层结构 | INIT**

（1） 整体结构

底层基于链表实现，复用 AbstractQueue 逻辑;

有类似虚拟节点的概念，初始时设置为 new Node(null);

count： 原子类记录存在多个锁控制同步的容器

```java
transient Node<E> head;
private transient Node<E> last;
final int capacity;
final AtomicInteger count = new AtomicInteger();     /* use for count */

public LinkedBlockingQueue(int capacity) {
    // ...
    last = head = new Node<E>(null);                   
}
```



（2） 节点

普通的单链表节点

```java
class Node<E> {
    E item;
    Node<E> next;
    Node(E x) { item = x; }
}
```





**同步控制**

两个锁实现，在全局性的操作时锁住整个，如 size()，性能比 ArrayBlockingQueue 高， 是锁分离思想、和减少锁粒度的两种思想的体现。

部分情况下需要全部锁住；

对于 size() 直接通过 AtomicInteger 实现，无需加锁获取；

能够高效的处理并发数据，还因为其  <u>对于生产者端和消费者端分别采用了独立的锁来控制数据同步</u>，这也意味着在高并发的情况下生产者和消费者可以并行地操作队列中的数据，以此来提高整个队列的并发性能。

```java
final ReentrantLock takeLock = new ReentrantLock();
final Condition notEmpty = takeLock.newCondition();
final ReentrantLock putLock = new ReentrantLock();
final Condition notFull = putLock.newCondition();
```



**take()**

使用 takeLock 可中断式获取锁实现；

同时控制该锁的条件 notEmpty；

```java
E take() throws InterruptedException {
    E x;
    int c = -1;
    final AtomicInteger count = this.count;
    final ReentrantLock takeLock = this.takeLock;
    takeLock.lockInterruptibly();        /* can interrupe, InterruptedException */
    try {
        while (count.get() == 0) {
            notEmpty.await();
        }
        x = dequeue();
        c = count.getAndDecrement();
        if (c > 1)
            notEmpty.signal();
    } finally {
        takeLock.unlock();
    }
    if (c == capacity)
        signalNotFull();
    return x;
}
```

**put()**

使用 putLock 可中断加锁实现；

控制该锁的条件 notFull 条件

```java
void put(E e) throws InterruptedException {
    if (e == null) throw new NullPointerException();
    // Note: convention in all put/take/etc is to preset local var
    // holding count negative to indicate failure unless set.
    int c = -1;
    Node<E> node = new Node<E>(e);
    final ReentrantLock putLock = this.putLock;
    final AtomicInteger count = this.count;
    putLock.lockInterruptibly();
    try {
        /*
         * Note that count is used in wait guard even though it is
         * not protected by lock. This works because count can
         * only decrease at this point (all other puts are shut
         * out by lock), and we (or some other waiting put) are
         * signalled if it ever changes from capacity. Similarly
         * for all other uses of count in other wait guards.
         */
        while (count.get() == capacity) {
            notFull.await();
        }
        enqueue(node);
        c = count.getAndIncrement();
        if (c + 1 < capacity)
            notFull.signal();
    } finally {
        putLock.unlock();
    }
    if (c == 0)
        signalNotEmpty();
}
```



### 其他

#### PriorityBlockingQueue

是一个 <u>支持优先级的无界队列</u>。默认情况下元素采取自然顺序升序排列。可以自定义实现compareTo()方法来指定元素进行排序规则，或者初始化PriorityBlockingQueue时，指定构造参数Comparator来对元素进行排序。需要注意的是不能保证同优先级元素的顺序。





#### SynchronousQueue

<u>是一个不存储元素的阻塞队列。每一个put操作必须等待一个take操作，否则不能继续添加元素。</u>
SynchronousQueue可以看成是一个传球手，负责把生产者线程处理的数据直接传递给消费者线程。队列本身并不存储任何元素，非常适合于传递性场景,比如在一个线程中使用的数据，传递给另外一个线程使用，SynchronousQueue的吞吐量高于LinkedBlockingQueue 和 ArrayBlockingQueue。





#### DelayQueue

用于缓存失效，定时任务；

是一个支持延时获取元素的无界阻塞队列。队列使用PriorityQueue来实现。队列中的元素必须实现Delayed接口，在创建元素时可以指定多久才能从队列中获取当前元素。只有在延迟期满时才能从队列中提取元素。我们可以将DelayQueue运用在以下应用场景：

 1. 缓存系统的设计：可以用DelayQueue保存缓存元素的有效期，使用一个线程循环查询DelayQueue，一旦能从DelayQueue中获取元素时，表示缓存有效期到了。

2.任务调度：使用DelayQueue保存当天将会执行的任务和执行时间，一旦从DelayQueue中获取到任务就开始执行，从比如TimerQueue就是使用DelayQueue实现的。





LinkedTransferQueue







LinkedBlockingDeque







## ConcurrentQueue

**底层结构**

head 与 tail 不是精确的指定到对应的逻辑语义，是在两者之间不断变化的。

通过一种 Hops 的优化技巧来提供性能

```java
transient volatile Node<E> head;
transient volatile Node<E> tail;
public ConcurrentLinkedQueue() {
    head = tail = new Node<E>(null);
}
    
static class Node<E> {
    volatile E item;
    volatile Node<E> next;
    boolean casItem(E cmp, E val);
    void lazySetNext(Node<E> val);
}
```



**Hops 设计**

能减少CAS更新tail节点的次
数，就能提高入队的效率，所以doug lea使用hops变量来控制并减少tail节点的更新频率，并不是每次节点入队后都将tail节点更新成尾节点，而是当tail节点和尾节点的距离大于等于常量HOPS的值（默认等于1）时才更新tail节点，tail和尾节点的距离越长，使用CAS更新tail节点的次数就会越少，但是距离越长带来的负面效果就是每次入队时定位尾节点的时间就越长，因为循环体需要多循环一次来定位出尾节点，但是这样仍然能提高入队的效率，因为从本质上来
看它通过增加对volatile变量的读操作来减少对volatile变量的写操作，而对volatile变量的写操作开销要远远大于读操作，所以入队效率会有所提升。



**offer**

hops 机制优化实现，两者相隔的越远性能就越高

从源代码角度来看，整个入队过程主要做两件事情：第一是定位出尾节点；第二是使用
CAS算法将入队节点设置成尾节点的next节点，如不成功则重试。



**poll()**

并不是每次出队时都更新head节点，当head节点里有元素时，直接弹出head
节点里的元素，而不会更新head节点。只有当head节点里没有元素时，出队操作才会更新head
节点。这种做法也是通过hops变量来减少使用CAS更新head节点的消耗，从而提高出队效率。让我们再通过源码来深入分析下出队过程。



# Lock
## Condition

与 Object.wait() , wait(timeout), notify(), notifyAll() 形成对应

由 AQS 框架中的内部类 ConditionObject 实现，具有 等待标识。



底层为 AQS 中的 ConditionObject


**AQS 内部类-ConditionObject**

实现了 Condition 接口，，因为Condition的操作需要
获取相关联的锁，所以作为同步器的内部类是一个比较合理的方式。

每个 Condition 对象都包含一个等待队列，该队列使 Condition 实现等待通知机制的关键。



控制

① 提供对于重新打断的支持

② 提供对于抛出 InterruptedException 的支持

③ 提供对于 Condition 在某个锁上的队列；

```java
class ConditionObject implements Condition, java.io.Serializable {
    transient Node firstWaiter;
    transient Node lastWaiter;
    /*
     * For interruptible waits, we need to track whether to throw
     * InterruptedException, if interrupted while blocked on
     * condition, versus reinterrupt current thread, if
     * interrupted while blocked waiting to re-acquire.
     */
    /** Mode meaning to reinterrupt on exit from wait */
    private static final int REINTERRUPT =  1;
    /** Mode meaning to throw InterruptedException on exit from wait */
    private static final int THROW_IE    = -1;
}
```



**等待队列**

等待队列是一个FIFO的队列，在队列中的每一个节点都包含一个线程的引用，该线程
就是在Condition对象上等待的线程，如果一个线程调用了Condition.await() 方
法，那么该线程将会释放锁，构造成节点加入等待队列并进入等待状态。这里的节点
Node使用的是AQS中定义的Node。也就是说AQS中的同步队列和Condition的等待队
列使用的节点类型都是AQS中定义的Node内部类（AbstractQueuedSynchronizer.Node）。
一个Condition对象包含一个等待队列，Condition拥有首节点和尾节点。当前线程调
用Condition.await() 方法，将会以当前线程构造节点，并将该节点从尾部加入到等待队列，等待队列的基本结构如下图：

![1553838808834](http://img.janhen.com/202101301726541553838808834.png)





（1）  Lock 与同步队列

我们知道在使用synchronized的时候，是使用的对象监视器模型的，即在Object的监
视器模型上，一个对象拥有一个同步队列和等待队列，而  <u>Lock可以拥有一个同步队列</u>
<u>和多个等待队列</u>  ，这是因为通过lock.newCondition() 可以创建多个Condition条
件，而这多个Condition对象都是在同一个锁的基础上创建的，在同一时刻也只能由一个线程获取到该锁。

![1553838928905](http://img.janhen.com/202101301727001553838928905.png)



（2） 等待在结构上的逻辑

当前线程调用Condition.await() 方法的时候，相当于将当前线程从同步队列的首
节点移动到Condition的等待队列中，并释放锁，同时线程变为等待状态。
当前线程加入到等待队列的过程如下：

可以看出同步队列的首节点并不是直接加入到等待队列的尾节点，而是封装成等待队列的节点才插入到等待队列的尾部的。

![1553839003852](http://img.janhen.com/202101301727511553839003852.png)



（3） 通知的实现

调用当前线程的 Condition.signal() 方法，将会唤醒在等待队列中等待时间最长的节点也就是首节点，在唤醒节点之前，会将该节点移到同步队列中。
节点从等待队列加入到同步队列的过程如下：

![1553839070089](http://img.janhen.com/202101301727031553839070089.png)

通过调用同步器的方法将等待队列中的头结点线程安全的移到同步队列的尾节点，当前线程在使用LockSupport唤醒该节点的线程。
被唤醒后的线程，将会从 await（） 方法中的while循环中退出，进而调用同步器的方法加入到获取同步状态的竞争中。
成功获取同步状态之后，被唤醒的线程从先前调用的await饭发个返回，此时该线程已经功的获取了锁。
Condition的 signalAll() 方法，相当于对等待队列中的每一个节点均执行一次
signal（） 方法，效果就是将等待队列中的所有节点全部移到同步队列中，并唤醒每个节点的线程。



**使用 Condition 实现阻塞队列**

```java
final Queue<Object> queue = new LinkedList<>();
final AtomicInteger count = new AtomicInteger(0);
int capacity = Integer.MAX_VALUE;
Lock mainLock = new ReentrantLock();
Condition notEmpty = mainLock.newCondition();
Condition notFull = mainLock.newCondition();
```




**与 Object 类锁方法区别**

1. Condition类的awiat方法和Object类的wait方法等效 
2.  Condition类的signal方法和Object类的notify方法等效 
3.  Condition类的signalAll方法和Object类的notifyAll方法等效 
4.  ReentrantLock类可以唤醒指定条件的线程，而object的唤醒是随机的




## *ReentrantLock

是很多工具类的实现

JDK7 中 ConcurrentHashMap.Segment 基于此类实现；

基于此实现具备了可中断及无阻塞获取锁，公平锁，多种 Condition 的特性；



**主要方法**

void lock(): 执行此方法时, <u>如果锁处于空闲状态, 当前线程将获取到锁</u>. 相反, 如果锁已经被其他线程持有, 将禁用当前线程, 直到当前线程获取到锁. 

boolean tryLock()：<u>如果锁可用, 则获取锁, 并立即返回true, 否则返回false</u>. 该方法和lock()的区别在于, tryLock()只是"试图"获取锁, 如果锁不可用, 不会导致当前线程被禁用, 当前线程仍然继续往下执行代码. 而lock()方法则是一定要获取到锁, 如果锁不可用, 就一直等待, 在未获得锁之前,当前线程并不继续向下执行. 

void unlock()：执行此方法时, 当前线程将释放持有的锁. 锁只能由持有者释放, 如果线程并不持有锁, 却执行该方法, 可能导致异常的发生. 

Condition newCondition()：<u>条件对象，获取等待通知组件</u>。该组件和当前的锁绑定，当前线程只有获取了锁，才能调用该组件的await()方法，而调用后，当前线程将缩放锁。

（） 公平性提供



（） tryLock、lock、lockInterrupibly 区别

1. tryLock能获得锁就返回true，不能就立即返回false，tryLock(long timeout,TimeUnit unit)，可以增加时间限制，如果超过该时间段还没获得锁，返回false 
2.  lock能获得锁就返回true，不能的话一直等待获得锁 
3.  lock和lockInterruptibly，如果两个线程分别执行这两个方法，但此时中断这两个线程，lock不会抛出异常，而lockInterruptibly会抛出异常。

**特性**

**（1） 可重入性**

AQS 中的 `state` 代表的语义为当前加了多少次锁，此外 ReentrantLock 是独占式的方式进行处理的。

当 `state` 为 0 时，代表无锁，否则加了多少次锁就要对应的释放多少次。



可重入锁一定程度上避免了死锁；



ReentrantLock

state 初始化为0，未锁定装填

A 线程 lock 时， 
![1552636782238](http://img.janhen.com/202101301728061552636782238.png)

可重入，+1，释放相对应的个数
![1552636786098](http://img.janhen.com/202101301728091552636786098.png)

transient 独占线程





（2） 公平性

通过 AQS 模板方法实现了 Sync, NofairSync, FairSync 分别实现不同的同步组件，Sync正是继承了AbstractQueuedSynchronizer 这个抽象类，而NonfairSync 和FairSync 又是继承了Sync 的两个静态内部类。



实现公平锁，需要在多线程环境下维护一个 FIFO 的双端队列，性能相比非公平性低；





**（） 等待通知模式的支持**

基于 AQS 中 ConditionObject 内部类实现

见 Condition

```java
public Condition newCondition() {
	return sync.newCondition();
}
final ConditionObject newCondition() {
	return new ConditionObject();
}
```





**与 synchronized 的比较**

Lock它缺少了（通过synchronized块或者方法所提供的）隐式获取释放锁的便捷性，

但是却拥有了锁获取与释放的可操作性、可中断的获取锁以及超时获取锁等多种synchronized关键字所不具备的同步特性




## ReentrantReadWriteLock

介绍： 在没有任何读写锁时，才能取得写锁；

（1） 特性：

实现了悲观锁，当读取多而写入少时，造成写的 "饥饿"；

基于 AQS 实现，同时实现独占和共享两种方式；

体现锁分离进行锁优化的思想；

>读读共享，写写互斥，读写互斥，写读互斥

（2） 读写

读写锁维护了一对锁：一个读锁和一个写锁。通过分离读锁和写锁，使得并发性相比
一般的排它锁有很大的性能提升。



**底层结构**

（1） 内部类

```
Sync
NofairSync
FairSync
ReadLock
WriteLock
```


（2） ReadWriteLock

提供了readLock() 和writeLock() 方法，类似于  <u>工厂方法模式的工厂接口</u>，而Lock就是返回的  <u>产品接口</u>。

而ReentrantReadWriteLock实现了ReadWriteLock接口，那么他就是  <u>具体的工厂接口实现类</u>，ReadLock和WriteLock就成了  <u>具体产品的实现类</u>，一个简单的工厂方法模式使用案例，值得学习。

![1553826265053](http://img.janhen.com/202101301727101553826265053.png)



**ReentrantReadWriteLock 的特性**

**（1） 锁的降级**
可将写锁降级为读锁，

（2） 公平性

（3） 可重入



**对于特性的实现**

（1） 使用一个 int 表示读写

如何在一个整型变量上维护多种状态，就需要"**按位切割使用**" 这个变量，读写锁将变
量切分成两个部分，**高16位表示读，低16位表示写**，划分方式如下图：

![1553828274145](http://img.janhen.com/202101301727131553828274145.png)



当前状态表示一个线程已经获取了写锁，且重入了两次，同时也获取了两次读锁。那
么读写锁是如何迅速确定读和写各自的状态那？答案就是"位运算" 。



如何通过位运算计算得出是读还是写获取到锁了那？
如果当前同步状态state不为0，那么先计算低16位写状态，如果低16为为0，也就是写状态为0则表示高16为不为0，也就是读状态不为0，则读获取到锁；如果此时低16为不为0则抹去高16位得出低16位的值，判断是否与state值相同，如果相同则表示写获取到锁。同样如果state不为0，低16为不为0，且低16位值不等于state，也可以通过state的值减去低16位的值计算出高16位的值。上述计算过程都是过位运算计算出来的。
上图中为什么表示当前状态有一个线程已经获取了写锁，且重入了两次，同时也获取了两次读锁



（2） 写锁的获取与释放

写锁是一个支持重进入的排它锁。如果当前线程已经获取了写锁，则增加写状态。如果当前线程在获取写锁时，读锁已经被获取（读状态不为0）或者线程不是已经获取写锁的线程，则当前线程进入等待状态。



（3） 读锁的获取与释放

读锁是一个支持重进入的共享锁。它能够被多个线程同时获取，在没有其他线写线程访问（写状态为0）时，读锁总是会被成功获取，而所作的也只是增加读状态。如果当前线程在获取读锁时，写锁已被其他线程获取，则进入等待状态。



（4） 锁降级

看到上述这两段似乎还是找不到为什么会出现高位和低位都不为0的情况怎样确定当前线程获取的是写锁的解答，这就需要我们从另一个需要注意的地方说起：锁降级, 何为锁降级，意思主要是  **为了保证数据的可见性** ，如有一个线程A已经获取了写锁，并且修改了数据，如果当前线程A不获取读锁而直接释放写锁，此时，另一个线程B获取到了写锁并修改了数据，那么当前线程A无法感知线程B的数据更新。

如果当前线程 A 获取读锁，即遵循降级的步骤，则线程B将会被阻塞，直到当前线程A使用数据并释放读锁之后，线程B才能获取写锁进行数据更新。



另外，**锁降级中读锁的获取是必要的！！！**



正是由于锁降级的存在，才会出现上图中高16位和低16为都不为0，但可以确定是写锁的问题。可以得出结论，如果高16为或者低16为为0，那么我们就可以判断获取到的是写锁或读锁；如果高16位和低16位都不为0那获取到的应该是写锁。就是**说如果当前线程已经获取到写锁的话，该线程也是可以通过CAS线程安全的增加读状态的，成功获取读锁。**

虽然，为了保证数据的可见性引入锁降级可以将写锁降级为读锁，但是却不可以锁升级，将读锁升级为写锁的，也就是不会出现：当前线程已经获取到读锁了，通过某种方式增加写状态获取到写锁的情况。不允许升级的原因也是保证数据的可见性，如果读锁已被多个线程获取，其中任意线程成功获取了写锁并更新了数据，则其更新对其他获取到读锁的线程是不可见的。







**总结**

1、读锁的重入是允许多个申请读操作的线程的，而写锁同时只允许单个线程占有，该
线程的写操作可以重入。
2、如果一个线程占有了写锁，在不释放写锁的情况下，它还能占有读锁，即写锁降级
为读锁。
3、对于同时占有读锁和写锁的线程，如果完全释放了写锁，那么它就完全转换成了读
锁，以后的写操作无法重入，在写锁未完全释放时写操作是可以重入的。
4、公平模式下无论读锁还是写锁的申请都必须按照AQS锁等待队列先进先出的顺
序。非公平模式下读操作插队的条件是锁等待队列head节点后的下一个节点是
SHARED型节点，写锁则无条件插队。
5、读锁不允许newConditon获取Condition接口，而写锁的newCondition接口实现方
法同ReentrantLock。


## StampedLock

乐观锁

// TODO ...




## LockSupport

不管是ReentrantReadWriteLock还是ReentrantLock，当需要阻塞或唤醒一个线程的
时候，都会使用LockSupport工具类完成相应的工作



LockSupport.park():
四种打断方式：

- unpart();
- 中断
- 定时时间到达
- 异常出现

```java
// unsafe.park
public native void park(isAbsolute, time)
```



 LockSupport 基于线程的锁
操作系统底层，由OS 决定，调用 Unsafe

wait 与 notify 必须配合 synchronized 使用
wait 方法释放锁，notify

nano 级别设置时间
park(),unpark(thread)

Condition



unpark(A) 方法先于指定线程 park() 操作

好处：

- **基于线程**级别的阻塞和唤醒，调用顺序可相反
  只用同一个线程即可使用






# Tools

## CountDownLatch

可通过 start, end 分别控制一起开始执行，一起结束。

用于并发控制
特性：
- 计数不可重置
- 等待所有请求(子任务)都被处理完后，进行统一处理



API：

- await()
- await(
- countDown()

使用场景：
等待所有子任务完成，之后进行统一处理；
在指定时间内未执行完也可进行处理；

作用于并发执行的线程，而非主线程的单线程；
同时 await(timeout, TimeUnion) 并发线程仍然会执行完

![1553694284812](http://img.janhen.com/202101301727171553694284812.png)

**底层结构**

（） 构造函数

基于 AQS 机制共享资源方式实现的；

```java
public CountDownLatch(int count) {
    if (count < 0) throw new IllegalArgumentException("count < 0");
    this.sync = new Sync(count);
}
```

（） countDown

通过 AQS 中的 state 来表示 count，共享资源方式获得；

```java
public void countDown() {
    sync.releaseShared(1);
}
```

（） await

```java
public void await() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}
```



阻塞的是主线程；







**应用场景**

（） 有一个任务A，它要等待其他4个任务执行完毕之后才能执行，此时就可以利用CountDownLatch来实现这种功能了。



（1）实现最大的并行性：

有时我们想同时启动多个线程，实现最大程度的并行性。例如，我们想测试一个单例类。如果我们创建一个初始计数为 1 的 CountDownLatch，并让所有线程都在这个锁上等待，那么我们可以很轻松地完成测试。我们只需调用 一次 countDown() 方法就可以让所有的等待线程同时恢复执行。
（2）开始执行前等待n个线程完成各自任务：

例如应用程序启动类要确保在处理用户请求前，所有N个外部系统已经启动和运行了。
（3）死锁检测：

一个非常方便的使用场景是，你可以使用n个线程访问共享资源，在每次测试阶段的线程数目是不同的，并尝试产生死锁。





**CountDownLatch 区别**

- ① CyclicBarrier 可重置，通过 reset() ，可用于处理复杂的业务场景，如出错重置重新执行；
- ② CyclicBarrier 提供执行完一个周期后的 Callback 函数，更加灵活；
- ④ CyclicBarrier还提供其他有用的方法，比如getNumberWaiting方法可以获得CyclicBarrier阻塞的线程数量。isBroken方法用来知道阻塞的线程是否被中断。比如以下代码执行完之后会返回true。
- ③ CountDownLatch会阻塞主线程，CyclicBarrier不会阻塞主线程，只会阻塞子线程。





（） 应用

表述的是Flume的文件输出流到HBase的时候，初始化HBase客户端的操
作，可以看到大数据框架Flume也是用到了CountDownLatch 。



（2） 不算实例的实例就是，如果我们操作一个非常耗时的数据库操作的时候，例
如一个查询，我需要把它分为三个线程分别去查询，等待所有的线程查询完之后，然
后需要组装查询完的数据





## CyclicBarrier

特性

- await(): 计数相加
- await(timeout, TimeUnit)

- 可重置，每一批线程到达 barrier 执行，之后自动重置执行下一批操作
- 指定到达 barrier 优先执行的 barrierAction



```java
final ReentrantLock lock = new ReentrantLock();
final Condition trip = lock.newCondition();
final int parties;
final Runnable barrierCommand;
Generation generation = new Generation();
int count;
```





## Semaphore

控制并发数，"交通路口可容纳的车的数量"

Semaphore是一种基于计数的信号量。它可以设定一个阈值，基于此，多个线程竞争获取许可信号，做完自己的申请后归还，超过阈值后，线程申请许可信号将会被阻塞。Semaphore可以用来构建一些对象池，资源池之类的，比如数据库连接池。



特性：
- 仅能提供优先访问的资源， 如 db 中控制并发访问数



原理：有大小限制的链表，可用 reentrantlock 实现，但复杂；

使用场景：
- 单个通行许可证；  作为互斥锁实现
- 多个通行许可证；
- 丢弃，尝试获取；


Semaphore类是一个计数信号量，必须由获取它的线程释放， 
通常用于限制可以访问某些资源（物理或逻辑的）线程数目。


一个信号量有且仅有3种操作，且它们全部是原子的：初始化、增加和减少 
增加可以为一个进程解除阻塞； 
减少可以让一个进程进入阻塞。



（） 实现互斥锁

计数为 1，类似互斥锁的机制，二元信号量，表示两种互斥状态。



```java
Semaphore semp = new Semaphore(5);
try {
    semp.acquire();
    try {
    	//...
    } catch (Exception e) {
    } finally {
        semp.release();        // need release
    }
} catch (InterruptedException e) {
}

```

**与 ReentrantLock 的对比**

Semaphore基本能完成ReentrantLock的所有工作，使用方法也与之类似，通过acquire()与release()方法来获得和释放临界资源。经实测，Semaphone.acquire()方法默认为可响应中断锁，与ReentrantLock.lockInterruptibly()作用效果一致，也就是说在等待临界资源的过程中可以被Thread.interrupt()方法中断。 

此外，Semaphore也实现了 <u>可轮询的锁请求与定时锁的功能</u>，除了方法名tryAcquire与tryLock不同，其使用方法与ReentrantLock几乎一致。Semaphore也提供了公平与非公平锁的机制，也可在构造函数中进行设定。 Semaphore的锁释放操作也由手动进行，因此与ReentrantLock一样，为避免线程因抛出异常而无法正常释放锁的情况发生，释放锁的操作也必须在finally代码块中完成。



**应用场景**

（） 若一个工厂有5台机器，但是有8个工人，一台机器同时只能被一个工人使用，只有使用完了，其他工人才能继续使用。那么我们就可以通过Semaphore来实现：





## Exchanger

通常用于两个线程之间交换数据，用于重新比对数据


# *Executor 

<img src="http://img.janhen.com/202101301727211553493883332.png" alt="1553493883332" style="zoom:67%;" />



## Executor 框架

Executor 框架的结构：

（1） Executor：一个接口，其定义了一个接收Runnable对象的方法execute，其
方法签名为void execute(Runnable command)；
（2）ExecutorService：是一个比Executor使用更广泛的子类接口，其提供了生命
周期管理的方法，以及可跟踪一个或多个异步任务执行状况返回Future的方法；
（3）AbstractExecutorService：ExecutorService执行方法的默认实现；
（4）ScheduledExecutorService：一个可定时调度任务的接口；
（5）ThreadPoolExecutor：线程池，可以通过调用Executors以下静态工厂方法
来创建线程池并返回一个ExecutorService对象：
（6）ScheduledThreadPoolExecutor：ScheduledExecutorService的实现，一个
可定时调度任务的线程池；



### **自带的线程池**

一般不推荐使用，建议自定义线程池

（1） newCachedThreadPool

创建一个可根据需要创建新线程的线程池，但是在以前构造的线程可用时将重用它们。对于执行很多短期异步任务的程序而言，这些线程池通常可提高程序性能。调用 execute 将重用以前构造的线程（如果线程可用）。如果现有线程没有可用的，则创建一个新线程并添加到池中。终止并从缓存中移除那些已有 60 秒钟未被使用的线程。因此，长时间保持空闲的线程池不会使用任何资源。

SynchronousQueue 吞吐量更大；

可能因为线程数量过多而 OOM；

```java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

（2） newFixedThreadPool

创建一个可重用固定线程数的线程池，以共享的无界队列方式来运行这些线程。在任意点，在大多数 nThreads 线程会处于处理任务的活动状态。如果在所有线程处于活动状态时提交附加任务，则在有可用线程之前，附加任务将在队列中等待。如果在关闭前的执行期间由于失败而导致任何线程终止，那么一个新线程将代替它执行后续的任务（如果需要）。在某个线程被显式地关闭之前，池中的线程将一直存在。

无界的 LinkedBlockingQueue，可能因为工作数太多而 OOM；

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
```

（3） newSingleThreadExecutor

这个线程池可以在线程死后（或发生异常时）重新启动一个线程来替代原来的线程继续执行下去！

借助单线程特性，持有阻塞队列，实现按序执行的处理。


（4） newScheduledThreadPool

创建一个线程池，它可安排在给定延迟后运行命令或者定期地执行。

① 指定延后执行；

② 给定延迟后按照给定周期进行执行；





### *线程池工作过程

> 1、线程池刚创建时，里面没有一个线程。任务队列是作为参数传进来的。不过，就算队列里面有任务，线程池也不会马上执行它们。

> 2、当调用 execute() 方法添加一个任务时，线程池会做如下判断： 
>
> a) 如果正在运行的线程数量小于 corePoolSize，那么马上创建线程运行这个任务； 
>
> b) 如果正在运行的线程数量大于或等于 corePoolSize，那么将这个任务放入队列； 
>
> c) 如果这时候队列满了，而且正在运行的线程数量小于 maximumPoolSize，那么还是要创建非核心线程立刻运行这个任务； 
>
> d) 如果队列满了，而且正在运行的线程数量大于或等于 maximumPoolSize，那么线程池会抛出异常RejectExecutionException。 

> 3、 当一个线程完成任务时，它会从队列中取下一个任务来执行。

> 4、一个线程无事可做，超过一定的时间（keepAliveTime）时，线程池会判断，如果当前运行的线程数大于 corePoolSize，那么这个线程就被停掉。所以线程池的所有任务完成后，它最终会收缩到 corePoolSize 的大小。

![1553499764484](http://img.janhen.com/202101301727241553499764484.png)



**submit() 的问题**

使用submit(Runnable task) 的时候，错误的堆栈信息跑出来的时候会被内部捕获到，所以打印不出来具体的信息让我们查看，解决的方法有如下两种：

（1） 处理取不出错误堆栈问题

① 使用execute（）代替submit（）；

② 使用 Future 接受，之后进行获取捕获异常实现；



（2） execute 与 submit() 的区别

① execute()方法用于提交不需要返回值的任务，所以无法判断任务是否被线程池执行成功。通过以下代码可知execute()方法输入的任务是一个Runnable类的实例。
② submit()方法用于提交需要返回值的任务。线程池会返回一个future类型的对象，通过这个future对象可以判断任务是否执行成功，并且可以通过future的get()方法来获取返回值，get()方法会阻塞当前线程直到任务完成，而使用get（long timeout，TimeUnit unit）方法则会阻塞当前线程一段时间后立即返回，这时候有可能任务没有执行完。





### **两级调度模型**

何为Executor框架的两级调度模型，简单的来说就是我们使用的**Java线程被一对一映射为本地操作系统线程**。Java线程启动的时候会启动一个本地操作系统线程，当该Java线程终止时，这个操作系统线程也会被回收。操作系统会调度所有线程并将它们分配给可用的CPU。
由此出现了应用程序通过Executor框架控制上层的调用，而下层的调度由操作系统的内核来控制，从而形成了两级的调度模型，并且下层的调度不受应用程序的控制，任务的两级调度模型如下图：

![1553839726394](http://img.janhen.com/202101301727271553839726394.png)



### **结构和使用流程**

![1553839790923](http://img.janhen.com/202101301727301553839790923.png)

1、主线程首先要创建实现Runnable或Callable接口的任务对象。工具类Executors可
以把一个Runnable对象封装成为一个Callable对象（`Executors.callable(Runnable task)` 或者`Executors.callable(Runnable task, Object result)`） ；
2、然后可以把Runnable对象直接交给ExecutorService执行（ `ExecutorService.execute(Runnable command)`） ，或者也可以把Runnable对象或Callable对象交给ExecutorService执行`（ExecutorService.submit(Runnable task) 或
ExecutorService.submit(Callable t)）` 。
3、如果执行submit方法，ExecutorService将返回一个实现Future接口的对象（到目前为止的JDK中返回的是FutureTask对象）。由于FutureTask实现了Runnable接口，程序员也可以创建FutureTask，然后直接交给ExecutorService执行。
4、最后主线程可以执行FutureTask.get()方法来等待任务执行完成。主线程也可以执行`FutureTask.cancel(boolean matInterruptIfRunning)` 来取消此任务的执行。





## ThreadPoolExecutor

线程池好处：

- 降低资源损耗： 重复利用已创建的线程降低创建和销毁的开销
- 提高响应速度： 相当于预初始化了一些线程，不需要等到线程创建再执行
- 便于监控管理与扩展： 若使用一个请求对应一个线程方式无线创建线程，会消耗系统资源，还会降低线程的稳定性。使用线程池可以统一进行分配、调优和监控。



### **底层结构**

（1） 基础结构

```java
final ReentrantLock mainLock = new ReentrantLock();
final HashSet<Worker> workers = new HashSet<Worker>();
final Condition termination = mainLock.newCondition();
int largestPoolSize;
long completedTaskCount;
```



（2） 状态 | 生命周期

- RUNNING:
- SHUTDOWN:    调用 shutdown() 实现 RUNNING -> SHUTDOWN
- STOP:     调用 shutdownNow() 可实现 RUNNING -> STOP, SHUTDOWN -> STOP
- TIDYING:   queue AND pool 为空
- TERMINATED:   terminated() 钩子方法被调用, 一般随着 Web 容器的销毁而关闭

```java
static final int RUNNING    = -1 << COUNT_BITS;
static final int SHUTDOWN   =  0 << COUNT_BITS;             
static final int STOP       =  1 << COUNT_BITS;
static final int TIDYING    =  2 << COUNT_BITS;
static final int TERMINATED =  3 << COUNT_BITS;
```



（3） 配置参数

```java
final BlockingQueue<Runnable> workQueue;  
volatile ThreadFactory threadFactory;
volatile RejectedExecutionHandler handler;
volatile long keepAliveTime;
volatile boolean allowCoreThreadTimeOut;
volatile int corePoolSize;
volatile int maximumPoolSize;
```



（4） 工作线程

基于 AQS 实现的，内部持有 Thread 和 Runnable

```java
class Worker extends AbstractQueuedSynchronizer
    implements Runnable
    final Thread thread;
    Runnable firstTask;
    volatile long completedTasks;
}
```





**方法**

（1） execute

进入 RUNNING 状态

（2） submmit

（3） shutdown

处理已提交的, 丢弃新来的任务

（4） shutdownNow

丢弃已提交的任务:
返回的为丢弃的任务??





### **线程池参数**

（1） corePoolSize

（2） maxPoolSize

CPU 密集型 NCPU + 1

IO 密集型 2*CPU | 

（3） BlockingQueue

共7 种 BlockingQueue，分为有界和无界的阻塞队列；

（4） RejectedExecutionHandler

（5） ThreadFactory

为线程确定池子中线程的优先级、名称、守护线程，未捕获异常的处理策略；

区分模块；

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.acc = System.getSecurityManager() == null ?
            null :
            AccessController.getContext();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```



**拒绝策略**

提供四种拒绝策略供选择

① AbortPolicy(default)： 直接抛出异常

② DiscardPolicy： 直接抛弃，什么都不做

③ CallerRunsPolicy： 交给调用者取处理

④ DiscardOldestPolicy： 丢弃最老的任务，入队等待处理

一般自己自定义拒绝策略，通过日志记录线程池的情况；



**阻塞队列**

![1553499807030](http://img.janhen.com/202101301727361553499807030.png)

**线程池使用情况监控**

```java
void beforeExecute(Thread t, Runnable r)
void afterExecute(Runnable r, Throwable t)
/* Statistics */
...
```





## ScheduledThreadPoolExecutor

定时调度任务，比 Timer实现效率高





## Callable | Future | FutureTask

**Callable**

Callable<V>接口类似于Runnable,两者都是为了哪些其实例可能被另一个线程执行的类设计的， 



（）与 Runnable 的比较

Runnable不会返回结果，并且无法抛出异常。 

```java
interface Callable<V> {
	V call() throws Exception;
}
```





**Future：**

在本次实例代码中还用到Future<V>接口 
      Future<V>接口表示 **异步计算** 的结果。它提供了检查计算是否完成的方法，以等待计算的完成， 
      并获取计算的结果。计算完成后只能使用get()方法来获取结果，如有必要计算完成前可以阻塞此方法。 
      取消则由cancel方法来执行。 
      实现此接口需要重写get()方法： 
          V  get() throws InterruptedException,ExecutionException 

（1） 查看状态

```java
boolean cancel(boolean mayInterruptIfRunning);
boolean isCancelled();
boolean isDone();
```



（2） 获取异步任务结果

```java
V get() throws InterruptedException, ExecutionException;
V get(long timeout, TimeUnit unit)
    throws InterruptedException, ExecutionException, TimeoutException;
```



综合 Future、Callable 两者的功能，异步任务的结果获取，出入异步的任务；

**底层结构**

```java
class FutureTask<V> implements RunnableFuture<V>
    volatile int state;
    Callable<V> callable;
    Object outcome; 
    volatile Thread runner;
    volatile WaitNode waiters;
}
```



（） 状态

```java
static final int NEW          = 0;
static final int COMPLETING   = 1;
static final int NORMAL       = 2;
static final int EXCEPTIONAL  = 3;
static final int CANCELLED    = 4;
static final int INTERRUPTING = 5;
static final int INTERRUPTED  = 6;
```





**方法**

（1） 监控方法

是 Future 接口提供

```java
boolean isCancelled()
boolean isDone()
boolean cancel(boolean mayInterruptIfRunning)
```



（2） 获取异步任务结果

提高两种方式获得，阻塞获得，超时获得；

```java
V get() 
V get(long timeout, TimeUnit unit)
```

