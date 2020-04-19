### Item8 避免使用终结方法和清除方法

> **Finalizers are unpredictable, often dangerous, and generally unnecessary.**Their use can cause erratic behavior, poor performance, and portability problems. Finalizers have a few valid uses, which we’ll cover later in this item, but as a rule, you should avoid them. As of Java 9, finalizers have been deprecated, but they are still being used by the Java libraries. The Java 9 replacement for finalizers is *cleaners*. **Cleaners are less dangerous than finalizers, but still unpredictable, slow, and generally unnecessary.**
>
> C++ programmers are cautioned not to think of finalizers or cleaners as Java’s analogue of C++ destructors. In C++, destructors are the normal way to reclaim the resources associated with an object, a necessary counterpart to constructors. In Java, the garbage collector reclaims the storage associated with an object when it becomes unreachable, requiring no special effort on the part of the programmer. C++ destructors are also used to reclaim other nonmemory resources. In Java, a try-with-resources or try-finally block is used for this purpose (Item 9).

终结方法（Finalizer，针对每个重写了finalize方法的对象，JVM会为之创建一个Finalizer对象）是不可预测的、非常危险的、通常情况下也是不必要的。他们的使用会导致行为不稳定、性能差、出现可移植问题。Finalizer只有一点点用，稍后我们将在本节中介绍，但是根据经验，还是要避免使用它。在java9里，已经使用清除方法（cleaner）替代了终结方法，但Finalizer在java类库里还是有使用。Cleaner没有Finalizer那么危险，但是也是不可预测、运行缓慢、一般来说没有必要的。

C++程序员要小心不要将Java里的Finalizers或者Cleaners认为是C++里的destructors相对应的东西。在C++里，destructor是用来回收一个对象绑定的资源的常用方法，和constructor相对应。但在java中，不需要程序员做出额外的努力，当对象变成不可达的时候，垃圾回收器会自动回收这个对象的存储空间。C++中destructor也可以用来回收不是存储空间的其他资源，在java中一般使用try-with-resources或者tr-finally结构来回收其他的资源（Item 9）。

> One shortcoming of finalizers and cleaners is that there is no guarantee they’ll be executed promptly [JLS, 12.6]. It can take arbitrarily long between the time that an object becomes unreachable and the time its finalizer or cleaner runs. This means that you should **never do anything time-critical in a finalizer or cleaner.** For example, it is a grave error to depend on a finalizer or cleaner to close files because open file descriptors are a limited resource. If many files are left open as a result of the system’s tardiness in running finalizers or cleaners, a program may fail because it can no longer open files.
>
> The promptness with which finalizers and cleaners are executed is primarily a function of the garbage collection algorithm, which varies widely across implementations. The behavior of a program that depends on the promptness of finalizer or cleaner execution may likewise vary. It is entirely possible that such a program will run perfectly on the JVM on which you test it and then fail miserably on the one favored by your most important customer.

Finalizer和Cleaner的一个缺点在于无法保证他们会被及时执行[JLS, 12.6]。从一个对象变成不可达到他的Finalizer或者Cleaner被执行，中间这段时间是任意长的。这就意味着，你不能在Cleaner或者Finalizer里做任何时间敏感的任务。比如在Cleaner和Finalizer里进行关闭文件的操作，就是一个非常严重的错误。因为打开的文件描述符是一个有限的资源，如果因为系统执行Finalizer或者Cleaner的拖拖拉拉导致很多的文件都没有被及时关闭，可能会因为不能正常打开文件而出现程序错误。

及时执行Finalizer和Cleaner是垃圾回收的一个主要的功能，但其具体实现在不同的JVM里不同。一个需要Finalizer和Cleaner方法及时执行的程序就会在不同的JVM里表现不同。这就可能出现这样一种情况：一个程序在你测试使用的JVM上运行非常完美，但是在你的重要的客户的机器上却完全无法运行。

