# Java线程的同步机制

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

同内部锁一样，显示锁同为一种互斥锁，能够保障相应的原子性、有序性、可见性。

Lock mutex = new ReentrantLock(boolean fair)

显示锁常用的接口

| 返回类型  |              方法名               |                             作用                             |
| :-------: | :-------------------------------: | :----------------------------------------------------------: |
|   void    |              lock()               |                            获取锁                            |
|   void    |        lockInterruptibly()        |                如果当前线程未被中断，则获取锁                |
| Condition |          newCondition()           |           返回绑定到此Lock实例上的新Condition实例            |
|  boolean  |             tryLock()             |      尝试获取锁，如果成功获取则返回true，否则返回false       |
|  boolean  | tryLock(long time, TimeUnit unit) | 尝试在指定时间内且当前线程没被中断的情况下获取锁，获取成功则返回true，否则返回false |
|   void    |             unlock()              |                            释放锁                            |

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



条件变量







## 内存屏障

