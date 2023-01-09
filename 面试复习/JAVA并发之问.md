# Java并发基础



## 线程与进程  

**进程**

+ 进程是程序的一次执行过程，是系统运行程序的基本单位，因此进程是动态的。系统运行一个程序即是一个进程从创建，运行到消亡的过程。

+ 在 Java 中，当我们启动 main 函数时其实就是启动了一个 JVM 的进程，而 main 函数所在的线程就是这个进程中的一个线程，也称主线程 。

**线程**

线程是一个比进程更小的独立调度和分派的基本单位，一个进程在其执行的过程中可以产生多个线程，每个线程共享其所附属的进程的所有资源，它同时也包含独立的**程序计数器**、**虚拟机栈**和**本地方法栈**，所以系统在产生一个线程，或是在各个线程之间作切换工作时，负担要比进程小得多，也正因为如此，线程也被称为轻量级进程 (LWP)，Java 程序天生就是多线程程序   



**线程与进程的关系、区别以及优缺点**

+ 从 **JVM 角度**，一个进程中可以有多个线程，多个线程共享进程的堆和方法区 (JDK1.8 之后的元空间)资源，但是每个线程有自己的程序计数器、虚拟机栈 和 本地方法栈
+ 线程和进程最大的不同在于基本上各进程是独立的，而各线程则不一定，因为同一进程中的线程极有可能会相互影响。线程执行开销小，但不利于资源的管理和保护；而进程正相反  



**为什么程序计数器、虚拟机栈和本地方法栈是线程私有的呢** 

程序计数器的作用  

1. 字节码解释器通过改变程序计数器来依次读取指令，从而实现代码的流程控制
2. 在多线程的情况下，程序计数器用于记录当前线程执行的位置，从而当线程被切换回来的时候能够知道该线程上次运行到哪儿了   

如果执行的是 native 方法，那么程序计数器记录的是 undefined 地址，只有执行的是 Java 代码时程序计数器记录的才是下一条指令的地址    

程序计数器私有主要是为了线程切换后能恢复到正确的执行位



**虚拟机栈和本地方法栈为什么是私有的** 

1. 虚拟机栈      每个 Java 方法在执行的同时会创建一个栈帧用于存储局部变量表、操作数栈、常量池引用等信息。从方法调用直至执行完成的过程，就对应着一个栈帧在 Java 虚拟机栈中入栈和出栈的过程。 
2. 本地方法栈  和虚拟机栈所发挥的作用非常相似，区别是： 虚拟机栈为虚拟机执行 Java 方法 （也就是字节码）服务，而本地方法栈则为虚拟机使用到的 Native 方法服务。在 HotSpot 虚拟机中和 Java 虚拟机栈合二为一。

为了保证线程中的局部变量不被别的线程访问到，虚拟机栈和本地方法栈是线程私有的

**堆区和方法区**

堆和方法区是所有线程共享的资源，其中堆是进程中最大的一块内存，主要用于存放新创建的对象 (**几乎**所有对象都在这里分配内存)，方法区主要用于存放已被加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。

`栈上分配 在栈上分配对象内存,随栈帧出栈而销毁 支持方法逃逸`



**查看进程线程**

+ Linux      ps -ef  |  grep 关键字   kill pid 

+ jdk自带   jps查看Java进程  

+ 查看进程的线程 top -H p pid 

+ jstack    jstack pid 用于生成java虚拟机当前时刻的线程快照

+ jconsole   win+R  jconsole 可视化监控工具 



**JAVA线程模型** 

JAVA是单进程多线程，JAVA线程是轻量级进程(LWP) 

![image-20210712221939114](G:\markdown图片\image-20210712221939114.png)

**线程优先级**

最低1  默认5  最高10，由于Java线程是被映射到系统的原生线程上来实现的，操作系统提供的线程优先级，并不与Java线程的优先级一 一 对应，有的操作系统还有 "优先级推进器"，所以并不能通过设置优先级来企图控制多线程执行

`优先级推进器 一个线程被执行得特别频繁时， 可能会越过线程优先级去为它分配执行时间， 从而减少因为线程频繁切换而带来的性能损耗 `



**JAVA线程调度的两种方式**

1. 协同式线程调度，线程的执行时间由线程本身控制，线程执行完工作取通知操作系统切换到另外一个线程，不需要线程同步，但是出现阻塞会导致系统崩溃 

2. 抢占式线程调度，线程执行时间由系统分配 



**创建线程启动线程的几种方式**

1. 直接使用 Thread 类创建线程 

2. 使用 Runnable 配合 Thread   

3. FutureTask 配合 Thread   

   1. 间接实现了Runable接口，所以也可以作为任务对象 

   2. 还实现了Future可以用来获取任务返回结果

   3. 传入一个Callable对象为要执行的对象 Callable call 方法有返回值，还可以抛出异常； 

**构造线程**  

+ 由父线程进行分配空间，继承父线程的优先级、是否为Daemon属性、类加载器、可继承的ThreadLocal，并为他分配线程ID 。  此时线程在堆中等待运行 

**启动线程**

+ 调用start()方法，父线程同步告知JVM，只要线程规划器空闲，立刻启动调用start()方法线程

+ Java 创建线程和启动  `调用本地方法 start0()`、`JVM 中 JVM_StartThread 的创建和启动`、`设置线程状态等待被唤醒`、`根据不同的OS启动线程并唤醒`、`最后回调 run() 方法启动 Java 线程` 

+ 调用 start() 方法线程，使线程处于就绪状态，当分配到时间片之后就可以开始运行；而直接调用run方法会以主线程的方式执行 

  

**Daemo线程**

+ 当没有非Daemo线程时，立刻退出，不再执行任何剩余的代码
+ 如果要设置线程为Daemo线程必须在线程启动前设置，否则会出现异常  t1.setDaemon(true)
+ 垃圾回收器线程就是一种守护线程    



**线程中常用的方法**

Thread类

+ 实例方法  start()、run() 、join([time]) 、getId() 、setName()、getName()  、get/set Priority() 、getState()  isInterrupted() 、isAlive()、interrupt()  
+ 静态方法  currentThread() 、yield() 、sleep(n)、interrupted() 返回打断标记并且清除标记

sleep() yield() 用于防止占满CPU，适用于无锁场景



**为什么Thread类的sleep()和yield()方法是静态的**

`sleep()`和`yield()`都是需要正在执行的线程调用的，那些本来就阻塞或者等待的线程调用这个方法是无意义的，所以这两个方法是静态的 



**线程运行原理**

每个线程启动后，虚拟机就会为其分配一块栈内存

+ 每个栈由多个栈帧（Frame）组成，对应着每次方法调用时所占用的内存  
+ 每个线程只能有一个活动栈帧，对应着当前正在执行的那个方法  

多线程，每个线程有自己独立的栈内存，里面有多个栈帧，他们之间互不干扰



**线程的生命周期与状态转换**

JAVA角度 

JAVA有  6 种不同状态  