> Tardy finalization is not just a theoretical problem. Providing a finalizer for a class can arbitrarily delay reclamation of its instances. A colleague debugged a long-running GUI application that was mysteriously dying with an OutOfMemoryError. Analysis revealed that at the time of its death, the application had thousands of graphics objects on its finalizer queue just waiting to be finalized and reclaimed. Unfortunately, the finalizer thread was running at a lower priority than another application thread, so objects weren’t getting finalized at the rate they became eligible for finalization. The language specification makes no guarantees as to which thread will execute finalizers, so there is no portable way to prevent this sort of problem other than to refrain from using finalizers. Cleaners are a bit better than finalizers in this regard because class authors have control over their own cleaner threads, but cleaners still run in the background, under the control of the garbage collector, so there can be no guarantee of prompt cleaning.
>
> Not only does the specification provide no guarantee that finalizers or cleaners will run promptly; it provides no guarantee that they’ll run at all. It is entirely possible, even likely, that a program terminates without running them on some objects that are no longer reachable. As a consequence, you should **never depend on a finalizer or cleaner to update persistent state.** For example, depending on a finalizer or cleaner to release a persistent lock on a shared resource such as a database is a good way to bring your entire distributed system to a grinding halt.

Finalizer的执行会被延迟，不仅仅是一个理论上的问题。一个重写了finalize方法的对象的回收过程，也会相应地被延迟任意长时间（因为重写了finalize方法的对象，需要等待Finalizer执行完才能进行回收）。一个同事最近在调试一个运行了很久的GUI应用程序，程序总是出现莫名其妙的OutOfMemoryError，然后死掉。通过分析发现，在程序死掉的时候，程序里有成千上万个图形对象在Finalizer队列里等待被执行finalize以及被清除。不幸的是，Finalizer线程的优先级比应用线程低得多，因此对象执行finalize的速度赶不上对象进入Finalizer队列的速度。语言规范也没有保证哪个线程一定会执行Finalizer，因此除了避免使用Finalizer外，没有更好的办法可以避免这个问题。在这种情况下，Cleaner比Finalizer稍微好一些，因为程序员自己可以控制cleaner线程，但是清除方法仍然在后台运行，受垃圾回收器的控制，因此也不能保证及时被清除。

这个规范不仅仅没有保证Finalizer和Cleaner能被及时执行，甚至都没有保证它们会被执行。很有可能出现这样的情况：程序结束了，但一些已经无法访问了的对象的终结方法没有被执行。因此，**永远不要依赖Finalizer或者Cleaner去更新重要的状态**。比如，如果依赖一个Finalizer或者Cleaner释放一个共享资源（比如数据库）上的永久锁，会很容易让整个分布式一同崩溃掉。

> Don’t be seduced by the methods System.gc and System.runFinalization. They may increase the odds of finalizers or cleaners getting executed, but they don’t guarantee it. Two methods once claimed to make this guarantee: System.runFinalizersOnExit and its evil twin, Runtime.runFinalizersOnExit. These methods are fatally flawed and have been deprecated for decades [ThreadStop].
>
> Another problem with finalizers is that an uncaught exception thrown during finalization is ignored, and finalization of that object terminates [JLS, 12.6]. Uncaught exceptions can leave other objects in a corrupt state. If another thread attempts to use such a corrupted object, arbitrary nondeterministic behavior may result. Normally, an uncaught exception will terminate the thread and print a stack trace, but not if it occurs in a finalizer—it won’t even print a warning. Cleaners do not have this problem because a library using a cleaner has control over its thread.

不要被System.gc 和 System.runFinalization方法迷惑了，他们只能增加Finalizer和Cleaner被执行的概率，也无法保证一定会执行。有两个方法声称可以保证Finalizer和Cleaner被执行，一个是System.runFinalizersOnExit ，另一个是他臭名昭著的孪生兄弟Runtime.runFinalizersOnExit，这两个方法有致命的缺陷，已经被废弃很久了（deprecated)。

Finalizer的另外一个问题是，如果在执行finalize方法时抛出异常，这个异常会被忽略，然后finalize方法也会终止。没有捕获的异常会让一些对象处在损坏（corrupt)的状态。如果另外一个线程又企图使用这个损坏的对象，就会出现不可预测的结果。一般情况下，一个没有被捕获的异常会使得线程终止，然后打印异常堆栈，但是在Finalizer线程里，不会这样，甚至连异常都不会打印。Cleaner就不会有这种问题，因为使用Cleaner的类库会控制自己的线程。

