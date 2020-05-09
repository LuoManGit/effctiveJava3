## 8 方法

> **T**HIS chapter discusses several aspects of method design: how to treat parameters and return values, how to design method signatures, and how to document methods. Much of the material in this chapter applies to constructors as well as to methods. Like Chapter 4, this chapter focuses on usability, robustness, and flexibility.

本章主要讨论方法设计的以下几个方面：如何处理参数和返回值，如何设计方法签名，以及如何给方法编写文档。本章中的大部分内容对构造器和普通的方法都适用。和第四章一样，本章也专注于可用性、健壮性和灵活性。

### Item49 检查参数的有效性

> Most methods and constructors have some restrictions on what values may be passed into their parameters. For example, it is not uncommon that index values must be non-negative and object references must be non-null. You should clearly document all such restrictions and enforce them with checks at the beginning of the method body. This is a special case of the general principle that you should attempt to detect errors as soon as possible after they occur. Failing to do so makes it less likely that an error will be detected and makes it harder to determine the source of an error once it has been detected.

大部分的方法和构造器对于传递给其参数的值都有一个限制。比如，常见的有索引值必须是非负数，对象引用也必须是非null的。也应该在文档中说明这些限制，并且在方法体的开头进行检查。这是“在发生错误后，要尽快检查到错误”这一原则的一个特例。如果做不到的话，就会很难检测到错误，或者检测到了错误却很难确定错误的源头。

> If an invalid parameter value is passed to a method and the method checks its parameters before execution, it will fail quickly and cleanly with an appropriate exception. If the method fails to check its parameters, several things could happen. The method could fail with a confusing exception in the midst of processing. Worse, the method could return normally but silently compute the wrong result. Worst of all, the method could return normally but leave some object in a compromised state, causing an error at some unrelated point in the code at some undetermined time in the future. In other words, failure to validate parameters, can result in a violation of *failure atomicity* (Item 76).

如果给方法传递了一个异常的参数，这个方法在执行之前进行了参数检测，那么这个方法就会很快失败，并抛出合适清晰的异常。如果这个方法没有进行参数检测的话，就可能会出现好几种情况。比如，这个方法会在运行的中间抛出一个让人迷惑的异常；更糟糕的是，这个方法会正常返回，但是计算的值却是错误的；最糟糕的是，这个方法会返回正常的值，但是把一个对象置成了不正确的状态，导致在未来某个不确定的时候里，某段不相关的代码出现错误。也就是说，没有正确检查参数，会导致违背“失败原子性”（详见Item76）。

> For public and protected methods, use the Javadoc @throws tag to document the exception that will be thrown if a restriction on parameter values is violated (Item 74). Typically, the resulting exception will be IllegalArgumentException, IndexOutOfBoundsException, or NullPointerException (Item 72). Once you’ve documented the restrictions on a method’s parameters and you’ve documented the exceptions that will be thrown if these restrictions are violated, it is a simple matter to enforce the restrictions. Here’s a typical example:

对于公有的，或者受宝华的方法，使用Javadoc标签@throws来用文档说明，在违背了参数规定的时候会抛出的异常。常见的异常有IllegalArgumentException, IndexOutOfBoundsException, 或者 NullPointerException（Item72）。一旦你使用文档说明了方法参数的限制，你就应该说明违背了这些限制会抛出什么异常，这样就很容易强制执行这些限制了。下面是一个典型的示例：

```java
/**
* Returns a BigInteger whose value is (this mod m). This method
* differs from the remainder method in that it always returns a * non-negative BigInteger.
*
* @param m the modulus, which must be positive
* @return this mod m
* @throws ArithmeticException if m is less than or equal to 0 */
public BigInteger mod(BigInteger m) { 
  if (m.signum() <= 0)
       throw new ArithmeticException("Modulus <= 0: " + m);
       ... // Do the computation
}
```

> Note that the doc comment does *not* say “mod throws NullPointerException if m is null,” even though the method does exactly that, as a byproduct of invoking m.signum(). This exception *is* documented in the class-level doc comment for the enclosing BigInteger class. The class-level comment applies to all parameters in all of the class’s public methods. This is a good way to avoid the clutter of documenting every NullPointerException on every method individually. It may be combined with the use of @Nullable or a similar annotation to indicate that a particular parameter may be null, but this practice is not standard, and multiple annotations are in use for this purpose.

