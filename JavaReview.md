## 一、集合

### 1.ArrayList与LinkendList区别

|                          | ArrayList | LinkendList |
| ------------------------ | --------- | ----------- |
| 数据结构                 | 数组      | 双向链表    |
| 插入和删除受元素位置影响 | &radic;   | &times;     |
| 支持快速随机访问         | &radic;   | &times;     |
| 内存空间连续             | &radic;   | &times;     |

>  补充：RandomAccess是标记接口，如Collections.binarySearch方法会判断传入的list是否实现RandomAccess判断其是否支持快速随机访问，调用不同的方法遍历(索引/迭代器)

list遍历方式选择：

1. 实现了 RandomAccess 接口的list，优先选择普通 for 循环，其次 foreach
2. 未实现 RandomAccess接口的list，优先选择iterator遍历（foreach遍历底层也是通过iterator实现的），大size的数据，千万不要使用普通for循环

### 2.HashMap与HashTable区别

|                     | HashMap                   | HashTable |
| ------------------- | ------------------------- | --------- |
| 线程安全且效率低    | &times;                   | &radic;   |
| 数据结构            | 数组+链表+红黑树（大于8） | 数组+链表 |
| 容量始终为2的幂次方 | &radic;                   | &times;   |
| 初始容量和扩容量    | 16，2n                    | 11，2n+1  |
| 支持Null Key        | &radic;                   | &times;   |

> 

---------------------



## 二、线程

### 1.图解进程与线程的关系

![](images/JVM运行时数据区域.png)

### 2.线程死锁

产生线程死锁的四个必备条件及如何避免：

1. 互斥条件（无法破坏）
2. 请求与保持条件：进程因请求资源而阻塞时，对已获资源保持不放（一次申请完资源）
3. 不剥夺条件:线程已获资源在末使用完之前不能被其他线程强行剥夺（主动释放资源）
4. 循环等待条件（有序申请资源）

### 3.双重校验锁实现对象单例

```java
public class Singleton {

    private volatile static Singleton uniqueInstance;

    private Singleton() {
    }

    public static Singleton getUniqueInstance() {
       //第一次判断防止实例化后还进入判断，提升效率
        if (uniqueInstance == null) {
            //类对象加锁
            synchronized (Singleton.class) {
                if (uniqueInstance == null) {
                    uniqueInstance = new Singleton();
                }
            }
        }
        return uniqueInstance;
    }
}
```

uniqueInstance 采用 volatile 关键字修饰很有必要， uniqueInstance = new Singleton()分为三步执行：

1. 为 uniqueInstance 分配内存空间
2. 初始化 uniqueInstance
3. 将 uniqueInstance 指向分配的内存地址

多线程下JVM的重排序会导致出现问题，如线程T1执行了1和3，T2会返回未被初始化的对象，**volatile可以禁止JVM的重排序**。

### 4.synchronized关键字

![](images/二  Synchronized 关键字使用、底层原理、JDK1.6 之后的底层优化以及 和ReenTrantLock 的对比.png)

### 5.volatile关键字

volatile作用：

1. 可见性（从主存读取，修改后刷新到主存）
2. 防止指令重排序

volatile关键字和synchronized关键字比较：

|                  | volatile         | synchronized   |
| ---------------- | ---------------- | -------------- |
| 线程同步实现级别 | 轻量级、性能更好 | 重量级         |
| 修饰             | 变量             | 方法、代码块   |
| 作用             | 可见性           | 可见性、原子性 |
| 是否阻塞         | &times;          | &radic;        |

### 6.ThreadLocal

实现每个线程都有自己的专属本地变量（线程局部变量）

原理：

每个线程都维护有一个ThreadLocal.ThreadLocalMap，存储以ThreadLocal为key的键值对

```java
public class ThreadLocalExample1 {
    public static void main(String[] args) {
        ThreadLocal threadLocal1 = new ThreadLocal();
        ThreadLocal threadLocal2 = new ThreadLocal();
        Thread thread1 = new Thread(() -> {
            threadLocal1.set(1);
            threadLocal2.set(1);
        });
        Thread thread2 = new Thread(() -> {
            threadLocal1.set(2);
            threadLocal2.set(2);
        });
        thread1.start();
        thread2.start();
    }
}
```

