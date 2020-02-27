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

1. 可见性
2. 防止指令重排序

-------------------------



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

![](assets/阿里巴巴Java开发手-Arrays.asList()方法.png)

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
