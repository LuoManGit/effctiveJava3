## 10 异常

> **W**HEN used to best advantage, exceptions can improve a program’s readability, reliability, and maintainability. When used improperly, they can have the opposite effect. This chapter provides guidelines for using exceptions effectively.

使用时充分发挥异常的优点的话，可以提高程序的可读性，可靠性和可维护性。当使用不当的时候，可能会造成相反的结果。本章提供了一些高校使用异常的指导原则。

### Item69 只针对异常的情况使用异常

> Someday, if you are unlucky, you may stumble across a piece of code that looks something like this:

有时候，你运气特别差，可能会碰到这样子的代码：

```java
// Horrible abuse of exceptions. Don't ever do this!
try {
		int i = 0;
    while(true)
        range[i++].climb();
} catch (ArrayIndexOutOfBoundsException e) {
}
```

> What does this code do? It’s not at all obvious from inspection, and that’s reason enough not to use it (Item 67). It turns out to be a horribly ill-conceived idiom for looping through the elements of an array. The infinite loop terminates by throwing, catching, and ignoring an ArrayIndexOutOfBoundsException when it attempts to access the first array element outside the bounds of the array. It’s supposed to be equivalent to the standard idiom for looping through an array, which is instantly recognizable to any Java programmer:

这段代码在干嘛？看起来很不明显，这也是我们不要使用这种代码的原因。事实证明，这是一种循环编译数组中的元素的很糟糕的方法。当企图访问数组范围外的第一个元素的时候，无限循环会通过抛出，捕获，然后忽略ArrayIndexOutOfBoundsException来终止。它和使用for循环遍历数组的标准模式是等价的，而后者对于程序员来说，要更好识别一些：

```java
for (Mountain m : range)
       m.climb();
```

> So why would anyone use the exception-based loop in preference to the tried and true? It’s a misguided attempt to improve performance based on the faulty reasoning that, since the VM checks the bounds of all array accesses, the normal loop termination test—hidden by the compiler but still present in the for-each loop—is redundant and should be avoided. There are three things wrong with this reasoning:

那么，为什么还会有人用这种基于异常的循环，而不是被证明有效的方法呢？这是一种被误导了的做法，企图根据错误的依据来提高性能，其依据是这样的：因为VM在每次访问数组的时候都会检测数组的边界，因此他们认为普通的循环终止测试是多余的，应该避免。循环终止测试在for-each循环中被编译器隐藏了，但仍然确实存在。这个依据有三点错误如下：

> - Because exceptions are designed for exceptional circumstances, there is little incentive for JVM implementors to make them as fast as explicit tests.
> - Placing code inside a try-catch block inhibits certain optimizations that JVM implementations might otherwise perform.
> - The standard idiom for looping through an array doesn’t necessarily result in redundant checks. Many JVM implementations optimize them away.

- 因为异常只是为异常的情况设计的，因此JVM实现者很难有心思把他们做得像显式的测试那么快。
- 把这段代码放在一个try-catch块里，妨碍了JVM实现可能对其进行其他方面的优化。
- 对数组进行标准的for循环遍历，并不一定会生成多余的检测，因为很多JVM实现会将它优化掉。

> In fact, the exception-based idiom is far slower than the standard one. On my machine, the exception-based idiom is about twice as slow as the standard one for arrays of one hundred elements.

事实上，基于异常的模式要比标准模式慢得多，在作者的机器上，对于一个100个元素的数组，基于异常的模式比标准模式慢了2倍多。

> Not only does the exception-based loop obfuscate the purpose of the code and reduce its performance, but it’s not guaranteed to work. If there is a bug in the loop, the use of exceptions for flow control can mask the bug, greatly complicating the debugging process. Suppose the computation in the body of the loop invokes a method that performs an out-of-bounds access to some unrelated array. If a reasonable loop idiom were used, the bug would generate an uncaught exception, resulting in immediate thread termination with a full stack trace. If the misguided exception-based loop were used, the bug-related exception would be caught and misinterpreted as a normal loop termination.

这种基于异常的循环不仅仅让人搞不清楚它的意图以及会降低性能，而且还不能保证正常工作。如果for循环中有一个bug，在控制流里面使用的异常可能会掩盖掉这个bug，使得debug的过程变得格外地困难。假如这个循环体中的计算调用了一个方法，这个方法又会会执行一个对其他不相关数组的越界访问。如果使用的是一个合理的循环模式，这个bug就会生成一个未捕获的异常，导致这个线程就会带着所有的栈轨迹立即终止。如果是使用的这种基于异常的循环，这个bug的相关异常可能会被捕获，然后被误认为是一个正常的循环终止。

