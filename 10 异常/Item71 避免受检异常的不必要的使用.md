### Item71 避免受检异常的不必要的使用

> Many Java programmers dislike checked exceptions, but used properly, they can improve APIs and programs. Unlike return codes and unchecked exceptions, they *force* programmers to deal with problems, enhancing reliability. That said, overuse of checked exceptions in APIs can make them far less pleasant to use. If a method throws checked exceptions, the code that invokes it must handle them in one or more catch blocks, or declare that it throws them and let them propagate outward. Either way, it places a burden on the user of the API. The burden increased in Java 8, as methods throwing checked exceptions can’t be used directly in streams (Items 45–48).

很多的Java程序员不喜欢受检异常，但是如果使用得当的话，受检异常可以改善API和程序。不同于返回码和非受检异常，它强迫程序员必须要处理这些问题，增强了可靠性。也就是说，在API中，受检异常的过度使用，会使得API的使用不是很方便。如果一个方法抛出了受检异常，那么调用这个方法的代码就必须要在一个或者多个catch块里，处理这写受检异常，或者把受检异常抛出，让他们继续传播。无论是那种方法，这都加重了使用API的用户的负担。这个负担在Java8中变得更重了，因为在stream中无法直接直接将受检异常抛出（Item45-48)。

> This burden may be justified if the exceptional condition cannot be prevented by proper use of the API *and* the programmer using the API can take some useful action once confronted with the exception. Unless both of these conditions are met, an unchecked exception is appropriate. As a litmus test, ask yourself how the programmer will handle the exception. Is this the best that can be done?

如果正确使用API也不能阻止异常条件的出现，并且一旦出现异常，使用API的程序员也能采取有用的措施，那么这个负担就是应当的。如果这两个条件都不满足，那么就应该使用非受检异常。做一个测试，问问你自己，程序员将如何处理异常。下面这种做法是最好的吗？

```java
} catch (TheCheckedException e) {
       throw new AssertionError(); // Can't happen!
}
```

> Or this?

或者这个？

```java
} catch (TheCheckedException e) {
       e.printStackTrace();        // Oh well, we lose.
       System.exit(1);
}
```

> If the programmer can do no better, an unchecked exception is called for.

如果程序员不能有更好地处理异常的方式，那么就应该使用非受检异常。

> The additional burden on the programmer caused by a checked exception is substantially higher if it is the *sole* checked exception thrown by a method. If there are others, the method must already appear in a try block, and this exception requires, at most, another catch block. If a method throws a single checked exception, this exception is the sole reason the method must appear in a try block and can’t be used directly in streams. Under these circumstances, it pays to ask yourself if there is a way to avoid the checked exception.

如果一个方法抛出了一个唯一的受检异常，那么这个受检异常给程序员带来的额外负担就要更重一些。如果还有其他的异常，那么这个方法就本来就需要放在一个try块里，那么这个异常最多就需要新增一个catch块。如果这个方法只抛出了一个受检异常，那么这个异常就是这个方法必须出现在try块里的唯一的原因，也使得这个方法不能用在stream里（*在stream里会使用lambda表达式，lambda表达式里不能直接调用抛出受检异常的方法，必须写在try-catch块里，否则编译会报错*）。在这些情况下，你就需要问一下自己是不是有其他的方法来避免这个受检异常。

> The easiest way to eliminate a checked exception is to return an *optional* of the desired result type (Item 55). Instead of throwing a checked exception, the method simply returns an empty optional. The disadvantage of this technique is that the method can’t return any additional information detailing its inability to perform the desired computation. Exceptions, by contrast, have descriptive types, and can export methods to provide additional information (Item 70).

消除受检异常的最简单的方法是返回一个想要的结果类型的Optional（Item55）。这个方法会通过简单地返回一个空的optional来代替抛出受检异常。这种方法的缺点在于，这个方法不能返回任何的额外信息，来说明它为啥无法完成你想要的计算。同时，Exception还有具有描述能力的类型，并且还可以导出一些方法来提供更多的信息（Item70）。

> You can also turn a checked exception into an unchecked exception by breaking the method that throws the exception into two methods, the first of which returns a boolean indicating whether the exception would be thrown. This API refactoring transforms the calling sequence from this:

你也可以通过把这个抛出异常的方法分为两个方法，来把这个受检异常转换为非受检异常。第一个方法返回一个boolean来表示应该抛出异常。这种API重构，把如下所示的调用序列：

```java
// Invocation with checked exception
   try {
       obj.action(args);
   } catch (TheCheckedException e) {
       ... // Handle exceptional condition
}
```

> into this:

转换为这种：

```java
// Invocation with state-testing method and unchecked exception
   if (obj.actionPermitted(args)) {
       obj.action(args);
   } else {
       ... // Handle exceptional condition
}
```

> This refactoring is not always appropriate, but where it is, it can make an API more pleasant to use. While the latter calling sequence is no prettier than the former, the refactored API is more flexible. If the programmer knows the call will succeed, or is content to let the thread terminate if it fails, the refactoring also allows this trivial calling sequence:

这种重构方法有的时候不太合适，但是如果在合适的地方，它确实可以使得API使用起来更加舒服。虽然后面这个调用序列并没有比前面那个好看，但是重构后的API更加灵活一些。如果程序员知道这个调用一定会成功，或者当它失败的时候，允许线程终止，那么这个重构就可以使用下面这样简洁的调用序列：

```java
obj.action(args);
```

> If you suspect that the trivial calling sequence will be the norm, then the API refactoring may be appropriate. The resulting API is essentially the state-testing method API in Item 69 and the same caveats apply: if an object is to be accessed concurrently without external synchronization or it is subject to externally induced state transitions, this refactoring is inappropriate because the object’s state may change between the calls to actionPermitted and action. If a separate actionPermitted method would duplicate the work of the action method, the refactoring may be ruled out on performance grounds.

如果你怀疑这个简单的调用序列是否合适，那么这个API重构可能就是合适的。得到的API本质上和Item69里的状态检测方法一样，同样地，其告诫也是适用的：如果一个对象需要在不需要额外的同步的情况下并发访问，或者它的状态可以被外界改变，那么这个重构就是不合适的，因为这个对象的状态可能在actionPermitted和action的调用中间被改变。如果单独的actionPermitted方法重复了action方法的工作，从性能角度上来说，这个重构也是不合适的。

> In summary, when used sparingly, checked exceptions can increase the reliability of programs; when overused, they make APIs painful to use. If callers won’t be able to recover from failures, throw unchecked exceptions. If recovery may be possible and you want to *force* callers to handle exceptional conditions, first consider returning an optional. Only if this would provide insufficient information in the case of failure should you throw a checked exception.

总结一下，如果使用得当，受检异常可以增加程序的可读性；如果过度使用的话，会使得API使用起来很痛苦。如果调用中不能从失败中进行恢复的话，就抛出非受检异常。如果可以进行恢复，你也希望强迫调用者处理异常情况的话，首先考虑返回一个Optional。只有当失败时，需要提供额外的信息的情况，你才应该抛出受检异常。