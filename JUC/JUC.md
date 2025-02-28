# JUC

## 一. 概念及内容

JUC 指 JDK 中的 `java.util.concurrent` 包，提供了一些用于并发编程的工具和类，使得多线程编程变得更加简洁和安全。JUC 包包括了各种线程同步、执行器、并发集合等内容。主要分为以下几个部分：

### 并发集合类

并发集合是为了在多线程环境下操作集合时提供更好的性能和安全性。常见的并发集合类有：

- **`CopyOnWriteArrayList`** 和 **`CopyOnWriteArraySet`**：这两种集合是线程安全的，它们通过复制数组的方式来保证写操作不会影响正在进行的读操作，因此适用于读多写少的场景。
- **`ConcurrentHashMap`**：一个线程安全的哈希表，支持高效的并发操作。相比于`HashMap`，它在进行插入、删除、查找等操作时，不需要加锁整个表格，性能较好。
- `BlockingQueue`：常用的阻塞队列接口及其实现类。主要有：
  - **`ArrayBlockingQueue`**：基于数组的阻塞队列。
  - **`LinkedBlockingQueue`**：基于链表的阻塞队列，适合生产者-消费者模式。
  - **`PriorityBlockingQueue`**：优先级队列，线程按优先级获取元素。
  - **`DelayQueue`**：支持延时任务的队列。

### 锁

JUC包中的锁比传统的`sychronized`关键字更为灵活、功能更强大。常见的锁类包括：

- **`ReentrantLock`**：一个可重入的显式锁，它提供了比`synchronized`更多的功能，例如尝试锁（`tryLock()`）和定时锁（`lock(long time, TimeUnit unit)`）。
- **`ReentrantReadWriteLock`**：一个读写锁，允许多个线程同时读取，但在写锁被持有时，不能有任何线程进行读取或写入。
- **`StampedLock`**（JDK 1.8新增）：提供了比`ReentrantReadWriteLock`更高效的读写锁实现，支持乐观读模式。

### 执行器（Executors）

`Executor`框架简化了多线程的管理，常见的类有：

- `ExecutorService`：一个线程池接口，提供了比 Thread 类更灵活的线程管理方式。主要的实现类有：
  - **`ThreadPoolExecutor`**：线程池的核心实现，支持线程池大小的动态调整等。
  - **`ScheduledThreadPoolExecutor`**：支持定时任务调度的线程池。
- `Executors`：工厂类，用于创建常见的线程池。常用的工厂方法有：
  - `newFixedThreadPool(int n)`：创建一个固定线程数的线程池。
  - `newCachedThreadPool()`：创建一个可缓存的线程池，线程池的大小根据任务数量变化。
  - `newSingleThreadExecutor()`：创建一个单线程池。
  - `newScheduledThreadPool(int corePoolSize)`：创建一个定时调度线程池。

### 并发工具类

- **`CountDownLatch`**：一个同步工具类，允许一个或多个线程等待其他线程完成一项操作。通常用于多线程任务完成的协调。
- **`CyclicBarrier`**：一个屏障，多个线程可以到达一个屏障点后一起继续执行，适用于线程之间的同步。
- **`Semaphore`**：一种计数信号量，用于控制对共享资源的访问权限，可以用来限制同时访问某些资源的线程数量。
- **`Exchanger`**：用于两个线程之间交换数据的工具类，两个线程可以通过`exchange()`方法交换数据。
- **`Phaser`**（JDK 1.8新增）：类似于`CyclicBarrier`和`CountDownLatch`，但它更加灵活，支持更复杂的多阶段同步。

### 原子类

`java.util.concurrent.atomic`包提供了一些线程安全的原子变量类，用于实现无锁的线程安全操作。常见的原子类有：

- **`AtomicInteger`**、**`AtomicLong`**、**`AtomicBoolean`**、**`AtomicReference`**：用于对基本类型（如`int`、`long`、`boolean`）和对象进行原子操作（如`get()`、`set()`、`incrementAndGet()`等）。
- **`AtomicIntegerArray`** 和 **`AtomicReferenceArray`**：支持原子操作的数组类。

### ForkJoinPool

`ForkJoinPool`是 JDK 1.7 引入的一个线程池，设计用于并行任务的分治处理。它能够将一个大任务分解成多个小任务并行处理，然后将结果汇总。它特别适用于可以被递归分解的任务，如大规模的数据处理、图像处理等。

## 二. 内容详解

### Executors 类

`Executors` 类提供了一些静态工厂方法，用于创建不同类型的线程池。线程池的作用是管理和复用线程，避免频繁地创建和销毁线程，从而提升多线程程序的性能。`Executors` 类提供的几种常见线程池：

