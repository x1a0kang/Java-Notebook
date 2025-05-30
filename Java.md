# Java

## 基础

### String累加原理

* Java的+运算符是专门为String类重载过的，使用+时实际是新建了一个StringBuilder对象，完成后再调用toString方法生成新的String对象；
* 但是在循环内累加时，创建StringBuilder是在循环内部，因此会创建多个StringBuilder，影响性能，因此应该在循环外创建StringBuilder对象使用；

### 字符串常量池

* a = "ddd"，会直接使用常量池内的对象，如果常量池中没有，则创建一个；
* a = new String("ddd")，会先new一个对象，再将这个对象指向常量池中的对象，如果常量池中没有，则创建一个；

### 反射

允许程序在运行时动态**检查、访问和操作**类、接口、字段、方法及构造函数的机制。通过反射，开发者可以在不预先知道类具体结构的情况下，动态加载类、创建对象、调用方法或访问字段。

* 优点：**动态性**，可以在运行时动态获取对象的字段，方法等；**灵活性**，突破访问限制；
* 缺点：有一定的**安全**隐患，可以获得私有属性和方法，可以无视泛型的安全检查；**性能**比直接调用慢2-100倍；**代码可读性下降**；

### 泛型

是Java的一种特性，允许类或方法在定义时使用类型参数，在使用时再指定具体类型的特性；主要用途有：

* 增加可重用性：可以定义通用逻辑，传入不同类型时都可以处理；
* 消除类型转换：指定类型后，可以省去类型转换，如List不传具体类型时，获取元素都要进行转换；
* 验证类型正确性：编译器会对泛型参数检测，如果确定了泛型的类型后，传入其他类型的对象就会报错；      
* Java会在编译期对泛型进行类型擦除，将泛型替换为具体的类型；

### I/O流：

* inputStream和outputStream是字节流，Reader/Writer是字符流；
* 字节流是基础，适合处理所有类型的二进制文件，如图像，音频，视频，或直接操作字节的场景，如网络通信，加密数据；
* 字符流是高级封装，适合处理文本文件。因为文本文件的编码格式，在使用字节流时，如果没有正确分段或解码，会造成文本乱码，字符流在读写时自动完成编码转换，可以避免乱码。
* 此外字符流相比字节流增加了缓冲区，可以减少IO的次数，提升性能；也有有缓冲区的字节流（BufferedInputStream）

### BIO/AIO/NIO：

* 首先IO的过程是把数据从内核空间拷贝到用户空间或者反过来，而内核态和用户态的转换是比较耗费时间的，所以才有几种IO模型；

* BIO：程序发起请求后，直到数据从内核空间拷贝到用户空间，都是阻塞的，性能很低；
* NIO：程序通过selector监听多个事件，直到操作系统准备好任意一个事件后通知程序准备就绪，程序再发起read请求，拷贝的过程仍然是阻塞的；**Netty**；
* AIO：程序发起请求后立即返回，数据拷贝完成后，操作系统会通知请求程序（Linux支持不完善）；
* BIO和NIO的区别是，从发起请求到准备就绪是否是阻塞的，AIO和NIO的区别在于从内核空间拷贝到用户空间是不是阻塞的
* Java中`FileInputStream`/`FileOutputStream`等流类默认BIO，`FileChannel`和`ByteBuffer`是NIO，NIO在Java中是Channel，Buffer，Selector；

### new一个object，在Linux上运行占多大内存

* 对象的内存结构分为**对象头，实例数据和对其填充**三部分。

* 对象头**Mark Word**存储哈希码、GC分代年龄、锁状态等运行时数据，8字节。**类指针**（默认开启压缩4字节）。
* **实例数据**根据内容的不同占据的空间也不同，如果是int是4字节，long是8字节，引用类型根据实际对象判断。
* **对齐填充**确保对象总大小是**8字节的整数倍**，例如，对象头+实例数据为18字节时，需填充6字节至24字节。
* 如果是空对象，8字节对象头，4字节类指针，0字节实例数据，为了满足8的整数倍，填充4字节，总共16字节。

### 单例和原型模式

* 单例模式：**全局共享的无状态对象**。适用于工具类、线程池、数据库连接池等无需维护状态且频繁访问的对象。
* 默认Bean作用域为单例，如`@Service`、`@Repository`注解标注的类。
* 原型模式：**需要隔离状态的对象**，如会话状态对象。或**频繁创建销毁的临时对象**。
* 单例对象要保证无状态，如果有可变状态，要注意多线程的安全问题；
* 原型模式如果频繁创建，要注意内存占用的问题，可以使用对象池减少开销；

