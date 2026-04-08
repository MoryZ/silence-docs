# （九）并发处理

**1.【强制】获取单例对象需要保证线程安全，其中的方法也要保证线程安全。**

资源驱动类、工具类、单例工厂类都需要注意。

**2.【强制】创建线程或线程池时请指定有意义的线程名称，方便出错时回溯。**

1）创建单条线程时直接指定线程名称

正例：

    Thread t = new Thread();
    t.setName("cleanup-thread");

2）线程池则使用 Spring 或自行封装的`ThreadFactory`，指定命名规则。

正例：

    // Spring 或自行封装的 ThreadFactory
    ThreadFactory threadFactory = new CustomizableThreadFactory(threadNamePrefix);

    ThreadPoolExecutor executor = new ThreadPoolExecutor(..., threadFactory, ...);

**3.【强制】线程资源必须通过线程池提供，不允许在应用中自行显式创建线程。**

线程池的好处是减少在创建和销毁线程上所消耗的时间以及系统资源的开销，解决资源不足的问题。如果不使用线程池，有可能造成系统创建大量同类线程而导致消耗完内存或者“过度切换”的问题。

**4.【强制】线程池不允许使用`Executors`去创建，规避资源耗尽的风险。**

`Executors`返回的线程池对象的弊端如下：

1）`FixedThreadPool`和`SingleThreadPool`：

允许的*请求队列长度*为`Integer.MAX_VALUE`，可能会堆积大量的请求，从而导致 OOM。

2）`CachedThreadPool`：

允许的*创建线程数量*为`Integer.MAX_VALUE`，可能会创建大量的线程，从而导致 OOM。

2）`ScheduledThreadPool`：

允许的*请求队列长度*为`Integer.MAX_VALUE`，可能会堆积大量的请求，从而导致 OOM。

应通过`new ThreadPoolExecutor(xxx, xxx, xxx, xxx)`这样的方式，更加明确线程池的运行规则，合理设置 Queue 及线程池的 core size 和 max size。

推荐使用 Spring 中的`ThreadPoolExecutorFactoryBean`或`ThreadPoolTaskExecutor`创建线程池，结合 Spring 容器更好的管理线程任务的生命周期。

**5.【强制】正确停止线程。**

`Thread.stop()`不推荐使用，强行的退出太不安全，会导致逻辑不完整，操作不原子，该方法已被标记为`Deprecated`。

停止单条线程，执行`Thread.interrupt()`。

停止线程池：

-   `ExecutorService.shutdown()`: 不允许提交新任务，等待当前任务及队列中的任务全部执行完毕后退出；

-   `ExecutorService.shutdownNow()`: 通过`Thread.interrupt()`试图停止所有正在执行的线程，并不再处理还在队列中等待的任务。

最优雅的退出方式是先执行`shutdown()`，再执行`shutdownNow()`。

注意，`Thread.interrupt()`并不保证能中断正在运行的线程，需编写可中断退出的`Runnable`，见规则 6。

**6.【强制】编写可停止的Runnable。**

执行`Thread.interrupt()`时，如果线程处于`sleep()`，`wait()`，`join()`，`lock.lockInterruptibly()`等 blocking 状态，会抛出`InterruptedException`，如果线程未处于上述状态，则将线程状态设为 interrupted。

反例：如下的代码无法中断线程:

    public void run() {

        while (true) { // 无判断线程状态。
            sleep();
        }

        public void sleep() {
           try {
               Thread.sleep(1000);
           } catch (InterruptedException e) {
               logger.warn("Interrupted!", e); // WRONG，吃掉了异常，interrupt状态未再传递
           }
        }
    }

**6.1. 正确处理`InterruptException`。**

因为`InterruptException`异常是个必须处理的 Checked Exception，所以`run()`所调用的子函数很容易吃掉异常并简单的处理成打印日志，但这等于停止了中断的传递，外层函数将收不到中断请求，继续原有循环或进入下一个堵塞。

正例：正确处理是调用`Thread.currentThread().interrupt()`将中断往外传递。

    public void run() {

        while (true) { // WRONG，无判断线程状态。
            sleep();
        }

        public void sleep() {
           try {
               Thread.sleep(1000);
           } catch (InterruptedException e) {
               logger.warn("Interrupted!", e); // WRONG，吃掉了异常，interrupt状态未再传递
           }
        }
    }

