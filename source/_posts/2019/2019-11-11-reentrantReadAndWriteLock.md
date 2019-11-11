---
layout: post
title: ReentrantReadAndWriteLock的实现原理
category: 并发
tags: thread
date: 2019-11-11
---
### 读写锁和普通锁的区别
读写锁可以并发读，普通锁只能串行读取。但读写锁不能并发写，写和读也不能同时存在。

### 演示实例
```java
    public static ReentrantReadWriteLock lock = new ReentrantReadWriteLock();

//    public static ReentrantLock lock = new ReentrantLock();
    public static Lock readLock = lock.readLock();
    public void handleRead() throws InterruptedException {
        readLock.lock();
        System.out.println("====");
        Thread.sleep(1000);
        readLock.unlock();

//        lock.lock();
//        Thread.sleep(1000);
//        lock.unlock();
    }
   @Test
    public void testContext() throws Exception {
        CountDownLatch countDownLatch = new CountDownLatch(10);
        long startTime = System.currentTimeMillis();
        ExecutorService executorService = Executors.newFixedThreadPool(10);
        for (int i=0;i<10;i++) {
            executorService.submit(()-> {
                try {
                    handleRead();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                countDownLatch.countDown();
                    }
            );
        }
executorService.shutdown();
        countDownLatch.await();
        long endTime = System.currentTimeMillis();
        System.out.println("over"+ (endTime-startTime));
}

```
输出结果：
```java
over1026
```
打开注释代码，屏蔽read锁代码，运行结果为：
```java
over10031
```
从结果上可以看出读锁的效率大大增加。

### ReadLock实现原理
介绍ReadLock前，先看下相关参数的意义。
```java
//读写锁用state记录两种锁的状态
// 高16位来记录读锁(共享锁)的state，低16记录写锁(独占锁)的state
//读锁占用的位数
static final int SHARED_SHIFT   = 16;
//每次让读锁状态加1， 则让state加1<<16
static final int SHARED_UNIT    = (1 << SHARED_SHIFT);
//最大可重入数，也就是int_MAX
static final int MAX_COUNT      = (1 << SHARED_SHIFT) - 1;
//写锁标记，作getState&EXCLUSIVE_MASK 大于1就是重入了
static final int EXCLUSIVE_MASK = (1 << SHARED_SHIFT) - 1;

//获取读锁的重入数
static int sharedCount(int c)    { return c >>> SHARED_SHIFT; }
//获取写锁的重入数
static int exclusiveCount(int c) { return c & EXCLUSIVE_MASK; }
```
举个例子：比如，现在当前，申请读锁的线程数为13个，写锁1个，那state怎么表示？
上文说过，用一个32位的int类型的高16位表示读锁线程数，13的二进制为 1101,那state的二进制表示为
00000000 00001101 00000000 00000001，十进制数为851969， 接下在具体获取锁时，需要根据这个851968这个值得出上文中的 13 与 1。要算成13，只需要将state 无符号向右移位16位置，得出00000000 00001101，就出13，根据851969要算成低16位置，只需要用该00000000 00001101 00000000 00000001 & 111111111111111（15位），就可以得出00000001,就是利用了1&1得1,1&0得0这个技巧。

#### 构造函数
ReentrantReadWriteLock默认是非公平锁
```java
    public ReentrantReadWriteLock() {
        this(false);
    }
    public ReentrantReadWriteLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
        readerLock = new ReadLock(this);
        writerLock = new WriteLock(this);
    }
```

