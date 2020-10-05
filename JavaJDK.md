 

# java.lang.Math

四舍五入：Math.round(11.5) 等于多少？Math.round(-11.5)等于多少？

答：Math.round(11.5)的返回值是12，Math.round(-11.5)的返回值是-11。四舍五入的原理是在参数上加0.5然后进行下取整。

 

Math.floor()向下取整

 

Math.round() 四舍五入取整，Math.floor(x+1/2)

 

# 软件包 java.lang.reflect 

| ` `[Method](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/lang/reflect/Method.html) | **[getMethod](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/lang/Class.html#getMethod(java.lang.String, java.lang.Class...))**`(`[String](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/lang/String.html)` name,  `[Class](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/lang/Class.html)`... parameterTypes)`          返回一个 `Method` 对象，它反映此 `Class` 对象所表示的类或接口的指定**公共**成员方法。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ` `[Object](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/lang/Object.html) | **[invoke](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/lang/reflect/Method.html#invoke(java.lang.Object, java.lang.Object...))**`(`[Object](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/lang/Object.html)` obj,  `[Object](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/lang/Object.html)`... args)`          对带有指定参数的指定对象调用由此 `Method` 对象表示的底层方法。 |

 

Class<? extends List> class1 = list.getClass();

Method method = class1.getMethod("add",Object.class);

method.invoke(list, new Integer.valueOf(3));//绕过编译器，在Sting集合中加入Integer

 

 

 

# String

Thus the length of the **substring** is **endIndex-beginIndex**.

Examples: 

 "hamburger".substring(4, 8) returns "urge"

 "smiles".substring(1, 5) returns "mile"

 

**[String](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/lang/String.html#String(char[], int, int))**`(char[] value, int offset, int count)` 
      分配一个新的 `String`，它包含取自字符数组参数一个子数组的字符。

 

构造函数和subString方法里面参数注意区分，一个长度为**endIndex-beginIndex**，另一个为count

 

String.format("%.2f", f);

 

 

# Java.util.Arrays

| `static `[String](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/lang/String.html) | **[toString](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/Arrays.html#toString(java.lang.Object[]))**`(`[Object](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/lang/Object.html)`[] a)`          返回指定数组内容的字符串表示形式。//打印数组 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |

 

| `static int` | **[binarySearch](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/Arrays.html#binarySearch(byte[], int, int, byte))**`(byte[] a, int fromIndex, int toIndex, byte key)`         使用二分搜索法来搜索指定的 byte 型数组的范围，以获得指定的值。 |
| ------------ | ------------------------------------------------------------ |
|              |                                                              |

 

| `static void` | **[sort](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/Arrays.html#sort(java.lang.Object[], int, int))**`(`[Object](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/lang/Object.html)`[] a,  int fromIndex, int toIndex)`         根据元素的[自然顺序](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/lang/Comparable.html)对指定对象数组的指定范围按升序进行排序。 |
| ------------- | ------------------------------------------------------------ |
|               |                                                              |

 

| `static void` | **[fill](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/Arrays.html#fill(int[], int))**`(int[] a, int val)`         将指定的  int 值分配给指定 int 型数组的每个元素。 |
| ------------- | ------------------------------------------------------------ |
|               |                                                              |

 

 

# Java.util.Collections

**[sort](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/Collections.html#sort(java.util.List))**`(`[List](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/List.html)` list)` 
      根据元素的*自然顺序* 对指定列表按升序进行排序。

 

 

# Java.util.ArrayList

System.out.printf(list);//直接打印list，以数组形式打印在控制台

 

 

# java.util.PriorityQueue<E>

**[PriorityQueue](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/PriorityQueue.html#PriorityQueue())**`()` 
      使用默认的初始容量（**11**）创建一个 `PriorityQueue`，并根据其[自然顺序](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/lang/Comparable.html)对元素进行排序。

**[PriorityQueue](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/PriorityQueue.html#PriorityQueue(int))**`(int initialCapacity)` 
      使用指定的初始容量创建一个 `PriorityQueue`，并根据其[自然顺序](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/lang/Comparable.html)对元素进行排序。最小堆

**[PriorityQueue](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/PriorityQueue.html#PriorityQueue(int, java.util.Comparator))**`(int initialCapacity, `[Comparator](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/Comparator.html)`[E](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/PriorityQueue.html)`> comparator)` 
      使用指定的初始容量创建一个 `PriorityQueue`，并根据指定的比较器对元素进行排序。

 

 

 

 

 

11、swtich 是否能作用在byte 上，是否能作用在long 上，是否能作用在String上？

答：在Java 5以前，switch(expr)中，expr只能是byte、short、char、int。从Java 5开始，Java中引入了枚举类型，expr也可以是enum类型，从Java 7开始，expr还可以是字符串（String），但是长整型（long）在目前所有的版本中都是不可以的。

 

9、解释内存中的栈(stack)、堆(heap)和静态区(static area)的用法。

答：通常我们定义一个基本数据类型的变量，一个对象的引用，还有就是函数调用的现场保存都使用内存中的栈空间；而通过new关键字和构造器创建的对象放在堆空间；程序中的字面量（literal）如直接书写的100、"hello"和常量都是放在静态区中。栈空间操作起来最快但是栈很小，通常大量的对象都是放在堆空间，理论上整个内存没有被其他进程使用的空间甚至硬盘上的虚拟内存都可以被当成堆空间来使用。

 

 

7、int和Integer有什么区别？

答：Java是一个近乎纯洁的面向对象编程语言，但是为了编程的方便还是引入了基本数据类型，但是为了能够将这些基本数据类型当成对象操作，Java为每一个基本数据类型都引入了对应的包装类型（wrapper class），int的包装类就是Integer，从Java 5开始引入了自动装箱/拆箱机制，使得二者可以相互转换。

 

\1.     **public** **static** **void** main(String[] args) { 

\2.       Integer a = **new** Integer(3); 

\3.       Integer b = 3;         // 将3自动装箱成Integer类型 

\4.       **int** c = 3; 

\5.       System.out.println(a == b);   // false 两个引用没有引用同一对象 

\6.       System.out.println(a == c);   // true a自动拆箱成int类型再和c比较 

\7.     } 

\8.   } 

 

 

**Integer** 类：

public class Test03 {

  public static void main(String[] args) {

​    **Integer** f1 = 100, f2 = 100, f3 = 150, f4 = 150;

​    System.out.println(f1 == f2);       **//true**

​    System.out.println(f3 == f4);       **//false**

  }

}

 

简单的说，如果整型字面量的值在**-128****到127**之间，那么不会new新的Integer对象，而是直接引用常量池中的Integer对象，所以上面的面试题中f1==f2的结果是true，而f3==f4的结果是false。

 

 

问题：在System.out.println()里面,System, out, println分别是什么?

答案：System是系统提供的预定义的**final****类**，out类变量，是一个**PrintStream****对象**，println是out对象里面一个重载的方法。

 

 

问题：public static void写成static public void会怎样？

答案：程序正常编译及运行。

 

 

问题，变量声明和定义有什么不同？

答案：声明变量我们只提供变量的类型和名字，并没有进行初始化。

定义包括声明和初始化两个阶段String s;只是变量声明，String s = new String(“bob”); 或者String s =“bob”;是变量定义。

 

问题：Java支持哪种参数传递类型?

答案：Java参数都是进行传值。对于对象而言，传递的值是对象的引用，也就是说原始引用和参数引用的那个拷贝，都是指向同一个对象。

 

问题：怎么判断数组是null还是为空?

答案：输出array.length的值，如果是0,说明数组为空。如果是null的话，会抛出空指针异常。

 

问题：方法可以同时即是static又是synchronized的吗?

答案：可以。如果这样做的话，JVM会获取和这个对象关联的java.lang.Class实例上的锁。这样做等于：synchronized(XYZ.class) { }，获取类锁（class锁）

 

 

 

 

 

 

# 同步包java.util.concurrent 

在并发编程中很常用的实用工具类。 

| **接口摘要**                                                 |                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **[BlockingDeque](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/concurrent/BlockingDeque.html)** | 支持两个附加操作的 [Queue](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/Queue.html)，这两个操作是：获取元素时等待双端队列变为非空；存储元素时等待双端队列中的空间变得可用。 |
| **[BlockingQueue](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/concurrent/BlockingQueue.html)** | 支持两个附加操作的 [Queue](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/Queue.html)，这两个操作是：获取元素时等待队列变为非空，以及存储元素时等待空间变得可用。 |
| **[Callable](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/concurrent/Callable.html)** | 返回结果并且可能抛出异常的任务。                             |
| **[Executor](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/concurrent/Executor.html)** | 执行已提交的 [Runnable](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/lang/Runnable.html) 任务的对象。 |
| **[ExecutorService](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/concurrent/ExecutorService.html)** | [Executor](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/concurrent/Executor.html) 提供了管理终止的方法，以及可为跟踪一个或多个异步任务执行状况而生成 [Future](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/concurrent/Future.html) 的方法。 |
| **[Future](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/concurrent/Future.html)** | `Future` 表示异步计算的结果。                                |
| **[RunnableFuture](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/concurrent/RunnableFuture.html)** | 作为 [Runnable](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/lang/Runnable.html) 的 [Future](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/concurrent/Future.html)。 |
| **[ScheduledExecutorService](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/concurrent/ScheduledExecutorService.html)** | 一个 [ExecutorService](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/concurrent/ExecutorService.html)，可安排在给定的延迟后运行或定期执行的命令。 |
| **[ScheduledFuture](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/concurrent/ScheduledFuture.html)** | 一个延迟的、结果可接受的操作，可将其取消。                   |
| **[ThreadFactory](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/concurrent/ThreadFactory.html)** | 根据需要创建新线程的对象。                                   |

 

## 包java.util.concurrent.locks

为锁和等待条件提供一个框架的接口和类，它不同于内置同步和监视器。

| **接口摘要**                                                 |                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **[Condition](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/concurrent/locks/Condition.html)** | `Condition` 将 `Object` 监视器方法（[wait](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/lang/Object.html#wait())、[notify](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/lang/Object.html#notify()) 和  [notifyAll](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/lang/Object.html#notifyAll())）分解成截然不同的对象，以便通过将这些对象与任意  [Lock](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/concurrent/locks/Lock.html) 实现组合使用，为每个对象提供多个等待 set（wait-set）。 |
| **[Lock](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/concurrent/locks/Lock.html)** | `Lock` 实现提供了比使用 `synchronized` 方法和语句可获得的更广泛的锁定操作。 |
| **[ReadWriteLock](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/concurrent/locks/ReadWriteLock.html)** | `ReadWriteLock` 维护了一对相关的[锁](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/concurrent/locks/Lock.html)，一个用于只读操作，另一个用于写入操作。 |


 **接口 Lock**

**所有已知实现类：** 

[ReentrantLock](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/concurrent/locks/ReentrantLock.html), [ReentrantReadWriteLock.ReadLock](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/concurrent/locks/ReentrantReadWriteLock.ReadLock.html), [ReentrantReadWriteLock.WriteLock](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/concurrent/locks/ReentrantReadWriteLock.WriteLock.html) 

 

**Java.util.concurrent.locks.ReentrantLock**

| ` void`                                                      | **[lock](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/concurrent/locks/Lock.html#lock())**`()`         获取锁。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ` void`                                                      | **[lockInterruptibly](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/concurrent/locks/Lock.html#lockInterruptibly())**`()`         如果当前线程未被[中断](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/lang/Thread.html#interrupt())，则获取锁。 |
| ` `[Condition](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/concurrent/locks/Condition.html) | **[newCondition](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/concurrent/locks/Lock.html#newCondition())**`()`         返回**绑定到此 `Lock` 实例**的新 [Condition](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/concurrent/locks/Condition.html) 实例。 |
| ` boolean`                                                   | **[tryLock](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/concurrent/locks/Lock.html#tryLock())**`()`         仅在调用时锁为空闲状态才获取该锁。 |
| ` boolean`                                                   | **[tryLock](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/concurrent/locks/Lock.html#tryLock(long, java.util.concurrent.TimeUnit))**`(long time, `[TimeUnit](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/concurrent/TimeUnit.html)` unit)`         如果锁在给定的等待时间内空闲，并且当前线程未被[中断](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/lang/Thread.html#interrupt())，则获取锁。 |
| ` void`                                                      | **[unlock](mk:@MSITStore:E:\API\java\API 文档\JDK6API中文参考手册[沈东良制]\JDK6API中文参考手册[沈东良制].chm::/java/util/concurrent/locks/Lock.html#unlock())**`()`         释放锁。 |

 

 

# Java8

## Lambda表达式与函数式接口

Lambda表达式（也称为闭包）是整个Java 8发行版中最受期待的在Java语言层面上的改变，Lambda表示函数书写，而函数作为参数传递进方法中）。

Lambda表示函数：

(parameters) -> expression 或 (parameters) ->{ statements; }

| 1.简化接口实现，这里接口只能有一个方法，即函数式接口  MathOperation subtraction = (a, b) -> a - b;//实现operation方法     interface MathOperation {   int operation(int a, int b);   }     **函数式接口**  函数式接口(Functional Interface)就是一个具有一个方法的普通接口。函数式接口可以被隐式转换为lambda表达式。函数式接口可以现有的函数友好地支持 lambda。 |
| ------------------------------------------------------------ |
| 2.简化匿名内部类的书写         Collections.sort(list, new Comparator<String>() {        public int compare(String s1, String s2) {        return s1.compareTo(s2);        }       });          Collections.sort(list, (s1, s2) -> s1.compareTo(s2)); |

 

 

## Stream

Java 8 API添加了一个新的抽象称为流Stream，可以让你以一种声明的方式处理集合中的数据。

这种风格将要处理的元素集合看作一种流， 流在管道中传输， 并且可以在管道的节点上进行处理， 比如筛选， 排序，聚合等。

元素流在管道中经过中间操作（intermediate operation）的处理，最后由最终操作(terminal operation)得到前面处理的结果。

 

+--------------------+    +------+  +------+  +---+  +-------+

| stream of elements +-----> |filter+-> |sorted+-> |map+-> |collect|

+--------------------+    +------+  +------+  +---+  +-------+

以上的流程转换为 Java 代码为：

List<Integer> transactionsIds = 

widgets.stream()

​       .filter(b -> b.getColor() == RED)

​       .sorted((x,y) -> x.getWeight() - y.getWeight())

​       .mapToInt(Widget::getWeight)

​       .sum();

 

 

**在 Java 8** **中,** **集合接口有两个方法来生成流：**

stream() − 为集合创建串行流。

parallelStream() − 为集合创建并行流。

 

List<String> strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");

List<String> filtered = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.toList());

 

Stream（流）是一个**来自数据源的元素队列并支持聚合操作。**

**元素**是特定类型的对象，形成一个队列。 Java中的Stream并不会存储元素，而是按需计算。

**数据源流**的来源。 可以是集合，数组，I/O channel， 产生器generator 等。

**聚合操作**类似SQL聚合函数语句一样的操作，比如filter, map, reduce, find, match, sorted等。

**内部迭代**： 以前对集合遍历都是通过Iterator或者For-Each的方式, 显式的在集合外部进行迭代， 这叫做外部迭代。 Stream提供了**内部迭代**的方式， 通过访问者模式(Visitor)实现。

生成流

 

## 接口的默认方法与静态方法

Java 8用默认方法与静态方法这两个新概念来扩展接口的声明。[默认方法](http://www.javacodegeeks.com/2014/04/java-8-default-methods-what-can-and-can-not-do.html)使接口可以包含实现代码，也就是目前Java8新增的功能。默认方法与抽象方法不同之处在于抽象方法必须要求实现，但是默认方法则没有这个要求。

另一个有趣的特性是接口可以声明（并且可以提供实现）静态方法。原来只可有静态常量。

 

## Java 8 方法引用

方法引用通过Clazz :: method来指向一个方法。

方法引用可以使语言的构造更紧凑简洁，减少冗余代码。方法引用使用一对冒号 :: 。

 

## 注解

**重复注解**

自从Java 5引入了[注解](http://www.javacodegeeks.com/2012/08/java-annotations-explored-explained.html)机制，这一特性就变得非常流行并且广为使用。然而，使用注解的一个限制是相同的注解在同一位置只能声明一次，不能声明多次。Java 8打破了这条规则，引入了重复注解机制，这样相同的注解可以在同一地方声明多次。

重复注解机制本身必须用@Repeatable注解。

 

扩展注解的支持

Java 8扩展了注解的上下文。现在几乎可以为任何东西添加注解：局部变量、泛型类、父类与接口的实现，就连方法的异常也能添加注解。

 

 

## Java编译器的新特性

很长一段时间里，Java程序员一直在发明不同的方式使得[方法参数的名字能保留在Java字节码](http://www.javacodegeeks.com/2014/04/constructormethod-parameters-metadata-available-via-reflection-in-jdk-8.html)中，（字节码中只有参数类型（int,int,String））并且能够在运行时获取它们（比如，[Paranamer类库](https://github.com/paul-hammant/paranamer)）。最终，在Java 8中把这个强烈要求的功能添加到语言层面（通过反射API与Parameter.getName()方法）与字节码文件（通过新版的javac的–parameters选项）中。

 

 

 

 

 

 

 

 

 

 

 

 

 