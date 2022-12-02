## 11 并发

> **T**HREADS allow multiple activities to proceed concurrently. Concurrent programming is harder than single-threaded programming, because more things can go wrong, and failures can be hard to reproduce. You can’t avoid concurrency. It is inherent in the platform and a requirement if you are to obtain good performance from multicore processors, which are now ubiquitous. This chapter contains advice to help you write clear, correct, well-documented concurrent programs.

线程机制允许多个活动实行进行。并发编程要比单线程编程要困难得多，因为有很事情都可能错，并且这些失败也很难重现。但是你不能避免并发。因为它是平台固有的，并且如果你想从多核处理器中获得更好的性能，就需要并发编程，多核处理器现在已经很普遍了。本章包含一些的一些建议可以帮助你写清晰、正确、文档组织良好的并发程序。

### Item78 同步访问共享的可变数据

> The synchronized keyword ensures that only a single thread can execute a method or block at one time. Many programmers think of synchronization solely as a means of *mutual exclusion*, to prevent an object from being seen in an inconsistent state by one thread while it’s being modified by another. In this view, an object is created in a consistent state (Item 17) and locked by the methods that access it. These methods observe the state and optionally cause a *state transition*, transforming the object from one consistent state to another. Proper use of synchronization guarantees that no method will ever observe the object in an inconsistent state.

关键字synchronized可以保证在同一个时刻，只有一个线程在执行一个方法或者代码块。很多程序员都只是把同步理解为一种互斥，当一个对象被一个线程修改的时候，可以组织其他的线程看到这个对象的不一致状态。按照这个观点，对象创建的时候处于一个一致性状态（Item17），并且锁定了访问这个对象的方法，这些方法可以观察到这个状态，并且有可能造成状态转换，将对象的状态从一个一致性状态转移到另一个一致性状态。合理的使用同步，可以保证没有方法可以观察到对象处在不一致状态中。

> This view is correct, but it’s only half the story. Without synchronization, one thread’s changes might not be visible to other threads. Not only does synchroniza- tion prevent threads from observing an object in an inconsistent state, but it ensures that each thread entering a synchronized method or block sees the effects of all previous modifications that were guarded by the same lock.

这种观点是正确的，但是并不全面。没有同步的话，一个线程的变化，就不能被其他的线程看到。同步不仅仅可以阻止线程看到对象处在不一致状态，还可以保证每个线程在进入同步方法或者代码块的时候，可以看到由同一锁保护的代码的之前的修改。

> The language specification guarantees that reading or writing a variable is *atomic* unless the variable is of type long or double [JLS, 17.4, 17.7]. In other words, reading a variable other than a long or double is guaranteed to return a value that was stored into that variable by some thread, even if multiple threads modify the variable concurrently and without synchronization.

Java语言规范保证了对一个变量的读写是原子性的，除非这个变量的类型是long或者double [JLS, 17.4, 17.7]。换句话说，读取一个既不是long也不是double的变量的时候，可以保证返回的值是其他线程保存在里面的，即使多个线程在没有同步的情况下，并发地修改了这个变量。

> You may hear it said that to improve performance, you should dispense with synchronization when reading or writing atomic data. This advice is dangerously wrong. While the language specification guarantees that a thread will not see an arbitrary value when reading a field, it does not guarantee that a value written by one thread will be visible to another. **Synchronization is required for reliable communication between threads as well as for mutual exclusion.** This is due to a part of the language specification known as the *memory model*, which specifies when and how changes made by one thread become visible to others [JLS, 17.4; Goetz06, 16].

你可能听过这样的说法，为了提高性能，在读或者写原子数据的时候，你应该省略掉同步。这个建议是非常危险，并且错误的。虽然语言规范保证了线程在读取一个域的值的时候，不会看到任意的数据，但是它没有保证另一个线程写入的值对该线程可见。**为了进行可靠的通信以及互斥访问，同步还是很有必要的。**这是源于Java语言规范中的内存模型，它规定了一个线程的修改在什么时候以及如何做才能对其他的线程可见 [JLS, 17.4; Goetz06, 16]。

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

这种优化被称为提升，OpenJDK Server VM也确实是这么做的。结果是一个活性失败：这个程序并没有得到任何的提升。一种解决这个问题的方法是同步访问stopRequested域，这个程序就会如预期的那样，在一秒左右结束：

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

