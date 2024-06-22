# Netty学习笔记

## NioEventLoopGroup

类关系梳理

### Executor 

该接口只有一个 void execute(Runnable command); 方法，表示执行已经提交的Runnable任务的对象，此接口提供了一种任务提交与任务如何运行分离的方法，包括了线程的使用和调度。通常使用 anExecutor 而不是显式创建线程。
```Java
Executor executor = anExecutor(); 
executor. execute(new RunnableTask1()); 
executor. execute(new RunnableTask2()); 
...
```

事实上，Executor接口并不严格要求异步执行，最简单的情况下，Runnable对象可以直接在 caller 的线程中执行

```Java
class DirectExecutor implements Executor {   
    public void execute(Runnable r) {     
        r. run();   
    } 
}
```

也可以通过创建一个新的线程进行执行

```Java
class ThreadPerTaskExecutor implements Executor {
    public void execute(Runnable r) {
        new Thread(r).start();
    }
}
```

许多实现类，对任务的安排方式和执行事件进行了限制

下边这个接口，序列化任务，并交由另一个执行器对象实现

```Java
class SerialExecutor implements Executor {
    final Queue<Runnable> tasks = new ArrayDeque<>();
    final Executor executor;
    Runnable active;

    SerialExecutor(Executor executor) {
        this.executor = executor;
    }

    public synchronized void execute(Runnable r) {
        tasks.add(() -> {
            try {
                r.run();
            } finally {
                // 第二次执行的时候，队列为空了，不在执行
                scheduleNext();
            }
        });
        if (active == null) {
            scheduleNext();
        }
    }

    protected synchronized void scheduleNext() {
        if ((active = tasks.poll()) != null) {
            executor.execute(active);
        }
    }
}
```

### ExecutorService
这个接口继承了Executor，我们说接口定义了一种规范，因此，这个接口也具有**执行提交的任务的能力**
这个接口定义了这么一种Executor(提供了可以终止的方法，以及可以生成 Future 类来追踪一个或多个异步任务执行的方法)，并不是说这个接口就是定义了线程池，

**方法主要分为这么几类：**
- 提交任务的方法
  - submit 单个
  - invokeAll 集合，invokeAll 方法会执行一组任务，并返回包含每个任务结果的 Future 对象的列表。所有任务都执行完毕后才会返回。
  - invokeAny 提交结合，只返回最先完成的非异常返回的结果，其余未完成的会被取消
- 终止任务的方法
  终止后，执行者没有正在执行的任务，没有等待执行的任务，也无法提交新任务。应关闭未使用的 ExecutorService 资源，以便回收其资源。
  - shutdown 允许先前提交的方法可以在终止之前执行。
  - shutdownNow 组织等待任务启动，并尝试停止当前正在指定的任务
  - isShutdown 检查任务是否**开始**关闭，isShutdown 只检查线程池是否已经**开始**关闭过程，不保证所有任务都已经完成。
  - isTerminated 如果线程池已完全终止，返回 true；否则返回 false。
  - awaitTermination ExecutorService 中的 awaitTermination 方法用于阻塞当前线程，直到线程池中的所有任务都执行完成，或者等待超时，或者当前线程被中断，以先到者为准。这个方法通常与 shutdown 方法结合使用，以实现线程池的优雅关闭。true：如果在超时之前所有任务都执行完毕。 false：如果在超时之前仍然有任务没有完成。
  
  ```Java
  // 线程优雅关闭
  import java.util.concurrent.ExecutorService;
  import java.util.concurrent.Executors;
  import java.util.concurrent.TimeUnit;
  
  public class ExecutorServiceExample {
    public static void main(String[] args) {
    // 创建一个固定大小的线程池
    ExecutorService executorService = Executors.newFixedThreadPool(2);
        // 提交一些任务给线程池执行
        executorService.submit(() -> {
            try {
                Thread.sleep(2000); // 模拟耗时任务
                System.out.println("Task 1 completed");
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                System.out.println("Task 1 interrupted");
            }
        });
  
        executorService.submit(() -> {
            try {
                Thread.sleep(1000); // 模拟耗时任务
                System.out.println("Task 2 completed");
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                System.out.println("Task 2 interrupted");
            }
        });
  
        // 启动线程池的关闭过程
        executorService.shutdown();
        try {
            // 等待所有任务完成，或者等待最长 5 秒钟
            if (!executorService.awaitTermination(5, TimeUnit.SECONDS)) {
                // 如果在超时之前线程池没有终止，强制关闭
                executorService.shutdownNow();
            }
        } catch (InterruptedException e) {
            // 当前线程被中断，重新发起线程池关闭
            executorService.shutdownNow();
            // 保留中断状态
            Thread.currentThread().interrupt();
        }
  
        // 使用 isTerminated 方法检查线程池是否完全终止
        if (executorService.isTerminated()) {
            System.out.println("Executor service terminated");
        } else {
            System.out.println("Executor service not terminated yet");
        }
  
        System.out.println("Executor service terminated");
    }
  }
  
  ```
  
### ScheduledExecutorService

