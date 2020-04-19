### Item21 为后代设计接口

> Prior to Java 8, it was impossible to add methods to interfaces without breaking existing implementations. If you added a new method to an interface, existing implementations would, in general, lack the method, resulting in a compile-time error. In Java 8, the *default method* construct was added [JLS 9.4], with the intent of allowing the addition of methods to existing interfaces. But adding new methods to existing interfaces is fraught with risk.

在Java8之前的版本中，要为已经存在的接口添加方法，又不破坏现有的实现，是不可能的。如果你让接口中添加了一个新的方法，一般来说，已经存在的实现中是不存在这个方法的，就会出现编译问题。在Java8中，增加了默认方法（default method）构造，目的就是要允许给已经存在的接口添加方法。但是，往已经存在的接口中添加实现还是要冒风险的。

> The declaration for a default method includes a *default implementation* that is used by all classes that implement the interface but do not implement the default method. While the addition of default methods to Java makes it possible to add methods to an existing interface, there is no guarantee that these methods will work in all preexisting implementations. Default methods are “injected” into existing implementations without the knowledge or consent of their implementors. Before Java 8, these implementations were written with the tacit understanding that their interfaces would *never* acquire any new methods.

默认方法的实现中包含一个默认实现，那些实现了这个接口的类，不用实现这个默认方法，也可以使用这些默认实现。在Java中，默认方法的添加使得给已存在的接口添加方法成为可能，但是不能保证这些方法可以在所有的已经存在的实现中正常工作。因为这些方法是直接被注入到这些已经存在实现中的，它们的实现者并不知道，也没有许可。在Java8之前，编写这些实例的时候，是默认他们的接口不会添加任何新的方法的。

> Many new default methods were added to the core collection interfaces in Java 8, primarily to facilitate the use of lambdas (Chapter 6). The Java libraries’ default methods are high-quality general-purpose implementations, and in most cases, they work fine. But **it is not always possible to write a default method that maintains all invariants of every conceivable implementation.**

在Java8中，为了使lamdba（第6章）的使用更加便利，给核心的集合接口添加了很多新的默认方法。在java类库中的默认方法都是高质量的通用实现，在大部分情况下，它们都能很好的工作。但是**并不是总能编写一个默认方法，能够覆盖其所有可以想到的实现的变体**。

> For example, consider the removeIf method, which was added to the Collection interface in Java 8. This method removes all elements for which a given boolean function (or *predicate*) returns true. The default implementation is specified to traverse the collection using its iterator, invoking the predicate on each element, and using the iterator’s remove method to remove the elements for which the predicate returns true. Presumably the declaration looks something like this:

比如在Java8中添加到集合接口中的removeIf方法。这个方法删除集合中所有“根据给定的boolean函数（或者断言）返回为true”的元素。其默认实现明确指出，它使用集合的迭代器来遍历集合，然后再每一个元素上调用断言，当断言返回true的时候，使用迭代器的remove方法来删除这个元素。其声明大致如下：

```java
// Default method added to the Collection interface in Java 8
   default boolean removeIf(Predicate<? super E> filter) {
       Objects.requireNonNull(filter);
       boolean result = false;
       for (Iterator<E> it = iterator(); it.hasNext(); ) {
           if (filter.test(it.next())) {
               it.remove();
               result = true;
           }
			}
       return result;
   }
```

> This is the best general-purpose implementation one could possibly write for the removeIf method, but sadly, it fails on some real-world Collection implementations. For example, consider org.apache.commons.collections4.collection.SynchronizedCollection. This class, from the Apache Commons library, is similar to the one returned by the static factory Collections.synchronizedCollection in java.util. The Apache version additionally provides the ability to use a client-supplied object for locking, in place of the collection. In other words, it is a wrapper class (Item 18), all of whose methods synchronize on a locking object before delegating to the wrapped collection.

这是一个可以为removeIf方法写得最佳的通用实现了，但是很遗憾地是，在现实中的一些集合实现上却不能正常工作。比如org.apache.commons.collections4.collection.SynchronizedCollection，这个类来自Apache的Commons类库，和java.util中的静态工厂方法Collections.synchronizedCollection 返回的对象类有点相似。Apache的版本添加了一个可以使用客户端提供的对象来锁定的功能。换句话说，这是一个包装类（Item18），所有的方法在委托给包装的集合之前，都先在锁定对象上进行同步。

> The Apache SynchronizedCollection class is still being actively maintained, but as of this writing, it does not override the removeIf method. If this class is used in conjunction with Java 8, it will therefore inherit the default implementation of removeIf, which does not, indeed *cannot*, maintain the class’s fundamental promise: to automatically synchronize around each method invocation. The default implementation knows nothing about synchronization and has no access to the field that contains the locking object. If a client calls the removeIf method on a SynchronizedCollection instance in the presence of concurrent modification of the collection by another thread, a ConcurrentModificationException or other unspecified behavior may result.

