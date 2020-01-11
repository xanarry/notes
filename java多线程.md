# Java线程的同步机制

## 可见性

在多线程环境下，一个线程对共享变量进行更新之后，后续访问该变量的线程可能无法立即读取到这个更新之后的结果，甚至永远得不到这个更新之后的结果。可见性就是一个线程对共享变量的更新结果对于其他访问的线程立即可见。

产生可见性问题的原因在于：计算机的缓存结构（寄存器，高速缓存，内存）之间，为了提高访问速度，存在相同的数据副本，一旦数据更新，在没有触发某些事件的情况下，数据没有做全局更新（缓存同步），所以导致后续的访问可能仍然是旧值。



## 有序性

有序性：即程序执行的顺序按照代码的先后顺序执行。

Java内存模型中的程序天然有序性可以总结为一句话：如果在本线程内观察，所有操作都是有序的；如果在一个线程中观察另一个线程，所有操作都是无序的。前半句是指“线程内表现为串行语义”，后半句是指“指令重排序”现象和“工作内存主主内存同步延迟”现象。

有序性的语意有几层，
1. 最常见的就是保证多线程运行的串行顺序
2. 防止重排序引起的问题
3. 程序运行的先后顺序。比方JVM定义的一些Happens-before规则



## 原子性

原子性是指针对共享变量的操作，该操作对于线程来讲，要么是一次性做完，要么就像是什么都没有发生一样。



## 重排序

重排序是指编译器可能改变两个操作的先后顺序；处理器可能不是完全依照程序的目标代码所指定的顺序执行执行。另外，一个处理器上执行的多个操作，从其他处理器的角度来看，其顺序可能与目标代码所指定的顺序不一致。

**重排序是对内存访问相关操作所做的一种优化，可以在不影响单线程程序正确性的情况下提升程序的性能**







## 内部锁：synchronize关键词

Java平台中的任何一个对象都有唯一一个与之关联的锁，这种锁被称为监视器(Monitor)或者内部锁(Intrinsic Lock)。内部锁是一种互斥锁，能够保障原子性、有序性、可见性。

**内部锁的使用不会导致锁的泄露**(申请了锁L却没有释放锁L)，因为Java编译器对内部锁编译为字节码的时候，对临界区可以抛出而代码有没有处理的异常进行了特殊处理，使得该临界区中的代码在任何情况下都能保证锁的释放。

### 同步方法

同步方法指用synchronize关键词修饰的方法，这样的方法能够保证任意时刻，都只有唯一一个进程进入该方法体。

```java
class Klass {
    private int cnt;
    public synchronized void inc() { //整个函数都为临界区
        cnt++;
    }
}
```



###　同步代码块

当某个代码块被synchronize修饰时，则称其为同步代码块，在修饰代码块时，必须为synchronize关键词提供一个对象用户获取锁与释放锁。

当同步代码块在实例方法中时，通常给synchronize传入this即可，让其使用当前对象上的锁。

```java
class Klass {
    private int cnt;
    public void inc() {
        //do something
        synchronized(this) {
            //临界区
            cnt++;
        }
        //do something
    }
}
```

当同步代码块在静态方法中时(没有当前对象)，我们可以给synchronize传入当前类的class，因为class本身也是类。

```java
class Klass {
    private static int cnt;
    public static void inc() {
        //do something
        synchronized(Klass.class) {
            //临界区
            cnt++;
        }
        //do something
    }
}
```



除了为synchronize传入当前类的class对象或者其当前实例，synchronize也可以被传入任何实例对象。但通常要求传递给synchronize关键词的对象要用final修饰。形如：`private final Object lock = new Object()`，因为在代码的执行过程当中，一旦锁的对象被修改，那么该代码块也就同步在了不同的锁上，从而没法保障原子性、有序性、可见性。最终使得数据在多个线程方法的情况下无法保证一致性和正确性。

使用任意对象锁对象：

```java
class Klass {
    private static int cnt;
    private final String lock = "a lock object"
    public static void inc() {
        //do something
        synchronized(lock) {
            //临界区
            cnt++;
        }
        //do something
    }
}
```