### 单例模式

* **私有构造方法**：确保外部代码不能通过构造器创建类的实例。
* **私有静态实例变量**：持有类的唯一实例。
* **公有静态方法**：提供全局访问点以获取实例，如果实例不存在，则在内部创建。

饿汉式

```java
public class Singleton {
    private static final Singleton instance = new Singleton();

    private Singleton() {}

    public static Singleton getInstance() {
        return instance;
    }
}
```

懒汉式（双重检查锁）：因为 `instance = new Singleton()` 并不是一个原子操作，可能会被重排序，导致其他线程获取到未初始化完成的实例。

```java
public class Singleton {
    private static volatile Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

静态内部类：当第一次加载 Singleton 类时并不会初始化 SingletonHolder，只有在第一次调用 getInstance 方法时才会导致 SingletonHolder 被加载，从而实例化 instance。

```java
public class Singleton {
    private Singleton() {}

    private static class SingletonHolder {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return SingletonHolder.INSTANCE;
    }
}
```

### 适配器模式

* 目标接口，被适配接口，适配器三者；
* 适配器实现的方式有两种：
  * 类适配器：通过继承被适配者，实现目标接口；（需要语言支持多继承）
  * 对象适配器：通过组合被适配者的实例，实现目标接口；（更灵活，更推荐）
* 适配器模式的使用场景：
  * 继承遗留系统，旧系统接口和新需求不兼容；
  * 统一多源接口，比如统一支付宝和微信的支付接口；
  * 复用代码，对原有代码进行改造；
  * 兼容性扩展，如Java把Stream适配为Reader；

### 代理模式

**使用代理对象来代替对真实对象的访问，这样就可以在不修改原目标对象的前提下，提供额外的功能操作，扩展目标对象的功能，且屏蔽对目标对象的访问。**

* 静态代理：对目标对象的增强是手动完成的，举例就是一个对象类，需要编写一个代理类，和目标对象类实现同一个接口，代理类内注入目标对象，调用代理类的接口，代理类内部再调用目标对象类，实现屏蔽。从 JVM 层面来说， **静态代理在编译时就将接口、实现类、代理类这些都变成了一个个实际的 class 文件。**
* 动态代理：不需要针对每个目标类都单独创建一个代理类，并且也不需要我们必须实现接口，我们可以直接代理实现类。从 JVM 层面来说，**动态代理是在运行时动态生成类字节码，并加载到 JVM 中的**。

### JDK动态代理

**在 Java 动态代理机制中 `InvocationHandler` 接口和 `Proxy` 类是核心。**

使用步骤：

1. 定义一个接口及其实现类；
2. 自定义 `InvocationHandler` 并重写`invoke`方法，在 `invoke` 方法中我们会调用原生方法（被代理类的方法）并自定义一些处理逻辑；
3. 通过 `Proxy.newProxyInstance(ClassLoader loader,Class<?>[] interfaces,InvocationHandler h)` 方法创建代理对象；

**JDK 动态代理有一个最致命的问题是其只能代理实现了接口的类。**

### CGLIB动态代理

CGLIB 通过继承方式实现代理。例如 Spring 中的 AOP 模块中：如果目标对象实现了接口，则默认采用 JDK 动态代理，否则采用 CGLIB 动态代理。**在 CGLIB 动态代理机制中 `MethodInterceptor` 接口和 `Enhancer` 类是核心。**

使用步骤：

1. 定义一个类；
2. 自定义 `MethodInterceptor` 并重写 `intercept` 方法，`intercept` 用于拦截增强被代理类的方法，和 JDK 动态代理中的 `invoke` 方法类似；
3. 通过 `Enhancer` 类setClassLoader，setSuperclass，setCallback，后 `create()`创建代理类；

## 并发

### 线程的六种状态

* new：刚创建，但是没有调用start
* runnable：可以运行或者正在运行
* blocked：竞争锁失败时自动进入（被动），锁释放时被自动唤醒
* waiting：等待，主动调用了wait()，或者join()，需要调用notify唤醒
* time-waiting：有时间的等待状态
* terminated：终止

### 死锁

多个线程互相持有其他线程需要的资源，导致所有线程都互相等待无法向下执行；

* 互斥：资源不可共享
* 请求与保持：获取到的资源不会释放；
* 不可抢夺：已获取的资源不会被系统回收，只能由线程主动释放。请求某个资源失败时，释放所有已经获得的资源；
* 循环依赖：多个线程间行程循环的等待关系

* 避免死锁：
  * 一次性申请所有需要的资源，如果申请不到所有资源，就阻塞直到所有资源可用；
  * 获取资源增加超时时间，如果获取失败，释放已有资源并重试，如使用tryLock；
  * 比较重要的还是代码规范，不要加锁执行长时间的事务，尽量减少加锁的时间。不要嵌套锁，在持有一个锁时去获取另一个锁；
* 死锁检测：
  * jstack命令：输出中会有deadlock及相关信息，并导致CPU、内存等消耗过高

### JMM

Java内存模型，是进程和线程之间的内存关系，并不真实存在，与**并发编程相关**；

* 主内存：进程中的内存，储存所有实例变量，所有线程创建的变量都在主内存中；
* 本地内存：每个线程有一份私有的本地内存，保存本地变量，共享变量的副本，线程间需要通信，需要通过主内存实现；
* volatile关键字就是JMM是体现之一，因为可能共享变量被某一个线程修改了，但是另一个线程不知道，volatile关键字修饰的变量，就是要每次使用时都去主存获取，保证每次获取的都是最新的变量；

### volatile的实现原理

* 可见性：所有线程读取volatile修饰的变量时，必须从主内存中获取；修改也会刷新到主内存中；
* 防止指令重排：内存屏障storeload，loadload，storestore，loadstore，在操作volatile修饰的变量的指令前后加上内存屏障，防止指令被重排导致问题；
* 在写操作之后加上写屏障，保证写入主存，在读操作前加上读屏障，保证从主存读取

### volatile和synchronized对比

* volatile只能修饰变量，synchronized可以修饰代码或方块；
* volatile只能保证可见性，synchronized可以保证可见性和原子性；

### synchronized和ReentrantLock

* 使用方法不同，synchronized是关键字，用在方法或代码块上，ReentrantLock是类，用lock，unlock方法配置try catch使用；
* ReentrantLock可以实现公平锁，synchronized不可以；
* ReentrantLock可以在等待锁时响应中断，lockinterruptibly()
* ReentrantLock获取锁可以超时失败，失败后会返回false，线程可以执行后续逻辑，比如高并发下获取超时可以触发降级逻辑，synchronized不可以；
* ReentrantLock可以利用Condition实现多条件通知，精准控制通知，比synchronized的notify和notifyall更灵活；

### synchronized锁升级

* 只有一个线程重复访问同步代码，在对象头的Mark Word中保存线程id，直接进入；
* 有线程竞争时，升级为轻量级锁，等待的线程会自旋，CAS把Mark Word中的id改成自己的线程ID，默认自旋十次；
* 自旋获取失败或更多线程竞争（自旋线程超过CPU核数的一半）时，升级为重量级锁，线程阻塞等待锁释放；
* 升级为重量级锁后，没有降级机制；

### 非公平锁相比公平锁的优点

* 减少上下文切换成本：如果是公平锁，需要依次唤起在阻塞状态的线程，需要进行用户态到内核态的切换，这比新线程直接抢占增加了切换的成本；
* 速度快：减少了上下文切换成本，处理的速度也响应增加，尤其是在处理时间短，高并发的场景下更加明显；

### ThreadLocal

* ThreadLocal实现：线程创建一个ThreadLocal对象，就是在线程内部自己保存一个ThreadLocalMap，用来保存ThreadLocal变量，有私有的本地变量，与其他线程之间互不干扰，保证线程安全，通过 ThreadLocal 的get获取该线程的本地副本，通过 ThreadLocal 的set修改本地副本。
* ThreadLocal实现：每个线程有个ThreadLocalMap，key是ThreadLocal对象，value是对象的值；
* ThreadLocal内存泄漏：ThreadLocalMap里，key是弱引用，value是强引用；如果key不被其他强引用指向，且线程持续存在，比如线程池内的线程，GC时会把key回收，但value不会回收，导致ThreadLocalMap中的entry不被回收，造成泄漏；建议使用完ThreadLocal后，调用remove方法；

### 线程池

限制和管理资源的一种方式，提供多个线程的池子。推荐使用ThreadPoolExecutor创建，也可以用Executors创建。

spring boot项目的用**ThreadPoolTaskExecutor**，是ThreadPoolExecutor的增强，由spring管理生命周期，自动初始化和销毁，可以定义线程名，可以配置@Async注解使用

* corePoolSize：核心线程数，根据CPU密集型和IO密集型选择数量，CPU密集型可以设为核数+1，IO密集型可以设为核数*2；
* maxPoolSize：最大线程数；
* workQueue：排队队列；一般LinkedBlockingQueue或array，可以设为无界，大小根据实现需求确定；
* keepAliveTime：非核心线程空闲时间；
* threadFactory：线程工厂，自定义线程时用到；
* handler：拒绝策略，包括：拒绝并报错，丢弃，请求线程自行处理，丢弃最早未处理的任务；
* 线程回收没有区分核心还是非核心线程；

### LinkedBlockingQueue和ArrayBlockingQueue



### CompleteableFuture

实现异步任务编排，如某个任务在其他任务之后执行。方法有allOf，then等。

runAsync，supplyAsync有返回值，thenApply，thenRun，allof用于定时任务。

### AQS

抽象队列同步器，是Java并发的一个核心组件，提供了Semaphore，CountdownLatch，CyclicBarrier三种常用的组件；

* 原理是CLH锁：将多线程组织成一个队列，每个节点自旋访问前一个节点的状态，前一个节点释放后，后一个节点获取锁。AQS优化为一个先进先出的双向队列，节点不一直自旋，前一个节点释放后，唤醒下一个节点；
* 有一个state值，为0表示无锁，线程获取锁+1，释放-1，可以重入，每次重入+1，释放-1；
* Semaphore：state为N，acquire获取，state-1，release释放，state+1，state小于等于0，阻塞排队，每次修改state都是CAS操作；可以用来控制数据库连接，网络连接；
* CountdownLatch：state为N，主线程wait，每个线程执行结束后countdown，直到state为零，主线程继续执行；
* CyclicBarrier：state为N，每个线程执行await方法表示到达屏障，state-1，当state为0时，所有线程继续执行。屏障打开后自动重置，可以重复使用。
* ReentrantLock也是基于AQS原理的，没有获得锁的线程会先自旋尝试获取锁，如果多次尝试无法获取，包装一个CLH队列的node节点，加入CLH队列，会将线程挂起进入waiting状态。队列中的线程，只能按照进入队列的顺序获取锁，这是公平锁的保证，非公平锁是指新获取锁的线程，可以直接抢占锁，比队列中的线程更提前获取到锁。

## JVM

* JVM运行时区域：
  * 堆：线程共享。包含字符串常量池，主要作用就是存放对象，**几乎所有新创建的对象都在堆内分配空间（如果方法中的对象没有被外部引用或返回，可以直接在栈上分配）**；
  * 方法区：线程共享。Java8的实现是元空间，位于本地内存中，不在JVM中，包含运行时常量池；
  * 程序计数器：线程私有。记录线程执行的位置，在线程切换时帮助线程定位到正确的位置，**唯一一个不会OOM的区域**；
  * 虚拟机栈：线程私有。每个方法在执行时都是生成一个栈帧，栈帧压入虚拟机栈，执行后弹出；
  * 本地方法栈：线程私有。和虚拟机栈类似，不过针对的是本地方法，如文件操作，系统时间；
  * 直接内存：不在JVM内，用于NIO等堆外内存；

* 虚拟机栈/本地方法栈：深度超过最大深度时，抛出StackOverFlowError。申请更多空间失败时，OOM；
  * 局部变量表：储存方法中需要用到的局部变量；
  * 操作数栈：储存方法运行时产生的中间结果和临时变量；
  * 动态链接：储存方法运行时需要调用的其他方法的引用；
  * 方法出口；

* 堆：线程共享。包含字符串常量池，主要作用就是存放对象，**几乎所有新创建的对象都在堆内分配空间（如果方法中的对象没有被外部引用或返回，可以直接在栈上分配）**；
  * 新生代：Eden区，S0，S1，8:1:1。新对象会在Eden区分配，经过垃圾回收后如果仍然存在，会被移动到S区；
  * 老年代：新生代中的对象年龄超过一定阈值时（经过多少次垃圾回收），会被移动到老年代，大对象也可能直接被分配到老年代。年龄阈值有参数可以调整，且在运行过程中JVM也会动态调整；

* 方法区（元空间）：存放类结构信息，运行时常量池，静态变量，方法字节码和本地机器码，类加载器和class实例引用（用于反射）等元数据；
  * 之前的永久代在JVM内，受JVM的大小限制，元空间移动到了本地内存，大小只受机器内存限制，溢出概率大大降低；
  * 之前永久代在jvm内，其中的类元数据生命周期长，不易回收，占用内存空间，可能导致GC频繁，元空间因为在本地内存，就没有这种问题；
  * 之前的永久代无法动态调整大小，如果设置不当，可能导致内存溢出或内存不够，而元空间可以动态伸缩；
  * 运行时常量池：编译期生成的字面量如数值常量和符号引用如方法名，字段名等组成的常量池表
  
* 对象的创建过程：
  * 类加载检查：JVM检查是否类是否被加载过，如果没有，先执行类加载过程；
  * 分配内存：指针碰撞（内存规整时，用分界指针分隔已用和空闲内存）和空闲列表（可用内存的列表，选一块大小够用的内存）；
  * 初始化零值；
  * 设置对象头：类信息，年龄信息，哈希码等；
  * 执行init方法，把变量设置为预期值；

* 空间分配担保：在Minor GC前判断老年代是否有足够的空间存放存活下来的对象。老年代的连续空间大于新生代对象总大小或历次晋升的平均大小，就进行Minor GC，否则Full GC。
* 对象死亡的判断方法：
  * 引用计数法：对象加一个计数器，记录被引用的次数；无法解决循环引用，不被使用；
  * 可达性分析：从root出发，通过引用搜索，如果一个对象没到到达root的引用链，则需要被回收。作为root的对象：虚拟机栈、本地方法栈、方法区中类静态属性、方法区中常量引用的对象，被同步锁持有的对象；被判定需要被回收的对象会进行二次判定，如果还是没有和root建立关联，就会被回收；

* 垃圾收集算法：
  * 标记清除：标记出要清除的对象，然后清除。会产生大量内存碎片；
  * 复制算法：把内存分为两块区域，每次只使用其中一块，把存活的对象复制到一块区域，然后清除当前块。复制的效率不高，且每次只使用一半内存，不适合存活对象多的老年代；
  * 标记整理：标记出存活的对象，向内存的一端移动，然后清除边界外的对象；整理的效率也不高；
  * Java分代将对象的存活时间区分开，这样可以在不同的代采用合适的算法。比如年轻代对象存活数量少，适合使用复制算法，老年代存活数量多但是GC不频繁，适合使用标记整理。

* 垃圾收集器
  * Serial：单线程，在收集时要STW，新生代用标记复制算法，老年代用标记整理算法；
  * ParNew：多线程收集，收集时要STW，其他和serial一样；
  * **Parallel Scavenge**：是新生代收集器，使用复制算法。关注吞吐量，会寻找最合适的停顿时间或最大吞吐量，是JDK8的默认收集器；
  * **Parallel Old**：老年代收集器，Parallel Scavenge的老年代版本，使用标记整理算法，是JDK8的默认收集器；
  * CMS：老年代收集器，目标是最短停顿时间，使用标记清除算法，已被JDK14移除；初始标记（STW），并发标记（不STW），重新标记（STW），并发清除（**不STW**）；
  * **G1**：把内存分为多个region，新生代和老年代不需要隔离，它们都是一部分region（不一定连续）的集合。新生代使用复制算法，老年代使用标记整理算法。从JDK9起成为默认收集器；
    * 并行：初始标记（STW），并发标记（不STW）并发是指应用线程和回收线程同时进行，最终标记（STW），筛选回收（**STW**）； 
    * Minor GC：Eden区满执行，新生代Eden和S区一起标记，存活的复制到另一个S区；
    * Mixed GC：当堆内存使用率达到阈值时执行，一定会回收年轻代，也会回收部分老年代的region；
    * Full GC：当老年代满，或担保失败时触发；
    * 停顿预测：可以通过参数设置预期停顿时间，G1会根据设置的时间计算合理的回收时间，G1有一个优先列表，会根据时间从优先级最高的开始清理；
    * 回收价值定义：G1通过并发标记阶段统计每个Region中存活对象的比例，存活率越低（垃圾越多）的Region回收价值越高
    * 回收成本：结合回收所需时间（如复制存活对象耗时）和释放空间量综合评估价值
    * 回收流程：G1在后台维护一个**Region优先级列表**，根据统计的垃圾比例和回收成本动态排序，每次垃圾回收时，按用户设定的最大停顿时间选择能在此时间内回收的高价值Region组合
  * ZGC：暂停时间很短，染色指针，读屏障等技术；
  
* minor GC，major GC和full GC：
  * minor GC是年轻代的GC，当Eden区满时触发，serial，parnew，parallel，G1都有；

  * major GC是老年代的GC，当老年代满时触发，CMS独有的GC，其他收集器的老年代回收属于full GC；

  * full GC是全局的GC，老年代空间不足、显式调用`System.gc()`、空间分配担保失败时触发，各种old收集器都有；

* ==类加载过程==：加载（双亲委派），连接（验证、准备、解析），初始化

* ==类的初始化过程==

* 类加载器：**`BootstrapClassLoader`**，**`ExtensionClassLoader`**，**`AppClassLoader`**，可以自定义类加载器；

* 双亲委派模型：
  * 每个类加载器都会委派给父类加载，如果不行，再自己加载。
  * 避免重复加载。父加载器加载了，子加载器就不能加载。
  * 避免核心类被篡改。如果定义了一个新类，也叫String，加载时没有双亲委派模型，那就会和Java原生的String冲突，导致核心类被篡改。
* 打破双亲委派模型
  * 自定义类加载器，重写loadClass方法，如果自定义类加载器，但不想打破双亲委派模型，就重写findClass方法即可；
  * 例子**Tomcat**的类加载机制，Tomcat有自定义的webappClassLoader，会先尝试从特定的目录加载，加载不到再委派给父加载器，但是java的核心类还是通过双亲委派机制加载；

* 注解的底层原理：反射

## 集合

* arraylist扩容，是扩大为现在的1.5倍，再把原数组的数值拷贝到新数组中；
* fail-fast，当一个线程在遍历时，另一个线程修改了集合对象的内容，就会抛出异常；
* fail-safe，当一个线程遍历集合时，是对原集合的拷贝进行遍历，所以另一个线程的修改并不会影响；
* hashmap：
  * 链表长度大于8时（数组长度大于64），改为红黑树，红黑树的查询效率是logn，插入效率是logn
  * 初始容量是16，扩容每次会变成原来的两倍；
  * 为什么hashmap的容量都是二的N次幂，因为为了高效计算元素在hashmap中的位置，计算方法用了位运算，二的N次幂让计算更高效；
  * 扩容发生在实际大小大于容量*负载因子时；
  * 1.8之前是头插法，之后是尾插法，头插法可能在扩容时，导致链表上的元素前后位置不一致，在多线程时可能导致死循环；
  * 可能死锁？

* ConcurrentHashMap：
  * 对于读，加了volatile关键字保证可见性，node节点中的value和next加了volatile关键字；
  * 对于写，先尝试CAS写入，如果不成功，就使用synchronized代码块处理；

* copyonwrite数组：用于读多写少的场景
  * 用volatile修饰数组array，保证可见性
  * 读时不加锁，写时拷贝一个新数组，用reentranceLock保证线程安全；
  * 读到的可能是老数据，但是保证了读的性能
  * 数组比较大时，写会消耗性能；


## Spring

* spring：核心是控制反转（IOC）和面向切面编程（AOP）

### IOC控制反转

* 是指把类的控制权由用户（程序员）的手中交给spring，由spring来控制类的创建，注入，回收等，由spring来控制类的生命周期；

* IOC容器就是控制反转的载体，实际上是一个ConcurrentHashMap，存放着各种对象；通俗解释就是没有控制反转的情况下，要使用一个类，需要程序员在代码中显式地new一个对象，有了IOC容器后，改为从IOC容器中取即可；

* 依赖注入DI：是IOC的实现方式，类似方法区和元空间，利用注入把目标对象传递给需要它的类。有构造函数注入，setter注入，字段注入（resource或autowired注解）。

* bean：指被容器管理的对象，哪些类要交给spring管理，需要通过配置或注解指明。如@Bean，@component，@service，@controller等；

### AOP面向切面编程

* 面向多个类或对象执行过程中的公共行为，提取抽象成一个切面Aspect类，类似volatile关键字防止指令重排，系统会在前后加入内存屏障，切面是在多个类的某个流程中切入；

* Advice是要执行的操作。有五种类型：before，after，after returning，after throwing，around，around可以控制目标方法的执行；

* pointcut切点，控制哪些类要被切面增强，可以是表达式，可以是注解；

* AOP可以将分散在多个类或对象中的公共行为抽离出来，减少重复代码；

* 应用场景：日志记录，限流，权限控制，事务控制（transaction注解就是通过AOP实现的），性能统计（调用的次数或时间）

* 实现方式：动态代理。AspectJ和spring AOP的区别，AspectJ是在编译时增强，不需要通过代理，因此性能更高，功能强大但更为复杂，除方法外，还可以切入字段，构造器，静态方法等。

### resource和autowired

* resource是by name，autowired是by type；
* resource是JDK提供的注解，autowired是spring提供的注解；
* 当一个接口有多个实现类时，resource搭配name属性使用，autowired搭配qualify注解使用
* idea对autowired有下划线提示：因为idea不推荐字段注入，resource不提示是因为它是Java提供的注解，使用spring以外的IOC时同样可以生效，而autowired不行。

### bean的生命周期

* 实例化：创建bean的实例，通过反射；

* 属性赋值：设置bean的属性，如依赖注入的对象，resource，value注入的值等

* 各种aware接口，如果实现了这些接口，会调用对应的方法

* 如果实现了beanpostprocessor接口，先调用postProcessBeforeInitialization()方法（遍历所有切面，为@async，@transaction等创建动态代理）

* 如果实现了InitializingBeanInitializingBean接口，执行afterPropertiesSet方法

* 如果实现了beanpostprocessor接口，再调用postProcessAfterInitialization()方法

* 此时bean已经可用，会保存在上下文中，直到上下文被销毁时销毁

* 销毁：如果bean实现了disposableBean接口，执行destroy方法，或者在配置文件中定义了销毁方法的话，执行该方法

### spring MVC

现在前后端分离的情况下，只要返回JSON数据即可，view由前端实现

* dispatch servlet拦截客户端请求

* dispatch servlet调用handlerMapping找到对应的handler（controller）

* dispatch servlet调用handlerAdapter适配器执行handler（适配器模式）

* handler处理完后返回一个JSON数据

* 适配器解决多种handler（controller接口，requestmapping注解）的兼容问题，同时处理参数和返回值。

### spring循环依赖

* 两个或多个不同的类互相持有对方的引用，形成环型。通过三级缓存来解决：

* 一级缓存：保存生成完成的单例bean；

* 二级缓存：保存原始bean，属性尚未填充，但已有堆内的位置，即三级缓存中objectFactory生成的对象；

* 三级缓存：保存objectFactory，可以通过getObject方法产生对象，原始bean或AOP的代理对象；

* 当发生循环依赖时，创建A时发现依赖B，去创建B，创建B时又发现依赖A，这时一二级缓存中还没有A，于是B去三级缓存获取A的objectFactory，通过getObject方法产生对象，并加入二级缓存，这时B就创建完成了，A就可以从一级缓存中获取到B；

* 为什么要三级缓存：
  * 代理对象：objectFactory可以判断是否经过AOP增强，如果有就创建代理，没有就创建bean，并放入二级缓存，后面如果还要创建代理，就可以从二级缓存中取，保证多次代理是同一个对象。
  * 如果缺少二级缓存，就不能暴露未完全初始化的bean，而某些代理逻辑不能在三级缓存中暴露，可能导致报错；
* 新版本变化：在2.6版本以后，spring默认禁止循环依赖，新增配置项，是否禁止循环依赖，如果没有打开，程序无法启动；

### spring事务

* 分为两种：编程式事务TransactionTemplate和声明式事务（@Transaction），必须是public方法；
* spring事务是依赖数据库事务的，如果spring事务中不涉及数据库操作，或者数据库不支持事务，spring中的事务是不生效的；
* spring事务遵循acid，隔离等级也是四个：读未提交，读已提交，可重复读，串行化。默认是遵循数据库的默认级别，如MySQL是可重复读；
* 当一个事务方法调用另一个事务方法时，需要传播机制：required：没有事务，创建事务，有事务，加入事务；new：无论有没有事务，创建新事务；nested：有事务，创建新事务作为当前事务的子事务，没有等价于required；mandatory：有事务就加入，没有就报错；
* 事务失效的情况：
  * 未捕获异常：事务中发生了未捕获的异常，未被处理或者传播到事务边界之外，事务失效，所有数据库操作回滚；
  * 非受检异常：默认回滚策略是只有在遇到`RuntimeException`(运行时异常) 或者 `Error` 时才会回滚事务，可以通过 `rollbackFor` 和 `noRollbackFor` 属性来指定哪些异常需要回滚，哪些异常不需要回滚；
  * 传播属性设置不当：如果多个事务间调用，传播属性设置不正确可能导致事务失效；
  * 在非公开方法：注解使用到private，或非public方法上，事务会失效；
  * 跨方法调用事务：一个事务内部调用另一个方法，但这个方法没有开启事务，可能导致外部事务失效；
  * **spring事务使用this是不能生效的**，因为事务是通过代理来实现的，只有通过代理对象调用才有事务的规则。


### spring boot优点

* 开箱即用：starter提供了很多功能，减少了很多工作量
* 自动配置：可以根据依赖提供大量自动配置常见组件，提供合适的默认配置，也可以配置覆盖，省去了很多配置工作；
* 内嵌web服务器Tomcat，大大简化了运行和部署；
* 对构建工具支持完善，如对maven和gradle提供了专门的插件，使打包运行等更方便
* 监控方便，actuator模块对监控提供了很大的便利；

### spring boot配置文件优先级

* jar包外config目录
* jar包外根目录，jar包同级目录
* jar包内config目录，resources/config
* jar包内根目录，resources

### spring boot全局异常处理

* @ControllerAdvice+@ResponseBody或@RestControllerAdvice
* @ExceptionHandler

### 注解的原理

* 运行时基于反射机制获取注解，
* 用@interface关键字定义注解
* 元注解有：@Retention，定义注解的生命周期；@Target：定义注解的作用目标；@Document：是否包含在javadoc中；@Inherited：允许子类继承父类注解；
* 可以配合aop使用，如限流，分布式锁；

### spring boot启动过程

* `@SpringBootConfiguration`（标识主配置类）、`@ComponentScan`（包扫描）和 **`@EnableAutoConfiguration`**（触发自动装配的核心注解）
* 入口@SpringBootApplication，在main方法上，main方法内调用SpringApplication.run(Application.class, args)；
* run方法内，新建一个SpringApplication对象，初始化，创建一个应用监听器；
* 然后加载spring boot配置环境，包括命令行参数，环境变量，配置文件，加载完成后发布事件，通知监听器；
* 再然后加载应用上下文，ApplicationContext，负责加载bean，管理bean的生命周期的重要类，作为run方法的返回对象；
* 自动配置机制，通过@EnableAutoConfiguration加载自动配置类，spring boot的一些starter。
* 上下文刷新，bean实例化和依赖注入，执行@PostConstructor；
* 实现commandLineRunner/applicationRunner接口的run方法，执行启动后任务，比如数据初始化，发布完成事件；

### spring boot自动装配

* 核心是注解@SpringBootApplication，包含了**`@EnableAutoConfiguration`**（触发自动装配的核心注解）
* **`@EnableAutoConfiguration`**通过@Import注解加载配置类，启动自动装配流程
* 获取需要自动装配的配置类，读取META-INF/spring.factories，所有starter内部都有这个spring.factories，spring boot会读取所有starter的spring.factories进行加载
* 加载spring.factories中符合条件的配置类，有各种条件注解，比如在web项目中哪些不用加载；

### spring启动加载数据的方法

* @PostConstructor
* 实现InitializingBean，重写afterPropertiesSet方法，在bean属性注入后执行，早于@PostConstructor，这两个是bean加载后执行；
* 实现CommandLineRunner/ApplicationRunner接口，重写run方法，这两个是应用启动后执行，可以接收命令号的参数
* @EventListener监听spring事件，ApplicationReadyEvent或者ContextRefreshEvent

### @Asyn注解的实现

* **动态代理**，首先要在启动类上@EnableAsync
* 如果一个类或者方法使用了@Async，AsyncBeanPostProcess类就会生成这个类的动态代理，该类的方法在执行时，都会被代理对象拦截，其中使用了@Async的方法会异步执行
* @Async注解使用线程池的线程进行异步处理，可以在@Async注解上指定线程池，如果没有指定，默认使用名为taskExecutor的线程池，如果程序中也没有这个名称的线程池，就会创建一个SimpleAsync线程池，但这个线程池不会复用线程，每有一个任务就会创建一个线程，可能会导致线程过多，内存溢出。

* 定时任务调度：scheduled，cron。分布式调度：quartz（不好用，维护数据库，没有可视化），xxl-job，powerjob；

### spring中的设计模式

1. 工厂模式：Bean的创建与管理，通过`BeanFactory`和`ApplicationContext`接口集中管理Bean的生命周期，隐藏对象实例化细节。
2. 单例模式：Bean的默认作用域，Spring容器通过`ConcurrentHashMap`缓存单例Bean实例，确保全局唯一性。
3. 代理模式：AOP（面向切面编程），通过**JDK动态代理**（基于接口）和**CGLIB代理**（基于子类）为对象添加横切逻辑（如事务、日志）。
4. 模板方法模式：各种template，jdbc，mongo，Redis，kafka等等，标准化流程，封装通用操作。
5. 适配器模式：`HandlerAdapter`将不同类型的控制器（如`@Controller`、`HttpRequestHandler`）适配为统一接口，Reader字符流对字节流Stream类的适配。
6. 策略模式：动态选择算法实现。`CglibAopProxy`和`JdkDynamicAopProxy`分别作为AOP代理策略。
7. 观察者模式：spring是事件驱动模型，EventListener；