#### newFixedThreadPool(int n)

~~~java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
}
~~~

创建一个固定大小的线程池，线程池中有固定数量的线程，所有提交的任务都将由这些线程来执行。如果有任务提交，而线程池中的线程都在忙碌中，那么新的任务将会进入等待队列，直到有线程空闲出来。

特点：

- 线程池大小固定。
- 适用于负载均衡的任务。
- 当任务数较多时，队列可能会变得非常大（无界阻塞队列）。

#### newCachedThreadPool()

~~~java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
~~~

创建一个可缓存的线程池，线程池的大小是可以动态调整的。如果线程池中有空闲的线程，可以重用它们；如果没有空闲线程且任务数较多，则会创建新的线程来处理任务。线程空闲超过一定时间后销毁。

特点：

- 线程池大小不固定。
- 适用于任务量变化较大或者执行时间较短的任务。
- 当有大量短时间任务时，线程池中可能会创建大量线程。

`SynchronousQueue`：是一个特殊的阻塞队列，不会存储任何元素。每次提交任务时，必须有一个线程能够立即执行该任务，否则任务会被阻塞，直到有线程可用为止。它非常适合用于 **任务量动态变化** 的场景，尤其是当每个任务的执行时间较短且线程池中的线程数量不确定时。特点：

- **无缓存**： `SynchronousQueue` 本身不存储任何元素，它是一个“零容量”队列。当线程调用 `put()` 方法时，它会等待另一个线程调用 `take()` 方法来接收任务。反之亦然，当线程调用 `take()` 方法时，它会等待另一个线程调用 `put()` 方法来提供任务。因此，队列不会保存任务，必须有线程立刻消费任务。
- **每个任务都有一个线程来处理**： 每个任务在提交时都需要立刻有线程可用。如果线程池中没有空闲线程（即没有线程可以执行新的任务），任务就会被阻塞，直到有线程可用。这个特性使得 `newCachedThreadPool()` 在面对快速增长的任务时，能够灵活地创建新线程来处理任务。
- **线程池大小动态调整**： `SynchronousQueue` 配合 `newCachedThreadPool()` 使用时，线程池中的线程数量是动态变化的。它会根据任务的数量来增加线程数。只要有任务需要执行，`newCachedThreadPool()` 会创建一个新线程并立即执行该任务。线程在空闲超过一定时间后会被销毁，从而避免过多的资源浪费。
- **不会将任务保存在队列中**： 由于 `SynchronousQueue` 不存储任何任务，它的工作方式与一般的队列（如 `LinkedBlockingQueue`）不同，后者会缓存提交的任务，直到有线程可以执行。因此，`SynchronousQueue` 更加关注任务的及时执行。**
- **适用于高并发、小任务的场景**： 由于线程池的线程数量会随着任务的增加而增加，而线程在任务执行完毕后会被销毁，因此这种队列适用于任务量波动较大且任务执行时间较短的场景。例如，短小的异步任务，或者需要在短时间内并发执行大量任务的场景。

#### newSingleThreadExecutor()

~~~java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
~~~

创建一个单线程的线程池，线程池中只有一个线程。所有提交的任务将由这一个线程按顺序执行，保证任务的顺序性。如果有多个任务，它们会被放入任务队列中，依次执行。

**特点**：

- 线程池只有一个线程，任务串行执行。
- 适用于需要按顺序执行的任务。

#### newScheduledThreadPool(int corePoolSize)

~~~java
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}
public ScheduledThreadPoolExecutor(int corePoolSize) {
    super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
          new DelayedWorkQueue());
}
~~~

创建一个支持定时和周期性任务执行的线程池。可以安排任务在指定的延迟后执行，或者按照固定的频率执行。

**特点**：

- 适用于定时任务或周期性任务。
- 线程池大小可调（指定核心池大小）。

#### newWorkStealingPool()（JDK 1.8 引入）

~~~java
public static ExecutorService newWorkStealingPool(int parallelism) {
    return new ForkJoinPool
        (parallelism,
         ForkJoinPool.defaultForkJoinWorkerThreadFactory,
         null, true);
}
public ForkJoinPool(int parallelism,
                        ForkJoinWorkerThreadFactory factory,
                        UncaughtExceptionHandler handler,
                        boolean asyncMode) {
    this(checkParallelism(parallelism),
         checkFactory(factory),
         handler,
         asyncMode ? FIFO_QUEUE : LIFO_QUEUE,
         "ForkJoinPool-" + nextPoolId() + "-worker-");
    checkPermission();
}
~~~

