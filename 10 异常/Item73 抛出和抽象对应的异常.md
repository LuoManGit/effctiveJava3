### Item73 抛出和抽象对应的异常

> It is disconcerting when a method throws an exception that has no apparent connection to the task that it performs. This often happens when a method propagates an exception thrown by a lower-level abstraction. Not only is it disconcerting, but it pollutes the API of the higher layer with implementation details. If the implementation of the higher layer changes in a later release, the exceptions it throws will change too, potentially breaking existing client programs.

如果一个方法抛出的异常和它自己执行的任务没什么关联，就会让人很迷惑。当一个方法传递了底层抽象的异常的时候，就会发生这样的情况。这不仅仅会让人迷惑，还会污染具有具体实现的高层API。如果在后面的版本中这个高层的实现被修改了，这个抛出的异常也会发生改变，可能会破坏已经存在的客户端程序。

> To avoid this problem, **higher layers should catch lower-level exceptions and, in their place, throw exceptions that can be explained in terms of the higher-level abstraction.** This idiom is known as *exception translation*:

为了避免这个问题，**高层应该捕获底层的异常，然后抛出可以按照高层抽象解释的异常**。这种方式被称为“异常转译”，代码如下：

```java
// Exception Translation
   try {
       ... // Use lower-level abstraction to do our bidding
   } catch (LowerLevelException e) {
       throw new HigherLevelException(...);
}
```

> Here is an example of exception translation taken from the AbstractSequentialList class, which is a *skeletal implementation* (Item 20) of the List interface. In this example, exception translation is mandated by the specification of the get method in the List<E> interface:

下面是一个来自AbstractSequentialList类的异常转译的例子，AbstractSequentialList类是List接口的骨架实现（Item20）。在这个例子中，根据List接口的规范要求，get方法需要使用异常转译，代码如下：

```java
/**
* Returns the element at the specified position in this list. 
* @throws IndexOutOfBoundsException if the index is out of range 
* ({@code index < 0 || index >= size()}).
*/
   public E get(int index) {
       ListIterator<E> i = listIterator(index);
       try {
           return i.next();
       } catch (NoSuchElementException e) {
           throw new IndexOutOfBoundsException("Index: " + index);
       }
}
```

> A special form of exception translation called *exception chaining* is called for in cases where the lower-level exception might be helpful to someone debugging the problem that caused the higher-level exception. The lower-level exception (the *cause*) is passed to the higher-level exception, which provides an accessor method (Throwable’s getCause method) to retrieve the lower-level exception:

异常转译的一种特殊形式被称为“异常链”。这种形式在底层异常对于调试导致高层异常的问题有帮助的时候，就很有用。底层异常（原因）可以传递到高层异常里，然后通过一个访问方法（Throwable的getClause方法）来获取底层异常，代码如下：

```java
// Exception Chaining
   try {
       ... // Use lower-level abstraction to do our bidding
       } catch (LowerLevelException cause) { 
     throw new HigherLevelException(cause);
}
```

> The higher-level exception’s constructor passes the cause to a *chaining-aware* superclass constructor, so it is ultimately passed to one of Throwable’s chaining- aware constructors, such as Throwable(Throwable):

这个高层异常的构造器传递一个原因到支持异常链的超类构造器中，也就是说，这个原因最后会被传递到一个Throwable的支持异常链的构造器中，比如Throwable(Throwable)：

```
// Exception with chaining-aware constructor
   class HigherLevelException extends Exception {
       HigherLevelException(Throwable cause) {
           super(cause);
       }
}
```

> Most standard exceptions have chaining-aware constructors. For exceptions that don’t, you can set the cause using Throwable’s initCause method. Not only does exception chaining let you access the cause programmatically (with getCause), but it integrates the cause’s stack trace into that of the higher-level exception.

大部分的标准异常都有支持异常链的构造器。对于那些不支持的异常，你可以通过使用Throwable的initCause方法来设置原因。异常链不仅仅可以让你通过程序的方法访问到原因，还可以把这个原因的异常栈融入到高层异常中。

> **While exception translation is superior to mindless propagation of excep- tions from lower layers, it should not be overused.** Where possible, the best way to deal with exceptions from lower layers is to avoid them, by ensuring that lower-level methods succeed. Sometimes you can do this by checking the validity of the higher-level method’s parameters before passing them on to lower layers.

**虽然异常转译，比直接把底层的异常直接传递要好一些，但是也不应该被滥用。**如果有可能的话，最好的处理异常的方法就是，保证底层方法成功执行，来避免抛出异常。有时候你可以在调用底层方法之前，检查器参数的合理性，以保证底层方法成功执行。

> If it is impossible to prevent exceptions from lower layers, the next best thing is to have the higher layer silently work around these exceptions, insulating the caller of the higher-level method from lower-level problems. Under these circumstances, it may be appropriate to log the exception using some appropriate logging facility such as java.util.logging. This allows programmers to investigate the problem, while insulating client code and the users from it.

如果不能阻止异常从底层抛出，其次的做法是让高层悄悄地处理这些异常，使高层的调用者和这些底层的问题，隔绝开来。在这些情况下，可以使用一些合适的日志工具比如java.util.logging来记录这些异常。这样可以方便程序员定位问题，也能使客户端代码和用户与这些问题分隔开。

> In summary, if it isn’t feasible to prevent or to handle exceptions from lower layers, use exception translation, unless the lower-level method happens to guarantee that all of its exceptions are appropriate to the higher level. Chaining provides the best of both worlds: it allows you to throw an appropriate higher-level exception, while capturing the underlying cause for failure analysis (Item 75).

总结一下，如果不能阻止也不能处理来自底层的异常的话，一般情况下就使用异常转译，除非可以保证底层方法的异常对于高层来说都是合适的，才可以将异常从底层传播到高层。异常链对于底层和高层异常提供了最好的处理:它既允许你抛出合适的高层异常，又能捕获底层的异常进行失败分析（Item75）。