-   [Sonar-2142: "InterruptedException" should not be ignored](https://rules.sonarsource.com/java/RSPEC-2142)

**6.2. 主循环及进入阻塞状态前要判断线程状态。**

正例：

    public void run() {
        try {
            while (!Thread.isInterrupted()) {
                // do stuff
            }
        } catch (InterruptedException e) {
            logger.warn("Interrupted!", e);
        }
    }

其他如`Thread.sleep()`的代码，在正式 sleep 前也会判断线程状态。

**7.【强制】Runnable中必须捕获一切异常。**

如果`Runnable`中没有捕获`RuntimeException`而向外抛出，会发生下列情况：

1.  `ScheduledExecutorService`执行定时任务，任务会被中断，该任务将不再定时调度，但线程池里的线程还能用于其他任务。

2.  `ExecutorService`执行任务，当前线程会中断，线程池需要创建新的线程来响应后续任务。

3.  如果没有在`ThreadFactory`设置自定义的`UncaughtExceptionHanlder`，则异常最终只打印在`System.err`，而不会打印在项目的日志中。

可参考 Spring 中的`TaskUtils.decorateTaskWithErrorHandler`对`Runnable`进行包装，默认会捕获所有异常并打印日志。对于定时任务可优先考虑使用`ThreadPoolTaskScheduler`，该类可以指定一个`ErrorHandler`，对于定时任务抛出的异常会统一捕获并处理，默认为打印错误日志。

**8.【强制】全局的非线程安全的对象可考虑使用ThreadLocal存放。**

全局变量包括单例对象，`static`成员变量。

著名的非线程安全类包括`SimpleDateFormat`，MD5 / SHA1 的`Digest`。

对这些类，需要每次使用时创建。

但如果创建有一定成本，可以使用ThreadLocal存放并重用。

`ThreadLocal`变量需要定义成`static`，如果需要时在每次使用前重置。

正例：

    private static final ThreadLocal<MessageDigest> SHA1_DIGEST = ThreadLocal.withInitial(() -> {
        try {
            return MessageDigest.getInstance("SHA");
        } catch (NoSuchAlgorithmException e) {
            throw new UndeclaredThrowableException("...", e);
        }
    });

    public void digest(byte[] input) {
        MessageDigest digest = SHA1_DIGEST.get();
        digest.reset();
        return digest.digest(input);
    }

-   [Sonar-2885: Non-thread-safe fields should not be static](https://rules.sonarsource.com/java/RSPEC-2885)

-   Facebook-Contrib: Correctness - Field is an instance based ThreadLocal variable

**9.【强制】JDK8 后`ThreadLocal`必须使用`withInitial`静态方法初始化。**

**10.【强制】必须回收自定义的`ThreadLocal`变量记录的当前线程的值，尤其在线程池场景下，线程经常会被复用，如果不清理自定义的`ThreadLocal`变量，可能会影响后续业务逻辑和造成内存泄露等问题。尽量在代码中使用`try-finally`块进行回收。**

正例：

    objectThreadLocal.set(userInfo);
    try {
        // ...
    } finally {
        objectThreadLocal.remove();
    }

**11.【强制】高并发时，同步调用应该去考量锁的性能损耗。能用无锁数据结构，就不要用锁；能锁区块，就不要锁整个方法体；能用对象锁，就不要用类锁。**

尽可能使加锁的代码块工作量尽可能的小，避免在锁代码块中调用 RPC 方法。

**12.【强制】对多个资源、数据库表、对象同时加锁时，需要保持一致的加锁顺序，否则可能会造成死锁。**

线程一需要对表 A、B、C 依次全部加锁后才可以进行更新操作，那么线程二的加锁顺序也必须是 A、B、C，否则可能出现死锁。

**13.【强制】在使用阻塞等待获取锁的方式中，必须在`try`代码块之外，并且在加锁方法与`try`代码块之间没有任何可能抛出异常的方法调用，避免加锁成功后，在`finally`中无法解锁。**

1.  在`lock`方法与`try`代码块之间的方法调用抛出异常，无法解锁，造成其它线程无法成功获取锁。

2.  如果`lock`方法在`try`代码块之内，可能由于其它方法抛出异常，导致在`finally`代码块中，`unlock`对未加锁的对象解锁，它会调用 AQS 的`tryRelease`方法(取决于具体实现类)，抛出`IllegalMonitorStateException`异常。

3.  在`Lock`对象的`lock`方法实现中可能抛出 Unchecked 异常，产生的后果与说明二相同。

正例：

    Lock lock = new XxxLock();
    // ...
    lock.lock();
    try {
        doSomething();
        doOthers();
    } finally {
        lock.unlock();
    }

反例：

    Lock lock = new XxxLock();
    // ...
    lock.lock();
    try {
        // 如果此处抛出异常，则直接执行 finally 代码块
        doSomething();
        // 无论加锁是否成功，finally 代码块都会执行
        lock.lock();
        doOthers();
    } finally {
        lock.unlock();
    }

**14.【强制】在使用尝试机制来获取锁的方式中，进入业务代码块之前，必须先判断当前线程是否持有锁。锁的释放规则与锁的阻塞等待方式相同。**

`Lock`对象的`unlock`方法在执行时，它会调用 AQS 的`tryRelease`方法(取决于具体实现类)，如果当前线程不持有锁，则抛出`IllegalMonitorStateException`异常。

正例：

    Lock lock = new XxxLock();
    // ...
    boolean isLocked = lock.tryLock();
    if (isLocked) {
        try {
            doSomething();
            doOthers();
        } finally {
            lock.unlock();
        }
    }

**15.【强制】并发修改同一记录时，避免更新丢失，需要加锁。要么在应用层加锁，要么在缓存加锁，要么在数据库层使用乐观锁，使用`version`作为更新依据。**

如果每次访问冲突概率小于 20%，推荐使用乐观锁，否则使用悲观锁。乐观锁的重试次数不得小于 3 次。

**16.【强制】多线程并行处理定时任务时，`Timer`运行多个`TimeTask`时，只要其中之一没有捕获抛出的异常，其它任务便会自动终止运行，使用`ScheduledExecutorService`则没有这个问题。**

**17.【推荐】选择分离锁，分散锁甚至无锁的数据结构。**

-   分离锁：

    1） 读写分离锁`ReentrantReadWriteLock`，读读之间不加锁，仅在写读和写写之间加锁；

    2） Array Base 的 queue 一般是全局一把锁，而 Linked Base 的 queue 一般是队头队尾两把锁。

-   分散锁（又称分段锁）：

    1）如 JDK7 的 `ConcurrentHashMap`，分散成 16 把锁；

    2）对于经常写，少量读的计数器，推荐使用 JDK8 的`LongAdder`对象性能更好（内部分散成多个 counter，减少乐观锁的使用，取值时再相加所有 counter ）。

-   无锁的数据结构：

    1）完全无锁无等待的结构，如 JDK8 的`ConcurrentHashMap`；

    2）基于 CAS 的无锁有等待的数据结构，如`AtomicXXX`系列。

**18.【推荐】资金相关的金融敏感信息，使用悲观锁策略。**

乐观锁在获得锁的同时已经完成了更新操作，校验逻辑容易出现漏洞，另外，乐观锁对冲突的解决策略有较复杂的要求，处理不当容易造成系统压力或数据异常，所以资金相关的金融敏感信息不建议使用乐观锁更新。

正例：悲观锁遵循*一锁二判三更新四释放*的原则。

**19.【推荐】使用`CountDownLatch`进行异步转同步操作，每个线程退出前必须调用`countDown`方法，线程执行代码注意 catch 异常，确保`countDown`方法被执行到，避免主线程无法执行至`await`方法，直到超时才返回结果。**

注意，子线程抛出异常堆栈，不能在主线程 try-catch 到。

**20.【推荐】避免`Random`实例被多线程使用，虽然共享该实例是线程安全的，但会因竞争同一 seed 导致的性能下降。**

`Random`实例包括`java.util.Random`的实例或者`Math.random()`的方式。

正例：在 JDK7 之后，可以直接使用 API `ThreadLocalRandom`，而在 JDK7 之前，需要编码保证每个线程持有一个单独的`Random`实例。

**21.【推荐】通过双重检查锁(double-checked locking)，实现延迟初始化需要将目标属性声明`volatile`型，(比如修改 helper 的属性声明为 `private volatile Helper helper = null;`)。**

正例：

    public class LazyInitDemo {
        private volatile Helper helper = null;
        public Helper getHelper() {
            if (helper == null) {
                synchronized(this) {
                    if (helper == null) {
                        helper = new Helper();
                    }
                }
            }
            return helper;
        }
        // other methods and fields...
    }

通过双重检查锁（double-checked locking）实现延迟初始化存在隐患，需要将目标属性声明为`volatile`，为了更高的性能，还要把`volatile`属性赋予给临时变量，写法复杂。

所以如果只是想简单的延迟初始化，可用下面的静态类的做法，利用 JDK 本身的`class`加载机制保证唯一初始化。

正例：

    private static class LazyObjectHolder {
        static final LazyObject instance = new LazyObject();
    }

    public void myMethod() {
        LazyObjectHolder.instance.doSomething();
    }

-   [Sonar-2168: Double-checked locking should not be used](https://rules.sonarsource.com/java#RSPEC-2168)

**22.【参考】`volatile`解决多线程内存不可见问题对于一写多读，是可以解决变量同步问题，但是如果多写，同样无法解决线程安全问题。**

如果是`count++`操作，使用如下类实现：

正例：

    AtomicInteger count = new AtomicInteger();
    count.addAndGet(1);

如果是 JDK8，推荐使用`LongAdder`对象，比`AtomicLong`性能更好(减少乐观锁的重试次数)。

**23.【参考】`HashMap`在容量不够进行`resize`时由于高并发可能出现死链，导致 CPU 飙升，在开发过程中注意规避此风险。**

**24.【参考】`ThreadLocal`对象使用`static`修饰，`ThreadLocal`无法解决共享对象的更新问题。**

这个变量是针对一个线程内所有操作共享的，所以设置为静态变量，所有此类实例共享此静态变量，也就是说在类第一次被使用时装载，只分配一块存储空间，所有此类的对象(只要是这个线程内定义的)都可以操控这个变量。