### 内部锁的调度

内部锁仅支持非公平调度。虚拟机为每个内部锁分配一个入口集合(Entry Set)，记录正在等待该锁的线程，该集合中线程的状态为BLOCKED，当锁被释放时，虚拟机从集合中随机选取一个线程调度。











## 显示锁：Lock接口

同内部锁一样，显示锁同为一种互斥锁，能够保障相应的原子性、有序性、可见性。不同的是，显示锁提供了更丰富的接口，支持更灵活的操作和对锁的相关信息进行监控。

Lock mutex = new ReentrantLock(boolean fair)



### 显示锁常用的接口

| 返回类型  |              方法名               |                             作用                             |
| :-------: | :-------------------------------: | :----------------------------------------------------------: |
|   void    |              lock()               |                            获取锁                            |
|   void    |        lockInterruptibly()        |                如果当前线程未被中断，则获取锁                |
| Condition |          newCondition()           |           返回绑定到此Lock实例上的新Condition实例            |
|  boolean  |             tryLock()             |      尝试获取锁，如果成功获取则返回true，否则返回false       |
|  boolean  | tryLock(long time, TimeUnit unit) | 尝试在指定时间内且当前线程没被中断的情况下获取锁，获取成功则返回true，否则返回false |
|   void    |             unlock()              |                            释放锁                            |



### 模板代码

显示锁的使用模板：

```java
private final Lock mutex = new ReentrantLock();
mutex.lock();
try {
    //临界区代码块
} finally {
    mutex.unlock(); //unlock总是放在finally块中，避免因为临界区中代码发生异常而导致锁泄露
}
```



使用tryLock()的模板：

```java
private final Lock mutex = new ReentrantLock();
if (mutex.tryLock()) {
    //成功获取锁
    try {
        //临界区代码块
    } finally {
        mutex.unlock(); //unlock总是放在finally块中，避免因为临界区中代码发生异常而导致锁泄露
    }
} else {
    //获取锁失败，执行其他操作
}


```



### 显示锁的调度

ReentranLock既支持非公平锁也支持公平锁调度，默认为非公平锁，但可以通过构造函数传入参数设置。公平锁的调度的公平性往往增加了线程暂停和唤醒的可能性，既上下文切换代价。

**公平锁的适用场景：**适合于锁被持有时间相对长或者线程申请锁的平均时间间隔较长的情况。避免出现饥饿现象。



## 读写锁

读写锁是一种改进型排他锁，允许多个线程同时进行读操作，但仅允许一个线程进行写操作，存在读操作时不可写，反之亦然。

### 读写锁的使用方法

```java
private final ReadWriteLock readWriteLock = new ReentrantReadWriteLock();
private final Lock rlock = readWriteLock.readLock();
private final Lock wlock = readWriteLock.writeLock();

public void reader() {
    rlock.lock();
    try {
        //read
    } finally {
        rlock.unlock();
    }
}

public void writer() {
    wlock.lock();
    try {
        //write
    } finally {
        wlock.unlock();
    }
}
```

### 读写锁使用场景

- 只读操作比写操作要频繁
- 读线程持有锁的时间比较长



## 内部锁与显示锁的比较

1. **调度公平性**：显示锁默认为非公平锁，但可以通过传入参数设置为公平锁。

2. **锁的性能**：设置为公平调度的显示锁，其背后存在更多的线程上下文切换开销，性能不如非公平锁、内部锁。

3. **使用便捷性**：内部锁的申请与释放只能在一个方法上或者方法内部进行，简单易用，不存在锁泄露的问题；显   示锁使用更加灵活，可以跨方法申请与释放锁，但过程需要程序员自己把控，使用不好存在锁泄露的可能性。





## 内存屏障

内存屏障(Memery Barrier/Fence)是被插入到两个指令序列之间使用的，其作用是进制编译器、处理器对指令重排序，从而保证有序性。内存屏障在两个指令之间就如一个屏障一样隔绝两条指令。



### 内存屏障的分类

#### 按可见性保障划分