> **There is a** **severe** **performance penalty for using finalizers and cleaners.** On my machine, the time to create a simple AutoCloseable object, to close it using try-with-resources, and to have the garbage collector reclaim it is about 12 ns. Using a finalizer instead increases the time to 550 ns. In other words, it is about 50 times slower to create and destroy objects with finalizers. This is primarily because finalizers inhibit efficient garbage collection. Cleaners are comparable in speed to finalizers if you use them to clean all instances of the class (about 500 ns per instance on my machine), but cleaners are much faster if you use them only as a safety net, as discussed below. Under these circumstances, creating, cleaning, and destroying an object takes about 66 ns on my machine, which means you pay a factor of five (not fifty) for the insurance of a safety net *if* you don’t use it.

**使用Finalizer和Cleaner还有一个严重的性能问题**。在作者的机器上，常见一个简单的AutoCloseable的对象，使用try-with-resources来关闭，然后进行垃圾回收，花费的时间为12ns。使用Finalizer花费的时间增加到了550ns。也就是说，创建和销毁一个Finalizer对象使得速度慢了近50倍。这主要是因为Finalizer抑制了有效的垃圾回收（*在正式垃圾回收前，先要执行finalize方法*）。如果你使用Cleaner来清除类的所有的实例的话，它的速度和Finalizer差不多（在作者的机器上需要500ns），但是如果Cleaner只是作为后面说到的安全网就会快得多。在前面的情景下，在作者的机器上，创建、清除、销毁一个对象花费了66ns，也就意味着你如果没有使用它（这个它是指什么？？？）,为了保证安全网，将花费5倍的代价。

> **Finalizers have a serious security problem: they open your class up to finalizer attacks.**The idea behind a finalizer attack is simple: If an exception is thrown from a constructor or its serialization equivalents—the readObject and readResolve methods (Chapter 12)—the finalizer of a malicious subclass can run on the partially constructed object that should have “died on the vine.” This finalizer can record a reference to the object in a static field, preventing it from being garbage collected. Once the malformed object has been recorded, it is a simple matter to invoke arbitrary methods on this object that should never have been allowed to exist in the first place.**Throwing an exception from a constructor should be sufficient to prevent an object from coming into existence; in the presence of finalizers, it is not.**Such attacks can have dire consequences. Final classes are immune to finalizer attacks because no one can write a malicious subclass of a final class.**To protect nonfinal classes from finalizer attacks, write a final finalize method that does nothing.**

**Finalizer还有一个严重的安全问题：为Finalizer攻击打开了类的大门**。Finalizer攻击的原理很简单：如果构造器或者序列化对等体（readObject和readResolve方法）抛出异常，恶意子类可以在构造了一半，夭折了的对象上运行finalize方法，这个finalize方法可以把这个对象的引用记录在一个静态域里，使得这个对象无法被垃圾回收掉。一旦这个畸形的对象被记录下来，就可以很简单的去调用这个本来就不该存在的对象的方法，**一般来说，抛出异常就足够防止这个对象继续存在了，但是有了Finalizer后，这一点就做不到了**。这种攻击可能造成致命的结果。final类不会受到Finalizer攻击，因为没有人可以写出一个final类的恶意子类。**对于非final类来说，可以写一个啥也不做的final的finalize方法，来防止Finalizer攻击。**

> So what should you do instead of writing a finalizer or cleaner for a class whose objects encapsulate resources that require termination, such as files or threads? Just **have your class implement** **AutoCloseable****,** and require its clients to invoke the close method on each instance when it is no longer needed, typically using try-with-resources to ensure termination even in the face of exceptions (Item 9). One detail worth mentioning is that the instance must keep track of whether it has been closed: the close method must record in a field that the object is no longer valid, and other methods must check this field and throw an IllegalStateException if they are called after the object has been closed.

对于那些确实包含需要手动结束的资源（比如文件或者线程）的类，我们应该采用什么方法来替代Finalizer或者Cleaner呢？**让这些类实现AutoCloseable接口**，然后客户端在不需要这些类的实例的时候，调用close方法，手动关闭这些类，通常使用try-with-resources来确保即使抛异常也能关闭资源。一个需要注意的细节是：这个实例必须要记录自己是否已经被关闭了：必须在一个域里记录这个对象已经不可用了，其他的方法也需要检测这个域，当这个对象已经被关闭后，需要抛出IllegalStateException。

