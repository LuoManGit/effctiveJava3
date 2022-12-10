## 11 并发

> **T**HREADS allow multiple activities to proceed concurrently. Concurrent programming is harder than single-threaded programming, because more things can go wrong, and failures can be hard to reproduce. You can’t avoid concurrency. It is inherent in the platform and a requirement if you are to obtain good performance from multicore processors, which are now ubiquitous. This chapter contains advice to help you write clear, correct, well-documented concurrent programs.

线程机制允许多个活动实行进行。并发编程要比单线程编程要困难得多，因为有多很地方都可能出错，并且这些错误也很难重现。但是你不能避免并发。因为它是平台固有的，并且如果你想要更好地利用多核处理器中的性能，就需要并发编程，毕竟多核处理器现在已经很普遍了。本章包含一些的一些建议可以帮助你写清晰、正确、文档组织良好的并发程序。

### Item78 同步访问共享的可变数据

> The synchronized keyword ensures that only a single thread can execute a method or block at one time. Many programmers think of synchronization solely as a means of *mutual exclusion*, to prevent an object from being seen in an inconsistent state by one thread while it’s being modified by another. In this view, an object is created in a consistent state (Item 17) and locked by the methods that access it. These methods observe the state and optionally cause a *state transition*, transforming the object from one consistent state to another. Proper use of synchronization guarantees that no method will ever observe the object in an inconsistent state.

关键字synchronized可以保证在同一个时刻，只有一个线程在执行一个方法或者代码块。很多程序员都只是把同步理解为一种互斥，当一个对象被一个线程修改的时候，可以组织其他的线程看到这个对象的不一致状态。按照这个观点，对象创建的时候处于一个一致性状态（Item17），并且锁定了访问这个对象的方法，这些方法可以观察到这个状态，并且有可能造成状态转换，将对象的状态从一个一致性状态转移到另一个一致性状态。合理的使用同步，可以保证没有方法可以观察到对象处在不一致状态中。

> This view is correct, but it’s only half the story. Without synchronization, one thread’s changes might not be visible to other threads. Not only does synchronization prevent threads from observing an object in an inconsistent state, but it ensures that each thread entering a synchronized method or block sees the effects of all previous modifications that were guarded by the same lock.

这种观点是正确的，但是并不全面。没有同步的话，一个线程的修改，可能对其他线程不可见。同步不仅仅可以阻止线程看到对象处在不一致状态，还可以保证每个线程在进入同步方法或者代码块的时候，可以看到由同一锁保护的代码的所有之前的修改。

> The language specification guarantees that reading or writing a variable is *atomic* unless the variable is of type long or double [JLS, 17.4, 17.7]. In other words, reading a variable other than a long or double is guaranteed to return a value that was stored into that variable by some thread, even if multiple threads modify the variable concurrently and without synchronization.

Java语言规范保证了对一个变量的读写是原子性的，除非这个变量的类型是long或者double [JLS, 17.4, 17.7]。换句话说，读取一个既不是long也不是double的变量的时候，可以保证返回的值是其他线程保存在里面的，即使有多个线程在没有同步的情况下，并发地修改了这个变量。*（译者注：原子性 指的是 一个操作不会分割为几步，像一个原子一样。但是不能保证，不同线程读到的数据的一致性。比如说，可能存在，int变量x，初始值是3，线程A修改了int变量x的值为5，在这之后，线程B读到变量x的值还是3。为了要保证线程B正确地读到线程A的值，就需要保证 变量x 读写操作的可见性，可见性在下面会介绍。）* 

> You may hear it said that to improve performance, you should dispense with synchronization when reading or writing atomic data. This advice is dangerously wrong. While the language specification guarantees that a thread will not see an arbitrary value when reading a field, it does not guarantee that a value written by one thread will be visible to another. **Synchronization is required for reliable communication between threads as well as for mutual exclusion.** This is due to a part of the language specification known as the *memory model*, which specifies when and how changes made by one thread become visible to others [JLS, 17.4; Goetz06, 16].

