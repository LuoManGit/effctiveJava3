### Item72 优先使用标准的异常

> An attribute that distinguishes expert programmers from less experienced ones is that experts strive for and usually achieve a high degree of code reuse. Exceptions are no exception to the rule that code reuse is a good thing. The Java libraries provide a set of exceptions that covers most of the exception-throwing needs of most APIs.

专家和经验少的程序员之间的差别主要在于专家都致力于高度的代码可用，一般情况确实可以做到。代码复用是一件好事，异常也不例外。Java类库提供了一组异常，覆盖了大部分的需要抛出异常的API的需求。

> Reusing standard exceptions has several benefits. Chief among them is that it makes your API easier to learn and use because it matches the established conventions that programmers are already familiar with. A close second is that programs using your API are easier to read because they aren’t cluttered with unfamiliar exceptions. Last (and least), fewer exception classes means a smaller memory footprint and less time spent loading classes.

重用标准的异常有几个好处。其中最大的好处是，它会使得你的API学习和使用起来更加简单，因为它满足了程序员早就熟悉的建立好的约定。第二大好处是使用你的API的程序可读性会更高，因为它们不会出现一些陌生的异常。最后的（也是最不重要的）好处是，写的异常类越少，意味着在加载类的时候花费的内存大小和时间就越少。

> The most commonly reused exception type is IllegalArgumentException (Item 49). This is generally the exception to throw when the caller passes in an argument whose value is inappropriate. For example, this would be the exception to throw if the caller passed a negative number in a parameter representing the number of times some action was to be repeated.

最常见的重用异常类型是IllegalArgumentException（Item49）。当调用者传入的参数的值不合适的时候，通常会抛出这个异常。比如，如果调用者给一个带边某个操作执行多少次的参数传入一个负数的话，就应该抛出这个异常。

> Another commonly reused exception is IllegalStateException. This is generally the exception to throw if the invocation is illegal because of the state of the receiving object. For example, this would be the exception to throw if the caller attempted to use some object before it had been properly initialized.

另外一个常见的重用异常类型是IllegalStateException。当因为接受对象的状态问题导致调用非法的时候，通常会抛出这个异常。比如，当调用者企图去使用一些还没有初始化的对象的时候，就应该抛出这个异常。

> Arguably, every erroneous method invocation boils down to an illegal argument or state, but other exceptions are standardly used for certain kinds of illegal arguments and states. If a caller passes null in some parameter for which null values are prohibited, convention dictates that NullPointerException be thrown rather than IllegalArgumentException. Similarly, if a caller passes an out-of-range value in a parameter representing an index into a sequence, IndexOutOfBoundsException should be thrown rather than IllegalArgumentException.

可以说，所有错误的方法调用都可以归咎于非法的参数或者状态，但是有一些其他的标准异常用在一些特定的非法参数和状态中。如果调用者给一个不能为null的参数传递了一个null，按照约定就应该抛出NullPointerException而不是IllegalArgumentException。同样地，如果调用者给一个代表序列的索引的参数，传了一个超出范围的值，应该抛出IndexOutOfBoundsException而不是IllegalArgumentException。

> Another reusable exception is ConcurrentModificationException. It should be thrown if an object that was designed for use by a single thread (or with external synchronization) detects that it is being modified concurrently. This exception is at best a hint because it is impossible to reliably detect concurrent modification.

另一个值得重用的异常是ConcurrentModificationException。当一个设计只能用于单个线程的对象，或者只能与外部同步机制一起使用的对象，被检测到被并发的修改的时候，就应该抛出这个异常。这个异常顶多就是一个提示，因为不可能可靠地检测到并发修改。

> A last standard exception of note is UnsupportedOperationException. This is the exception to throw if an object does not support an attempted operation. Its use is rare because most objects support all of their methods. This exception is used by classes that fail to implement one or more *optional operations* defined by an interface they implement. For example, an append-only List implementation would throw this exception if someone tried to delete an element from the list.

最后一个需要注意的标准异常是UnsupportedOperationException。当一个对象不支持请求的方法的时候，就抛出这个异常。它比较少使用，因为大部分的对象都支持自己的方法，这个异常主要用在一些没有很好实现接口所定义的一个或者多个可选操作的类里。比如，一个只支持追加操作的列表，当有人企图少出列表中的元素的时候，就可以抛出这个异常。

> **Do** **not** **reuse** **Exception, RuntimeException, Throwable, or Error directly.** Treat these classes as if they were abstract. You can't reliably test for these exceptions because they are superclasses of other exceptions that a method may throw.
>
> This table summarizes the most commonly reused exceptions:

不要直接重用Exception, RuntimeException, Throwable, 或者Error。把这些类当做是抽象类，你不能直接可靠地测试这些异常，因为它们是一个方法可能抛出的其他异常的父类。

下面这个表格中总结了最常见的可重用的异常：

| 异常                            | 使用场合                             |
| ------------------------------- | ------------------------------------ |
| IllegalArgumentException        | 不合适的非空的参数                   |
| IllegalStateException           | 方法调用中的对象状态不合适           |
| NullPointerException            | 在禁止为null的参数为null             |
| IndexOutOfBoundsException       | 索引参数超出范围                     |
| ConcurrentModificationException | 不允许并发修改的对象，检测到并发修改 |
| UnsupportedOperationException   | 对象不支持这个方法                   |

> While these are by far the most commonly reused exceptions, others may be reused where circumstances warrant. For example, it would be appropriate to reuse ArithmeticException and NumberFormatException if you were implementing arithmetic objects such as complex numbers or rational numbers. If an exception fits your needs, go ahead and use it, but only if the conditions under which you would throw it are consistent with the exception’s documentation: reuse must be based on documented semantics, not just on name. Also, feel free to subclass a standard exception if you want to add more detail (Item 75), but remember that exceptions are serializable (Chapter 12). That alone is reason not to write your own exception class without good reason.

虽然这些是最常重用的异常，但是其他异常在适合的情况下也能重用。比如，当你在实现一些算术对象，比如复数和有理数的时候，你就可以重用ArithmeticException和NumberFormatException。如果一个异常满足你的要求，并且你抛出这个异常的条件和这个异常的文档中说明的条件一致的话，就可以直接使用这个异常。注意的是，重用对象必须基于文档的含义，而不是名字含义。并且，如果你想添加更多的细节的话，你可以放心地对标准异常进行子类化，但是记住异常时可以序列化的（第12章）。这也正是如果没有正当理由，不要编写自己的异常类的原因。

> Choosing which exception to reuse can be tricky because the “occasions for use” in the table above do not appear to be mutually exclusive. Consider the case of an object representing a deck of cards, and suppose there were a method to deal a hand from the deck that took as an argument the size of the hand. If the caller passed a value larger than the number of cards remaining in the deck, it could be construed as an IllegalArgumentException (the handSize parameter value is too high) or an IllegalStateException (the deck contains too few cards). Under these circumstances, the rule is to **throw** **IllegalStateException** **if no argument values would have worked, otherwise throw** **IllegalArgumentException.**

选择重用哪个异常是很复杂的，因为上面的表格中的“使用场景”也并不是完全互斥的。比如，有一个对象带边一组纸牌，假如有一个方法发牌，有一个参数表示一手牌的个数，假如这个调用者传入的参数比这副牌剩余的数量大，那么它就可以被解释为一个IllegalArgumentException（因为参数值太大了），或者是一个IllegalStateException（因为这幅牌的数量太少了）。在这种情况下，通用规则是：**如果没有相关的参数值，就抛出IllegalStateException，否则就抛出IllegalArgumentException。**