只集成了 ExecutorService 接口，在 ExecutorService 定义的能力的基础上，提供了再给定延迟或者周期性执行任务的能力。

**有几个需要注意的地方：**
- 所有的 schedule 方法都只接受相对时间或者周期作为参数，而不是绝对时间
- Executor 定义的 execute 方法和 submit 都视为请求延迟为0
- schedule 方法，也允许零延迟和负延迟，但不允许周期，被视为立即执行

schedule方法的执行会以守护线程的方法执行

一个简单的实现类
```Java
package com.yinxl.nettylearning.demos;

import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

public class ScheduledExecutorServiceDemo {
    public static void main(String[] args) {
        ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(1);

        // 1. 在5秒后执行一次任务
        scheduler.schedule(new RunnableTask(), 5, TimeUnit.SECONDS);
        System.out.println("task1 executed");

        // 2. 在3秒后执行任务，然后每10秒执行一次
        scheduler.scheduleAtFixedRate(new RunnableTask(), 3, 10, TimeUnit.SECONDS);
        System.out.println("task2 executed");

        // 3. 在2秒后执行任务，然后每5秒执行一次（在每次执行完成后再延迟5秒）
        scheduler.scheduleWithFixedDelay(new RunnableTask(), 2, 5, TimeUnit.SECONDS);
        System.out.println("task3 executed");

    }
}

class RunnableTask implements Runnable {
    @Override
    public void run() {
        System.out.println("Task executed at: " + System.currentTimeMillis());
    }
}
```

### EventExecutorGroup

Netty中用于管理一组 EventExecutor 的接口，它是处理并发任务和事件调度的核心组件之一。 next()方法提供要使用的 EventExecutor。除此之外，还需要负责管理的 EventExecutor 的生命周期，并允许以全局的方式来关闭他们

集成了 `ScheduledExecutorService, Iterable<EventExecutor>` 接口

因此，这个接口首先具有这样的能力：
- 返回一个迭代器
- 提供了执行 Runnable 类型任务的能力，execute, submit
- 提供了延时或周期性执行 Runnable 任务的能力
这些是所继承的接口提供的能力，但并不是完全继承，抛弃了原有的 shutdown 和 shutdownNow 方法

这个接口主要是提供了管理 EventExecutor 接口的能力，这个接口后边在介绍，可以理解为事件执行器。
- isShuttingDown 当 EventExecutorGroup 管理的所有 EventExecutor 正在优雅的关闭或者已经关闭，会返回true。
- shutdownGracefully 一旦调用本方法，isShuttingDown 会返回 true，执行程序开始准备关闭自身。**执行器停止接受新的任务，并且在完成所有正在执行的任务和排队的任务后关闭线程池。** 与 shutdown 方法有所不同，shutdown Gracefully 会有一个quietPeriod的概念，参数：
  - quietPeriod: 安静期是指在指定的时间段内，如果没有新的任务提交，那么线程池就认为可以开始关闭。**这是为了避免在关闭过程中有新的任务突然提交，导致关闭过程被打断。**
  - timeout: 超时时间是指从开始关闭到最终强制关闭的最大时间。如果在这个时间内仍有任务未完成，系统将强制关闭，以防止线程池无限期地运行下去。
  - unit: 指定 quietPeriod 和 timeout 的时间单位，如 TimeUnit.SECONDS。
- terminationFuture() 返回一个 Future 对象，如果当前管理的所有 EventExecutor 都**已经关闭**时，返回的对象会获得通知。
- EventExecutor next(); 返回一个管理的 EventExecutor 

**这里有个问题，不是说开始关闭以后就不在接受新的任务了吗，为什么还会害怕关闭过程中有新的任务提交导致关闭过程被打断？**

为了更准确地解释 quietPeriod 的作用，我们需要更详细地理解 shutdownGracefully 的工作机制。尽管 shutdownGracefully 方法会停止接受新的任务，但安静期的设计是为了确保系统在关闭之前有足够的时间完成现有任务，并处理潜在的、可能会在短时间内提交的新任务。

**shutdownGracefully 的工作机制**

当调用 shutdownGracefully 方法时，它实际上进入了一个逐步关闭的过程，该过程可以分为以下几个步骤：

- 停止接受新任务: 一旦调用 shutdownGracefully 方法，EventExecutorGroup 将不再接受任何新的任务提交。

- 执行现有任务: 它将继续执行已经提交但尚未开始的任务，以及正在运行的任务。

- 安静期检查: 安静期是**为了处理那些可能在关闭请求发出前后短时间内提交的新任务**。具体地说，在安静期内，如果没有新的任务提交，并且所有现有任务都完成了，EventExecutorGroup 就会继续关闭过程。

- 超时检查: **如果在安静期内有新任务提交，系统会重置安静期计时，直到安静期内没有新任务提交为止。如果整个关闭过程超过了指定的超时时间（timeout），无论是否完成所有任务，系统都会强制关闭，以避免无限期等待。**

**所以这里又有个问题，为什么要特殊处理关闭请求前后短时间内提交的新任务，不应该执行到shutdownGracefully方法就立即停止提交吗？**