你可能听过这样的说法，为了提高性能，在读或者写原子数据的时候，你应该省略掉同步。这个建议是非常危险，并且错误的。虽然语言规范保证了线程在读取一个域的值的时候，不会看到随机的数据，但是它没有保证另一个线程写入的值对该线程可见。**为了进行可靠的通信以及互斥访问，同步还是很有必要的。**这是源于Java语言规范中的内存模型，它规定了一个线程的修改在什么时候以及如何做才能对其他的线程可见 [JLS, 17.4; Goetz06, 16]。

> The consequences of failing to synchronize access to shared mutable data can be dire even if the data is atomically readable and writable. Consider the task of stopping one thread from another. The libraries provide the Thread.stop method, but this method was deprecated long ago because it is inherently *unsafe*—its use can result in data corruption. **Do not use** **Thread.stop**.A recommended way to stop one thread from another is to have the first thread poll a boolean field that is initially false but can be set to true by the second thread to indicate that the first thread is to stop itself. Because reading and writing a boolean field is atomic, some programmers dispense with synchronization when accessing the field:

如果对共享可变数据的访问同步失败的话，即使这个数据是原子读写的，那后果也是非常严重的。比如从一个线程停止另一个线程的例子。虽然类库提供了Thread.stop()的方法，但是这个方法很久以前就不推荐使用了，因为它存在内在的不安全问题——它的使用会导致数据遭到破坏。**因此不要使用Thread.stop**。对于从一个线程中停止另一个线程，有一种推荐的方法是让第一个线程给一个boolean域设为false，然后第二个线程可以把这个域设为true，然后第一个线程就可以把自己终结掉了。因为读和写一个boolean域是原子性的，一些程序员就会在访问这个属性的时候不使用同步，如下：

```java
 // Broken! - How long would you expect this program to run?
   public class StopThread {
       private static boolean stopRequested;
       public static void main(String[] args)
               throws InterruptedException {
           Thread backgroundThread = new Thread(() -> {
               int i = 0;
               while (!stopRequested)
                   i++;
           });
           backgroundThread.start();
           TimeUnit.SECONDS.sleep(1);
           stopRequested = true;
       }
}

```

> You might expect this program to run for about a second, after which the main thread sets stopRequested to true, causing the background thread’s loop to terminate. On my machine, however, the program *never* terminates: the background thread loops forever!

你可能希望这个程序可以允许1秒钟左右，在主线程设置了stopRequested为true以后，就会导致后台线程中的循环终止。然而，在作者的机器上，这个程序永远都不会终止，这个后台线程的循环会一直运行。

> The problem is that in the absence of synchronization, there is no guarantee as to when, if ever, the background thread will see the change in the value of stopRequested made by the main thread. In the absence of synchronization, it’s quite acceptable for the virtual machine to transform this code:

这里的问题就是缺少了同步操作，因此，没有办法保证background线程，何时能看到stopRequested值被主程序修改了。缺少同步的话，虚拟机会把下面这个代码：

```java
while (!stopRequested)
       i++;
```

> into this code:

转换成这样：

```java
if (!stopRequested)
       while (true)
					i++;
```

> This optimization is known as *hoisting*, and it is precisely what the OpenJDK Server VM does. The result is a *liveness failure*: the program fails to make progress. One way to fix the problem is to synchronize access to the stopRequested field. This program terminates in about one second, as expected:

这种优化被称为提升，OpenJDK Server VM也确实是这么做的。结果是一个**活性失败** (liveness failure)：这个程序并没有得到任何的提升。一种解决这个问题的方法是同步访问stopRequested域，这个程序就会如预期的那样，在一秒左右结束：

```java
// Properly synchronized cooperative thread termination
   public class StopThread {
       private static boolean stopRequested;
       private static synchronized void requestStop() {
           stopRequested = true;
       }
       private static synchronized boolean stopRequested() {
           return stopRequested;
       }
       public static void main(String[] args)
               throws InterruptedException {
           Thread backgroundThread = new Thread(() -> {
               int i = 0;
							 while (!stopRequested()) 
                 i++;
           });
           backgroundThread.start();
           TimeUnit.SECONDS.sleep(1);
					 requestStop(); 
       }
}
```

> Note that both the write method (requestStop) and the read method (stopRequested) are synchronized. It is *not* sufficient to synchronize only the write method! **Synchronization is not guaranteed to work unless both read and write operations are synchronized.** Occasionally a program that synchronizes only writes (or reads) may *appear* to work on some machines, but in this case, appearances are deceiving.