> The moral of this story is simple: **Exceptions are, as their name implies, to be used only for exceptional conditions; they should never be used for ordinary control flow.** More generally, use standard, easily recognizable idioms in preference to overly clever techniques that purport to offer better performance. Even if the performance advantage is real, it may not remain in the face of steadily improving platform implementations. The subtle bugs and maintenance headaches that come from overly clever techniques, however, are sure to remain.

这个故事的教训很简单：**异常，就像他们的名字所说的那样，只能被用在异常的情况下，绝不应该用作普通的流控制。**更加通俗地说，使用简单容易识别的模式，而不是使用哪些企图提供更好的性能的聪明过头的技术。即使其性能优势是真的，但是随着平台实现的不断升级，这种优势可能会没有了。但是来自这种技术的细微的bug以及难以维护的问题，却会一直存在。

> This principle also has implications for API design. **A well-designed API must not force its clients to use exceptions for ordinary control flow.** A class with a “state-dependent” method that can be invoked only under certain unpredictable conditions should generally have a separate “state-testing” method indicating whether it is appropriate to invoke the state-dependent method. For example, the Iterator interface has the state-dependent method next and the corresponding state-testing method hasNext. This enables the standard idiom for iterating over a collection with a traditional for loop (as well as the for-each loop, where the hasNext method is used internally):

这个原则对于API设计也有一些指示。**一个设计良好的API不应该强迫其客户端使用异常来做正常的流控制。一个包含“依赖状态”的方法的类，依赖状态的方法即只能在某些特定的不可预测的环境下调用的方法，通常需要提供一个单独的“状态检测”方法，来表明掉用这个依赖状态的方法是否合适。比如，Iterator接口有一个依赖状态的方法next和一个相关的状态检测方法hasNext。这样就可以使用传统的for循环来对集合进行标准的遍历（也可以使用for-each循环，其内部也使用了hasNext方法）:

```java
for (Iterator<Foo> i = collection.iterator(); i.hasNext(); ) {
       Foo foo = i.next();
       ...
}
```

> If Iterator lacked the hasNext method, clients would be forced to do this instead:

如果Iterator没有hasNext方法的话，客户端就只能这样去做了：

```java
// Do not use this hideous code for iteration over a collection!
   try {
       Iterator<Foo> i = collection.iterator();
       while(true) {
           Foo foo = i.next();
           ... 
       }
   } catch (NoSuchElementException e) {
   }
```

> This should look very familiar after the array iteration example that began this item. In addition to being wordy and misleading, the exception-based loop is likely to perform poorly and can mask bugs in unrelated parts of the system.

这个看起来和本节开头中的数组迭代的例子很像。除了代码啰嗦让人误解以外，这种基于异常的循环的性能也差，还可能覆盖掉系统中其他不相关部分的bug。

> An alternative to providing a separate state-testing method is to have the state-dependent method return an empty optional (Item 55) or a distinguished value such as null if it cannot perform the desired computation.

除了提供状态检测的方法外，还有一种可替换的方法，如果这个状态依赖的方法不能执行想要的计算，那么可以返回一个空的optional（Item55）或者一个不同的值，比如null。

> Here are some guidelines to help you choose between a state-testing method and an optional or distinguished return value. If an object is to be accessed concurrently without external synchronization or is subject to externally induced state transitions, you must use an optional or distinguished return value, as the object’s state could change in the interval between the invocation of a state-testing method and its state-dependent method. Performance concerns may dictate that an optional or distinguished return value be used if a separate state-testing method would duplicate the work of the state-dependent method. All other things being equal, a state-testing method is mildly preferable to a distinguished return value. It offers slightly better readability, and incorrect use may be easier to detect: if you forget to call a state-testing method, the state-dependent method will throw an exception, making the bug obvious; if you forget to check for a distinguished return value, the bug may be subtle. This is not an issue for optional return values.

这里有一些指导方针，帮助你在状态检测方法，返回Optional或者其他不同的值之间做选择。如果一个对象需要在没有额外的同步下并发访问，或者其状态可以被外界改变，那么你就必须使用返回optional或者其他不同的值，因为在调用状态检测方法和依赖状态的方法之间，这个对象的状态可能会改变。在对性能比较关心的情况下，如果这个状态检测方法会重复状态依赖的方法的工作，那么也优先选择返回optional或者其他不同的值。如果其他的条件都相同，那么状态检测方法稍微比返回其他不同值优先一些。状态检测方法可以提供更高的可读性，使用不正确也很容易发现：比如如果你忘记了调用状态检测方法，那么依赖状态的方法就会抛出异常，这个bug就会很明显；如果你忘记了检测这个其他的不同的返回值，这个bug就很难发现。Optional返回值没有这些问题。

> In summary, exceptions are designed for exceptional conditions. Don’t use them for ordinary control flow, and don’t write APIs that force others to do so.

总结一下，异常是专门为异常情况设计的。不要用来做普通的流控制，也不要编写强迫别人这样做的API。