创建一个工作窃取线程池，该线程池使用一种基于工作窃取算法的线程池策略。如果某个线程池中的任务较少，它会从其他池中“窃取”任务来执行。适用于任务数量不均匀的场景。

**特点**：

- 使用工作窃取算法进行任务分配。
- 适合高吞吐量的任务，能平衡任务的负载。

### 锁

JUC 提供了比传统的 `synchronized` 更灵活、更高效的控制机制。

#### ReentrantLock

`ReentrantLock` 是一种可重入的互斥锁，提供比 `synchronized` 更灵活的锁定功能，具有以下特点：

- **可重入性**：同一个线程可以多次获得锁，而不会导致死锁。它会记录获取锁的次数，锁会被释放的次数与获取的次数匹配时才会真正释放。
- **可中断性**：`ReentrantLock` 可以在等待锁时被中断，而 `synchronized` 无法中断等待的线程。
- **公平性**：`ReentrantLock` 提供了公平锁（`fair = true`）的选择。公平锁保证线程获取锁的顺序与请求锁的顺序一致，避免了线程饥饿问题。
- **锁的条件变量**：`ReentrantLock` 与 `Condition` 对象配合使用，能够实现复杂的线程同步行为。

`ReentrantLock` 与 `synchronized` 对比：

| 特性               | `ReentrantLock`         | `synchronized`           |
| ------------------ | ----------------------- | ------------------------ |
| **可重入性**       | ✅ 是                    | ✅ 是                     |
| **是否可中断**     | ✅ `lockInterruptibly()` | ❌ 不可中断               |
| **公平锁支持**     | ✅ 支持                  | ❌ 仅非公平               |
| **Condition 支持** | ✅ 支持多个 `Condition`  | ❌ 只能 `wait()/notify()` |
| **性能**           | ✅ 更高效（自旋 + CAS）  | ❌ 低（JVM 内部实现）     |

**推荐使用：**

- **简单场景**：用 `synchronized`，JVM 优化后开销不大。
- **复杂同步需求（可中断、公平锁、多个条件变量）**：用 `ReentrantLock`。

源码分析：

- 可重入：

  ~~~java
  private final Sync sync;
  ~~~

  `Sync` 继承 `AbstractQueuedSynchronizer`（AQS）。

  AQS 维护了 `state` 变量，表示锁的持有状态：

  - `state == 0`：锁空闲，可获取。

  - `state > 0`：已被某个线程持有，若同一线程再次获取，`state` 递增。

    ~~~java
    protected final boolean tryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            if (compareAndSetState(0, acquires)) { // CAS 更新 state
                setExclusiveOwnerThread(current);
                return true;
            }
        } else if (current == getExclusiveOwnerThread()) { // 当前线程已持有锁
            int nextc = c + acquires;
            setState(nextc);
            return true;
        }
        return false;
    }
    ~~~

  - **CAS（Compare-And-Swap）机制** 确保只有一个线程能成功获取锁，避免并发问题。

  - **持有锁的线程可以再次获取锁，并递增 `state` 计数**。 

- 可中断

  - `lockInterruptibly()` 允许线程等待锁时被 `interrupt()` 终止。

    ~~~java
    public void lockInterruptibly() throws InterruptedException {
        sync.acquireInterruptibly(1);
    }
    ~~~

    `AQS` 相关实现：

    ~~~java
    public final void acquireInterruptibly(int arg) throws InterruptedException {
        if (Thread.interrupted()) // 如果线程被中断，直接抛异常
            throw new InterruptedException();
        if (!tryAcquire(arg)) // 如果获取不到锁，就进入等待队列
            doAcquireInterruptibly(arg);
    }
    ~~~