注意，这个文档说明里并没有说“如果m为空的话，mod方法会抛出NullPointerException。”即使这个方法在调用m.signum()的时候，确实会抛出这个异常。这个异常被写在BigInteger类的类级别的文档说明上了。类级别的文档是针对该类的所有的公有方法的所有参数的。这样可以很好地避免了在每个方法上都分别写NullPointerException。它可以和@Nullable或者其他类似的注解一起使用，来表示这个某个特定的注解可以为空，但是在实际引用中，这并不是唯一的标准，有很多的注解都可以达到这个目的。

> **The** **Objects.requireNonNull** **method, added in Java 7, is flexible and convenient, so there’s no reason to perform null checks manually anymore.** You can specify your own exception detail message if you wish. The method returns its input, so you can perform a null check at the same time as you use a value:

**在Java7中新增了Objects.requireNonNull方法，灵活又方便，因此没有必要在手动进行null检查了**。你还可以指定你想要的异常细节信息，这个方法会返回它的输入值，因此你可以在null检查的同时，使用这个值。如下：

```java
// Inline use of Java's null-checking facility
   this.strategy = Objects.requireNonNull(strategy, "strategy");
```

> You can also ignore the return value and use Objects.requireNonNull as a freestanding null check where that suits your needs.

你也可以忽略这个返回值，然后按照你的需求，使用Objects.requireNonNull做独立的null检查。

> In Java 9, a range-checking facility was added to java.util.Objects. This facility consists of three methods: checkFromIndexSize, checkFromToIndex, and checkIndex. This facility is not as flexible as the null-checking method. It doesn’t let you specify your own exception detail message, and it is designed solely for use on list and array indices. It does not handle closed ranges (which contain both of their endpoints). But if it does what you need, it’s a useful convenience.

在Java9中，在java.util.Objects中新增了一个范围检查技术，这个技术包括三个方法：checkFromIndexSize, checkFromToIndex, 和 checkIndex。这个方法没有null检查方法那么灵活，不能自己制定异常细节信息，而且他是专门为列表和数组索引设计的。它也不处理关闭范围（即包括两个端点的范围）。但是如果你确实需要的，它是一个很好用的方便工具。

> For an unexported method, you, as the package author, control the circumstances under which the method is called, so you can and should ensure that only valid parameter values are ever passed in. Therefore, nonpublic methods can check their parameters using *assertions,* as shown below:

对于一个不用导出的方法，你作为这个包的作者，可以控制这个方法被调用的环境，也能确定这个只会传入合法的参数值。因此非公开的方法可以使用断言来检查他们的参数，比如：

```java
// Private helper function for a recursive sort
private static void sort(long a[], int offset, int length) {
		assert a != null;
		assert offset >= 0 && offset <= a.length;
		assert length >= 0 && length <= a.length - offset; 
  	... // Do the computation
}
```

> In essence, these assertions are claims that the asserted condition *will* be true, regardless of how the enclosing package is used by its clients. Unlike normal validity checks, assertions throw AssertionError if they fail. And unlike normal validity checks, they have no effect and essentially no cost unless you enable them, which you do by passing the -ea (or -enableassertions) flag to the java command. For more information on assertions, see the tutorial [Asserts].

本质来说，这些断言是在生成被断言的条件将为真，无论其外围包的客户端如何使用它。和普通有效性检查不一样的是，断言如果失败的话，会抛出AssertionError。还有一点和普通有效性检查不一样的是，它没什么影响也不会有什么消耗，除非你通过给java 解释器传递一个-ea (或者 -enableassertions)标志，来启用他们。更多关于断言的信息，请查看教程[Asserts]。

> It is particularly important to check the validity of parameters that are not used by a method, but stored for later use. For example, consider the static factory method on page 101, which takes an int array and returns a List view of the array. If a client were to pass in null, the method would throw a NullPointerException because the method has an explicit check (the call to Objects.requireNonNull). Had the check been omitted, the method would return a reference to a newly created List instance that would throw a NullPointerException as soon as a client attempted to use it. By that time, the origin of the List instance might be difficult to determine, which could greatly complicate the task of debugging.