- 加载屏障(Load Barrier)：作用为从内存中加载数据刷新处理器缓存

- 存储屏障(Story Barrier)：作用为将处理器缓存中的数据冲刷到内存。

#### 按有序性保障划分

- 获取屏障(Aquire Barrier)

  获取屏障的使用方式是在一个**读操作之后**插入内存屏障，其作用是禁止该读操作与**其后面的任何读写**操作进行重排序。**保证操作所访问的数据是最新的**

  ![](./assets/aquire-barrier.png)

- 释放屏障(Release Barrier)

  释放屏障的使用方式是在一个**写操作之前**插入内存屏障，其作用是禁止该读操作与**其前面的任何读写**操作进行重排序。**保证对数据的操作都结束之后，才会进行写入操作**

  ![](./assets/release-barrier.png)

### 内存屏障对synchronize的保障

从下图可以看出：

- 在进入synchronize代码块时，系统为了保证临界区访问到的是相对较新的数据，使用了**加载屏障**来更新数据，并使用**获取屏障**来防止代码重拍下保证只有在数据加载完毕之后才能进行相关的数据操纵。
- 退出临界区时，内存屏障做了进入临界区的相反操纵，首先用**释放屏障**隔绝前面访问数据的代码与后面写入数据到内存的代码（**存储屏障**），保证退出临界区后，其他进程能从内存中得到修改后的数据。



![](assets/synchronize-mem-barrier.png)



### 内存屏障对volatile的保障

volatile修饰的变量一般作为通知信息的变量，在多线程环境下，如何保证通知变量在逻辑上的正确性（防止被重排序）以及变量的值一定是最新的值（可见性），通过内存屏障的配合使用，能达成相应目标。



内存屏障对**写volatile变量**的保护：

**写volatile变量操作与该操作之前的任何读、写操作不会被重排序。**

![](assets/set-volatile.png)



内存屏障对**读volatile变量**的保护：

**读volatile变量操作与该操作之后的任何读、写操作不会被重排序。**



![](assets/get-volatile.png)



## volatile关键词

volatile字面意思即为**易变的**，在多线程环境下，虚拟机可以保障使用volatile修饰的变量可见性，顺序性和long/double读写操作的原子性（64为虚拟机下，double/long不用volatile已经可以得到原子性保证）。

**注意**：volatile仅仅保障对其修饰变量的读写操作本身的原子性，并不表示对volatile变量的整个复制操作一定具有原子性。如果是对于引用类型变量的修饰，volatile关键词只是保证线程能够读取到一个指向对象的相对新的内存地址（引用），但对这个地址指向对象实例的值本身是否较新并没有保证。

例如：表达式`volatileVar = sharedVar+1`，sharedVar为共享变量，在表达式赋值的同时，可能有其他线程修改了该变量的子，也就失去了原子性。



### volatile的开销

volatile修饰的变量的开销介于普通变量与加锁变量之间。因为volatile变量没有锁，不存在上下文切换过程，所以比加锁的操纵更快；但因为volatile需要通过内存屏障保证变量的可见性与有序性，导致编译器对指令的优化不及普通变量，所以其性能低于普通变量。



### volatile的适用场景

- 使用volatile修饰的变量作状态标志
- 使用volatile变量替换锁：利用volatile变量的读写操纵具有原子性，可以把一组volatile变量封装成一个对象，那么对这些变量的更新操作就能通过新建一个对象，并替换掉就对象的引用来实现。这个过程中保证了原子性和可见性，从而避免了锁的使用。





## 单例模式

单例模式保证在同一个运行时环境下，一个单例类只存在唯一对象实例。



下面看看单例模式常见的实现方式：

### 加锁方式

```java
class Singleton {
    private static Singleton instance = null;
    private Singleton() {
        //定义私有构造函数禁止外部调用产生新实例
    }

    /**
     * 定义synchronized修饰的公开方法，保证在多线程环境下，
     * 所有需要该类对象的代码通过调用这个函数取得正确的对象实例
     */
    public synchronized Singleton getInstance() {
        if (instance == null) { //如果首次被调用，创建对象
            instance = new Singleton();
        }
        //返回实例
        return instance;
    }
}
```