安静期的设计考虑到了可能存在的 race condition（竞争条件），即在发出关闭请求时，系统可能仍然有少量的任务正在提交。如果没有安静期，可能会立即关闭线程池，而这些新提交的任务还没有来得及开始处理。这种情况在高并发环境下尤为常见。

总结一下，shutdownGracefully 方法的安静期是为了处理关闭请求前后短时间内提交的新任务，确保在高并发环境下系统能够正确地完成所有任务并优雅地关闭，而不是突然中断现有工作。这个机制有助于在关闭期间确保系统的稳定性和数据完整性。


### EventExecutor

上边讨论了 EventExecutorGroup 类，接下来让我们看看被管理的 EventExecutor 类

EventExecutor 的主要作用是执行事件任务，通常与 Netty 的 EventLoop 紧密结合，用于处理 I/O 操作和任务调度。



### Future 接口

java.util.concurrent 包中 Future 接口表示异步计算的结果，提供了用于检查计算是否完成、等待计算完成以及检索计算结果的方法。
- cancel
- isCancelled
- isDone
- get()
- get(long timeout, TimeUnit unit)

FutureTask类，

### Promise类

Promise类扩展了Netty的Future接口，额外具有手动设置操作的结果或失败原因

功能:
- 手动设置结果
- 回调机制，来自Future
- 链式调用

其实就是Future添加了手动设置成功或失败的能力

### ProgressivePromise



### MultithreadEventExecutorGroup 类

这个类是 AbstractEventExecutorGroup 抽象实现类，而类 AbstractEventExecutorGroup 是EventExecutorGroup接口的初步抽象实现类。

这个类提供了EventExecutorGroup的多线程版本基础实现

```Java
// 包含了所有的EventExecutor实例
private final EventExecutor[] children;
// children的不可变版本，通常用于提供线程安全的访问，避免外部代码修改EventExecutor集合
private final Set<EventExecutor> readonlyChildren;
// terminatedChildren 原子变量用于跟踪已经终止的 EventExecutor 实例数量
private final AtomicInteger terminatedChildren = new AtomicInteger();
// terminationFuture 是一个 Promise，用于表示 MultithreadEventExecutorGroup 的终止状态。
// 当所有 EventExecutor 实例都终止后，会触发 terminationFuture，从而通知所有等待者 MultithreadEventExecutorGroup 已经完全关闭。
// 可以通过 terminationFuture 添加监听器，在终止后执行特定操作。
private final Promise<?> terminationFuture = new DefaultPromise(GlobalEventExecutor.INSTANCE);
// chooser 是一个用于在多个 EventExecutor 实例之间选择的组件。它根据具体的负载均衡策略，选择一个 EventExecutor 来执行任务。不同的选择器策略可以实现不同的负载均衡效果，常用的策略包括轮询（Round Robin）和随机选择（Random）。
private final EventExecutorChooserFactory.EventExecutorChooser chooser;
```



### EventLoopGroup

主要方法：
1. EventLoop next()：
   返回下一个要使用的 EventLoop 实例。通常实现中会使用负载均衡策略（如轮询）来选择 EventLoop 实例。
2. ChannelFuture register(Channel channel)：
   将一个 Channel 注册到 EventLoop 中。返回一个 ChannelFuture，可以用来检查注册操作的状态。 
3. ChannelFuture register(ChannelPromise promise)：
   使用 ChannelPromise 将一个 Channel 注册到 EventLoop 中。这允许在注册完成后执行一些操作。 
4. ChannelFuture register(Channel channel, ChannelPromise promise)（已废弃）：
   这是一个已废弃的方法，之前用于将 Channel 注册到 EventLoop，同时使用 ChannelPromise。

### 为什么`EventExecutor`会继承`EventExecutorGroup`接口

**EventExecutorGroup 接口**

表示一个执行器组。这个组可以管理多个 EventExecutor 实例，并提供一些用于管理这些执行器的方法，例如：
- next()：返回下一个执行器。
- iterator()：返回一个遍历所有执行器的迭代器。
- shutdownGracefully()：优雅地关闭所有执行器。
- isShuttingDown()、isShutdown()、isTerminated()：检查执行器组的状态。

**EventExecutor 接口**

EventExecutor 继承了 EventExecutorGroup 接口，并扩展了它，表示一个具体的执行器。作为一个执行器，它提供了执行单个任务或多个任务的方法，例如：

- inEventLoop()：检查当前线程是否在事件循环中。
- schedule()：调度一个任务在将来某个时间执行。

**继承的原因**

- 通过继承，EventExecutor 和 EventExecutorGroup 可以通过相同的接口进行操作。无论是单个执行器还是执行器组，用户都可以使用相同的方法来管理它们。这样简化了 API 的设计和使用。
- 灵活性：在某些情况下，一个执行器也可以作为一个执行器组来使用。例如，只有一个执行器的执行器组。在这种情况下，EventExecutor 作为一个特殊的 EventExecutorGroup，允许在需要时替换或扩展实现，而不改变接口的使用方式。
- 代码复用：通过继承，EventExecutor 可以直接使用 EventExecutorGroup 中定义的方法和逻辑，减少代码重复，提高代码的维护性和一致性。