![](images/68747470733a2f2f63732d6e6f7465732d313235363130393739362e636f732e61702d6775616e677a686f752e6d7971636c6f75642e636f6d2f36373832363734632d316266652d343837392d616633392d6539643732326139356433392e706e67.png)

内存泄漏问题：

ThreadLocalMap中的key为ThreadLocal的弱引用，而value是强引用，ThreadLocal在没有被外部强引用的情况，GC时key会被回收，value不会，ThreadLocalMap产生key为null的键值对，会造成内存泄漏，调用set()、get()、remove()的时候会清理掉key为null的记录，使用完手动调用remove()方法可避免内存泄漏

应用场景：

- 每个线程需要有自己单独的实例
- 实例需要在多个方法中共享，但不希望被多线程共享

1）存储用户session

```java
private static final ThreadLocal threadSession = new ThreadLocal();

public static Session getSession() throws InfrastructureException {
    Session s = (Session) threadSession.get();
    try {
        if (s == null) {
            s = getSessionFactory().openSession();
            threadSession.set(s);
        }
    } catch (HibernateException ex) {
        throw new InfrastructureException(ex);
    }
    return s;
}
```

2）解决线程安全问题

```java
public class DateUtil {
    private static ThreadLocal<SimpleDateFormat> format1 = new ThreadLocal<SimpleDateFormat>() {
        @Override
        protected SimpleDateFormat initialValue() {
            return new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        }
    };

    public static String formatDate(Date date) {
        return format1.get().format(date);
    }
}
```

原本非线程安全的`SimpleDateFormat`，这里是线程安全的

### 7.线程池

Executor框架结构：

1. 任务（Runnable/Callable）
2. 任务的执行（Executor）
3. 异步计算的结果（Future）

使用示意图：

![](images/68747470733a2f2f696d67636f6e766572742e6373646e696d672e636e2f6148523063446f764c3231354c574a73623263746447387464584e6c4c6d397a6379316a626931695a576c716157356e4c6d467361586c31626d4e7a4c6d4e76625338784f4330314c544d774c7a67304f44.jpg)

- **Callable接口用于处理Runnable接口不支持的用例，有返回结果和异常抛出**
- **submit方法相比execut方法**
  - **会返回实现了Future接口的对象，Runnable对象的返回结果为null**
  - **方便异常处理**
- **主线程可以执行 `FutureTask.get()`方法来等待任务执行完成。主线程也可以执行 `FutureTask.cancel（boolean mayInterruptIfRunning）`来取消此任务的执行**

ThreadPoolExecutor执行流程图：

![](images/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d372f254535253942254245254538254137254133254537254241254246254537254138253842254536254231254130254535254145253945.png)

`ThreadPoolExecutor`3个核心参数：

1. **`corePoolSize` :** 最小可以同时运行的线程数量。

2. **`maximumPoolSize` :** 队列存满的时候，当前可以同时运行的线程数量。

3. **`workQueue`:** 当前运行的线程数达到`corePoolSize` ，新任务会进入队列。

`ThreadPoolExecutor`其他常见参数:

1. **`keepAliveTime`**:当线程数量大于 `corePoolSize` 时，多余的空闲线程的最大存活时间 。
2. **`unit`** : 时间单位。
3. **`threadFactory`** :executor 创建新线程的时候会用到。
4. **`handler`** :饱和策略。

Executors的常见线程池对比：

|                      | corePoolSize | maximumPoolSize  | **`workQueue`**  | keepAliveTime |
| -------------------- | ------------ | ---------------- | ---------------- | ------------- |
| FixedThreadPool      | n            | n                | Intger.MAX_VALUE | 0             |
| SingleThreadExecutor | 1            | 1                | Intger.MAX_VALUE | 0             |
| CachedThreadPool     | 0            | Intger.MAX_VALUE | 0                | 60            |

**不推荐使用的原因都与Intger.MAX_VALUE有关**

线程池大小确定：

- CPU密集型任务（n+1）：n为CPU核心数
- I/O密集型任务（2n）：系统大部分时间处理I/O交互，不会占用CPU处理

### 8.乐观锁与悲观锁：