- 公平锁 & 非公平锁

  ~~~java
  public ReentrantLock(boolean fair) {
      sync = fair ? new FairSync() : new NonfairSync();
  }
  ~~~

  - **公平锁**：线程获取锁按照**先来先得**的顺序执行（FIFO）；先检查等待队列中有没有排队的线程 (`hasQueuedPredecessors()`)，如果有，就不能插队。

    ~~~java
    static final class FairSync extends Sync {
        protected final boolean tryAcquire(int acquires) {
            Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (!hasQueuedPredecessors() && compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            return false;
        }
    }
    ~~~

  - **非公平锁**（默认）：线程可以**直接尝试获取锁**，提高吞吐量，但可能造成线程饥饿。

    ~~~java
    static final class NonfairSync extends Sync {
        protected final boolean tryAcquire(int acquires) {
            if (compareAndSetState(0, acquires)) { // 直接 CAS 争抢锁
                setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }
            return false;
        }
    }
    ~~~

- `Condition` 变量

  `ReentrantLock` 支持多个 `Condition` 变量，可以精细控制线程的唤醒，而 `synchronized` 只能用 `wait()/notify()`。

  ~~~java
  public final void await() throws InterruptedException {
      if (Thread.interrupted())
          throw new InterruptedException();
      Node node = addConditionWaiter(); // 添加到等待队列
      fullyRelease(node); // 释放锁
      acquireQueued(node, 0); // 重新获取锁
  }
  ~~~

- 释放锁

  ~~~java
  protected final boolean tryRelease(int releases) {
      int c = getState() - releases;
      if (Thread.currentThread() != getExclusiveOwnerThread())
          throw new IllegalMonitorStateException();
      boolean free = false;
      if (c == 0) {
          free = true;
          setExclusiveOwnerThread(null); // 清空持有锁的线程
      }
      setState(c);
      return free;
  }
  ~~~

  - 递减 `state` 计数，直到 `state == 0`，彻底释放锁。
  - 唤醒等待队列中的线程，重新竞争锁。

#### ReadWriteLock、ReentrantReadWriteLock

ReadWriteLock 是一个接口，定义了读写锁的基本行为；ReentrantReadWriteLock 是 ReadWriteLock 的一个实现，可重入，并通过 readLock() 和 writeLock() 提供分别对读和写操作的锁定。

- **读操作**：多个线程可以同时访问缓存并且不会互相阻塞。
- **写操作**：每次只有一个线程可以修改缓存，其他线程必须等待写锁释放。

读锁（ReadLock）：

~~~java
public class ReadLock implements Lock {
    public void lock() {
        sync.acquireShared(1);  // 获取共享锁
    }
    
    public void unlock() {
        sync.releaseShared(1);  // 释放共享锁
    }
}
~~~

写锁（WriteLock）：

~~~java
public class WriteLock implements Lock {
    public void lock() {
        sync.acquireExclusive(1);  // 获取独占锁
    }
    
    public void unlock() {
        sync.releaseExclusive(1);  // 释放独占锁
    }
}
~~~

ReentrantReadWriteLock 特别适用于**读多写少**的场景。比如：

- 高并发的缓存系统

  在缓存系统中，通常会有大量的读取操作，尤其是对于一些热点数据。为了提高并发性能，可以使用 `ReentrantReadWriteLock` 来确保多个线程能够同时读取缓存，但更新缓存的写操作必须是独占的。

- 数据库管理系统的缓存

  在数据库缓存中，读操作非常频繁，而写操作较少。`ReentrantReadWriteLock` 可以用于对数据库表数据的缓存控制，提高查询效率。

- 配置文件读取

  配置文件通常被读取多次，而修改配置的操作相对较少。在这种场景下，使用读写锁可以让多个线程并行读取配置文件，而不需要每次读取时都加写锁。

- 共享资源的访问

  比如在一些实时计算系统中，多个线程需要对共享数据进行计算或查询，但这个共享数据的修改相对较少。`ReentrantReadWriteLock` 能够确保多个线程同时读取数据，但在有修改时会阻塞其他读线程，确保数据的一致性。

- 文件系统的读写管理

  当有大量的读操作需要频繁访问文件时，`ReentrantReadWriteLock` 是一个理想的选择。它可以确保读操作的并发性，同时当有写操作时，独占访问文件。

使用时的注意点：

- 写操作的饥饿问题

  - 如果写操作的频率较低，而读操作非常频繁，写线程可能会一直得不到执行，造成写饥饿。

  - 为了避免写饥饿，可以使用公平锁（`ReentrantReadWriteLock(true)`）来确保写操作有机会执行。公平锁的作用是保证请求锁的线程按照请求的顺序获得锁。

- 不适用于写操作频繁的场景

  - 如果读写操作几乎一样频繁或者写操作占主导地位，使用 `ReentrantReadWriteLock` 可能会导致性能下降，因为每次获取写锁时都必须等待所有的读锁释放，这可能会降低并发性能。

- 需要小心死锁

  - 如果多个线程交替获取读锁和写锁时，可能会导致死锁。例如，线程 A 获取了读锁，线程 B 获取了写锁，线程 A 再次尝试获取写锁，而线程 B 再次尝试获取读锁，这会导致死锁。

- 锁的持有时间

  - 如果持有锁的时间太长（特别是写锁），可能会阻塞其他线程，降低并发性能。因此，在使用读写锁时，要确保锁持有的时间尽量短。

#### StampedLock



#### Lock、Condition



#### 其他同步工具