![Java 线程的状态 ](https://i.loli.net/2021/08/31/BJ2cvtzgfHmEx6P.png)

![无标题](https://i.loli.net/2021/08/31/JPKNQbiEeIUMAno.png)

JVM 为什么把  READY 和 RUNNING 状态统称为 **RUNNABLE（运行中）** 状态  

+ 现在的 **时分**（time-sharing）**多任务**（multi-task）操作系统架构通常都是用所谓的 "**时间分片**（time quantum or time slice "方式进行**抢占式**（preemptive）轮转调度（round-robin式）。这个时间分片通常是很小的，一个线程一次最多只能在 CPU 上运行比如 10-20ms 的时间（此时处于 running 状态），也即大概只有 0.01 秒这一量级，时间片用后就要被切换下来放入调度队列的末尾等待再次调度（也即回到 ready 状态）。线程切换的如此之快，区分这两种状态就没什么意义了。 

  

**RUNABLE 涵盖了阻塞(IO操作)    JAVA中的运行、可运行(未获得CPU)**



**操作系统线程状态**

+ 就绪状态，运行状态，阻塞状态（三种最基本状态）  初始状态，终止状态

  

**JAVA线程之间如何通信** 

1. volatile  和 sychronized 

   + volatile写-读内存语义 线程A写一个volatile变量，随后线程B读这个volatile变量，这个过程实质上是线程A通过主内存向线程B发送消息。
   + 锁的内存语义 线程A释放锁，随后线程B获取这个锁，这个过程实质上是线程A通过主内存向线程B发送消息。

2. 等待通知机制 

   + 等待通知机制依赖于同步机制(从wait()方法返回的前提是获得了调用对象的锁)，目的是为了保证等待线程从wait() 返回时能感知到通知线程对变量做出的修改 

3. 管道输入 /  输出流  

   + PipedInputStream  PipedOutputStream   PipedReader   PipedWriter

**JAVA线程的中断**

+ 调用interrupt()方法
+ 运行的线程被打断只会设置打断标志，不会退出
+ park线程，被打断可以继续运行，中断标志为true，不会抛出异常
+ Thread.sleep(long millis)被打断抛出异常后又会清除掉打断标志，可以在catch块中再次打断以设置打断标志 
+ 中断标志位 true  park不会生效  利用 Thread.interrupted() 清除打断标志  
+ suspend()  resume()  线程被挂起，不会释放锁资源，容易造成死锁
+ stop 方法会真正杀死线程，如果这时线程锁住了共享资源，被杀死后就再也没有机会释放锁，其它线程将永远无法获取锁 
+ System.exit (int) 终止当前正在运行的Java虚拟机，会让整个程序都停止  

**interrupt方法**

+ sleep wait join 被interrupt  isInterrupted()返回假 会抛出异常 InterruptedException: sleep interrupted，会在抛出异常后继续运行  可以在处理异常中重新设置标志
+ 正常运行的线程不会暂停，但是设isInterrupted() 为真，可以根据标记优雅的停止线程 

**如何优雅的退出线程**

+ 两阶段终止模式

  

**并行与并发**

并发（concurrent）是同一时间应对（dealing with）多件事情的能力  

并行（parallel）是同一时间动手做（doing）多件事情的能力

**为什么要使用多线程呢**  

1. 线程是一个比进程更小的独立调度和分派的基本单位，线程间的切换和调度的成本远远小于进程
2. 编写的代码很容易映射到 JAVA线程模型 
3. 更好的利用多核CPU，多核 CPU 时代意味着多个线程可以同时运行
4. 多线程并发编程正是开发高并发系统的基础 
5. 异步调用   避免阻塞主线程，耗时间操作可以开一个新线程执行，避免阻塞主线程



**多线程可能带来什么问题** 

1. 导致线程不安全，没有正确同步的多线程会存在数据竞争，产生违反直觉的结果 `一个线程写,另外一个线读,可能出现先读后写`
2. 多线程由于有线程的创建和上下文切换开销，不一定比单线程快。
3. 多线程可能发生死锁 
4. 多线程的活跃性问题
5. 由于任务拆分的不合理，导致效率反而降低    fork join  
6. 并发编程会受到硬件资源和软件资源的限制，当然可以考虑横向纵向拓展，资源复用。 

**什么是线程安全**

当多个线程同时访问一个对象时， 如果不用考虑这些线程在运行时环境下的调度和交替执行， 也不需要进行额外的同步，或者在调用方进行任何其他的协调操作， 调用这个对象的行为都可以获得正确的结果， 那就称这个对象是线程安全的    



**线程安全的等级**

不可变、 绝对线程安全、 相对线程安全`我们常说的线程安全`、 线程兼容和线程对立  

**不可变**             final   修饰的基本数据类型   String 、Integer  

String 的不可变性是 final+ 有修改发生时候保护性拷贝(创建新的对象副本从而避免共享)  

**绝对线程安全**  不管运行时环境如何， 调用者都不需要任何额外的同步措施  

**相对线程安全**  我们常说的线程安全  JUC、HashTable 

**线程兼容**         对象本身并不是线程安全的， 但是可以通过在调用端正确地使用同步手段来保证对象在并发环境中可以安全地使用  

**线程对立**        无法在多线程环境中并发使用代码   suspend()和resume()  



**JDK线程安全类**

+ String、Integer、StringBuffer、Random、Vector 、Hashtable 、java.util.concurrent 包下的类  

  

**实现线程安全的方法**  

1. 互斥同步  synchronized  ReentrantLock    

2. 非阻塞同步   CAS实现的原子操作  

3. 无同步方案  线程本地存储  



**原子操作的实现原理**

处理器如何实现原子操作 

+ 锁总线、锁缓存

JAVA如何实现原子操作

1. 循环CAS，但也存在一些问题，1.ABA  2.竞争大时候，开销大   3.只能保证一个共享变量的原子操作
2. 使用锁   

**CAS循环时间长占用资源大问题**

如果JVM 能支持处理器提供的pause指令，那么效率会有一定的提升 

第一延迟流水线执行指令，使CPU不会消耗过多的执行资源，延迟的时间取决于具体实现的版本，在一些处理器上延迟时间为0；第二避免在退出循环的时候因内存顺序冲突而引起CPU流水线被清空，从而提高CPU执行效率。



## ThreadLocal  

`线程本地变量`

TheadLocal可以称为线程本地变量，可以在指定的线程内存储数据，数据存储之后，只有指定的线程才能得到存储的数据   

+ 单例例 Bean 的线程安全可以借助ThreadLocal  
+ 解决SimpleDateFormat线程安全问题 
+ 数据库连接，Session  管理

**ThreadLocal原理**

+ Thread类有一个成员变量：`threadlocals`。每一个线程中都会有一个属于自己的ThreadLocal值，也就是说每new出来一个线程，内部就会有一个`threadlocals`。多个线程之间的threadlocals相互隔离与独立。
+ ThreadLocal有一个静态内部类ThreadLocalMap，它是一个哈希表只适用于维护线程本地值，它的键值对是Entry对象
+ Entry类继承了弱引用，Entry的key是弱引用  

**ThreadLocal 内存泄露问题**

+  弱引用的目的是为了防止内存泄露，如果是强引用那么ThreadLocal对象除非线程结束否则始终无
   法被回收，弱引用则会在下一次GC的时候被回收。 
+  key被回收之后，如果value没有被断开引用，就会导致我们再也访问不到 value，GC也无法回收value，发生内存泄漏 

ThreadLocalMap 实现中已经考虑了这种情况，在调用 `set()`、`get()`、`remove()` 方法的时候，会清理掉 key 为 null 的记录。使用完 `ThreadLocal`方法后 最好手动调用`remove()`方法 



**变量的线程安全**

**实例变量和静态变量是否线程安全**

1. 如果它们没有共享，则线程安全  
2. 被共享了  如果只有读操作，则线程安全，如果有读写操作，则这段代码是临界区，需要考虑线程安全   

静态变量 i++   不是原子性操作    静态变量被类的实例共享 

```java
getstatic i // 获取静态变量i的值
iconst_1 // 准备常量1  
iadd // 自增     isub // 自减
putstatic i // 将修改后的值存入静态变量i
```

**局部变量**

i++    是原子操作         线程有各自的栈帧，局部变量是栈帧中的，是私有的

```java
 iinc  // i++   
```

局部变量是线程安全的，但局部变量引用的对象不一定线程安全

如果该对象没有逃离方法的作用范围，它是线程安全的；如果该对象逃离方法的作用范围，需要考虑线程安全 ； 例如创建子类，子类覆盖父类方法，在方法中创建新的线程来修改局部变量，暴露给其他的线程，线程逃逸  所以 private final  可以一定程度保证线程安全，开闭原则的闭



**临界区**   

一段代码块内如果存在对**共享资源**的多线程读写操作，称这段代码块为**临界区**  

**竞态条件**   

多个线程在临界区内执行，由于代码的**执行序列不同**而导致结果无法预测，称之为发生了**竞态条件**  

**如何避免发生竞态条件**     1. 阻塞的方式 使用锁     2. 非阻塞 原子变量   





**上下文切换** 

CPU通过时间片分配算法来循环执行任务，当前任务执行一个时间片后会切换到下一个任务。但是，在切换前会保存上一个任务的状态，以便下次切换回这个任务时，可以再加载这个任务的状态。所以任务从保存到再加载的过程就是一次上下文切换 

状态信息包括程序计数器(记住下一条 JVM 指令的执行地址)、虚拟机栈中每个栈帧的信息，如局部变量、操作数栈、返回地址等 



**什么时候会发生上下文切换** 

被动

1. 线程时间片用完
2. GC时 
3. 更高优先级线程需要运行
4. 线程被终止  

主动

1. 主动让出 CPU，比如调用了 sleep()，wait() 、yield() 

2. 请求 IO，线程被阻塞 

3. 运行结束  



**如何减少上下文切换**

+ 无锁并发    将数据ID按照Hash算法取模分段，不同的线程处理不同的数据
+ CAS算法    
+ 使用最少的线程数  
+ 使用协程 



**什么是线程死锁  如何避免死锁**  

死锁 : 当线程A持有独占资源1，并尝试去获取独占资源2的同时，线程B持有独占资源2，并尝试获取独占资源1的情况下，就会发生AB两个线程由于互相持有对方需要的资源，而发生的阻塞现象，我们称为死锁。



**产生死锁的必要条件** 

1. 互斥条件   进程访问的是临界资源，即在一段时间内某资源只由一个进程占用。  JAVA线程是 LWP
2. 请求和保持条件   获取了资源，申请另外资源失败阻塞，但不释放已有的资源
3. 不可抢占条件  
4. 环路等待条件      存在进程资源环   



**处理死锁的几种方式**

1. 预防死锁  导致系统资源利用率和系统吞吐量降低。  
2. 避免死锁  银行家算法  (计算机使用的)
3. 检测死锁  
4. 解除死锁  撤销或挂起一些进程，以便回收一些资源，再将这些资源分配给已处于阻塞状态的进程  

**如何预防和避免线程死锁**

**如何预防死锁**  

1. **破坏请求与保持条件** ：一次性申请所有的资源。
2. **破坏不剥夺条件** ：占用部分资源的线程进一步申请其他资源时，如果申请不到，可以主动释放它占有的资源。
3. **破坏循环等待条件** ：靠按序申请资源来预防。按某一顺序申请资源，释放资源则反序释放。破坏循环等待条件。

**如何避免死锁 **

银行家算法



**JAVA定位死锁**

1. jstack 

2. jconsole 检测死锁选项



**多线程的活跃性问题**

+ 死锁
+ **活锁**  两个线程互相改变结束条件，导致两个线程都得不到结束，通过增加随机睡眠时间来避免
+ **饥饿**  一个线程由于优先级太低，始终得不到 CPU 调度执行，也不能够结束 ，通过ReentrantLock 公平模式



## 并发编程模式

 **终止模式 之 两阶段终止模式**

优雅的终止线程 

![image-20210712000146117](G:\markdown图片\NHBJo2k1UrKhYy8.png)

**注意** 

1. 运行的线程被打断只会设置打断标志，不会退出

2. Thread.sleep(long millis)被打断抛出异常后又会清除掉打断标志，可以在catch块中再次打断以设置打断标志

3. 打断park方法暂停的线程  
   + 被打断可以继续运行，中断标志为true，不会抛出异常 
   + 中断标志位true park不会生效  利用 Thread.interrupted() 清除打断标志 

**park & unpark 原理**

```java
每个线程都有一个自己的 Parker 对象 由 _counter , _cond 和 _mutex 组成
park方法会检查 _counter , 如果为 0 , 这时，获得 _mutex 互斥锁,线程进入 _cond 条件变量阻塞设置 _counter = 0
 
    
调用 Unsafe.unpark(Thread_0) 方法，设置 _counter 为 1 
唤醒 _cond 条件变量中的 线程恢复运行
设置 _counter 为 0 
    
    
调用 Unsafe.unpark(Thread_0) 方法 , 设置 _counter 为 1
当前线程调用 Unsafe.park() 方法, 检查 _counter 本情况为 1,继续运行,无需阻塞,设置 _counter 为 0
```

**同步模式 之 保护性暂停**

+ JDK 中，join 的实现、Future 的实现，因为要等待另一方的结果，因此归类到同步模式 
+ 有一个结果需要从一个线程传递到另一个线程，让他们关联同一个对象，t1  wait() 直到  t2 给对象赋值，就像 Future 的 get()      
+ 如果有结果不断从一个线程到另一个线程那么可以使用消息队列 (生产者/消费者）

![image-20210906213406669](G:\markdown图片\image-20210906213406669.png)

**join 方法及原理** 

等待调用join的线程结束，利用的保护性暂停模式，他等待的条件是线程结束 isAlive() 为 false 

缺点

+ 需要外部共享变量，不符合面向对象封装的思想
+ 必须等待线程结束，不能配合线程池使用  

**Future原理**

任务没有执行完毕，调用get线程进入等待状态，当任务执行完毕调用set方法设置结果并唤醒等待的线程，get线程拿到结果返回  

**Future的get方法相比join优点**

+ 可以方便配合线程池使用
+ 不需要用外部共享变量来传递结果  

**同步模式之 Balking**

Balking （犹豫）模式用在一个线程发现另一个线程或本线程已经做了某一件相同的事，那么本线程就无需再做
了，直接结束返回。  实现线程安全的单例  

**设计模式——享元模式** 

包装类 、 String 串池、BigDecimal(保护性拷贝)、BigInteger(保护性拷贝) 不可变类，体现了享元模式，是线程安全的。

享元模式 vs 单例、缓存、对象池 

# JAVA并发进阶 

## synchronized  

synchronized关键字解决的是多个线程之间访问资源的同步性，synchronized关键字可以保证被它修饰的方法或者代码块在任意时刻只能有一个线程执行。 另外，在 Java 早期版本中，synchronized属于重量级锁，效率低下，每个对象都与一个monitor关联，monitor 是依赖于底层的操作系统的Mutex Lock 来实现的，Java 的线程是映射到操作系统的原生线程之上的。如果要阻塞或唤醒一条线程，则需要进行系统调用， 这就不可避免地陷入用户态到核心态的转换中， 进行这种状态转换需要耗费很多的处理器时间，如果只是简单的同步块，状态转换消耗的时间甚至会比用户代码本身执行的时间还要长 ，这也是为什么早期的synchronized 效率低的原因。庆幸的是在 Java 6 之后 Java 官方对从 JVM 层面对synchronized 较大优化，所以现在的 synchronized 锁效率也优化得很不错了。JDK1.6对锁的实现引入了大量的优化，如自旋锁、适应性自旋锁、锁消除、锁粗化、偏向锁、轻量级锁等技术来减少锁操作的开销  



**synchronized可见性保证**

同步块的可见性是由 "对一个变量执行unlock操作之前， 必须先把此变量同步回主内存中（执行store、 write操
作)"  这条规则获得的  

**synchronized有序性 **

synchronized有序性是由 "一个变量在同一个时刻只允许一条线程对其进行lock操作"  这条规则获得的，这个规则决定了持有同一个锁的两个同步块只能串行地进入，这个规则保证了有序性   



**sychronized如何实现可重入**

Synchronized进过编译，会在同步块的前后分别形成monitorenter和monitorexit这个两个字节码指令。在执行monitorenter指令时，首先要尝试获取对象锁。如果这个对象没被锁定，或者当前线程已经拥有了那个对象锁，把锁的计算器加1，相应的，在执行monitorexit指令时会将锁计算器就减1，当计算器为0时，锁就被释放了。如果获取对象锁失败，那当前线程就要阻塞，直到对象锁被另一个线程释放为止  



**synchronized无法禁止指令重排，却能保证有序性**

sychronized  只能保证一个线程获取锁，那相当就是单线程了，JMM的as-if-serial语义保证，不管怎么重排(存在数据依赖性不会重排),（单线程）程序的执行结果不能被改变。编译器,处理器,runtime都遵守 



**synchronized实现懒汉式单例**

如果反复调用，性能低下 

```java
public class IdGenerator {
    private AtomicLong id = new AtomicLong(0);
    private static IdGenerator instance;
    private IdGenerator() {
    }
    public static synchronized IdGenerator getInstance() {
        if (instance == null) {
            instance = new IdGenerator();
        }
        return instance;
    }
    public long getId() {
        return id.incrementAndGet();
    }
}
```

**DCL**  

双重检查锁定是一种延迟初始化技术  

双重检查锁定，可以用来实现既支持延迟加载又支持高并发的单例模式  

**基于volatile的延迟初始化** 

通过将延迟初始化对象声明为volatile，既可以对静态字段实现延迟初始化，也可以对实例字段实现延迟初始化

```java
public class Singleton {
    
    private static volatile Singleton singleton;

    private Singleton() {
    }

    public static Singleton getSingleton() {
        if (singleton == null) {
            synchronized (Singleton.class) {
                if (singleton == null) {
                    singleton = new Singleton();

                }
            }
        }
        return singleton;
    }
}

singleton 采用 volatile 关键字修饰也是很有必要的, singleton = new Singleton(); 这段代码其实是分为三步执行 
1.为 singleton 分配内存空间
2.初始化 singleton
3.将 singleton 指向分配的内存地址
    
但是由于 JVM 具有指令重排的特性,执行顺序有可能变成 1->3->2。指令重排在单线程环境下不会出现问题,但是在多线程环境下会导致一个线程获得还没有初始化的实例。 
 
使用 volatile 可以禁止 JVM 的指令重排,保证在多线程环境下也能正常运行。
```

**基于类初始化的延迟初始化** 

只能对静态字段使用延迟初始化 

允许 2，3 重排序但是不允许别的线程看到，利用初始化锁实现 

```java
public class IdGenerator {
    private final AtomicLong id = new AtomicLong(0);
    
    private static class SingletonHolder {
        private static final IdGenerator instance = new IdGenerator();
    }

    public static IdGenerator getInstance() {
        return SingletonHolder.instance;
    }

    public long getId() {
        return id.incrementAndGet();
    }
}
```

类需要被立即初始化JVM指令

1. T是一个类，创建T的实例 new 
2. T中声明的静态方法被调用 invokestatic
3. 类的静态变量被赋值 putstatic
4. 类的静态变量被使用 getstatic  (字符串常量除外) 

初始化执行流程其实是JVM类加载初始化阶段执行类构造器 <clinit> 方法  

+ 通过获取Class对象的初始化锁LC，来进行同步。当线程A获取LC锁，将初始化状态设置为初始化中，释放锁。另外一个线程B获取 LC，发现初始化状态为初始化中，释放锁，到LC对应的条件队列中进行等待。线程A又获取LC锁，进行对象初始化，分配存储空间、初始化对象、赋值给引用变量（这个过程可以重排，但是其他线程看不到），设置初始化状态为初始完毕，唤醒条件队列中等待的线程，释放锁。其他所有线程依次获取LC锁发现已经初始化完毕，释放锁，也初始化完毕 ，整个过程通过初始化锁LC的获取释放，由happen-before的锁规则保证可见性。



**synchronized 关键字的底层原理**



**Monitor 原理** 



每个 Java 对象都可以关联一个 Monitor 对象，如果使用 synchronized 给对象上锁（重量级）之后，该对象头的Mark Word 中就被设置指向  Monitor 对象` (OS的，不是Java的)`的指针  

![image-20210628115833039](G:\markdown图片\XWlYBsMr9f7gdaR.png)

```java
刚开始 Monitor 中 Owner 为 null当 Thread-2 执行 synchronized(obj) 就会将 Monitor 的所有者 Owner 置为 Thread-2，Monitor中只能有一个 Owner
在 Thread-2 上锁的过程中，如果 Thread-3，Thread-4，Thread-5 也来执行 synchronized(obj)，就会进入 EntryList BLOCKED 
Thread-2 执行完同步代码块的内容，然后唤醒 EntryList 中等待的线程来竞争锁,竞争是非公平的(JDK底层实现决定)
图中 WaitSet 中的 Thread-0，Thread-1 是之前获得过锁，但条件不满足进入 WAITING 状态的线程

synchronized 必须是进入同一个对象的 monitor 才有上述的效果
不加 synchronized 的对象不会关联监视器,不遵从以上规则 
```



**修饰方法的synchronized与修饰同步块的sychronized 实现不同**  

**修饰同步代码块**

`synchronized` 同步语句块的实现使用的是 `monitorenter` 和 `monitorexit` 指令，其中 `monitorenter` 指令指向同步代码块的开始位置，`monitorexit` 指令则指明同步代码块的结束位置，并且如果有异常表，在异常表中有跳转到 monitorexit的项，确保抛出异常也能释放；

线程执行到 monitorenter 指令时，将会尝试获取对象对应Moniter的所有权，即尝试获得对象的锁。当一个对象获取到Moniter的所有权，对象便处于锁定状态，会将对象头MarkWord中的  ptr_to_heavyweight_monitor 指针指向 Monitor；执行monitorexit时会重置MarkWord ，并唤醒EntryList

```c
Code:
stack=2, locals=3, args_size=1
0: getstatic #2 // <- lock引用 (synchronized开始)
3: dup 复制一份,释放使用
4: astore_1 // lock引用 -> slot 1 
5: monitorenter // 将 lock对象 MarkWord 置为 Monitor 指针
6: getstatic #3 // <- i
9: iconst_1 // 准备常数 1
10: iadd // +1
11: putstatic #3 // -> i
14: aload_1 // <- lock引用
15: monitorexit // 将 lock对象 MarkWord 重置, 唤醒 EntryList
16: goto 24
19: astore_2 // e -> slot 2
20: aload_1 // <- lock引用
21: monitorexit // 将 lock对象 MarkWord 重置, 唤醒 EntryList
22: aload_2 // <- slot 2 (e)
23: athrow // throw e
24: return
Exception table: 异常表 
from to target type
6 16 19 any 同步代码块内容
19 22 19 any 
```

**synchronized 修饰方法** 

方法级的同步是隐式的，无须通过字节码指令来控制，它实现在方法调用和返回操作之中。当方法调用时， 调用指令将会检查方法的ACC_SYNCHRONIZED访问标志是否被设置，如果设置了，执行线程就要求先成功持有管程， 然后才能执行方法；当方法完成（无论是正常完成还是非正常完成） 时释放管程。 在方法执行期间， 执行线程持有了管程， 其他任何线程都无法再获取到同一个管程。 如果一个同步方法执行期间抛出了异常， 并且在方法内部无法处理此异常， 那这个同步方法所持有的管程将在异常抛到同步方法边界之外时自动释放 



**JDK 1.6  synchronized 的优化**

JDK1.6 对锁的实现引入了大量的优化，如偏向锁、轻量级锁、自旋锁、适应性自旋锁、锁消除、锁粗化等技术来减少锁操作的开销 

![image-20210628113952826](G:\markdown图片\MbCYyjhNk9qs5KD.png)

![image-20210628114009196](G:\markdown图片\pRHBYLbhagmjcXf.png)

​      Klass Word 对象的类型指针

**Mark Word**   

+ 32bit 

![image-20210627222528248](https://i.loli.net/2021/09/02/RYCMctNgl2fxzLd.png)

![image-20210627222607657](G:\markdown图片\LhSi8jFt2IyxRGg.png)



线程ID是操作系统给线程设置的唯一表示，不是JAVA中的线程 ID 

**轻量级锁** 

加锁

+ 加锁时候，在线程栈帧生成锁记录(Lock record)，让锁记录中 Object reference 指向锁对象，并尝试用 CAS 替换 Object 的 Mark Word，将 Mark Word 的值存入锁记录，这个过程叫Displaced Mark Word，如果 CAS 替换成功，对象头中存储了 锁记录地址和状态 00，表示由该线程给对象加锁，这个如果 CAS 失败，有两种情况  
  1. 如果是其它线程已经持有了该 Object 的轻量级锁，这时表明有竞争，**进入锁膨胀过程**
  2. 如果是本线程持有，执行了 synchronized 锁重入，那么再添加一条 Lock Record 作为重入的计数，锁记录地址和状态为 null 

`每一个Lock Record 包含了 Object reference 和  锁记录地址 和 状态`  

**在displaced Mark Word这个过程中，如果CAS失败，为什么虚拟机还要检查对象的Mark Word是否指向当前线程的栈帧**

如果这个线程对这个锁进行了重入，由于Lock Record不是同一个，所以JVM并不能知道当前占据锁的线程是谁，所以还必须要检查一下Mark Word的指针是否是指向当前这个线程，如果是， 说明当前线程已经拥有了这个对
象的锁， 那直接进入同步块继续执行就可以了，否则就说明这个锁对象已经被其他线程抢占了  

**锁膨胀**   

如果在尝试加轻量级锁的过程中，CAS 操作无法成功，这时一种情况就是有其它线程为此对象加上了轻量级锁（有竞争），这时需要进行锁膨胀，将轻量级锁变为重量级锁

锁膨胀流程  为Object 对象申请 Monitor 锁，让Object  ptr_to_heavyweight_monitor 指向重量级锁地址然后，获取锁失败的线程进入 Monitor 的 EntryList  BLOCKED (阻塞)    

![image-20210628113412760](G:\markdown图片\6pwuo3efTWFqdty.png)

![image-20210628113433799](G:\markdown图片\7fzStPgElZNukMR.png)

轻量级解锁

+ 当退出 synchronized 代码块 (解锁时) 锁记录 的值不为 null，这时使用 CAS 将 Mark Word 的值恢复给对象头
  成功，则解锁成功，失败，说明轻量级锁进行了锁膨胀或已经升级为重量级锁，**进入重量级锁解锁流程**    

重量级锁解锁

+ 按照 Monitor 地址找到 Monitor 对象，设置 Owner 为 null，唤醒 EntryList 中 BLOCKED 线程  

**偏向锁** 

适用于冲突很少，就一个线程自己

轻量级锁在没有竞争时 (就自己这一个线程) ，每次进入仍然需要执行 CAS 操作 (CAS底层是Lock开头的CPU指令，会影响性能) 。Java 6 中引入了偏向锁来做进一步优化：只有**第一次**使用 CAS 将线程 ID 设置到对象的 Mark Word 头，之后发现这个线程 ID 是自己的就表示没有竞争，不用重新 CAS。以后只要不发生竞争，这个对象就归该线程所有。

**synchronized 锁优先级： 偏向锁  >  轻量级锁  >  重量级锁**

**偏向锁撤销**   

可偏向 转换为 不可偏向 

1. -XX:-UseBiasedLocking 关闭偏向锁，一上来就是轻量级锁
2. 调用对象notify/wait()只有重量级锁有，所以会把轻量级和偏向锁升级为重量级锁
3. 其它线程使用对象(**注意没有竞争**)，因为偏向锁是对象被一个线程专门持有，升级为轻量级锁 。  优化：批量重偏向
4. 对象的hashCode()方法会撤销掉偏向锁，Mark Word中的hash码默认是0，只有第一次调用了对象hashCode()，才会产生hash码，填充到对象头中。轻量级锁hash码存在线程栈帧的锁记录中，重量级锁存放在Moniter中，解锁的时候还会还原回去，而偏向锁没有额外的存储空间 



**批量重偏向** 

如果对象虽然被多个线程访问，但没有竞争，这时偏向了线程 T1 的对象仍有机会重新偏向 T2，重偏向会重置对象的 Thread ID 

当撤销偏向锁阈值超过 20 次后，JVM  会这样觉得，我是不是偏向错了呢，于是会在给这些对象加锁时重新偏向至加锁线程 

**批量撤销** 

当撤销偏向锁阈值超过 40 次后，JVM 会这样觉得，自己确实偏向错了，根本就不该偏向。整个类的所有对象都会变为不可偏向的，这个类新建的对象也是不可偏向的。如果没有超过阈值，最后新建仍然是可偏向。 



**自旋锁**

`多核CPU自旋才有意义，单核纯属浪费`

重量级锁竞争的时候，还可以使用自旋来进行优化，如果当前线程自旋成功（即这时候持锁线程已经退出了同步块，释放了锁），这时当前线程就可以避免阻塞和唤醒 

在JDK 6中就已经改为默认开启了，之前可用-XX:+UseSpinning开启                                                                              JDK  7 之后不能控制是否开启自旋功能JVM自己决定



**适应性自旋锁** 

在 JDK 6  之后自旋锁是自适应的，比如对象刚刚的一次自旋操作成功过，那么认为这次自旋成功的可能性会高，就多自旋几次；反之，就少自旋甚至不自旋，比较智能 



**锁粗化**

如果在一个操作序列中对同一个对象反复加锁对同一个对象反复加锁和解锁，JVM会将将会把加锁同步的范围扩展（粗化） 到整个操作序列的外部 

**锁消除**

根据逃逸技术分析，如果加锁的对象不会逃逸处出方法，也就是完全不会逃逸，就没必要加锁，JIT即时编译器就会把synchronized优化掉    

**锁升级过程** 

![image-20210831223454707](https://i.loli.net/2021/09/02/ADIWX8ZHkS2bzcF.png)





首先最开始对象是无锁状态，把对象是否偏向设置为1，锁标志位还是01，并把 MarkWord 的线程 ID 改为当前线程ID，此时对象处于偏向锁状态，一个线程继续对该该对象加锁，发现是偏向锁状态，判断偏向锁线程是否是这个线程，如果是则是直接进入，如果偏向线程不是当前线程，也就是存在锁竞争，那么就撤销偏向锁

偏向锁的撤销，需要等待全局安全点（这个时间点上没有正在执行的字节码），首先，它会暂停拥有偏向锁的线程，然后检测持有偏向锁的线程是否还活着，如果线程不处于活动状态，则将对象头设置为无锁状态，如果线程仍然活着，那么会将偏向锁升级为轻量级锁

**锁降级**

锁降级是指把持住（当前拥有的）写锁，再获取到读锁，随后释放（先前拥有的）写锁  

**锁降级中读锁的获取是否必要呢**

+ 是必要的，读锁与写锁互斥，如果释放读锁再获取写锁，这中间其他线程获取写锁修改了数据，当前线程无法感知线程的数据更新，获取读锁再释放写锁，其他线程写锁会被阻塞

**wait / notify原理**

![image-20210707103347628](G:\markdown图片\WrFJzAcPvDkpKMV.png)

+ obj.wait()             让获取到对象锁的线程进入waitset等待 
+ obj.wait(long n)   限时等待，超时间后唤醒 他的状态是TIMED_WAITING 
+ obj.notify()           在 object 上正在 waitSet 等待的线程中挑一个唤醒
+ obj.notifyAll()       让 object 上正在 waitSet 等待的线程全部唤醒 
+ **waitSet 中被唤醒，会重新进入EntryList重新竞争锁**  
+ 它们都是线程之间进行协作的手段，都属于 Object 对象的方法。必须获得此对象的锁，才能调用这几个方法




**为什么wait()、notify()、notifyAll()被定义在Object类中而不是在Thread类中**

JAVA提供的锁是对象级的而不是线程级的，每个对象都有锁，wait，notify和notifyAll 都是锁级别的操作，所以把他们定义在Object类中因为锁属于对象。



**避免虚假唤醒** 

`使用wait notify要避免虚假唤醒` 

+ 虚假唤醒 notify 只能随机唤醒一个 WaitSet 中的线程，这时如果有其它线程也在等待，那么就可能唤醒不了正确的线
  程  可以使用notifyAll   while + wait，当条件不成立，再次 wait   



**sleep(long n) 和 wait(long n) 的区别**

+  sleep 是 Thread 方法，而 wait 是 Object 的方法 
+  sleep 不需要强制和 synchronized 配合使用，但 wait 需要和 synchronized 一起用 
+  sleep 在睡眠的同时，不会释放对象锁的，但 wait 在等待的时候会释放对象锁  sleep 在sychronized 中调用不会释放锁
+  它们状态 都是TIMED_WAITING 



**sleep 方法和 wait  方法区别和共同点**

1. 两者最主要的区别在于：sleep() 方法没有释放锁，而 wait()  方法释放了锁  
2. 两者都可以暂停线程的执行，它们状态 都是 TIMED_WAITING
3. wait()  通常被用于线程间交互/通信，sleep() `yeid` 通常被用于暂停执行释放CPU 
4. wait() 方法被调用后，线程不会自动苏醒，需要别的线程调用同一个对象上的 notify()或者 notifyAll() 方法，sleep() 方法执行完成后，线程会自动苏醒 



**park、unpark 与 wait，notify，notifyAll区别**

1. wait，  notify 和 notifyAll 必须配合 Object Monitor 一起使用，而 park，unpark 不必  
2. park & unpark 是以线程为单位来【阻塞】和【唤醒】线程，更加精确
3. park & unpark 可以先 unpark，而 wait & notify 不能先 notify   



## volatile  

一旦一个共享变量（类的成员变量、类的静态成员变量）被volatile修饰之后，那么就具备了两层语
义  

(1) 保证了不同线程对这个变量进行操作时的可见性，即一个线程修改了某个变量的值，这新值对
其他线程来说是立即可见的，volatile关键字会强制将修改的值立即写入主存  

(2) 禁止进行指令重排序  



**为什么会出现可见性问题**   

+ 为了解决CPU运算速率与内存读写速率不匹配的矛盾 
+ CPU 的速度是极高的，如果 CPU 需要存取数据时都直接与内存打交道，在存取过程中，CPU 将一直空闲，这是一种极大的浪费，所以，为了提高处理速度，CPU 不直接和内存进行通信，而是在 CPU 与内存之间加入了多级缓存，它们比内存的存取速度高得多，这样就解决了 CPU 运算速度和内存读取速度不一致问题。由于 CPU 与内存之间加入了**缓存**，在进行数据操作时，先将数据从内存拷贝到缓存中，CPU 直接操作的是缓存中的数据。但在多处理器下，将可能导致各自的缓存数据不一致（这也是**可见性问题**的由来），为了保证各个处理器的缓存是一致的，就会实现缓存一致性协议，而**嗅探是实现缓存一致性的常见机制**     

**volatile 可见性原理** 

+ 对 volatile变量写操作之后，JVM会向处理器发送一条Lock前缀的指令，它会锁定这块内存区域并将其写回内存，    并使用缓存一致性机制来确保原子性，缓存一致性机制能阻止同时修改由两个以上的处理器缓存的内存数据，此操作也被称为缓存锁定    
+ 再由处理器**总线嗅探机制**使其它处理器相应的缓存无效，下次必须从主内存读取，这样就保证了内存可见性      



**嗅探机制工作原理**：

+ 每个处理器通过监听在总线上传播的数据来检查自己的缓存值是不是过期了，如果处理器发现自己缓存行对应的内存地址修改，就会将当前处理器的缓存行设置无效状态，当处理器对这个数据进行修改操作的时候，会重新从主内存中把数据读到处理器缓存中    

+ 基于 CPU 缓存一致性协议，JVM 实现了 volatile 的可见性，但由于总线嗅探机制，会不断的监听总线，如果大量使用 volatile 会引起总线风暴。所以，volatile 的使用要适合具体场景，不要随便使用 



**总线风暴** 

+ 基于 CPU 缓存一致性协议，JVM 实现了 volatile 的可见性，但如果在短时间内产生大量的CAS操作再加上 volatile的嗅探机制则会不断地占用总线带宽，导致总线流量激增，就会产生总线风暴





**既然CPU有缓存一致性协议（MESI），为什么JMM还需要volatile关键字 ？** 

单靠MESI并不能满足volatile，它只是volatile **语义实现** 的一部分， volatile 除了保证内存可见性，还可以限制JIT及时编译器指令重排，提供有序性的保证   



**内存屏障**

存储缓存和失效队列的引入在提升MESI协议实现的性能同时，也带来了一些问题。由于**MESI的高速缓存一致性是建立在强一致性的总线串行事务上的，而存储缓存和失效队列将事务的强一致性弱化为了最终一致性，使得在一些临界点上全局的高速缓存中的数据并不是完全一致的。** 所以CPU的设计者提供了**内存屏障**机制将对共享变量读写的高速缓存的强一致性控制权交给了程序的编写者或者编译器

CPU提供了三个汇编指令串行化运行读写指令达到实现保证读写有序性的目的

+ SFENCE：在该指令前的写操作必须在该指令后的写操作前完成
+ LFENCE：在该指令前的读操作必须在该指令后的读操作前完成
+ MFENCE：在该指令前的读写操作必须在该指令后的读写操作前完成

**缓存一直性协议是为了保证并发场景下线程可见性（实际所说的cpu 指令重排序并不是真正的排序，也是由于共享变量不是实时可见导致的），实际上是对 汇编指令上加了 锁操作的指令的变量进行应用, 表现到 java 层面是使用 volatile 关键字修饰的变量**  volatile指令重排，使用的是sfence指令实现了Store Barrier，相当于StoreStore Barriers， volatile保证内存可见性使用的Lock前缀指令 



**volatile 使用的优化**

+ 字节填充    Lock指令会锁缓存，通过字节填充避免要同时修改的volatile变量处于同一个缓存行



**synchronized  关键字和  volatile  关键字的区别**

1. `volatile 关键字是线程同步的轻量级实现，所以 `volatile `性能肯定比`synchronized`关键字要好。但是 `volatile` 关键字只能用于变量而 `synchronized 关键字可以修饰方法以及代码块  

2. `volatile` 不能保证数据的原子性`synchronized` 都能保证

3. `volatile`关键字主要用于解决变量在多个线程之间的可见性，而 `synchronized` 关键字解决的是多个线程之间访问资源的同步性



**synchronized 和 ReentrantLock 的区别**

1. 两者都是可重入锁 

   一个线程获得了某个对象的锁，此时这个对象锁还没有释放，它可以再次获取这个锁

2. synchronized 依赖于 JVM 而  ReentrantLock 依赖于 AQS 同步器的API

3. ReentrantLock  比  synchronized 使用更加灵活

   + 等待可中断              lock.lockInterruptibly()
   
   + 可以设置超时时间   超时没获得锁，就执行其他逻辑，不再争抢锁
   
   + 可实现公平锁防止饥饿  synchronized 只能是非公平锁，ReentrantLock默认是非公平的
   
   + 支持多个条件变量   一个条件变量类似 sychronized的waitSet   
   
     条件变量使用 await()前需要获得锁，await()后会释放锁，进入ConditionObject，await()被 唤醒 /打断 /超时 后重新竞争锁，竞争到锁后，从await后继续执行 
   
   + 都支持可重入 
   
   
   
   

## JMM  

**并发编程的三个重要特性**

也是JMM关注的三个方面

1. 原子性  临界区代码对外不可分割，不会被线程切换所打断   synchronized 可以保证  
2. 可见性  当一个变量对共享变量进行了修改，那么另外的线程都是立即可以看到修改后的最新值。`volatile` 关键字可以保证共享变量的可见性，synchronized  也可以保证可见性 
3. 有序性  JVM 会在不影响正确性的前提下，可以调整语句的执行顺序    volatile对编译器定制的volatile重排序规则和处理器的内存屏障插入策略能保证有序性，synchronized则是由 "一个变量在同一个时刻只允许一条线程对
   其进行lock操作" 这条规则获得的， 这个规则决定了持有同一个锁的两个同步块只能串行地进入保证了有序性  



**指令重排**

处理器指令重排

+ 指令执行五个阶段：取指令 间指 执行  访内 写回   
+ 流水线技术 增加吞吐量
+ 指令重排序：在不改变程序结果的前提下，这些指令执行的各个阶段可以通过重排序和组合来实现指令级并行

JIT即时编译器指令重排

+ JVM 也会在不影响正确性的前提下，可以调整语句的执行顺序 

**说说JMM** 

+ JMM控制了Java线程之间的通信，决定了对一个共享变量的写入何时对另外一个线程可见，定义线程和主内存之间的抽象关系，通过控制主内存与每个线程的本地内存之间的交互，为程序员提供一致的内存可见性保证  

+ JMM是语言级的内存模型，他确保在不同的编译器和处理器平台上，通过禁止特定编译器处理器重排序为程序员提供一致的内存可见性保证  

  

![image-20210907002821601](https://i.loli.net/2021/09/07/xeZJq76ANoPOKsb.png)

为了支持 JMM，Java 定义了 8 种 **原子操作** （Action），用来控制主存与工作内存之间的交互 

1. **read**   读取：  作用于主内存，将共享变量从主内存传动到线程的工作内存中，供后面的 load 动作使用
2. **load**   载入：  作用于工作内存，把 read 读取的值放到工作内存中的副本变量中
3. **store** 存储： 作用于工作内存，把工作内存中的变量传送到主内存中，为随后的 write 操作使用
4. **write**  写入： 作用于主内存，把 store 传送值写到主内存的变量中。
5. **use**   使用： 作用于工作内存，把工作内存的值传递给执行引擎，当虚拟机遇到一个需要使用这个变量的指令，就会执行这个动作 
6. **assign** 赋值：作用于工作内存，把执行引擎获取到的值赋值给工作内存中的变量，当虚拟机栈遇到给变量赋值的指令，执行该操作。比如 `int i = 1;` 
7. **lock（锁定）** 作用于主内存，把变量标记为线程独占状态  
8. **unlock（解锁）** 作用于主内存，它将释放独占状态    

把一个变量数据从主内存复制到工作内存，要顺序执行 read 和 load；而把变量数据从工作内存同步回主内存，就要顺序执行 store 和 write 操作     

8 种基本操作的规则

1. read 和 load 与  store 和 write 必须成对出现
2. 变量在工作内存中改变后必须同步回主存，不允许丢弃assign
3. 不允许一个线程无原因地（没有发生过任何assign操作） 把数据从线程的工作内存同步回主内存中 
4. use store 之前需要执行   load  和 assign 
5. 一个变量在同一个时刻只允许一条线程对其进行lock操作，但lock操作可以被同一条线程重复执
   行多次 **有序性+ syhcronized可重入** 
6. 如果对一个变量执行lock操作， 那将会清空工作内存中此变量的值 ，在执行引擎使用这个变量
   前， 需要重新执行load或assign操作以初始化变量的值  
7. unlock之前必须 lock 
8. 对一个变量执行unlock操作之前， 必须先把此变量同步回主内存中（执行store、 write操作）

JMM 通过read、 load、 assign、 use、 store、write 与lock 和 unlock 操作可以实现**原子性**；通过volatile，sychronized（unlock操作之前，必须先把此变量同步回主内存中 (执行store、 write操作)  ，final (被final修饰的字段在构造器中一旦被初始化完成， 并且构造器没有把 " this " 的引用传递出去，那么在其他线程中就能看见final字段的值，原理是内存屏障)**可见性**；通过volatile 和 synchronized (规则**5**) 两个关键字来保证线程之间操作的**有序性** 



**happens-before**  `上述规则很复,通过happens-before来判断`  



## 线程池

**使用线程池的好处**

1. **降低资源消耗**。通过重复利用已创建的线程降低线程创建和销毁造成的消耗，因为JAVA线程与操作系统内核线程是1：1 的关系，每次创建和回收线程都要进行系统调用  
2. **提高响应速度** 当任务到达时，任务可以不需要的等到线程创建就能立即执行。
3. **提高线程的可管理性**  线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，监控和调优`beforeExecute，afterExecute，terminated 可自己拓展` 

### Executor 框架 

通过 Executor 来启动线程比使用 Thread 的 start 方法更好，除了更易管理，效率更好（用线程池实现，节约开销）外，还有助于避免 **this 逃逸** 问题 —— 构造函数返回之前其他线程就持有该对象的引用

Executor 框架不仅包括了线程池的管理，还提供了线程工厂、队列以及拒绝策略等，Executor 框架让并发编程变得更加简单 

队列的泛型是Runable， AbstractExecutorService  可以将 Callable 任务转化成Runable子类 FutureTask 

![image-20210909142958266](https://i.loli.net/2021/09/09/Y2GcbNrItoZBdQj.png)

#### Executor 三大部分组成

1. 任务(Runnable / Callable)

2. 任务的执行Executor 

   + **`ThreadPoolExecutor`**
   + **`ScheduledThreadPoolExecutor`**

3. 异步计算的结果(Future) 

   Future 接口以及 Future 接口的实现类 FutureTask类都可以代表异步计算的结果

将 **`Runnable`接口** 或 **`Callable` 接口** 的实现类提交给 **`ThreadPoolExecutor`** 或 **`ScheduledThreadPoolExecutor`** 执行。(调用 `submit()` 方法时会返回一个 **`FutureTask`** 对象）



#### Executor 框架的使用流程  

1. 主线程首先要创建实现 `Runnable` 或者 `Callable` 接口的任务对象 
2. 把创建完成的实现        `Runnable`/`Callable`接口的 对象直接交给 `ExecutorService` 执行，Executor.execute()，ExecutorService.submit()   
3. submit()   方法会返回一个实现 Future 接口的对象 
4. 主线程执行 `FutureTask.get()`获取执行结果 



### 创建线程池 

1. 通过**通过构造方法实现**   

   ThreadPoolExecutor executor= new ThreadPoolExecutor(.....) 

2. 通过 Executor 框架的工具类 Executors 来实现，该类中提供了众多工厂方法来创建各种用途的线程池  

   

**newFixedThreadPool**  

+ 没有救急线程，也不需要超时时间

+ LinkedBlockingQueue 阻塞无界队列，可以放任意数量的任务

+ **适用于任务量已知，相对耗时的任务，任务比较多时导致OOM**

  

**newCachedThreadPool** 

+ 全部都是救急线程， 60s 空闲生存时间 
+ SynchronousQueue 无边界的同步队列，必须是一个线程接收任务一个线程提交任务，这个队列专门和newCachedThreadPool搭配 
+ **整个线程池表现为线程数会根据任务量不断增长，没有上限，当任务执行完毕，空闲 1分钟后释放线
  程。 适合任务数比较密集，但每个任务执行时间较短的情况，线程较多耗尽CPU，任务较多OOM**



**newSingleThreadExecutor**

+ 只有一个核心线程 
+ LinkedBlockingQueue 阻塞无界队列

+ **希望多个任务排队执行。线程数固定为 1，任务数多于 1 时，会放入无界队列排队。任务执行完毕，这唯一的线程也不会被释放。 任务较多OOM  ** 

  那为什么不自己创建一个单线程串行执行任务 ？

+ 自己创建一个单线程串行执行任务，如果任务执行失败而终止那么没有任何补救措施，而线程池还会新建一
  个线程，保证线程池的正常工作 

+ 与newFixedThreadPool 区别 Executors.newFixedThreadPool(1) 初始时为1，可以强转为 ThreadPoolExecutor 对象调用 setCorePoolSize 等方法进行修改，而newSingleThreadExecutor 使用了装饰器模式只对外暴露了ExecutorService接口，只能使用该接口的方法，而不能使用实现类中特有的方法。



**Timer**

+ 缺点：一个线程串行，前一个任务的延迟或异常都将会影响到之后的任务  



**ScheduledThreadPoolExecutor**

+ DelayQueue 作为任务队列 DelayQueue基于优先级队列的DelayQueue 即延迟队列，也就是一个按延迟时间从小到大出队的PriorityQueue 

1. 延时 schedule方法 
2. 定时 
   + scheduleAtFixedRate 执行时间大于间隔，下一个任务会在这个任务执行完毕后立刻执行，但是不会和前面的一起执行，重叠在一起。
   + scheduleWithFixedDelay 从上一次任务执行结束之后开始算间隔时间 



### 线程池异常处理

+ 其他任务不会受到任务异常影响，所有线程池出现异常不会抛出。
+ 所有线程池出现异常不会抛出 
  + 必须自己 try catch 处理
  + Callable(有返回值) + Future get 方法获取异常信息(如果出现了异常)。 



### 线程池大小确定

+ CPU 密集型任务(N+1)  防止线程偶发的缺页中断，申请调页，这时候多的这个线程就可以利用CPU 

+ I/O 密集型任务(2N)   

  系统会用大部分的时间来处理 I/O 交互，而线程在处理 I/O 的时间段内不会占用 CPU 来处理，这时就可以将 CPU 交出给其它线程使用



### 线程池核心参数

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler)  
corePoolSize    核心线程数目 (最多保留的线程数)  
maximumPoolSize 最大线程数目  corePoolSize + 救急线程数量
keepAliveTime   生存时间 - 针对救急线程
unit            时间单位- 针对救急线程
workQueue       阻塞队列
threadFactory   线程工厂 - 可以为线程创建时起个好名字
handler 拒绝策略 
```

### 线程池的拒绝策略

+ AbortPolicy                让调用者抛出 RejectedExecutionException 异常，这是默认策略
+ CallerRunsPolicy      让调用者运行任务
+ DiscardPolicy            放弃本次任务
+ DiscardOldestPolicy 放弃队列中最早的任务，本任务取而代之

### 线程池状态

使用 int 的高 3 位来表示线程池状态，低 29 位表示线程数量  

这些信息存储在一个原子变量 ctl 中，目的是将线程池状态与线程个数合二为一，这样就可以用一次 CAS 原子操作
进行赋值   

|    状态名    | 高 3 位 | 接收新任 务 | 处理阻塞队列任 务 | 说明                                                         |
| :----------: | ------- | ----------- | ----------------- | ------------------------------------------------------------ |
|   RUNNING    | 111     | Y           | Y                 | 接收新任务，并处理阻塞队列中的任务                           |
|   SHUTDOWN   | 000     | N           | Y                 | 不会接收新任务，但会处理阻塞队列剩余任务                     |
|     STOP     | 001     | N           | N                 | 不接收新任务，会中断正在执行的任务，并抛弃阻塞队列任务，返回抛弃的任务 |
| TIDYING 过渡 | 010     | -           | -                 | 任务全执行完毕，活动线程为 0  即将进入 终结                  |
|  TERMINATED  | 011     | -           | -                 | 终结状态                                                     |

TERMINATED  >  TIDYING   >  STOP  >  SHUTDOWN  >  RUNNING  

![image](https://i.loli.net/2021/09/01/cWSjnXYaEDNqQ5R.png)

### 线程池原理

+ 线程池中刚开始没有线程，当一个任务提交给线程池后，线程池会创建一个新线程来执行任务
+ 当线程数达到 corePoolSize 并没有线程空闲，这时再加入任务，新加的任务会被加入 workQueue 队列排队，直到有空闲的线程。
+ 如果队列选择了有界队列，那么任务超过了队列大小时，会创建 maximumPoolSize - corePoolSize 数目的线程来救急。
+ 如果线程到达 maximumPoolSize 仍然有新任务这时会执行拒绝策略   

![image-20210901162131626](https://i.loli.net/2021/09/02/qYO2kX7mdDhpMIb.png)

#### 源码分析

ThreadPoolExecutor

##### execute 任务提交流程 

1. 获取并当前线程池状态

2. 当前线程数小于corePoolSize，则向workers添加一个核心线程，**成功**则返回

3. 1)  添加失败，重新获取线程池状态

   ​     如果线程池状态处于RUNNING，  offer() 方法将任务放入阻塞队列

   2)  加入队列成功

   ​	 第二次检查线程池状态，若加入阻塞队列后不是RUNNING了就从队列中移除，移除成功则执行拒绝策略

   ​	 移除失败，就给这个任务一个被执行的机会，若当前线程池工作线程为空，尝试新增Worker去消费阻塞队列的任务  

   3) 加入队列失败失败说明阻塞队列已满，则尝试添加新线程执行 (救急线程)， 添加失败则执行拒绝策略




##### addWorker

addWorker 这个方法主要用来创建新的工作线程，如果返回 true 说明创建和启动工作线程成功，否则的话返回的就是false 

1. 校验当前的线程池状态是否能接受新任务，不能则返回false 

2. 1）能则循环 + CAS 增加线程的个数    如果线程超过个数限制则返回 false 

   ​      CAS成功进入worker 创建流程

   ​      失败查看线程池是否变化，变化就 go to到for 外层重新获取线程池状态，没有变化内层重新CAS增加线程个数

    2)  创建worker，存放worker集合的HashSet不是线程安全的，为了防止add时候其他线程remove，需要加独占锁，再次检查线程池状态，如果线程池状态处于**非RUNNING 或 SHUTDOWN且当前Worker是为了用于消费阻塞队列缓存的任务**时候向workers新增worker，成功后释放独占锁，如果出现错误释放锁后执行finally方法  

3. 添加成功，启动工作线程，若添加失败从workers中移除工作线程并通过CAS减少线程个数(finally方法) 

**CAS 增加线程的个数这个逻辑会保证同时只有一个线程能执行创建worker，其他线程都将自旋等待** 



##### 任务执行流程分析

runWorker   

Worker继承了AQS，实现了Runable，任务执行由Worker来实现，它自己可能有任务，如果为空，就从队列中取任务 

启动工作线程会执行run方法，在run方法中执行 runWorker 方法

1. 不断从队列中获取任务执行，没有任务了就阻塞等待在阻塞队列直到获取到队列的任务，如果判断当前Worker需要被回收就返回null，只要getTask()返回null，就会进行回收Worker操作  **5** 
2. 获取到任务，执行前加锁  
3. task.run()
4. 执行完毕释放锁 
5. Worker退出，执行清理工作   



**详解Worker什么时候需要被回收** 

getTask方法中 

1. 判断线程池状态，如果线程池状态大于等于SHUTDOWN 并且状态大于等于STOP或者是SHUTDOWN且队列为空，则返回null，需要回收 

2. 超过了最大线程数且工作队列为空；超过了核心线程且获取超时(keepAliveTime) 



##### 正确关闭线程池步骤

调用shutDown()或者shutDownNow()，后调用awaitTermination等待线程池关闭。awaitTermination循环判断线程池是否达到终止状态，是返回，否则在条件变量termination上termination.awaitNanos(nanos)



###### **shutdown**

+ shutdown方法中断线程之前会调w.tryLock，运行线程会获取锁，tryLock会失败，所以不会对正在执行任务的线程进行中断，而会对空闲的线程进行中断；
+ 对于空闲的线程，是阻塞在getTask()的workQueue.take()方法中的，所以当线程被中断后，会捕捉InterruptedException异常，后判断线程池状态是SHUTDOWN并且队列为空则任务返回空，getTask方法便返回 null，Worker被回收



###### **shutdownNow**

它是对每个线程调用 t.interrupt() 方法来实现的，试图停止所有正在执行的线程，不过只是尽力尝试终止正在执行的线程，不保证一定能终止，因为不响应中断的线程无法终止或在移除。之后就会把那些未执行的任务进行**移除并且返回**。





### 线程池常见的对比

Runnable vs Callable

`Runnable`      接口不会返回结果或抛出检查异常，但是 Callable 接口可以，如果任务不需要返回结果或抛出异常推荐使用 `Executors`    可以实现将 `Runnable` 对象转换成 `Callable` 对象                                                                                       `Callable`      接口中的 call() 方法是有返回值的，和 FutureTask 配合可以用来获取异步执行的结果。   



execute() 方法和 submit() 方法的区别 

1. `execute()`方法用于提交不需要返回值的任务，所以无法判断任务是否被线程池执行成功与否；

2. `submit()`方法用于提交需要返回值的任务。线程池会返回一个 `Future` 类型的对象，通过这个 `Future` 对象可以判断任务是否执行成功，并且可以通过 `Future` 的 `get()`方法来获取返回值，`get()`方法会阻塞当前线程直到任务完成

   如果执行过程抛出异常，get()方法也会抛出异常 



isTerminated()  VS  isShutdown() 

+ isShutDown  当调用 shutdown()方法后返回为 true 

+ isTerminated 当调用 shutdown() 方法后，并且所有提交的任务完成后返回为 true 



# JUC源码

JUC包下三类关键词 

Blocking         大部分实现基于锁，并提供用来阻塞的方法  

CopyOnwrite  修改开销大         对读的性能要求很高，读多写少，弱一致性

Concurrent类型的容器

+ 内部很多操作使用cas优化，一般可以提供较高吞吐量

+ 弱一致性

  + size 操作未必是 100% 准确  
  + 遍历时弱一致性
  + 读取弱一致性

    

## Atomic 原子类

**基本类型**

AtomicInteger：整形原子类AtomicLong： 长整型原子类    AtomicBoolean：布尔型原子类  

**数组类型**

使用原子的方式更新数组里的某个元素

AtomicIntegerArray：整形数组原子类 AtomicLongArray ：长整形数组原子类

AtomicReferenceArray：引用类型数组原子类 

**引用类型**

1. `AtomicReference`：引用类型原子类 
2. `AtomicStampedReference`：AtomicStampedReference它内部不仅维护了对象值，还维护了一个时间戳（我这里把它称为时间戳，实际上它可以使任何一个整数，它使用整数来表示状态值）。当AtomicStampedReference对应的数值被修改时，除了更新数据本身外，还必须要更新时间戳。当AtomicStampedReference设置对象值时，对象值以及时间戳都必须满足期望值，写入才会成功。因此，即使对象值被反复读写，写回原值，只要时间戳发生变化，就能防止不恰当的写入 
3. `AtomicMarkableReference` ：原子更新带有标记位 的 引用类型     

**对象的属性修改类型**

- `AtomicIntegerFieldUpdater`：原子更新整形字段的更新器
- `AtomicLongFieldUpdater`：      原子更新长整形字段的更新器
- `AtomicReferenceFieldUpdater`：原子更新引用类型字段的更新器

### AtomicInteger  类的原理   

AtomicInteger 类主要利用 CAS + volatile 和  native 方法来保证原子操作  

CAS 的原理是拿期望的值和原本的一个值作比较，如果相同则更新成新的值。UnSafe 类的 objectFieldOffset() 方法是一个本地方法，这个方法是用来拿到 "原来的值"  的内存地址，返回值是 valueOffset 。value 是一个 volatile 变量，可以保变量的修改对其他线程可见



**JDK 8 引入了一个原子累加器 ** LongAdder 专门用作累加

LongAdder原理

+ 没有竞争在base上累加，有竞争创建累加单元，累加单元最多和CPU核心数一样，个数达到了核心数，就通过不断循环，每次循环就换累加单元，尝试累加，CHM也计数原理与之类似 
+ 使用了字节填充解决伪共享

**伪共享** 一个缓存行加入了多个对象，Core1要修改对象1，Core2要修改对象2，但是由于他们在一个缓存行，会导致Core1修改缓存行中对象1的时候，会导致Core2的缓存行失效，必须从内存读取最新值，导致效率降低 。

**填充字节** 

Java8中新增了一个注解：@sun.misc.Contended 用来解决累加单元Cell的伪共享，它的原理是在使用此注解的对象或字段的前后各增加 128 字节大小的padding，从而让 CPU 将对象预读至缓存时占用不同的缓存行，这样，不会造成对方缓存行的失效，需要注意的是此注解默认是无效的，需要在jvm启动时设置 -XX:-RestrictContended才会生效



## AQS

抽象队列同步器 ，AQS 就是一个抽象类，主要用来构建锁和同步器  

锁：ReentrantLock 读写锁   同步器：Semaphore ，门闩，栅栏 

**如何实现一个自定义同步组件**

+ 以实现一个独占锁为例子，实现一个静态内部类 Sync， 该类 继承 AQS 重写 AQS 提供的独占式获取与释放同步状态的tryAcquire，tryRelease，isHeldExclusively 等方法， 在重写的方法中，使用AQS提供的设置内部设置状态state的方法即可 
+ 自定义同步组件实现Lock接口，重写的方法中调用(组合委派的方式调用)内部类Sync的方法比如  tryLock 调用 acquire，acquire又调用 tryAcquired   

AQS 锁内存语义的实现 

+ 利用volatile变量(state)的写-读所具有的内存语义   
+ 利用CAS所附带的volatile读和volatile写的内存语义   

**对AQS的理解**

同步器是用来构建锁和同步器组件的基石，内部维护了一个FIFO的自旋队列来实现线程的排队，并使用volatile类型变量state来表示同步状态，利用 volatile 变量的写-读所具有的内存语义和利用 CAS所附带的 volatile 读和 volatile 写的内存语义来实现线程之间的通信保证内存可见性，它屏蔽同步状态管理、线程的排队等待唤醒的细节，简化了我们实现同步器组件的难度   

**AQS原理**

AQS 使用一个 int 成员变量来表示同步状态，通过内置的 FIFO 队列来完成获取资源线程的排队工作。AQS 使用 CAS 对同步状态进行原子操作 实现对其值的修改。

**结点*状态*waitStatus**    CANCELLED =  1(由于超时或中断，此节点被取消)  SIGNAL    = -1 CONDITION = -2  PROPAGATE = -3，刚加入默认是 0 

**AQS 对资源的共享方式**

1. 独占式 只有一个线程能获取锁，分为公平和非公平锁 

   公平锁：    lock acquire tryAcquire 尝试获取锁的线程不是老二就不会获取锁成功

   非公平锁： lock 方法一上来会CAS抢锁，抢锁失败才调用acquire，acquire中再次 tryAcquire CAS 抢锁  

非公平锁与公平锁只有两点区别，对于非公平锁如果这两次 CAS 都不成功，那么后面非公平锁和公平锁是一样的，都要进入到阻塞队列等待唤醒 

+ 公平锁能防止  "饥饿"  ，但是会进行大量的线程切换

+ 非公平锁，可能出现饥饿，但是线程切换次数少，吞吐量高  



2. Share（共享）

   多个线程可同时执行获取锁，同步器 Semaphore，CountDownLatch属于共享方式

3. 独占+共享

   ReentrantReadWriteLock 读锁属于共享方式，写锁属于独占方式
   
   

**两种模式对比** 

+ AQS 独占模式 锁只能被一个线程获取
  AQS 共享模式 锁可以被多个线程获取 
+ 独占模式只会把将要出队的节点线程唤醒 
  共享模式下除了把将要出队的节点的线程唤醒之外还会唤醒后续处于挂起状态的节点

独占式获取同步状态流程  

![image-20210909153554996](G:\markdown图片\image-20210909153554996.png)

独占式超时获取同步状态流程 与独占式获取同步状态类似，只是多了一个超时等待，每次自旋失败会判断是否超时，超时则退出，否则进入等待状态，等待状态可能被前驱节点唤醒或者被打断，若是被打断则抛出异常后退出 

**AQS的模板方法**

继承 AbstractQueuedSynchronizer 并重写同步器提供的操作state变量的**空方法**，tryAcquire，tryRelease，acquireShared，releaseShared，当我们调用同步器工具提供的使用接口，会先调用AQS模板方法，模板方法会调用具体同步器实现的方法，**如果未实现会抛出异常** 

具体同步器  ->  AQS模板  ->  子类具体实现方法  ->  AQS提供操作状态方法 

```java
acquire  -> acquireShared    - >  tryAcquireShared (子类实现 tryAcquireShared (F/N/Syn) 返回值大于等于0 获取锁成功 )  ->  doAcquireShared  

release  -> releaseShared    - >  tryReleaseShared(子类实现tryReleaseShared (Sync) 读写锁/CDL 释放完毕才返回 true Semaphore 释放成功就 返回 true )  -> doReleaseShared 

ReentrantLock 带超时的 tryLock
tryLock  -> tryAcquireNanos  ( AQS中实现 ) - > tryAcquire 失败再调用 doAcquireNanos 
    
不带超时
tryLock  -> nonfairTryAcquire(自己的Sync实现) 
注意:无论是公平还是非公平锁  tryLock 都是非公平的所以nonfairTryAcquire是在Sync里面实现,也就是自定义同步器的内部类中 

lock   -> acquire -> tryAcquire(自己实现) 失败 ——> nonfairTryAcquire(自己实现)
unlock -> release -> tryRelease(自己实现)  
```





### ConditionObject

AQS的内部类 类似于 sychronized 的waitSet

await()

+ **获取锁**调用 await()方法   Node node = addConditionWaiter(); 将线程加入条件变量的双向链表中(`该双向链表没有dummy`) 等待。节点Node 的 waitStatus 为 -2，调用fullyRelease 释放锁上所有重入次数，唤醒锁等待队列中下一个节点，挂起线程直到其它线程将该线程从等待队列移动**外部的锁等待队列**或者被打断 

signal()  

+ 锁的持有者调用signal()，doSignal(first) 将头节点从条件变量链表中断开，将下一个节点设置为first。transferForSignal 将节点加入锁等待队列(将waitStatus 由-2 改为0 ，enq(node)入队，enq(node)成功则返回其前一个结点将其waitStatus设置为-1，以唤醒后继节点) 

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## 锁

### ReentrantLock

会在第一次发生竞争的时候，创建伪元  

1. 加锁成功流程 lock()方法，CAS成功，设owner为当前线程
2. 加锁失败流程 lock()方法，CAS失败，调用AQS acquire，再次CAS抢锁，失败addWaiter()方法进入双向队列，队列最开始有个dummy头节点(**调用addWaiter()说明有竞争**，会创建一个dummy，以方便唤醒后续节点)，入队的节点依次排在尾部，入队后进入acquireQueued方法的for循环自旋。如果当前结点是头结点的后一个结点，他将尝试获取锁，获取锁失败，调用shouldParkAfterFailedAcquire()会将他前一个结点waitstatus设置为-1，负责唤醒他，第一次该方法返回false，第二次获取锁失败，该方法返回true，parkAndCheckInterrupt()进入等待状态。 
3. 解锁竞争成功 unlock()方法      调用AQS的release，先tryRelease()  设置锁的状态，设置owner为null，如果头节点的waitStatus!=0 也即是 -1，唤醒后继线程，被唤醒的线程的前一个结点当然是头节点，他会尝试获取锁，获取锁成功，将自己设置为头节点，thread 设置为null    
4. 解锁竞争失败  **仍然** parkAndCheckInterrupt() 进入等待状态，等待唤醒
5. 可重入原理   tryAcquire() 时候调用nonfairTryAcquire() ，getState()获取锁状态，为0则CAS抢锁(非公平锁的实现)，如果不等于0，且拥有所得线程是自己，则state++，tryRelease() 如果拥有所得线程是自己，就 state--, 直到减为0，返回 true，随后唤醒后继线程  
6. **默认不可打断**    被打断后，设置interrupted为true，在获取锁后返回，然后再设打断标记 
7. 可打断原理        lockInterruptibly()方法，调用AQS的acquireInterruptibly 在被打断后会抛出异常
8. tryRelease       在释放锁(state减为0) 的时候才会返回true
9. tryAcquire        在获取锁成功的时候才会返回true
10. 为什么要有 PROPAGATE  -3    只有doReleaseShared用到了，可以有效避免在高并发场景下可能出现的、线程没有被成功唤醒的情况出现 

**公平锁比非公平锁就多了两次CAS抢锁**  



**乐观锁与悲观锁**

悲观锁认为数据很容易被其他线程修改，所以在处理数据前先加锁，一般使用数据库提供的锁机制

乐观锁并不会使用数据库提供的锁机制，一般在表中添加版本字段version，更新前需要查询version，更新时携带version字段与数据库中的版本字段比较，相同则更新并将version+1，这样在多线程更新时同一时刻只有一个线程能成功，根据更新操作的返回值可以判断是否成功，成功则返回，失败则可以重试一定次数，类似于CAS自旋，不过不是死循环。





## 同步器

## CountDownLatch

CountDownLatch 构造方法，传入state的初始值，刚开始占有这么多次锁，子任务完成后释放锁，整个过程不会出现加锁的过程



latch.await()  

+ 调用 acquireSharedInterruptibly  调用 tryAcquireShared   如果当前 state=0 则 tryAcquireShared 返回 1，代表没有子任务，直接返回主线程不会被阻塞

+ 当 tryAcquireShared 返回 -1，代表还有子任务在执行，doAcquireSharedInterruptibly 执行主任务进入自旋并等待

latch.countDown()

+ 子任务执行完毕，调用 latch.countDown()，释放锁，最后一个子任务释放锁时候 state=0，tryReleaseShared 返回true，会执行doReleaseShared方法唤醒后继节点，也就是唤醒我们阻塞的主线程   
+ 主线程被唤醒，后执行 tryAcquireShared，返回1，唤醒后续可能存在的主线程(等待的主任务可能有多个)



## Semaphore

构造方法将资源数目赋值给同步器的state，state最多减少到0，后续线程加入AQS队列等待 

acquire()

+ acquire() 调用同步器的acquireSharedInterruptibly()，tryAcquireShared返回剩余的许可数。tryAcquireShared() 返回>=0  相当于加锁成功，结束。
+ tryAcquireShared()返回 < 0 ，获取锁失败 ，进doAcquireSharedInterruptibly()，添加在头节点dummy后(节点是SHARED类型)，自旋，获取锁失败，进入等待 

release()

+ release() 调用 AQS releaseShared()，调用子类实现的 tryReleaseShared()，释放许可成功，成功**就**执行doReleaseShared()，修改 waitStatus 唤醒后续节点  

  

**tryAcquireShared()**

state = 0，返回1，获取锁成功需要唤醒后续线程

state不等于 0 ，返回 -1，获取锁失败

返回 0，获取锁成功，不需要唤醒后续线程 ，针对 Semaphore



**tryReleaseShared()**

CountDownLatch 释放完毕才返回 true，Semaphore 释放成功就返回 true

```java
acquire  -> acquireShared    - >  tryAcquireShared(子类实现 tryAcquireShared (F/N/Syn) 返回值大于等于0 获取锁成功 )  ->  doAcquireShared  

release  -> releaseShared    - >  tryReleaseShared(子类实现tryReleaseShared (Sync) 读写锁/CDL 释放完毕才返回 true Semaphore 释放成功就 返回 true )  -> doReleaseShared 

ReentrantLock 带超时的 tryLock
tryLock  -> tryAcquireNanos  ( AQS中实现 ) - > tryAcquire 失败再调用 doAcquireNanos 
    
不带超时
tryLock  -> nonfairTryAcquire(自己的Sync实现) 
注意:无论是公平还是非公平锁  tryLock 都是非公平的所以nonfairTryAcquire是在Sync里面实现,也就是自定义同步器的内部类中 

lock   -> acquire -> tryAcquire(自己实现) 失败 ——> nonfairTryAcquire(自己实现)
unlock -> release -> tryRelease(自己实现)  
```

## 并发容器

所有的并发容器都不允许加入null 



### ConcurrentHashMap

线程安全的HashMap

#### ConcurrentHashMap 8

相比较于ConcurrentHashMap 7  实现了懒惰初始化  

红黑树

+ 链表长度超过 8，先扩容到64之后才会转红黑树，替换一维链表结构，加快查找效率。
+ 红黑树节点小于6，转回链表 

重要属性和内部类

```java
// 默认为 0
// 当初始化时, 为 -1,代表table还没创建
// 当扩容时, 为 - (1 + 扩容线程数<可以帮忙扩容>) 
// 当初始化或扩容完成后, 为下一次的扩容的阈值大小
private transient volatile int sizeCtl;
// 整个 ConcurrentHashMap 就是一个 Node[]
static class Node<K,V> implements Map.Entry<K,V> {}
// hash 表  
transient volatile Node<K,V>[] table;
// 扩容时的 新 hash 表
private transient volatile Node<K,V>[] nextTable;
// 扩容时如果某个 bin 迁移完毕, 用 ForwardingNode 作为旧 table  bin 的头结点
static final class ForwardingNode<K,V> extends Node<K,V> {}
// 用在 compute 以及 computeIfAbsent 时, 用来占位, 计算完成后替换为普通 Node
static final class ReservationNode<K,V> extends Node<K,V> {}
// 作为 treebin 的头节点, 存储 root 和 first 
static final class TreeBin<K,V> extends Node<K,V> {} 
// 作为 treebin 的节点, 存储 parent, left, right 
static final class TreeNode<K,V> extends Node<K,V> {}
```

  重要方法

```java
// 获取 Node[] 中第 i 个 Node  
static final <K,V> Node <K,V> tabAt(Node<K,V>[] tab, int i)
// cas 修改 Node[] 中第 i 个 Node 的值, c 为旧值, v 为新值
static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i, Node<K,V> c, Node<K,V> v)
// 直接修改 Node[] 中第 i 个 Node 的值, v 为新值 
static final <K,V> void setTabAt(Node<K,V>[] tab, int i, Node<K,V> v)  
```

  **构造方法**

`可以看到实现了懒惰初始化，在构造方法中仅仅计算了 table 的大小，以后在第一次使用时才会真正创建  `

```java
实现了懒惰初始化,仅仅计算  
public ConcurrentHashMap(int initialCapacity,
                             float loadFactor, int concurrencyLevel) {
        if (!(loadFactor > 0.0f) || initialCapacity < 0 || concurrencyLevel <= 0)
            throw new IllegalArgumentException();
        if (initialCapacity < concurrencyLevel)   // Use at least as many bins
            initialCapacity = concurrencyLevel;   // as estimated threads
        // tableSizeFor 仍然是保证计算的大小是 2^n, 即 16,32,64 ... hash表大小必须是2^n才能正常工作
        long size = (long)(1.0 + (long)initialCapacity / loadFactor);  8/0.75 +1  = 11.几
        int cap = (size >= (long)MAXIMUM_CAPACITY) ?
            MAXIMUM_CAPACITY : tableSizeFor((int)size);
        this.sizeCtl = cap;
  }
```

##### **put**

binCount 代表链表长度 

```java
   public V put(K key, V value) {
        return putVal(key, value, false); //true putIfAbsent 只有第一次put 才会将键值对放入map,后续不会用新值覆盖旧值。 false 会
    } 
   final V putVal(K key, V value, boolean onlyIfAbsent) {
       //普通hashmap 允许,ConcurrentHashMap 不允许
        if (key == null || value == null) throw new NullPointerException();
       // 其中 spread 方法会综合高位低位,具有更好的 hash 性
        int hash = spread(key.hashCode());
        int binCount = 0;
        for (Node<K,V>[] tab = table;;) {
            // f 是链表头节点
           // fh 是链表头结点的 hash
           // i 是链表在 table 中的下标
            Node<K,V> f; int n, i, fh;
            if (tab == null || (n = tab.length) == 0) 
                tab = initTable(); //懒惰初始化 —— 第一次 put没创建就创建 CAS创建,进入下次循环继续执行put
            //有无头节点, 没有头节点f,创建Node 使用CAS将头节点设置为新建的Node,失败进入下个循环。
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            //帮容扩容 是ForwardingNode 的 hash 说明其他线程在扩容,也去锁住某一个链表扩容,帮助扩容。
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            //桶下标冲突才会加锁,对链表头节点进行加锁。
            else {
                V oldVal = null;
               //对链表头节点进行加锁。
                synchronized (f) {
                    //再次判断头节点是否被移动过 
                    if (tabAt(tab, i) == f) {
                        //>=0普通节点
                        if (fh >= 0) {
                            binCount = 1;
                            //从头节点开始遍历
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                //找到相同key更新
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                Node<K,V> pred = e;
                                //没找到 创建一个新节点,追加在链表上。
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
                        //hash码小于0 看是不是红黑树
                        else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                //链表长度, binCount != 0 该链表有冲突
                if (binCount != 0) {
                    // 如果链表长度 >= 树化阈值(8), 进行链表转为红黑树(先扩容超过64再转红黑树)
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
       // 增加 size 计数     类似 LongAdder计数器方法(设置多个累加单元)
        addCount(1L, binCount);
        return null;
    }
```

##### **initTable**

```java
private final Node<K,V>[] initTable() {
        Node<K,V>[] tab; int sc;
       //hash表还没有创建
        while ((tab = table) == null || tab.length == 0) {
            //<0 其他线程在创建hash表还没创建完,让出cpu 
            if ((sc = sizeCtl) < 0) 
                Thread.yield(); // lost initialization race; just spin
            //SIZECTL 改为-1 -1表示正在创建hash表,其他线程CAS失败,
            else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                try {
                    if ((tab = table) == null || tab.length == 0) {
                        //没创建,创建    sc是创建的初始容量 sc没初始化  就用默认值DEFAULT_CAPACITY 16 ;
                        int n = (sc > 0) ? sc : DEFAULT_CAPACITY;    // 16
                        @SuppressWarnings("unchecked")
                        Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                        table = tab = nt;
                        //下次要扩容的阈值 
                        sc = n - (n >>> 2);
                    }
                } finally {  
                    // sizeCtl 恢复成正数,为下一次扩容的阈值  
                    sizeCtl = sc; 
                }
                break;
            }
        }
        return tab;
    }
```

##### **get**

```java
public V get(Object key) {
        Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
        int h = spread(key.hashCode());
    	//table不为空
        if ((tab = table) != null && (n = tab.length) > 0 &&
            //计算桶下标,根据桶下标找链表头节点。
            (e = tabAt(tab, (n - 1) & h)) != null) {
            //头节点hash码是否key哈希码,hash码相同,同一个对象或值相等返回value
            if ((eh = e.hash) == h) {
                if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                    return e.val;
            }
            //头节点hash码是负数 
            //1.链表在扩容中,扩容是从后往前扫描hash表,对每个链表进行扩容,处理过的链表头节点为ForwardingNode             hash码为-1,会调用ForwardingNode 的find去新的table中找key。
            //2.该位置链表转成了红黑树 TREEBIN hash码是-2,调用find去红黑树中查找。
            else if (eh < 0)
                return (p = e.find(h, key)) != null ? p.val : null;
            //正常链表查找
            while ((e = e.next) != null) {
                if (e.hash == h &&
                    ((ek = e.key) == key || (ek != null && key.equals(ek))))
                    return e.val;
            }
        }
        return null;
    }
```

##### **addCount**  

添加元素之后，用来维护计数

```java
1.维护计数
2.hash表计数超过阈值,他会扩容。 
// 增加 size 计数     类似 LongAdder计数器方法(设置多个累加单元)
addCount(1L, binCount); 

按 (1) -> (2) -> (3) -> (4) -> (5)
private final void addCount(long x, int check) {
         //多个累加单元
        CounterCell[] as; long b, s;
        // 已经有了 counterCells, 已经有竞争了,向 cell 累加。
        // 还没有在基础的数值上加  (还没有, 向 baseCount 累加)
        if ((as = counterCells) != null ||
            !U.compareAndSwapLong(this, BASECOUNT, b = baseCount, s = b + x)) {
            CounterCell a; long v; int m;
            boolean uncontended = true;
            //累加单元数组 counterCells 还没有
            if (as == null || (m = as.length - 1) < 0 ||
                // 还没有 累加单元 cell
                (a = as[ThreadLocalRandom.getProbe() & m]) == null ||
                !(uncontended =
                //或者累加失败
                U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))) {
                // 创建累加单元数组和cell,累加重试
                fullAddCount(x, uncontended);
                return;
            }
            //链表长度
            if (check <= 1)
                return;
            // 获取元素个数
            s = sumCount();
        }
         //链表长度>1 可能要扩容 
        if (check >= 0) {
            Node<K,V>[] tab, nt; int n, sc;
            //长度大于阈值,进行扩容 
            while (s >= (long)(sc = sizeCtl) && (tab = table) != null &&
                   (n = tab.length) < MAXIMUM_CAPACITY) {
                int rs = resizeStamp(n);
                //其他线程发现 ** sc小于0,在进行扩容  (2)
                if (sc < 0) {  //最开始table初始化后  sc是8,是正数,进入 
                     //newtable还没创建直接break ((nt = nextTable) == null) (3)
                    if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                        sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                        transferIndex <= 0) 
                        break;  //(4)
                    //newtable已经被创建了,帮忙扩容
                    if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                   transfer(tab, nt); //((nt = nextTable) == null),nextTable 赋值给了nt, 帮忙扩容 (5)
                }
                //CAS 将 SIZECTL 设置为负数,进入扩容   其他线程CAS失败再次循环 **  (1)
                else if (U.compareAndSwapInt(this, SIZECTL, sc,
                                             (rs << RESIZE_STAMP_SHIFT) + 2))
                    //tab原来hash表 null:新的hash表,因为是懒惰初始化,所以还没创建,故传null  
                    transfer(tab, null);
                s = sumCount();
            }
        }
    }
```

##### **size sumCount **

```java
public int size() {
    //获取计数值
        long n = sumCount();
        return ((n < 0L) ? 0 :
                (n > (long)Integer.MAX_VALUE) ? Integer.MAX_VALUE :
                (int)n);
    }


final long sumCount() {
        CounterCell[] as = counterCells; CounterCell a;
        long sum = baseCount;
        if (as != null) {
            //遍历所有累加单元数
            for (int i = 0; i < as.length; ++i) {
                if ((a = as[i]) != null)
                    sum += a.value;
            }
        }
        return sum;
    }
```

##### **transfer**

```java
//传过来的原始table    新table null   会延迟初始化 
private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
        int n = tab.length, stride; 
        if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
            stride = MIN_TRANSFER_STRIDE; // subdivide range
    	//创建table
        if (nextTab == null) {            // initiating
            try {
                @SuppressWarnings("unchecked")
                // 创建table容量直接*2 
                Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];
                //将创建的hashtable赋值给新table(nextTab)
                nextTab = nt;
            } catch (Throwable ex) {      // try to cope with OOME
                sizeCtl = Integer.MAX_VALUE;
                return;
            }
            nextTable = nextTab;
            transferIndex = n;
        }
        int nextn = nextTab.length;
    	//**	
        ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
        boolean advance = true;
        boolean finishing = false; // to ensure sweep before committing nextTab
       //开始节点搬迁, 以一个一个链表为单位。
        for (int i = 0, bound = 0;;) {
            Node<K,V> f; int fh;
            while (advance) {
                int nextIndex, nextBound;
                if (--i >= bound || finishing)
                    advance = false;
                else if ((nextIndex = transferIndex) <= 0) {
                    i = -1;
                    advance = false;
                }
                else if (U.compareAndSwapInt
                         (this, TRANSFERINDEX, nextIndex,
                          nextBound = (nextIndex > stride ?
                                       nextIndex - stride : 0))) {
                    bound = nextBound;
                    i = nextIndex - 1;
                    advance = false;
                }
            }
            if (i < 0 || i >= n || i + n >= nextn) {
                int sc;
                if (finishing) {
                    nextTable = null;
                    table = nextTab;
                    sizeCtl = (n << 1) - (n >>> 1);
                    return;
                }
                if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                    if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                        return;
                    finishing = advance = true;
                    i = n; // recheck before commit
                }
            }
            //f 链表头节点为null 说明该链表处理完了,将链表头设置为ForwardingNode hash为-1  ** 
            else if ((f = tabAt(tab, i)) == null)
                advance = casTabAt(tab, i, null, fwd);
            else if ((fh = f.hash) == MOVED) //已经是-1(ForwardingNode) 去处理下一个链表
                advance = true; // already processed
          //这个链表是有元素的 将头锁住,保证处理链表的安全性
            else {
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        Node<K,V> ln, hn;
                        //头节点hash>=0 普通节点
                        if (fh >= 0) {
                            int runBit = fh & n;
                            Node<K,V> lastRun = f;
                            for (Node<K,V> p = f.next; p != null; p = p.next) {
                                int b = p.hash & n;
                                if (b != runBit) {
                                    runBit = b;
                                    lastRun = p;
                                }
                            }
                            if (runBit == 0) {
                                ln = lastRun;
                                hn = null;
                            }
                            else {
                                hn = lastRun;
                                ln = null;
                            }
                            for (Node<K,V> p = f; p != lastRun; p = p.next) {
                                int ph = p.hash; K pk = p.key; V pv = p.val;
                                if ((ph & n) == 0)
                                    ln = new Node<K,V>(ph, pk, pv, ln);
                                else
                                    hn = new Node<K,V>(ph, pk, pv, hn);
                            }
                            setTabAt(nextTab, i, ln);
                            setTabAt(nextTab, i + n, hn);
                            setTabAt(tab, i, fwd);
                            advance = true;
                        }
                        //类型是不是TreeBin红黑树,是就走红黑树搬迁方法
                        else if (f instanceof TreeBin) {
                            TreeBin<K,V> t = (TreeBin<K,V>)f;
                            TreeNode<K,V> lo = null, loTail = null;
                            TreeNode<K,V> hi = null, hiTail = null;
                            int lc = 0, hc = 0;
                            for (Node<K,V> e = t.first; e != null; e = e.next) {
                                int h = e.hash;
                                TreeNode<K,V> p = new TreeNode<K,V>
                                    (h, e.key, e.val, null, null);
                                if ((h & n) == 0) {
                                    if ((p.prev = loTail) == null)
                                        lo = p;
                                    else
                                        loTail.next = p;
                                    loTail = p;
                                    ++lc;
                                }
                                else {
                                    if ((p.prev = hiTail) == null)
                                        hi = p;
                                    else
                                        hiTail.next = p;
                                    hiTail = p;
                                    ++hc;
                                }
                            }
                            ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                                (hc != 0) ? new TreeBin<K,V>(lo) : t;
                            hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                                (lc != 0) ? new TreeBin<K,V>(hi) : t;
                            setTabAt(nextTab, i, ln);
                            setTabAt(nextTab, i + n, hn);
                            setTabAt(tab, i, fwd);
                            advance = true;
                        }
                    }
                }
            }
        }
    }
```

### ConcurrentLinkedQueue

ConcurrentLinkedQueue 的设计与 LinkedBlockingQueue 非常像，也是   

+ 两把锁，同一时刻，可以允许两个线程同时（一个生产者与一个消费者）执行
+ 引入了 dummy 节点，让两把锁将来锁住的是不同对象，避免竞争
+ 只是这锁使用了 CAS 来实现 

入队 

```java
public boolean offer(E e) {
        checkNotNull(e);
        final Node<E> newNode = new Node<E>(e);
        //入队到链表尾
        for (Node<E> t = tail, p = t;;) {
            Node<E> q = p.next;
            // 如果没有next，说明到链表尾部了，就入队
            if (q == null) {
             // CAS更新p的next为新节点      
			// 如果成功了,就返回true     
			// 如果不成功就重新取next重新尝试
                if (p.casNext(null, newNode)) {
                    //有大于等于两跳就更新队尾
                    if (p != t) // hop two nodes at a time
                        casTail(t, newNode);  // Failure is OK.
                    return true;
                }
            }
         //如果p的next等于p,说明p已经被删除了(已经出队了) 重新设置p的值
         //或者队列刚初始化,返回(设置为) 头节点
            else if (p == q)
                p = (t != (t = tail)) ? t : head;
            else
          // t后面还有值，重新设置p的值
              p = (p != t && t != (t = tail)) ? t : q;
        }
    }
```

（1）定位到链表尾部，尝试把新节点放到后面；

（2）如果尾部变化了，则重新获取尾部，再重试；

**优化：tail代表尾节点，但并不是每次都更tail，当tail与真正尾节点距离大于等于1才会CAS更新 tail **

出队

```java
    public E poll() {
        restartFromHead:
        for (;;) {
            for (Node<E> h = head, p = h, q;;) {
                E item = p.item;
                if (item != null && p.casItem(item, null)) { 
                    if (p != h) // hop two nodes at a time
                        updateHead(h, ((q = p.next) != null) ? q : p);
                    return item;
                }
                else if ((q = p.next) == null) {
                    // 如果p的next为空,说明队列中没有元素了 返回null 
                    updateHead(h, p);
                    return null;
                }
                else if (p == q)
               //如果p等于p的next,说明p已经出队了,重试
                    continue restartFromHead;
                else
                    p = q;
            }
        }
    }
```

1. 定位到头节点，尝试更新其值为 null ；
2. 如果成功了，就成功出队；
3. 如果失败或者头节点变化了，就重新寻找头节点，并重试；
4. 整个出队过程没有一点阻塞相关的代码，所以出队的时候不会阻塞线程，没找到元素就返回null；

**优化：head代表头节点，但并不是每次都更head，当head与真头节点距离大于等于1才会CAS更新 head**







### CopyOnWriteArrayList

CopyOnWriteArraySet 是 CopyOnWriteArrayList 的封装，创建对象时候还是创建的CopyOnWriteArrayList 

 读不会加锁，写入也不会阻塞读取操作，只有写写需要进行同步，**修改开销大**

add方法

+ 写写互斥   JDK8 ReentrantLock        JDK 11 sychronized  
+ 适合『读多写少』的应用场景 

```java
public boolean add(E e) {
    //主要是为了写写互斥 读操作并未加锁
        synchronized (lock) {
          // 获取旧的数组
          Object[] es = getArray();
          int len = es.length;
          // 拷贝新的数组（这里是比较耗时的操作，但不影响其它读线程）
          es = Arrays.copyOf(es, len + 1);
          // 添加新元素
          es[len] = e;
          // 替换旧的数组
          setArray(es);
          return true;
     }
}
```

**弱一致性**

get方法弱一致性

+ Thread1 先get方法拿到旧数组的引用，还没有读取。Thread2 再romve(加锁了，使用了写时复制)，然后在新数组删除后 setArray(newElements)替换掉 array属。这时候 Thread1 使用原始的引用读取，读取到被删除的值 

迭代器弱一致性 

+ 迭代读取到被删除的值  



## 阻塞队列

阻塞队列 **BlockingQueue** 被广泛使用在“生产者-消费者”问题中，它提供了可阻塞的插入和移除的方法。当队列容器已满，生产者线程会被阻塞，直到队列未满；当队列容器为空时，消费者线程会被阻塞，直至队列非空时为止 

![image-20210904120519947](G:\markdown图片\image-20210904120519947.png)

put take 是可打断的

offer 成功返回true，失败返回false

poll 没有返回null  

offer (e,time,unit)   poll(time,unit) 支持超时，也一定是可中断的



### LinkedBlockingQueue 

底层是一个单向链表 ，可以当做无界队列也可以当做有界队列来使用，同样满足 FIFO 的特性，与 `ArrayBlockingQueue` 相比起来具有更高的吞吐量，为了防止 `LinkedBlockingQueue` 容量迅速增，损耗大量内存。通常在创建 `LinkedBlockingQueue` 对象时，会指定其大小，如果未指定，容量等于 `Integer.MAX_VALUE` 

单向链表的构成 head (**队头永远是dummy节点**)   +   last(tail)  

后继节点 next  的三种情况

1. 真正的后继节点
2. 指向自己，发生在出队时，help GC 
3. null，表示是没有后继节点 

核心成员变量

+ 两个锁，两个条件变量，如果没有传入队列大小，notFull 一般也用不上

**入队**

+  last = last.next = newNode   

**出队**

+ 将队头的下一个节点作为dummy节点，相当于出队了 

  具体流程

  + Node <E> h = head；  h用来help GC   Node <E> first = h.next；  h.next = h；// help GC  head = first；
    E x = first.item； first.item = null；return x；

**LinkedBlockingQueue 如何加锁**

两把锁和 dummy 节点

+ 当节点总数大于 2 时（包括 dummy 节点），putLock 保证的是 last 节点的线程安全，takeLock 保证的是
  head 节点的线程安全。两把锁保证了入队和出队没有竞争  

+ 当节点总数等于 2 时（即一个 dummy 节点，一个正常节点）这时候，仍然是两把锁锁两个对象，不会竞争  

+ 当节点总数等于 1 时（就一个 dummy 节点）这时 take 线程会被 notEmpty 条件阻塞，有竞争，会阻塞   

  

**put /take** 

take和put一样 只分析put 

```java
public void put(E e) throws InterruptedException {
        if (e == null) throw new NullPointerException();
        int c = -1;
        Node<E> node = new Node(e);
        final ReentrantLock putLock = this.putLock;
        final AtomicInteger count = this.count;
        putLock.lockInterruptibly();
        try {
            while (count.get() == capacity) {
                notFull.await(); //等待不满
            }
            enqueue(node); //last = last.next = node;
            c = count.getAndIncrement();
            //除了自己 put 以外, 队列还有空位, 由自己叫醒其他 put 线程
            //唤醒一个避免竞争,因为就算唤醒多个,也只有一个能获得锁。
            if (c + 1 < capacity)
                notFull.signal();
        } finally {
            putLock.unlock();
        }
        if (c == 0)
            //也是唤醒一个
            signalNotEmpty();
    }
```

**核心点** ： 除了自己 put 以外，队列还有空位，由自己叫醒其他 put 线程，唤醒一个避免竞争，因为就算唤醒多个，也只有一个能获得锁，take 也类似



### ArrayBlockingQueue 

基于循环数组，传入初始容量大小，后续不能改变

核心成员变量

+ 一个锁 ReentrantLock  默认是 **unfair**  
+ 两个条件变量   notEmpty 、notFull  



LinkedBlockingQueue 与 ArrayBlockingQueue 的性能比较  

+ Linked 支持有界，Array 强制有界
  Linked 实现是链表，Array 实现是数组
  Linked 是懒惰的，而 Array 需要提前初始化 Node 数组
  Linked 每次入队会生成新 Node，而 Array 的 Node 是提前创建好的 (对内存重用，减少垃圾回收)
  Linked 两把锁，Array 一把锁    (性能不如Linked)     



### PriorityBlockingQueue  

+  一个锁一个条件变量，因为他是无界队列，没有 notFull 条件
+ PriorityBlockingQueue (数组实现的二叉小根堆默认大小11)  

+ 添加元素不阻塞也不响应中断  

+  是 PriorityQueue 的线程安全版本，不可以插入 null 值，同时，插入队列的对象必须是可比较大小 (comparable) 



PriorityBlockingQueue 与 ArrayBlockingQueue 区别

+ 和ArrayBlockingQueue的实现机制相似，主要区别是用数组实现了一个二叉堆，从而实现按**优先级从小到大**出队列。另一个区别是没有 notFull 条件 (一个锁一个条件变量)，因为他是无界队列，当元素个数超出数组长度时，执行扩容操作  



PriorityBlockingQueue 扩容

+ 添加元素之前会获取锁，扩容发生在添加元素前，当队列已满，先释放锁(由于PriorityBlockingQueue 添加元素方法不阻塞)，通过 CAS 设置 allocationSpinLock，只允许一个线程扩容，扩容完毕之前，其他线程自旋，扩容完再获取锁，由获取锁的线程用新数组替换原来数组，并添加元素，添加完毕释放锁  



### DelayQueue  

DelayQueue基于优先级队列的     DelayQueue 即延迟队列，也就是一个按延迟时间从小到大出队的 PriorityQueue 

+ 核心成员 1 锁 1  条件队列    
+ 添加元素的方法不阻塞，不响应中断  



用一个Thread类型leader变量记录了等待堆顶元素的第1个线程通过 getDelay（..）可以知道堆顶元素何时到期，不必无限期等，可以使用 available.awaitNanos() ，`available是一个条件变量`，等待一个有限的时间，只有当发现还有其他线程也在等待堆顶元素 (leader！=NULL) 时才需要无限等待，available.await()



**DelayQueue 与 PriorityBlockingQueue 区别**  

+ 扩容时候相比较于 PriorityBlockingQueue 不释放锁，故扩容时候不能 take

+ PriorityBlockingQueue 增加堆数组的长度并不影响队列中元素的出队操作，所以不用加锁

+ DelayQueue要加锁，我觉得是按照时间来出队的并不能并发出队，故没释放锁  

  

### SynchronousQueue

同步的队列 

当一个线程往队列中写入一个元素时，写入操作不会立即返回，需要等待另一个线程来将这个元素拿走；同理，当一个读线程做读操作的时候，同样需要一个相匹配的写线程的写操作。这里的 Synchronous 指的就是读线程和写线程需要同步，一个读线程匹配一个写线程  

两个内部类

+ TransferQueue 单向链表    FIFO 对应着公平模式
+ TransferStack   栈               非公平模式



peek() 直接返回null  



put 与 take

+ put(E o) 和读操作 take() 都是调用 Transferer.transfer(…) 方法，区别在于take()方法调用的 transfer，第一个参数是为 null 值，put方法的transfer第一个参数是添加的元素 



TransferQueue 的 transfer 



有dummy节点  

![image-20210904165357327](G:\markdown图片\aZhoQKmHVcb53Wn.png)

```java
for (;;) {
    QNode t = tail;
    QNode h = head;
    if (t == null || h == null)         // saw uninitialized value
        continue;                       // spin

    if (h == t || t.isData == isData) { // empty or same-mode
        QNode tn = t.next;
        if (t != tail)                  // inconsistent read
            continue;
        if (tn != null) {               // lagging tail
            advanceTail(t, tn);
            continue;
        }
        if (timed && nanos <= 0)        // can't wait
            return null;
        if (s == null)
            s = new QNode(e, isData);
        if (!t.casNext(null, s))        // failed to link in
            continue;

        advanceTail(t, s);              // swing tail and wait
        Object x = awaitFulfill(s, e, timed, nanos);
        if (x == s) {         //  wait was cancelled
            clean(t, s);
            return null;
        }

        if (!s.isOffList()) {           // 从阻塞中唤醒 确认已经处在队列中的第一个元素
            advanceHead(t, s);          // unlink if head 前进头结点 
            if (x != null)              // and forget fields
                s.item = s;
            s.waiter = null;
        }
        //如果该线程是x为空 说明他被CAS交换了 put 方法返回 ; 不为空 说明该线程是take 返回他拿到的e 
        return (x != null) ? (E)x : e;
    } else {                            // complementary-mode
        QNode m = h.next;               // node to fulfill
        if (t != tail || m == null || h != head)
            continue;                   // inconsistent read
        Object x = m.item;
        if (isData == (x != null) ||    // m already fulfilled
            x == m ||                   // m cancelled
            !m.casItem(x, e)) {         // lost CAS
            advanceHead(h, m);          // dequeue and retry
            continue;
        }
        advanceHead(h, m);              // successfully fulfilled
        LockSupport.unpark(m.waiter);
        return (x != null) ? (E)x : e;
    }
}

Object awaitFulfill(QNode s, E e, boolean timed, long nanos) {
    final long deadline = timed ? System.nanoTime() + nanos : 0L;
    Thread w = Thread.currentThread();
    int spins = ((head.next == s) ?
                 (timed ? maxTimedSpins : maxUntimedSpins) : 0);
    for (;;) {
        if (w.isInterrupted())
            s.tryCancel(e);
        Object x = s.item;
        if (x != e)
            return x;
        if (timed) {
            nanos = deadline - System.nanoTime();
            if (nanos <= 0L) {
                s.tryCancel(e);
                continue;
            }
        }
        if (spins > 0)
            --spins;//自旋  
        else if (s.waiter == null)
            s.waiter = w; //自旋后还没返回则把当前节点的线程设置为当前线程 下面挂起
        else if (!timed)
            LockSupport.park(this);
        else if (nanos > spinForTimeoutThreshold) //如果设置了时间 短暂自旋后剩余时间还大于1000就挂起线程,自旋比挂起性好
            LockSupport.parkNanos(this, nanos);
    }
}
// 被取消  CAS 把节点的item 设置为this(节点类型), 从awaitFulfill方法中返回  
// 所以退出要么被匹配CAS 换成另外一个值才会退出,要么是中断取消   
if (x == s) {
clean(t, s);
}

CAS 把节点的item 设置为this(节点类型)
void tryCancel(Object cmp) {
    UNSAFE.compareAndSwapObject(this, itemOffset, cmp, this);
} 


clean(t, s);
对接点 s 进行清除, 若 s 不是链表的尾节点,则直接 CAS 进行节点的删除,若s是链表的最后一个节点, 则保存它的前驱为cleanMe,并删除先前保存的cleanMe 
    
awaitFulfill 自旋+阻塞等待,不断判断是否被匹配,匹配的话(CAS将item设置为null),退出等待 
 Object x = s.item;
     if (x != e)
   return x;
```



TransferStack 的  transfer 

![image-20210904170219929](https://i.loli.net/2021/09/04/7EMHCvdJl3e8GFV.png)

（1）如果栈中没有元素，或者栈顶元素跟将要入栈的元素模式一样，就入栈；

（2）入栈后自旋等待一会看有没有其它线程匹配到它，自旋完了还没匹配到元素就阻塞等待；

（3）阻塞等待被唤醒了说明其它线程匹配到了当前的元素，就返回匹配到的元素；

（4）如果两者模式不一样，且头节点没有在匹配中，就拿当前节点跟它匹配，匹配成功了就返回匹配到的元素；

（5）如果两者模式不一样，且头节点正在匹配中，当前线程就协助去匹配，匹配完成了再让当前节点重新入栈重新匹配

# 手撕代码

**两个线程按顺序交替输出1-100**  

**三个线程T1、T2、T3轮流打印ABC，打印n次，如ABCABCABCABC.......**

**两个线程交替打印1-100的奇偶数** 

**N个线程循环打印1-100**   

**A1B2C3D4...**

**Java生产者消费者的三种实现**   