![Lock流程图]((/image/2019-11-11-reentrantReadAndWriteLock-01.png)

#### Lock函数
```java
public void lock() {
    sync.acquireShared(1);
}
```
进一步看代码
```java
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}
```
- tryAcquireShared()是尝试去获取锁，如果获取成功，if不成立，直接返回
- doAcquireShared 如果获取不到锁加入阻塞队列

1、tryAcquireShared函数
```java
protected final int tryAcquireShared(int unused) {
    Thread current = Thread.currentThread();
    int c = getState(); //获取锁状态，即上述中提到的高位表示读线程个数，低位表示写线程个数
    //exclusiveCount 判断当前读线程数是否为0(判断方法：为右移16位得到高位数)
    //getExclusiveOwnerThread 判断当前独占锁是否为当前线程（如写线程正在进行写操作）
    if (exclusiveCount(c) != 0 &&
        getExclusiveOwnerThread() != current) 
        return -1;
    //获取锁状态
    int r = sharedCount(c);
    //read操作是否应该堵塞
    //锁状态是否超出最大限制
    //利用CAS进行锁状态高位加一
    if (!readerShouldBlock() &&
        r < MAX_COUNT &&
        compareAndSetState(c, c + SHARED_UNIT)) {
        //如果第一个线程 HoldCount是为了减少readHolds.get()的使用
        if (r == 0) {
            firstReader = current;
            firstReaderHoldCount = 1;
        } else if (firstReader == current) { //如果当前线程为重入线程，则firstReaderHoldCount++
            firstReaderHoldCount++;
        } else {
            HoldCounter rh = cachedHoldCounter;
            if (rh == null || rh.tid != getThreadId(current))
                cachedHoldCounter = rh = readHolds.get(); //使用ThreadLocal为每个线程保持一个线程计数器 
            else if (rh.count == 0)
                readHolds.set(rh);
            rh.count++; //HoldCounter为静态对象，rh++会使HoldCounter中的count+1 （如：非第一个线程重入）
        }
        return 1;
    }
    return fullTryAcquireShared(current);//完整版本的获取读，处理CAS脱靶 可重入读取在tryacquirered中没有处理
}
```
1.1、fullTryAcquireShared函数
完整版本的获取读，处理CAS脱靶 可重入读取在tryacquirered中没有处理,和TryAcquireShared 代码有冗余，利用for循环防止脱靶
```java
final int fullTryAcquireShared(Thread current) {

    HoldCounter rh = null;
    for (;;) {
        int c = getState();
        if (exclusiveCount(c) != 0) {
            if (getExclusiveOwnerThread() != current)
                return -1;
            // else we hold the exclusive lock; blocking here
            // would cause deadlock.
        } else if (readerShouldBlock()) {
            // Make sure we're not acquiring read lock reentrantly
            if (firstReader == current) {
                // assert firstReaderHoldCount > 0;
            } else {
                if (rh == null) {
                    rh = cachedHoldCounter;
                    if (rh == null || rh.tid != getThreadId(current)) {
                        rh = readHolds.get();
                        if (rh.count == 0)
                            readHolds.remove();
                    }
                }
                if (rh.count == 0)
                    return -1;
            }
        }
        if (sharedCount(c) == MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        if (compareAndSetState(c, c + SHARED_UNIT)) {
            if (sharedCount(c) == 0) {
                firstReader = current;
                firstReaderHoldCount = 1;
            } else if (firstReader == current) {
                firstReaderHoldCount++;
            } else {
                if (rh == null)
                    rh = cachedHoldCounter;
                if (rh == null || rh.tid != getThreadId(current))
                    rh = readHolds.get();
                else if (rh.count == 0)
                    readHolds.set(rh);
                rh.count++;
                cachedHoldCounter = rh; // cache for release
            }
            return 1;
        }
    }
}
```
2、doAcquireShared(arg)函数
当线程被阻塞时，执行此方法，把线程加入阻塞队列
```java
private void doAcquireShared(int arg) {
    final Node node = addWaiter(Node.SHARED); //加入阻塞队列，返回新节点
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor(); //获取前一个节点
            //如果是头节点 尝试获取锁
            if (p == head) { 
                int r = tryAcquireShared(arg);
                if (r >= 0) {
                    setHeadAndPropagate(node, r); //设置头节点和传播属性值
                    p.next = null; // 头结点设置为空，方便GC
                    if (interrupted)
                        selfInterrupt();
                    failed = false;
                    return;
                }
            }
            //设置前置节点状态，保证前置节点释放后，能唤醒当前节点
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```
2.1、 shouldParkAfterFailedAcquire函数
```java
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    int ws = pred.waitStatus;
    //  如果他的上一个节点的 ws 是 SIGNAL，他就需要阻塞。
    if (ws == Node.SIGNAL)
        // 阻塞
        return true;
    // 前任被取消。 跳过前任并重试。
    if (ws > 0) {
        do {
            // 将前任的前任 赋值给 当前的前任
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        // 将前任的前任的 next 赋值为 当前节点
        pred.next = node;
    } else { 
        // 如果没有取消 || 0 || CONDITION || PROPAGATE，那么就将前任的 ws 设置成 SIGNAL.
        // 为什么必须是 SIGNAL 呢？
        // 答：希望自己的上一个节点在释放锁的时候，通知自己（让自己获取锁）
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    // 重来
    return false;
}
```
该方法的主要逻辑就是将前置节点的状态修改成 SIGNAL。其中如果前置节点被取消了，就跳过他。

那么肯定，在前置节点释放锁的时候，肯定会唤醒这个节点。