> So what, if anything, are cleaners and finalizers good for? They have perhaps two legitimate uses. One is to act as a safety net in case the owner of a resource neglects to call its close method. While there’s no guarantee that the cleaner or finalizer will run promptly (or at all), it is better to free the resource late than never if the client fails to do so. If you’re considering writing such a safety-net finalizer, think long and hard about whether the protection is worth the cost. Some Java library classes, such as FileInputStream, FileOutputStream, ThreadPoolExecutor, and java.sql.Connection, have finalizers that serve as safety nets.
>
> A second legitimate use of cleaners concerns objects with *native peers*. A native peer is a native (non-Java) object to which a normal object delegates via native methods. Because a native peer is not a normal object, the garbage collector doesn’t know about it and can’t reclaim it when its Java peer is reclaimed. A cleaner or finalizer may be an appropriate vehicle for this task, assuming the performance is acceptable and the native peer holds no critical resources. If the performance is unacceptable or the native peer holds resources that must be reclaimed promptly, the class should have a close method, as described earlier.

那么Finalizer和Cleaner有什么好处呢？它们大概有两个合理的用途。第一个就是作为安全网（safety net）使用，以防使用资源的人忘记了去调用close方法。虽然不能保证Finalizer和Cleaner会及时地调用，但总比客户端无法正常结束的好。当你想写一个Finalizer作为安全网的时候，要好好考虑清楚为了这种保护而付出的代价值不值。在Java的类库里，有一些类，比如FileInputStream, FileOutputStream, ThreadPoolExecutor, 和java.sql.Connection,都使用了Finalizer作为安全网。

Cleaner第二个合理的用法与本地对等体（native peers）有关。本地对等体是一个本地对象（不是java对象），普通对象通过本地方法代理给本地对象。因为本地对等体不是一个普通的对象，所以垃圾回收器并不知道它，也无法在其对应的java对等体（java peer）被回收以后，将其回收。如果本地对等体不包含关键的资源，并且性能上可以接受，那么Finalizer或者Cleaner就是完成这项任务最适合的工具。但是如果这个本地对等体包含必须被及时回收的资源，或者性能要求跟高，那么这个类就必须像前面介绍的那样，有一个close方法。

> Cleaners are a bit tricky to use. Below is a simple Room class demonstrating the facility. Let’s assume that rooms must be cleaned before they are reclaimed. The Room class implements AutoCloseable; the fact that its automatic cleaning safety net uses a cleaner is merely an implementation detail. Unlike finalizers, cleaners do not pollute a class’s public API:

Cleaner使用起来有点复杂。下面就以一个简单的Room类为例示范一下。让我们假设房间在回收之前必须打扫干净。这个Room类实现了AutoCloseable接口，使用cleaner实现自动清除安全网只是一个实现细节。和Finalizer不一样，Cleaner不会污染这个类的公有API。

```java
// An autocloseable class using a cleaner as a safety net
public class Room implements AutoCloseable {
    private static final Cleaner cleaner = Cleaner.create();
    // Resource that requires cleaning. Must not refer to Room!
    private static class State implements Runnable {
        int numJunkPiles; // Number of junk piles in this room
        State(int numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }// Invoked by close method or cleaner
        @Override 
        public void run() {
            System.out.println("Cleaning room");
            numJunkPiles = 0;
        }
    } // The state of this room, shared with our cleanable
    private final State state;
    // Our cleanable. Cleans the room when it’s eligible for gc
    private final Cleaner.Cleanable cleanable;
    public Room(int numJunkPiles) {
        state = new State(numJunkPiles);
        cleanable = cleaner.register(this, state);
    } 
    @Override 
    public void close() {
        cleanable.clean();
    }
}
```

> The static nested State class holds the resources that are required by the cleaner to clean the room. In this case, it is simply the numJunkPiles field, which represents the amount of mess in the room. More realistically, it might be a final long that contains a pointer to a native peer. State implements Runnable, and its run method is called at most once, by the Cleanable that we get when we register our State instance with our cleaner in the Room constructor. The call to the run method will be triggered by one of two things: Usually it is triggered by a call to Room’s close method calling Cleanable’s clean method. If the client fails to call the close method by the time a Room instance is eligible for garbage collection, the cleaner will (hopefully) call State’s run method.