对那些在方法中没有使用，但是保存起来后面用的参数进行有效性检查非常重要。比如，Item20中的静态工厂方法，参数是一个int数组，返回这个数组的List视图。如果客户端传进来的是空，这个方法就会抛出NullPointerException，因为这个方法有一个明确的检查（调用了Object.requrieNonNull）。假如没有这个价差，这个方法就会返回一个新创建的List实例的引用，并且在客户端一使用这个List的时候就会抛出NullPointerException。在这个时候，List实例的源头很难确定，会使得debug任务非常复杂。

> Constructors represent a special case of the principle that you should check the validity of parameters that are to be stored away for later use. It is critical to check the validity of constructor parameters to prevent the construction of an object that violates its class invariants.

构造器正是 “应该检查保存起来后面使用的参数的有效性” 这一原则的一个特殊的例子。对构造器参数进行有效性检查是非常重要的，可以避免构造的对象违反了类的约束条件。

> There are exceptions to the rule that you should explicitly check a method’s parameters before performing its computation. An important exception is the case in which the validity check would be expensive or impractical *and* the check is performed implicitly in the process of doing the computation. For example, consider a method that sorts a list of objects, such as Collections.sort(List). All of the objects in the list must be mutually comparable. In the process of sorting the list, every object in the list will be compared to some other object in the list. If the objects aren’t mutually comparable, one of these comparisons will throw a ClassCastException, which is exactly what the sort method should do. Therefore, there would be little point in checking ahead of time that the elements in the list were mutually comparable. Note, however, that indiscriminate reliance on implicit validity checks can result in the loss of *failure atomicity* (Item 76).

“你应该在执行计算之前，明确地检查方法的参数”这一原则，有一些例外的情况。一个重要的例外情况就是有效性检查的代价非常昂贵或者无法进行，以及在执行计算的过程中隐式地进行了有效性检查。比如，给列表数据进行排序的方法，比如 Collections.sort(List)。这个列表中的所有的对象都必须是可以相互比较的。在进行排序的过程中，每个列表中的对象都需要和列表中的其他对象进行比较。如果有一些对象不可以相互比较，那么这些比较中的某一个就会抛出ClassCastException，这也正是sort方法应该做的。因此，提前检查这些方法是不是可以互相比较的就不是那么重要了。然而，需要注意的是，过度依赖这种隐式的有效性检查可能会失去“失败原子性（Item76）”。

> Occasionally, a computation implicitly performs a required validity check but throws the wrong exception if the check fails. In other words, the exception that the computation would naturally throw as the result of an invalid parameter value doesn’t match the exception that the method is documented to throw. Under these circumstances, you should use the *exception translation* idiom, described in Item 73, to translate the natural exception into the correct one.

有时候，计算时隐式执行的有效性检查，在检查失败的异常可能是错的。换句话说，就是，计算时因为参数异常抛出的异常，可能和文档中声明会抛出的异常不一致。在这种情况下，你可以使用Item73里介绍的异常转换技术，来把这个异常转换成正确的异常。

> Do not infer from this item that arbitrary restrictions on parameters are a good thing. On the contrary, you should design methods to be as general as it is practical to make them. The fewer restrictions that you place on parameters, the better, assuming the method can do something reasonable with all of the parameter values that it accepts. Often, however, some restrictions are intrinsic to the abstraction being implemented.

不要从本节中得出这样的结论：对参数的任何限制都是好事。相反地，你应该把把它设计得竟可能通用，和实际情况相符。假如这个方法能对其接受的所有值都做出正确的工作，那么参数上的限制越少就越好。然后，大部分时候，一些方法上的限制对于这个抽象的实现是固有的。

> To summarize, each time you write a method or constructor, you should think about what restrictions exist on its parameters. You should document these restrictions and enforce them with explicit checks at the beginning of the method body. It is important to get into the habit of doing this. The modest work that it entails will be paid back with interest the first time a validity check fails.

总结一下，每次你写方法或者构造器的时候，都应该思考一下参数上应该有的限制。你应该用文档说明这些限制，并且在方法体的开头使用明确的检查来强制执行这些限制。养成这样一个习惯非常重要。只要这个有效性检验有一次失败，你为此付出的工作就是很值得的。