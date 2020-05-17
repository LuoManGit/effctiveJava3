### Item61 基本类型优先于封装基本类型

> Java has a two-part type system, consisting of *primitives*, such as int, double, and boolean, and *reference types*, such as String and List. Every primitive type has a corresponding reference type, called a *boxed primitive*. The boxed primitives corresponding to int, double, and boolean are Integer, Double, and Boolean.

Java有两个类型系统，一个是基本类型，比如int，double，boolean；一个是引用类型，比如String和List。每一个基本类型都有一个对应的引用类型，称为封装基本类型。int，double，boolean是Integer，Double，和Boolean。

> As mentioned in Item 6, autoboxing and auto-unboxing blur but do not erase the distinction between the primitive and boxed primitive types. There are real differences between the two, and it’s important that you remain aware of which you are using and that you choose carefully between them.

正如Item6里面说的那样，自动装箱和自动拆箱并没有消除基本类型和封装基本类型之间的差别。两个类型之间有本质的区别，因此在你使用和选择使用哪种类型的时候，一定要注意到他们之间的区别。

> There are three major differences between primitives and boxed primitives. First, primitives have only their values, whereas boxed primitives have identities distinct from their values. In other words, two boxed primitive instances can have the same value and different identities. Second, primitive types have only fully functional values, whereas each boxed primitive type has one nonfunctional value, which is null, in addition to all the functional values of the corresponding primitive type. Last, primitives are more time- and space-efficient than boxed primitives. All three of these differences can get you into real trouble if you aren’t careful.

在基本类型和封装类型之间主要有三点不同。第一点是，基本类型只有值，而封装类型有不同于值的身份。也就是说，可以有两个封装类型，其值一样，但是是两个不同的对象；第二点是基本类型只有函数值，而封装类型除了有对应基本类型的函数值以外，还有一个非函数值null。最后，基本类型比封装类型在时间和空间上都更加节省。如果你不小心的话，这三点区别会给你带来麻烦。

> Consider the following comparator, which is designed to represent ascending numerical order on Integer values. (Recall that a comparator’s compare method returns a number that is negative, zero, or positive, depending on whether its first argument is less than, equal to, or greater than its second.) You wouldn’t need to write this comparator in practice because it implements the natural ordering on Integer, but it makes for an interesting example:

来看看下面这个比较器，设计来表示在integer值上的升序排列。（回顾一下，比较器的compare方法，根据第一个参数小于，等于，大于第二个参数，分别返回负数，0，整数）在实际中，你不需要编写这样的比较器，因为Integer已经实现了自然排序，但是它展示了一个有趣的例子：

```java
// Broken comparator - can you spot the flaw?
   Comparator<Integer> naturalOrder =
       (i, j) -> (i < j) ? -1 : (i == j ? 0 : 1);
```

> This comparator looks like it ought to work, and it will pass many tests. For example, it can be used with Collections.sort to correctly sort a million-element list, whether or not the list contains duplicate elements. But the comparator is deeply flawed. To convince yourself of this, merely print the value of naturalOrder.compare(new Integer(42), new Integer(42)). Both Integer instances represent the same value (42), so the value of this expression should be 0, but it’s 1, which indicates that the first Integer value is greater than the second!

