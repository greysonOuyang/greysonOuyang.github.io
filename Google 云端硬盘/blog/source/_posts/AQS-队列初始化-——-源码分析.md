---
title: AQS 队列初始化 —— 源码分析
date: 2020-02-28 21:46:51
tags: 高并发
toc: true
---

假设如下情景：线程 T1 已经获取锁 ， 线程 T2 此时进来 ， 首先判断锁的状态 ， 锁已经被占有，则入队。

Node 类的设计：

```
public class Node {
	volatile Node prev;
  volatile Node next;
	volatile Thread thread;
	int ws;      // 当前 Node 的状态
}
```

<!-- more -->

## 入队过程：

1. 根据线程来实例化一个 Node

   ```
    private Node addWaiter(Node mode) {
            // 根据当前线程实例化一个Node
            Node node = new Node(Thread.currentThread(), mode);
            // Try the fast path of enq; backup to full enq on failure
            /**
             * 1. 将尾部赋给一个临时变量，注意此时tail为空，因为tail此时还没有指向Node对象
             * 2. tail == null，因此执行 enq(node) 方法，这个node是创建的当前线程的node
             */
            Node pred = tail;
            if (pred != null) {
                node.prev = pred;
                if (compareAndSetTail(pred, node)) {
                    pred.next = node;
                    return node;
                }
            }
            enq(node);
            return node;
        }
   ```

2. 再次实例化一个 Thread 为 null 的 Node ， 我们暂时称其为 NullThreadNode

   注意：AQS队列中队头所指向的Node的Thread永远为空

3. 把 NullThreadNode 设置为 AQS 队列的头部和尾部

4. 队列初始化

5. 维护链表关系（入队）

   ```
    private Node enq(final Node node) {
            // 死循环判断
            for (;;) {
                /**
                 * 第一次循环：
                 *  tail赋值给临时变量t， 注意此时tail仍然是null, 进入if块
                 * 调用compareAndSetHead(new Node()) 创建了一个新的节点 ，这个节点的结构如下：
                 *         Node() {    // Used to establish initial head or SHARED marker
                 *         }
                 *   该节点为空，也就是我们称为 NullThreadNode 的节点
                 *   此时队列里有了第一个Node
                 */
                Node t = tail;
                if (t == null) { // Must initialize
                    if (compareAndSetHead(new Node()))
                        tail = head;
                } else {
                    /**
                     * 第二次循环：在第一次循环时head赋给了tail,此时tail 不为空，
                     * 进入else块，将线程T1入队，也就是维护链表关系
                     */
                    node.prev = t;
                    if (compareAndSetTail(t, node)) {
                        t.next = node;
                        return t;
                    }
                }
            }
        }
   ```

6. 自旋一次 ，判断能不能拿到锁

   注意此处，只有队列中第二个 Node 才有自旋的资格，第三个及其以后的 Node 很显然前边有优先级更高的线程在等待锁。

   ```
    final boolean acquireQueued(final Node node, int arg) {
            boolean failed = true;
            try {
                boolean interrupted = false;
                for (;;) {
                    final Node p = node.predecessor();
    								// 判断上一个节点是否是头部，即当前节点为队列中第二个节点，tryAcquire()尝试获取锁
                    if (p == head && tryAcquire(arg)) {
                        setHead(node);
                        p.next = null; // help GC
                        failed = false;
                        return interrupted;
                    }
    								/**
                     *  判断在一次自旋加锁失败后是否需要睡眠
                     * 	自旋第一次时shouldParkAfterFailedAcquire(p, node)返回false, 不会进入if块
                     * 	自旋第二次时，shouldParkAfterFailedAcquire(p, node)返回true,
                     * 	此时进入parkAndCheckInterrupt()方法，此时线程阻塞在 parkAndCheckInterrupt()
                     */
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

   `shouldParkAfterFailedAcquire()`  源码如下：

   pred 是上一个节点，在此时也就是队列中的第一个节点，即上面我们称之为 NullThreadNode 的节点，每个节点的waitStatus初始值为0，SIGNAL默认值为1。此方法在第一次被调用时会进入 else 块，将 ws 设置为 -1，在第二次被调用时ws == SIGNAL，返回true。

   ```
    private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
            int ws = pred.waitStatus;     // ws默认值为0，SIGNAL默认值为1
            if (ws == Node.SIGNAL)
                return true;
            if (ws > 0) {
                do {
                    node.prev = pred = pred.prev;
                } while (pred.waitStatus > 0);
                pred.next = node;
            } else {
                compareAndSetWaitStatus(pred, ws, Node.SIGNAL); // CAS设置ws
            }
            return false;
        }
   ```

7. 线程阻塞在 `parkAndCheckInterrupt()` 方法中

上述就是一个队列的入队过程，我们再进一步研究下，如果同时有大量的线程并发执行企图获取锁的情况是怎样的呢？

我们假设此时还要一个线程 T3 企图获取锁，我们只关注第七个步骤，在执行到 `shouldParkAfterFailedAcquire()` 方法时，在第一次被调用时判断 ws = 0（注意 ws 是上一个节点的waitStatus，此时也就是第二个节点的ws），因此执行 else 块并且将 ws 设置为 -1，在第二次循环时ws 已经是 -1 ，所以执行第一个 if 块，返回true；线程 T3 阻塞。

多个线程的执行过程同理，即重复上述步骤。