静态嵌套类State包含需要被cleaner清除的资源，在这个例子里，资源就是numJunkPiles域，numJunkPiles域代表房间的混乱程度。更确切地说，他可以是一个final的long，包含一个纸箱本地对等体的指针。State实现了Runnable接口，他的run方法最多被调用一次：通过在Room的构造器里将state实例注册到cleaner里后获得的cleanable来调用。run方法可能会在一下两种情况下被触发：第一种是客户端调用Room的close方法从而调用cleanable的clean方法。第二种是当客户端调用close方法失败，但Room实例有可以被垃圾回收时，cleaner可能会调用State的run方法。

> It is critical that a State instance does not refer to its Room instance. If it did, it would create a circularity that would prevent the Room instance from becoming eligible for garbage collection (and from being automatically cleaned). Therefore, State must be a *static* nested class because nonstatic nested classes contain references to their enclosing instances (Item 24). It is similarly inadvisable to use a lambda because they can easily capture references to enclosing objects.
>
> As we said earlier, Room’s cleaner is used only as a safety net. If clients surround all Room instantiations in try-with-resource blocks, automatic cleaning will never be required. This well-behaved client demonstrates that behavior:

有个很关键的点是State实例并没有引用Room实例。如果它引用了就将出现循环引用，从而导致room实例无法被垃圾回收（同时也就无法自动清除）。因此，State必须是静态嵌套类，因为非静态嵌套类包含对外部类的引用（Item24）。因此也不建议使用Lambda，因为它们很容易拥有对外部对象的引用。

正如我们前面说到的那样，Room类里的Cleaner只是用作安全网的，如果客户端将所有的Room实例都包含在try-with-resources块里，那么自动清除的功能就永远都不需要了。一个比较好的客户端示例如下：

```java
public class Adult {
       public static void main(String[] args) {
           try (Room myRoom = new Room(7)) {
               System.out.println("Goodbye");
					 } 
       }
}
```

> As you’d expect, running the Adult program prints Goodbye, followed by Cleaning room. But what about this ill-behaved program, which never cleans its room?

正如你所期待的那样，运行这个Adult程序，在输出”GoodBye“的后面紧跟这“Cleaning Room”。但是在下面这个表现糟糕的程序里会是怎么样呢？会不会永远都不清理房间呢？

```java
public class Teenager {
       public static void main(String[] args) {
           new Room(99);
           System.out.println("Peace out");
       }
}
```

> You might expect it to print Peace out, followed by Cleaning room, but on my machine, it never prints Cleaning room; it just exits. This is the unpredictability we spoke of earlier. The Cleaner spec says, “The behavior of cleaners during System.exit is implementation specific. No guarantees are made relating to whether cleaning actions are invoked or not.” While the spec does not say it, the same holds true for normal program exit. On my machine, adding the line System.gc() to Teenager’s main method is enough to make it print Cleaning room prior to exit, but there’s no guarantee that you’ll see the same behavior on your machine.
>
> In summary, don’t use cleaners, or in releases prior to Java 9, finalizers, except as a safety net or to terminate noncritical native resources. Even then, beware the indeterminacy and performance consequences.

你可能会期待输出”Peace out“ ，然后紧跟着”Cleaning Room“ 。但是在作者的机器上，它从来都没有打印过”Cleaning Room",就退出了。这就是我们前面说的不可预测性。Cleaner规范指出：“cleaner在System.exir期间的行为和具体的实现相关，并不保证清除方法是否会被调用。”虽然规范里并没有说，但在正常的程序退出中，就是不会答应“Cleaning Room"。在作者的机器上，在Teenager的main方法里添加依据System.gc()就可以在程序退出前打印”Cleaning Room“,但是不能保证在你的机器上也是这样的。

总的来说，除了作为安全网或者清除非关键的本地对等体以外，不要使用Cleaner或者在Java9发行之前的Finalizer。如果非要用，则要注意它们的不确定性和性能问题。

### 