这个实现可以保证线程安全，但有一个缺点在于，每次拿取实例都需要申请和释放锁，需要较大开销。为了避免这种开销，人们提出了一种新的方法（双重检查锁定），在要进入临界区前先判断实例是不是为null，为null才需要进入临界区生成对象，否则直接返回实例对象。

### 双重检查锁定

```java
class Singleton {
    private static Singleton instance = null;
    private Singleton() {
        //定义私有构造函数禁止外部调用产生新实例
    }

    /**
     * 定义synchronized修饰的公开方法，保证在多线程环境下，
     * 所有需要该类对象的代码通过调用这个函数取得正确的对象实例
     */
    public Singleton getInstance() {
        //如果为null，进入临界区生成对象
        if (instance == null) {         //#1
            synchronized(Singleton.class) {
                //进入临界区之后再次做判读，因为存在情况：
                //线程A，B都进入#1的if，此时，线程A进入临界区创建实例并返回，
                //线程B进入临界区不做判断的话，直接执行new，会再次产生一个对象，
                //破坏了模式
                if (instance == null) { //#2
                    instance = new Singleton();
                }
            }
        }
        //返回实例
        return instance;
    }
}
```



双重检查锁定方法避免了频繁的请求锁、释放锁操作。但是因为`#1`判断语句，**可能导致用户拿到的是一个为初始化的对象**。

我们看看一个new一个对象的过程，当执行`ObjectA  objA = new ObjectA()`时，实际上分为一下三个子步骤：

- 步骤一：`objRef = allocate(ObjectA.class);` 分配对象所需空间
- 步骤二：`invokeConstructor(objRef);` 调用构造函数初始化对象
- 步骤三：`objA = objRef;` 将对象引用写入变量objA

由于重排序的原因，步骤二和步骤三的顺序可能被交换，导致用户虽然拿到了一个为对象，但对象还没来得及初始化。该现象发生的时间点在：线程A进入临界区执行`instance = new Singleton();`，指令被重排序先执行了步骤三，还没来得及执行步骤二；这时线程B进入函数执行第一个判断语句`#1`，instance并不为空，直接返回对象，是未初始化的对象。

解决方法是对instance变量用volatile修饰，下面是正确的双重检查锁定单例模式：

### 正确的双重检查锁定

```java
class Singleton {
    //防止重排序
    private static volatile Singleton instance = null;
    private Singleton() {
        //定义私有构造函数禁止外部调用产生新实例
    }

    /**
     * 定义synchronized修饰的公开方法，保证在多线程环境下，
     * 所有需要该类对象的代码通过调用这个函数取得正确的对象实例
     */
    public Singleton getInstance() {
        //如果为null，进入临界区生成对象
        if (instance == null) {         //#1
            synchronized(Singleton.class) {
                //进入临界区之后再次做判读，因为存在情况：
                //线程A，B都进入#1的if，此时，线程A进入临界区创建实例并返回，
                //线程B进入临界区不做判断的话，直接执行new，会再次产生一个对象，
                //破坏了模式
                if (instance == null) { //#2
                    instance = new Singleton();
                }
            }
        }
        //返回实例
        return instance;
    }
}
```



### 使用静态变量实现单例模式

```java
class Singleton {
    private static Singleton instance = null;
    private Singleton() {}
    public static Singleton getInstance() {
        return instance;
    }
}
```



### 使用静态内部类实现的单例模式

```java
class Singleton {
    private Singleton() {}
    static class InstanceHolder {
        final static Singleton instance = new Singleton();
    }
    public static Singleton getInstance() {
        return InstanceHolder.instance;
    }
}
```





## CAS与原子变量

CAS(Compare And Swap)是对一种处理器指令的称呼，对于一些比较简单的原子操作，选择CAS来处理具有比加锁更好的性能。CAS能够将read-modify-write和check-and-act之类的操作转换为原子操作。



### CAS背后的逻辑