需要注意的是写方法（requestStop）和读方法（stopRequested）都是同步的。如果只是同步写方法的话，还不够！**只有读写操作都是同步的时候，才能保证同步起作用**。有的时候，程序只对写（或者读）操作进行了同步，在一些机器上也能正常工作，但是在这种情况下，这种表象有很大的欺骗性。

> The actions of the synchronized methods in StopThread would be atomic even without synchronization. In other words, the synchronization on these methods is used *solely* for its communication effects, not for mutual exclusion. While the cost of synchronizing on each iteration of the loop is small, there is a correct alternative that is less verbose and whose performance is likely to be better. The locking in the second version of StopThread can be omitted if stopRequested is declared volatile. While the volatile modifier performs no mutual exclusion, it guarantees that any thread that reads the field will see the most recently written value:

在StopThread类中的同步方法的操作，即使没有同步，也具有原子性。换句话说，就是这些方法上的同步只是用来做通信的，不是用来做互斥访问的。虽然在循环的每次迭代中同步的开销很小，但是还是有一种替代方法，可以更简洁，且效率也更高。如果stopRequested声明为volatile的话，上面这个版本的StopThread中的锁就可以省略掉了。虽然这个volatile操作符不执行互斥访问，但是它可以保证每个线程在读取这个域的时候，可以看到最近写入的值：

```java
// Cooperative thread termination with a volatile field
public class StopThread {
			 private static volatile boolean stopRequested;
       public static void main(String[] args)
               throws InterruptedException {
           Thread backgroundThread = new Thread(() -> {
               int i = 0;
               while (!stopRequested)
                   i++;
           });
           backgroundThread.start();
           TimeUnit.SECONDS.sleep(1);
           stopRequested = true;
       }
}
```

> You do have to be careful when using volatile. Consider the following method, which is supposed to generate serial numbers:

在使用volatile的时候，必须要小心，比如下面这个用来生成连续数值的方法：

```
/ Broken - requires synchronization!
   private static volatile int nextSerialNumber = 0;
   public static int generateSerialNumber() {
       return nextSerialNumber++;
}
```

> The intent of the method is to guarantee that every invocation returns a unique value (so long as there are no more than 2^32 invocations). The method’s state consists of a single atomically accessible field, nextSerialNumber, and all possible values of this field are legal. Therefore, no synchronization is necessary to protect its invariants. Still, the method won’t work properly without synchronization.

这个方法的目的 ，是想保证每次调用都返回一个不一样的值（只要这里的调用次数不超过 在2^32就行）。这个方法的状态里只包含了一个可以原子访问的变量 extSerialNumber，这个变量所有的值都是合法的。因此，看起来就不需要加同步来保护它的约束条件。但是这个方法，如果没有同步的话，它的运行，就会不符合我们的预期。

> The problem is that the increment operator (++) is not atomic. It performs *two* operations on the nextSerialNumber field: first it reads the value, and then it writes back a new value, equal to the old value plus one. If a second thread reads the field between the time a thread reads the old value and writes back a new one, the second thread will see the same value as the first and return the same serial number. This is a *safety failure*: the program computes the wrong results.

这里的问题在于，++这个自增符号，不是原子的。 它对 nextSerialNumber变量进行了两个操作，首先读取了这个值，然后 对读取到的值+1 以后，写入了这个值。 如果另外一个线程，在这儿两个步骤之间，读取了这个变量，这个线程就会和前面的线程看到同一个值，并且返回值也是一样的。 这里就有一个**安全性失败** （safety failure)： 这个程序就会计算出错误的结果来。

> One way to fix generateSerialNumber is to add the synchronized modifier to its declaration. This ensures that multiple invocations won’t be interleaved and that each invocation of the method will see the effects of all previous invocations. Once you’ve done that, you can and should remove the volatile modifier from nextSerialNumber. To bulletproof the method, use long instead of int, or throw an exception if nextSerialNumber is about to wrap.

一种修复 generateSerialNumber 的思路，是在这个方法的声明上 加上 synchronized修饰符。这样就可以包含智能，各个调用之间的执行不会交叉，每个调用都能看到前面其他调用的执行结果。 如果你加了synchronized 修饰符 的话，也就不需要对变量 nextSerialNumber 加 volatile的修饰符了。 为了防这个方法更加健壮，可以用long代替int，或者在nextSerialNumber 将要溢出的时候，直接抛出异常。

