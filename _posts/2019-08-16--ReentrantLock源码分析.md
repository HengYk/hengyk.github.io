---
layout: post
title: ReentrantLock使用和源码分析
date: '2019-08-16 21:41'
description: "ReentrantLock使用和源码分析"
tag: 源码系列文章（JAVA-CODE）
---

##### 参考博文

[ReentrantLock使用和源码分析](https://liuzhengyang.github.io/2017/05/26/reentrantlock/)



##### 总体设计

```java
public class ReentrantLock implements Lock, java.io.Serializable {
    
    private final Sync sync;
    /**
     * Base of synchronization control for this lock. Subclassed
     * into fair and nonfair versions below. Uses AQS state to
     * represent the number of holds on the lock.
     */
    abstract static class Sync extends AbstractQueuedSynchronizer {
        abstract void lock();
        ...
    }
    static final class NonfairSync extends Sync {
        ...
    }
    static final class FairSync extends Sync {
        ...
    }

    public ReentrantLock() {
        sync = new NonfairSync();
    }

    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }

    public void lock() {
        sync.lock();
    }

    public boolean tryLock() {
        return sync.nonfairTryAcquire(1);
    }

    public void unlock() {
        sync.release(1);
    }
}
```



##### Sync设计

> state：记录了锁重入的次数
>
> compareAndSetState()：描述CAS(Compare and swap)操作，继承自AQS
>
> setExclusiveOwnerThread()：记录当前占用排它锁的线程，继承自AQS
>
> setState()：更新锁重入的次数，继承自AQS

```java
 final boolean nonfairTryAcquire(int acquires) {
     final Thread current = Thread.currentThread();
     int c = getState();
     if (c == 0) {
         if (compareAndSetState(0, acquires)) {
             setExclusiveOwnerThread(current);
             return true;
         }
     }
     else if (current == getExclusiveOwnerThread()) {
         int nextc = c + acquires;
         if (nextc < 0) // overflow
             throw new Error("Maximum lock count exceeded");
         setState(nextc);
         return true;
     }
     
     return false;
}
```

```java
 // AbstractQueuedSynchronizer
 protected final boolean compareAndSetState(int expect, int update)
 {
     // See below for intrinsics setup to support this
     return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
 }
```

```java
 // AbstractQueuedSynchronizer
 protected final void setExclusiveOwnerThread(Thread thread) {
     exclusiveOwnerThread = thread;
 }
```

```java
 // AbstractQueuedSynchronizer
 protected final void setState(int newState) {
     state = newState;
 }
```

```java
protected final boolean tryRelease(int releases) {
     int c = getState() - releases;
     if (Thread.currentThread() != getExclusiveOwnerThread())
         throw new IllegalMonitorStateException();
     boolean free = false;
     if (c == 0) {
         free = true;
         setExclusiveOwnerThread(null);
     }
     setState(c);
     return free;
}
```

##### NonfairSync设计

```java
 // AbstractQueuedSynchronizer
 static final class NonfairSync extends Sync {
     final void lock() {
         if (compareAndSetState(0, 1))
             setExclusiveOwnerThread(Thread.currentThread());
         else
             acquire(1);
         }
     protected final boolean tryAcquire(int acquires) {
         return nonfairTryAcquire(acquires);
     }
 }
```

##### FairSync设计

```java
    static final class FairSync extends Sync {

        final void lock() {
            acquire(1);
        }

        protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
    }
```

```java
 // 如果当前线程不是队列首部的第一个线程，则返回true
 // 如果当前线程是队列首部的第一个线程，则返回false
 public final boolean hasQueuedPredecessors() {
     Node t = tail;
     Node h = head;
     Node s;
     return h != t &&
     ((s = h.next) == null || s.thread != Thread.currentThread());
}
```
