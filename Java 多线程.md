# Java 多线程

## AbstractQueuedSynchronizer详解

[Java AQS源码解读|掘金]: https://juejin.cn/post/6844904035862986765

------

**独占锁：**

实现方法：

- ReentrantLock

- ReentrantReadWriteLock.WriteLock

实现策略：

- tryAcquire(int)

- tryRelease(int)

- isHeldExclusively()

**共享锁：**

实现方法：

- CountDownLatch
- ReentrantReadWriteLock.ReadLock

- Semaphore

实现策略：

- tryAcquireShared(int)
- tryReleaseShared(int)

------

我们可以猜测出，AQS其实主要做了这么几件事情：

- 同步状态（state）的维护管理
- 等待队列的维护管理
- 线程的阻塞与唤醒

------

源码解读

```java
/**
 * 获取独占锁，忽略中断。
 * 首先尝试获取锁，如果成功，则返回true；否则会把当前线程包装成Node插入到队尾，在队列中会检测是否为head的直接后继，并尝试获取锁,
 * 如果获取失败，则会通过LockSupport阻塞当前线程，直至被释放锁的线程唤醒或者被中断，随后再次尝试获取锁，如此反复。被唤醒后继续之前的代码执行
 */
public final void acquire(int arg) {
	//tryAcquire(arg)：尝试获取锁，当成功时直接返回。失败则继续执行接下来的方法
	//addWaiter(Node.EXCLUSIVE)：添加由当前线程构造的独占模式的节点到CLH同步队列中
	//acuqireQueued(node,arg)：再挣扎一下。尝试自选获取锁，获取不到则老实进入CLH等待队列并堵塞
  if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg)){
    selfInterrupt();
  }
}
```


