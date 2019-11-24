# juc-aotmic
##说明：
 Java从JDK5之后,可以使用java.util.concurrent.atomic包在无锁的情况下进行原子操作，通过Java的Unsafe工具类型调用CPU的原子指令完成，因CPU架构和指令的不同，也有可能出现堵塞的问题，通過自旋 + CAS（乐观锁）保证原子性。
 
 其原理是使用CAS（Compare and Swap）算法，即比较再交换，在这个过程中，通过compareAndSwapInt比较更新value值，如果更新失败，重新获取旧值，然后更新。是一种无锁的非阻塞算法的实现，是硬件对于并发的支持,针对多处理器操作而设计的处理器中的一种特殊指令,用于管理对共享数据的并发访问。JUC包中的类使用CAS算法实现了一种乐观锁,JDK5之前是使用synchronized关键字保证同步，这是一种独占锁，也是是悲观锁。（可参考：悲观锁和乐观锁）

 原子操作只能使用在单线程中，在多线程中可能出现“ABA问题”，可以使用标记原子引用类型AtomicStampedReference或AtomicMarkableReference处理解决。
##18个操作类
###*基本类型
1. AotmicBoolean 原子更新布尔类型
2. AtomicInteger 原子更新整型
3. AtomincLong 原子更新长整型
####*原子整形的常用方法
* int get() 获取当前值
* void set(int newValue) 设置为给定值
* int getAndAdd(int delta) 以原子方式将给定值与当前值相加
* int decrementAndGet() 以原子方式将当前值减 1
* int incrementAndGet() 以原子方式将当前值加 1
* void lazySet(int newValue) 最后设置为给定值

  方法可以通过API查找。我们主要是了解，为什么原子类型的操作就是线程安全的？底层又是如何进行原子操作的呢？
对JVM有过一定了解的应该都知道CAS操作。CAS操作有3个操作数，内存值M，预期值E，新值U，如果M==E，则将内存值修改为B，否则啥都不做。就是当且仅当内存值与当前值一致，才进行值的更新。否则不更新。这样就保证了atomic包下面的操作的原子行。那它是如何实现的呢？看一看源码：
```bash
    public class AtomicInteger extends Number implements java.io.Serializable {
        private static final long serialVersionUID = 6214790243416807050L;
    
        // setup to use Unsafe.compareAndSwapInt for updates
        private static final Unsafe unsafe = Unsafe.getUnsafe();
        private static final long valueOffset;
    
        static {
          try {
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
          } catch (Exception ex) { throw new Error(ex); }
        }
    
        private volatile int value;
    
        /**
         * Creates a new AtomicInteger with the given initial value.
         *
         * @param initialValue the initial value
         */
        public AtomicInteger(int initialValue) {
            value = initialValue;
        }
        }
```
这是AtomicInteger中的一段源码。我们可以看到其中有一个Unsafe类，一个volatile 修饰的value. 这个value就是先保证了AtomicInteger操作的可见性。然后Unsafe保证对这个value值的操作都是CAS操作。这样就保证了其原子性。Unsafe源码的一部分：
```angular2
    /**
    * 比较obj的offset处内存位置中的值和期望的值，如果相同则更新。此更新是不可中断的。
    * 
    * @param obj 需要更新的对象
    * @param offset obj中整型field的偏移量
    * @param expect 希望field中存在的值
    * @param update 如果期望值expect与field的当前值相同，设置filed的值为这个新值
    * @return 如果field的值被更改返回true
    */
    public native boolean compareAndSwapInt(Object obj, long offset, int expect, int update);
```
###*原子更新数组类型
1. AtomicIntegerArray 原子更新整型数组
2. AtomicLongArray 原子更新长整型数组\
3. AtomicReferenceArray 原子更新引用数组
####*AtomicIntegerArray 常用的方法
+ int get(int i) 获取位置 i 的当前值
+ void set(int i, int newValue) 将位置 i 的元素设置为给定值
+ int length() 返回该数组的长度
+ int getAndAdd(int i, int delta) 以原子方式将给定值与索引 i 的元素相加
+ int getAndDecrement(int i) 以原子方式将索引 i 的元素减 1
+ int getAndIncrement(int i) 以原子方式将索引 i 的元素加 1
+ AtomicIntegerArray(int length) 创建给定长度的新 AtomicIntegerArray
+ AtomicIntegerArray(int[] array) 创建与给定数组具有相同长度的新 AtomicIntegerArray，并从给定数组复制其所有素

    先来看一下AtomicIntegerArray的部分源码：
###*引用类型
+ AtomicReference：引用类型原子类
+ AtomicStampedRerence：原子更新引用类型里的字段原子类
+ AtomicMarkableReference ：原子更新带有标记位的引用类型