> Better still, follow the advice in Item 59 and use the class AtomicLong, which is part of java.util.concurrent.atomic. This package provides primitives for lock-free, thread-safe programming on single variables. While volatile provides only the communication effects of synchronization, this package also provides atomicity. This is exactly what we want for generateSerialNumber, and it is likely to outperform the synchronized version:

更好的方法，是遵守59 小节给出的意见，使用java.util.concurrent.atomic里的AtomicLong类。对于单个变量，这个包里提供了比较原始的不加锁的线程安全的操作。 volatile 只保证了 可见性，这个包 还能保证原子性。因此，这个正是我们实现一个 generateSerialNumber 方法所需要使用的。它的执行效率比那个synchronized的版本还要好一些：

``` java
// Lock-free synchronization with java.util.concurrent.atomic
private static final AtomicLong nextSerialNum = new AtomicLong();
public static long generateSerialNumber() { return nextSerialNum.getAndIncrement();
}
```

> The best way to avoid the problems discussed in this item is not to share mutable data. Either share immutable data (Item 17) or don’t share at all. In other words, **confine mutable data to a single thread.** If you adopt this policy, it is important to document it so that the policy is maintained as your program evolves. It is also important to have a deep understanding of the frameworks and libraries you’re using because they may introduce threads that you are unaware of.

要避免本节谈论的问题，最好的办法就是不共享可变数据，共享不可变数据，或者干脆不共享。换句话说，就是**只在单个线程里定义可变数据** 。如果你采用了这个原则的话，那么为了让这个原则在程序迭代过程中，可以一直被遵守，那就非常有必要为它撰写相关的文档。同时，也需要对使用的框架和类库有深刻的理解，因为他们也可能会引入一些你知道的线程。

> It is acceptable for one thread to modify a data object for a while and then to share it with other threads, synchronizing only the act of sharing the object reference. Other threads can then read the object without further synchronization, so long as it isn’t modified again. Such objects are said to be *effectively immutable* [Goetz06, 3.5.4]. Transferring such an object reference from one thread to others is called *safe publication* [Goetz06, 3.5.3]. There are many ways to safely publish an object reference: you can store it in a static field as part of class initialization; you can store it in a volatile field, a final field, or a field that is accessed with normal locking; or you can put it into a concurrent collection (Item 81)

如果只有一个线程在短时间内修改 一个数据对象，然后和其他的线程一起读取这个对象，那么只对读取操作进行同步，是可以接受的。如果这个对象只有一次写入后，不会再被修改了，那么其他的线程在读取这个对象的时候，也可以不加同步。这种对象 被称为**等效不可变** ，把这样一个 等效不可变 的对象，从一个线程里传到其他线程的过程，称为**安全发布** 。有很多 方法 对一个对象的引用进行安全发布，比如保存作为类初始化的一部分，保存在一个类的静态域里；保存在volatile域，final域，或者加了普通的锁的域，或者放到并发集合里（81小结）

> In summary, **when multiple threads share mutable data, each thread that reads or writes the data must perform synchronization.** In the absence of synchronization, there is no guarantee that one thread’s changes will be visible to another thread. The penalties for failing to synchronize shared mutable data are liveness and safety failures. These failures are among the most difficult to debug. They can be intermittent and timing-dependent, and program behavior can vary radically from one VM to another. If you need only inter-thread communication, and not mutual exclusion, the volatile modifier is an acceptable form of synchronization, but it can be tricky to use correctly.

总的来说，**当多个线程共享可变数据的时候，每个线程的读写操作都必须加上同步**。如果缺了同步操作的话，就无法保证一个线程的操作，对另外一个线程是可见的。对于共享数据，如果同步失败的话，就会出现 活性失败或者安全性失败。这些问题是最难通过debug发现的。他们可能是随机的，和时间相关，在不同的操作系统上，可能表现也不一致。如果你只需要做线程间的通信，不需要互斥，那么volatile修饰符，是一个可以选择的同步方式，只是要正确地使用这个修饰符，还是需要一些技巧的。