```java
bool CAS(variable var, Object oldVal, Object newVal) {
	if (var == oldVal) { //check 检查变量值是否被其他线程修改过
        var.set(newVal); //act   更新变量值
        return true;     //更新成功
    }
    return false;        //更新失败
}
```



### 使用CAS实现一个原子自增操作

在使用CAS时，需要注意两点：

- CAS仅仅保障共享变量更新操作的原子性，它并不保障可见性，所以使用CAS的变量需要使用volatile修饰。
- 对变量的更新操作必须放在一个循环中。因为在一次值更新过程中，发现原有的值已经被其他线程修改过，导致这次更新失败，必须下次再来，直到更新成功。

```java
class CASBasedCounter {
    private volatile long count; //一定要使用volatile修饰
    private final AtomicLongFieldUpdater<CASBasedCounter> fieldUpdater;

    public CASBasedCounter() throws SecurityException, NoSuchFieldException {
        //告诉需要更新的变量名字, 底层代码实现通过名字找到变量所在的内存偏移
        fieldUpdater = AtomicLongFieldUpdater.newUpdater(CASBasedCounter.class, "count");
    }

    public long vaule() {
        return count;
    }

    public void increment() {
        long oldValue, newValue;
        //更新值需要放到循环中处理
        do {
            oldValue = count;       // 读取共享变量当前值
            newValue = oldValue + 1;// 计算共享变量的新值
        } while (/* 调用CAS来更新共享变量的值 */!compareAndSwap(oldValue, newValue));
    }

    //该方法是个实例方法，且共享变量count是当前类的实例变量，因此这里我们没有必要在方法参数中声明一个表示共享变量的参数
    private boolean compareAndSwap(long oldValue, long newValue) {
        //调用系统实现的CAS, CAS最底层的代码为本地方法实现
        boolean isOK = fieldUpdater.compareAndSet(this, oldValue, newValue);
        return isOK;
    }
}
```



### CAS中的ABA问题

在CAS对变量进行更新的时候可能出现这样一种问题：假如有三个线程TA,TB,TC，变量的初始值为A，线程TA要将Ａ值修改B值，在TA读了A值还没来得及修改成Ｂ值时，线程TB抢先把Ａ值修改为其他值，然后TC又把其他值修改回Ａ值。在这个场景下，TA误认为A值还是原理的A值，可以进行操作，但实际上A是经过修改过后的值。

针对这种问题，设计者为变量引入了一个修订号（也称时间戳），变量每被修改一次，修订号就增加1，这样就能清楚的判定变量是否被修改过。AtomicStampedReference类就是基于这种思想的一个实现类。



### 原子变量类

原子变量类是基于CAS实现的能够保证对共享变量进行read-modify-write更新操作原子性和可见性的一组工具类。原子变量一共有12个可分为四组，具体如下：

|     分组     |                              类                              |
| :----------: | :----------------------------------------------------------: |
| 基础数据类型 |            AtomicInteger/AtomicLong/AtomicBoolean            |
|    数组型    |   AtomicIntegerArray/AtomicLongArray/AtomicReferenceArray    |
|  字段更新器  | AtomicIntegerFeildUpdater/AtomicLongFeildUpdater/AtomicReferenceFeildUpdater |
|    引用型    | AtomicReference/AtomicStampedReference/AtomicMarkableReference |



## 对象的初始化安全：static与final

一个类被Java虚拟机加载之后，该类的所有静态变量值仍然都是其默认值（数值类全为0，引用类全为null，布尔类全为false），直到有个线程初次访问了该类的任意一个静态变量才使得这个类被初始化-类的静态代码块`static {}`被执行，类的所有静态变量被赋予初始值。



**static**关键词能够保证一个线程即使在未使用同步机制的情况下也总是可以读到一个类的静态变量初始化之后的值，而非默认值（包括引用类型变量，保证所引用的对象已经初始化完毕）。但这种可见性的保障仅限于线程初始读取该变量的值。后续如果有对变量进行更改，仍然需要同步机制。



**final**修饰的变量保证在其他线程中访问这些变量时，无论是简单变量还是引用变量，都是初始化之后的值（一种有序性保障）。

























# 线程间的协作