|          | 悲观锁                               | 乐观锁                                         |
| -------- | ------------------------------------ | ---------------------------------------------- |
| 思想     | 加独占锁供一个线程使用，阻塞其他线程 | 不加锁，更新的时候判断此期间是否被其他线程更改 |
| 应用场景 | 写多于读（资源竞争多）               | 读多于写（资源竞争少）                         |
| 实现     | synchronized、ReentrantLock          | CAS、版本号                                    |

CAS算法：

比较内存中的某个值V是否与预期值A相同，相同则将该值更新为新值B，不相同则不修改，具备原子性，其通过Unsafe类调用CPU指令实现

CAS缺点：

1. **ABA问题，可通过在变量上加版本号解决**，如1A>2B>3A，`AtomicStampedReference`也是通过版本号解决该问题
2. **自旋（不成功就循环直到成功）长时间不成功开销大**，限制自旋次数解决
3. 只能保证一个共享变量的原子性

### 9.Atomic原子类总结：

![](images/TIM截图20200229124055.png)

### 10.AQS同步组件总结：

![](images/TIM截图20200229124553.png)

AQS原理图：

![](images/640.webp)

> CLH(Craig,Landin,and Hagersten)队列是一个虚拟的双向队列（不存在队列实例，仅存在结点之间的关联关系）。AQS是将每条请求共享资源的线程封装成一个CLH锁队列的一个结点（Node）来实现锁的分配。

CountDownLatch和CyclicBarrier的区别：

| CountDownLatch                                       | CyclicBarrier                                |
| ---------------------------------------------------- | -------------------------------------------- |
| 减计数方式，为0时释放等待线程                        | 加计数方式，达指定值时释放等待线程           |
| 计数为0时无法重置，不可重复利用                      | 计数达到指定值时通过reset重置为0，可重复利用 |
| 调用countDown()方法计数减一，调用await()方法进行阻塞 | 调用await()方法计数加一，未到到指定值则阻塞  |

### 11.并发容器

- **ConcurrentHashMap:** 线程安全的 HashMap

- **CopyOnWriteArrayList:** 线程安全的 List，适合读多写少的场合

  - 读操作不加锁，写操作加锁并且新建数组做修改后替换
  - 存在脏读，最终一致性
  - 不同于ReentreentReadWriteLock的仅读读不阻塞

- **ConcurrentLinkedQueue:** 高效的并发队列，使用链表实现。可以看做一个线程安全的 LinkedList，这是一个非阻塞队列。

- **BlockingQueue:** 这是一个接口，JDK 内部通过链表、数组等方式实现了这个接口。表示阻塞队列，非常适合用于作为数据共享的通道。

  - **ArrayBlockingQueue**：有界队列、数组
  - **LinkedBlockingQueue**：有/无界队列、单向链表
  - **PriorityBlockingQueue**：无界、支持优先级（comparTo/Comparator）

- **ConcurrentSkipListMap:** 跳表的实现。这是一个 Map，使用跳表的数据结构进行快速查找。

  - 基于多指针有序链表实现

  ![](images/68747470733a2f2f63732d6e6f7465732d313235363130393739362e636f732e61702d6775616e677a686f752e6d7971636c6f75642e636f6d2f30656133376565322d633232342d346337392d623839352d6531333163363830356334302e706e67.png)

## 三、JVM

------------------------------



## 四、其他

### 1.Arrays.asList()使用指南

- 传递对象必须是对象数组，不能是基本类型

```java 
int[] myArray = { 1, 2, 3 };
List myList = Arrays.asList(myArray);
System.out.println(myList.size());//1
System.out.println(myList.get(0));//数组地址值
System.out.println(myList.get(1));//报错：ArrayIndexOutOfBoundsException
int [] array=(int[]) myList.get(0);
System.out.println(array[0]);//1
```

- 数组转为集合后，底层还是数组

![](images/阿里巴巴Java开发手-Arrays.asList()方法.png)

- 将数组转换为ArrayList

```java 
List list = new ArrayList<>(Arrays.asList("a", "b", "c"))
```

> 也可使用Java8的Stream

```java 
Integer [] myArray = { 1, 2, 3 };
List myList = Arrays.stream(myArray).collect(Collectors.toList());
//基本类型也可以实现转换（依赖boxed的装箱操作）
int [] myArray2 = { 1, 2, 3 };
List myList = Arrays.stream(myArray2).boxed().collect(Collectors.toList());
```