这个比较器看起来没什么问题，而且还通过了很多的测试。比如，它可以被用在Collections.sort里，来对一个有上百万元素的列表进行排序，不管这个列表有没有包含相同的元素。但是这个比较器有很大的问题。为了证明它有问题，你只需要打印一下naturalOrder.compare(new Integer(42),的结果。这两个Integer实例都表示了同一个值42，因此这个表达式的值应该是0，但是这个结果却是1，它表示第一个Integer值大于第二个Integer值！

> So what’s the problem? The first test in naturalOrder works fine. Evaluating the expression i < j causes the Integer instances referred to by i and j to be *auto-unboxed*; that is, it extracts their primitive values. The evaluation proceeds to check if the first of the resulting int values is less than the second. But suppose it is not. Then the next test evaluates the expressioni==j, which performs an *identity comparison* on the two object references. If i and j refer to distinct Integer instances that represent the same int value, this comparison will return false, and the comparator will incorrectly return 1, indicating that the first Integer value is greater than the second. **Applying the** **==** **operator to boxed primitives is almost always wrong.**

那么这是什么问题呢？第一个naturalOrder的测试能正常地工作。对表达式i<j的计算会使得i和j引用的Integer实例进行自动拆箱，就是取出它的基本类型值，这个比较过程就是来检测第一个得到的int值是不是小于第二个。但是，如果不小于的话，就会进行下一个表达式i==j的计算，这个比较就会直接在两个对象引用上进行相等比较，如果i和j引用的是不同的Integer实例，但是表示的是同一个int值的话，这个比较就会返回false。然后比较器就会错误地返回1，表示第一个Integer值大于第二个。**对封装类型使用==操作符大部分情况都是错的**。

> In practice, if you need a comparator to describe a type’s natural order, you should simply call Comparator.naturalOrder(), and if you write a comparator yourself, you should use the comparator construction methods, or the static compare methods on primitive types (Item 14). That said, you could fix the problem in the broken comparator by adding two local variables to store the primitive int values corresponding to the boxed Integer parameters, and performing all of the comparisons on these variables. This avoids the erroneous identity comparison:

在实际中，如果你需要一个比较器来描述一个类型的自然顺序，你应该直接调用Comparator.naturalOrder()方法，如果你想自己写一个比较器，你应该使用比较器构造方法，或者基本类型的静态比较方法（Item14）。也就是，使用两个局部变量来保存对应封装类型里的值，然后基于这两个变量进行所有的比较，就可以解决这些问题。这样可以避免错误的等值比较：

```java
Comparator<Integer> naturalOrder = (iBoxed, jBoxed) -> {
       int i = iBoxed, j = jBoxed; // Auto-unboxing
       return i < j ? -1 : (i == j ? 0 : 1);
};
```

> Next, consider this delightful little program:

然后，看看这个小可爱程序：

```java
public class Unbelievable {
       static Integer i;
       public static void main(String[] args) {
           if (i == 42)
               System.out.println("Unbelievable");
       } 
}
```

> No, it doesn’t print Unbelievable—but what it does is almost as strange. It throws a NullPointerException when evaluating the expression i == 42. The problem is that i is an Integer, not an int, and like all nonconstant object reference fields, its initial value is null. When the program evaluates the expression i == 42, it is comparing an Integer to an int. In nearly every case **when you mix primitives and boxed primitives in an operation, the boxed primitive is auto-unboxed.** If a null object reference is auto-unboxed, you get a NullPointerException. As this program demonstrates, it can happen almost anywhere. Fixing the problem is as simple as declaring i to be an int instead of an Integer.

它确实不会打印”Unbelievable“——但是它的行为还是很奇怪。在执行表达式i==42的时候，它抛出了一个NullPointerException异常。这个问题是i是Integer，不是int，就像所有的所有的非常量对象域一样，它的初始值是null。当程序执行i==42的时候，它将Integer和int比较，在几乎每一个这样的例子中，**当你把基本类型和封装类型混在一个操作符的时候，封装类型就会被自动拆箱。**如果一个null的对象引用被自动拆箱，就会抛出NullPointerException。正如这个程序表示的那样，它可能会发生在任何地方。要解决这个问题也很简单，只需将i声明为int，而不是Integer就好了。

> Finally, consider the program from page 24 in Item 6:

最后，来看看这个Item6里面的程序：

```java
// Hideously slow program! Can you spot the object creation?
   public static void main(String[] args) {
       Long sum = 0L;
       for (long i = 0; i < Integer.MAX_VALUE; i++) {
           sum += i;
       }
       System.out.println(sum);
   }
```

> This program is much slower than it should be because it accidentally declares a local variable (sum) to be of the boxed primitive type Long instead of the primitive type long. The program compiles without error or warning, and the variable is repeatedly boxed and unboxed, causing the observed performance degradation.

这个程序比预计的要慢很多，因为它不小心把一个本地变量sum声明成了封装类型Long，而不是基本类型long。这个程序在编译的时候不会有任何的error和warning，然后这个变量就被重复的装箱、拆箱，导致程序的性能出现显著的问题。

> In all three of the programs discussed in this item, the problem was the same: the programmer ignored the distinction between primitives and boxed primitives and suffered the consequences. In the first two programs, the consequences were outright failure; in the third, severe performance problems.

在本节中讨论的三个程序里，问题是一样的：程序员忽略了基本类型和封装类型之间的区别，然后导致出现问题。在前面两个程序中，其结果是彻底的失败；在第三个程序中，是严重的性能问题。

> So when should you use boxed primitives? They have several legitimate uses. The first is as elements, keys, and values in collections. You can’t put primitives in collections, so you’re forced to use boxed primitives. This is a special case of a more general one. You must use boxed primitives as type parameters in parameterized types and methods (Chapter 5), because the language does not permit you to use primitives. For example, you cannot declare a variable to be of type ThreadLocal<int>, so you must use ThreadLocal<Integer> instead. Finally, you must use boxed primitives when making reflective method invocations (Item 65).

那么你什么时候应该使用封装类型呢？他们有几个合法的使用特例。第一个是作为集合的元素、key和value。你不能往集合里往集合里放基本类型。这是一个更通用的使用场景的特例，在参数类型和方法中的类型参数必须是封装类型，因为Java语言不允许你使用基本类型。比如，你不能声明一个类型为ThreadLocal<int>的变量，因此你必须使用ThreadLocal<Integer>来代替。最后，在进行反射方法调用的时候（Item65）里必须使用封装类型。

> In summary, use primitives in preference to boxed primitives whenever you have the choice. Primitive types are simpler and faster. If you must use boxed primitives, be careful! **Autoboxing reduces the verbosity, but not the danger, of using boxed primitives.** When your program compares two boxed primitives with the == operator, it does an identity comparison, which is almost certainly *not* what you want. When your program does mixed-type computations involving boxed and unboxed primitives, it does unboxing, and **when your program does unboxing, it can throw a** **NullPointerException**. Finally, when your program boxes primitive values, it can result in costly and unnecessary object creations.

总结一下，当你面临选择的时候，优先使用基本类型，而不是封装类型。基本类型简单又快速。如果你必须要使用封装类型，要仔细一些！**自动装箱降低了封装类型使用的负责度，但是却没有减少其风险。**当你的程序在使用==对两个封装类型进行比较的时候，它做的是一个同一性比较，得到的结果往往不是你想要的。当你的程序在进行封装类型和基本类型之间的混合类型操作计算的时候，它会做拆箱。**当程序进行拆箱的时候，可能会抛出NullPointerException**。最后当你的程序对基本类型值进行封装的时候，可能会在成较高的资源消耗，并创建一些没有必要的对象。