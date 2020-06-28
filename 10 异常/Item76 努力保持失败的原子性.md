### Item76 努力保持失败的原子性

> After an object throws an exception, it is generally desirable that the object still be in a well-defined, usable state, even if the failure occurred in the midst of performing an operation. This is especially true for checked exceptions, from which the caller is expected to recover. **Generally speaking, a failed method invocation should leave the object in the state that it was in prior to the invocation**. A method with this property is said to be *failure-atomic.*

在一个对象抛出异常之后，通常希望这个对象还处在良好定义的，可用的状态下，即使 这个失败发生在执行操作的过程中。对于受检异常来说，尤为重要，因为调用者希望可以从中恢复异常。**通俗地说，一个失败的方法调用应该让这个对象保持在调用之前的状态。**具有这样的属性的方法被称为具有失败原子性。

> There are several ways to achieve this effect. The simplest is to design immutable objects (Item 17). If an object is immutable, failure atomicity is free. If an operation fails, it may prevent a new object from getting created, but it will never leave an existing object in an inconsistent state, because the state of each object is consistent when it is created and can’t be modified thereafter.

有几个方法可以达到这样的效果。最简单的方法就是设计不可变对象（Item17）。如果一个对象是不可变的，那么就自然拥有失败原子性。如果这个操作失败了，可能会阻止创建新的对象，但是也不会让一个已经存在的对象处于前后矛盾的状态，因为每个对象的状态始终和在创建的时候一致，以后也永远不会发生变化。

> For methods that operate on mutable objects, the most common way to achieve failure atomicity is to check parameters for validity before performing the operation (Item 49). This causes most exceptions to get thrown before object modification commences. For example, consider the Stack.pop method in Item 7:

对于可变对象上的操作，最常用的达到失败原子性的方法是在操作执行之前，检查参数的有效性（Item49）。这可以使得在对象被修改之前，就抛出大部分的异常。比如，下面这个Item7的Stack.pop方法：

```java
public Object pop() {
       if (size == 0)
           throw new EmptyStackException();
       Object result = elements[--size];
       elements[size] = null; // Eliminate obsolete reference
       return result;
}
```

> If the initial size check were eliminated, the method would still throw an exception when it attempted to pop an element from an empty stack. It would, however, leave the size field in an inconsistent (negative) state, causing any future method invocations on the object to fail. Additionally, the ArrayIndexOutOfBoundsException thrown by the pop method would be inappropriate to the abstraction (Item 73).

如果没有前面的大小检测，这个方法在企图从一个空的堆栈里弹出一个元素的时候，就会抛出一个异常。然而，它还会使得size这个域处在一个前后矛盾（为负数）的状态，导致后面调用这个对象的方法都会失败。此外，这个pop方法抛出的ArrayIndexOutOfBoundsException对于抽象来说，也不合适（Item73）。

> A closely related approach to achieving failure atomicity is to order the computation so that any part that may fail takes place before any part that modifies the object. This approach is a natural extension of the previous one when arguments cannot be checked without performing a part of the computation. For example, consider the case of TreeMap, whose elements are sorted according to some ordering. In order to add an element to a TreeMap, the element must be of a type that can be compared using the TreeMap’s ordering. Attempting to add an incorrectly typed element will naturally fail with a ClassCastException as a result of searching for the element in the tree, before the tree has been modified in any way.

另外一个和这个方法类似的获得失败原子性的方法是给计算排序，使得所有可能导致失败的操作都在修改对象的操作前面进行。这个方法是前面的方法的一个自然延伸，适合用在当参数的校验只能在执行过程中进行的时候。比如，TreeMap，它的元素按照某个顺序进行了排序。当企图往里面添加一个类型错误的元素的时候，会在tree里搜索这个元素的时候，就抛出ClassCastException，这是在tree被修改之前就发生的。

> A third approach to achieving failure atomicity is to perform the operation on a temporary copy of the object and to replace the contents of the object with the temporary copy once the operation is complete. This approach occurs naturally when the computation can be performed more quickly once the data has been stored in a temporary data structure. For example, some sorting functions copy their input list into an array prior to sorting to reduce the cost of accessing elements in the inner loop of the sort. This is done for performance, but as an added benefit, it ensures that the input list will be untouched if the sort fails.

第三种获得失败原子性的方法是，在临时复制的对象上执行操作，并在计算完成之后，用临时复制对象来代替原对象的内容。一旦数据保存在临时的数据结构中，计算就会很快完成的话，这种方法就会显得很自然。比如，一些排序函数会将他们的输入复制到数组里，以减少在排序的时候的内循环中访问元素都来的消耗。这是出于性能的考虑，但也获得了另一个优势，即使排序失败，也能保证输入列表保持原样。

> A last and far less common approach to achieving failure atomicity is to write *recovery code* that intercepts a failure that occurs in the midst of an operation, and causes the object to roll back its state to the point before the operation began. This approach is used mainly for durable (disk-based) data structures.

最后一个，也不是那么常见的获得失败原子性的方法是，编写恢复代码，由他来拦截计算郭晨各种出现的失败，并且让对象的状态回滚到操作执行之前。这种方法主要用于永久（基于磁盘）的数据结构。

> While failure atomicity is generally desirable, it is not always achievable. For example, if two threads attempt to modify the same object concurrently without proper synchronization, the object may be left in an inconsistent state. It would therefore be wrong to assume that an object was still usable after catching a ConcurrentModificationException. Errors are unrecoverable, so you need not even attempt to preserve failure atomicity when throwing AssertionError.

虽然，通常情况下，都希望达到失败原子性，但是有时候确实做不到。比如，如果有两个线程想并发的修改同一个对象，并且没有合适的同步，这个对象就很可能处在不一致的状态。因此，在捕获ConcurrentModificationException后，还假定这个对象是可用的，就是不正确的。Errors是不可恢复的，因此当抛出AssertionError的时候，也不需要努力去保持失败原子性。

> Even where failure atomicity is possible, it is not always desirable. For some operations, it would significantly increase the cost or complexity. That said, it is often both free and easy to achieve failure atomicity once you’re aware of the issue.

即使在可以实现失败原子性的地方，它也不总是人们所期望的那样。对于一些操作，为了实现失败原子性，可能会极大得增加开销和复杂性。也就是说，一旦你意识到了这个问题，那么达到失败原子性可能又简单又容易。

> In summary, as a rule, any generated exception that is part of a method’s specification should leave the object in the same state it was in prior to the method invocation. Where this rule is violated, the API documentation should clearly indicate what state the object will be left in. Unfortunately, plenty of existing API documentation fails to live up to this ideal.

总结一下，作为一个规则，每一个生成的异常都是方法的规范的一部分，应该让这个对象保持在调用方法之前的状态，如果违反了这个规则，那么API的文档应该指明这个对象将处于什么状态。不幸地是，目前大部分已经存在的API文档都没有做到这一点。