Apache的SynchronizedCollection类至今还在被维护。但是截止本书编写之前，这个类都还没有覆盖removeIf方法。如果这个类在在Java8环境下运行，它就会继承到removeIf方法的默认实现，但是这个实现没有，也不可能满足这个类的基本保证：为每一个方法提供自动的同步。但是默认的实现对于同步一无所知，也无法访问包换锁对象的域。如果客户端调用了一个SynchronizedCollection实例的removeIf方法，而另一个线程又并发地修改了这个集合，就会导致ConcurrentModificationException或者其他异常行为。

> In order to prevent this from happening in similar Java platform libraries implementations, such as the package-private class returned by Collections.synchronizedCollection, the JDK maintainers had to override the default removeIf implementation and other methods like it to perform the necessary synchronization before invoking the default implementation. Preexisting collection implementations that were not part of the Java platform did not have the opportunity to make analogous changes in lockstep with the interface change, and some have yet to do so.

为了防止类似的事情出现在Java平台类库的实现里，比如Collections.synchronizedCollection返回的包级私有类里，JDK的维护者必须要覆盖默认的removeIf方法，以及其他的类似需要在调用默认方法前执行必要同步的方法。而不属于Java平台的已经存在的类就没有机会和接口做同步的修改，有些现在已经修改了。

> **In the presence of default methods, existing implementations of an interface may compile without error or warning but fail at runtime.** While not terribly common, this problem is not an isolated incident either. A handful of the methods added to the collections interfaces in Java 8 are known to be susceptible, and a handful of existing implementations are known to be affected.

在使用默认方法的时候，虽然已经存在的接口的实现在编译的时候不会报错或者警告，但在运行的时候却会出错。虽然这个问题并不普遍，但也不是个例。在java8中，有一些被添加到集合中的方法被认为是容易受到感染的，也有一些已经存在的实现已经受到了影响。

> Using default methods to add new methods to existing interfaces should be avoided unless the need is critical, in which case you should think long and hard about whether an existing interface implementation might be broken by your default method implementation. Default methods are, however, extremely useful for providing standard method implementations when an interface is created, to ease the task of implementing the interface (Item 20).

除非确实非常需要，否则应该避免使用默认方法来给已经存在的接口添加新的方法。在添加的时候，也必须认真努力思考，是否有已存在的接口实现可能会被新增的默认方法实现破坏。然而，在创建接口的时候，为了是接口实现变得更容易，默认方法可以提供标准的方法实现，是非常有用的。

> It is also worth noting that default methods were not designed to support removing methods from interfaces or changing the signatures of existing methods. Neither of these interface changes is possible without breaking existing clients.

还需要注意的是，默认方法并不是设计来将方法从接口中删除，或者改变已经存在方法的结构的。在不破坏现有客户端的情况下，这两种接口改变都是不可能实现的。

> The moral is clear. Even though default methods are now a part of the Java platform, **it is still of the utmost importance to design interfaces with great care.** While default methods make it *possible* to add methods to existing interfaces, there is great risk in doing so. If an interface contains a minor flaw, it may irritate its users forever; if an interface is severely deficient, it may doom the API that contains it.

结论很明显：即使默认方法现在是Java平台中的一部分，**但是谨慎小心地设计接口仍然是至关重要的**。虽然默认方法可以给已存在的接口添加方法，但是这样做还是有很大的风险。就算一个接口只有一个小问题，也会让用户一直不愉快；假如接口有了严重的问题，就可能会摧毁包含它的API。

> Therefore, it is critically important to test each new interface before you release it. Multiple programmers should implement each interface in different ways. At a minimum, you should aim for three diverse implementations. Equally important is to write multiple client programs that use instances of each new interface to perform various tasks. This will go a long way toward ensuring that each interface satisfies all of its intended uses. These steps will allow you to discover flaws in interfaces before they are released, when you can still correct them easily. **While it may be possible to correct some interface flaws after an interface is released, you cannot count on it**.

因此，在你发布每个新的接口前，进行测试是至关重要的。多个程序员应该使用不同方式实现每一个接口，至少，应该有三种不同的实现方式。编写多个客户端程序来使用新接口的实例来执行不同的任务也是同等重要的。这些步骤都有助于保证每一个接口都满足其所有的预期的用法。这些方法帮助你在接口发布之前，发现问题，此时你还可以很容易地修改这些问题。**虽在接口发布后，也许还能改正问题，但是最好还是不要指望了**。

### 