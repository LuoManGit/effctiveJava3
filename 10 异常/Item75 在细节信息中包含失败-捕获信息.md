### Item75 在细节信息中包含失败-捕获信息

> When a program fails due to an uncaught exception, the system automatically prints out the exception’s stack trace. The stack trace contains the exception’s *string representation*, the result of invoking its toString method. This typically consists of the exception’s class name followed by its *detail message*. Frequently this is the only information that programmers or site reliability engineers will have when investigating a software failure. If the failure is not easily reproducible, it may be difficult or impossible to get any more information. Therefore, it is critically important that the exception’s toString method return as much information as possible concerning the cause of the failure. In other words, the detail message of an exception should *capture the failure* for subsequent analysis.

当程序因为一个没有捕获的异常而失败时，系统会自动打印异常的堆栈轨迹。这个堆栈轨迹会包含异常的字符串表达式，这个字符串是通过toString方法得到的。这个通常会包含异常的类名以及它的详细的信息。通常情况下，这是程序员或者网站可靠维护工程师用来调查软件失败原因的唯一信息。如果这个失败不容易再现，那么就很难或者不可能获得其他的信息了。因此，异常的toString方法返回与失败原因相关的信息越多越好。换句话说，异常的详细信息应该包含失败信息，以便后续的分析。

> **To capture a failure, the detail message of an exception should contain the values of all parameters and fields that contributed to the exception.** For example, the detail message of an IndexOutOfBoundsException should contain the lower bound, the upper bound, and the index value that failed to lie between the bounds. This information tells a lot about the failure. Any or all of the three values could be wrong. The index could be one less than the lower bound or equal to the upper bound (a “fencepost error”), or it could be a wild value, far too low or high. The lower bound could be greater than the upper bound (a serious internal invariant failure). Each of these situations points to a different problem, and it greatly aids in the diagnosis if you know what sort of error you’re looking for.

**为了包含失败，异常的细节信息应该包含所有域异常相关的参数和域的值。**比如，IndexOutOfBoundsException的详细信息就应该包含最低边界和最高边界，以及这个没有在两个边界之中的索引值。这个信息就包含了很多的失败信息。这是三个值中有一个或者全部都可能是错的。这个索引可能比最低边界小，或者和最高边界一样大（”越界错误“），或者可能是无效值，太大或者太小。最低边界比最高边界大（一个严重的内部约束错误）。这种场景中的每一种情况都代表了不同的问题，如果你知道是哪种问题，那么就能有助于解决这个问题。

> One caveat concerns security-sensitive information. Because stack traces may be seen by many people in the process of diagnosing and fixing software issues, **do not include passwords, encryption keys, and the like in detail messages.**

有一个关于安全敏感信息的警告。因为堆栈信息可以被参与诊断和修复软件问题的很多人都看到，**因此不要在详细信息中包含密钥密码和类似的信息。**

> While it is critical to include all of the pertinent data in the detail message of an exception, it is generally unimportant to include a lot of prose. The stack trace is intended to be analyzed in conjunction with the documentation and, if necessary, source code. It generally contains the exact file and line number from which the exception was thrown, as well as the files and line numbers of all other method invocations on the stack. Lengthy prose descriptions of the failure are superfluous; the information can be gleaned by reading the documentation and source code.

虽然在异常的详细信息中包含所有相关的数据非常重要，但是通常情况下，包含大量的描述性语言也不是那么重要。这个堆栈是用来和文档信息一起分析的，有必要的话，还需要结合源代码。堆栈信息中通常包含抛出异常的文档和具体的行数。冗长的失败描述很多余；这些信息可以通过阅读文档和源代码获取到。

> The detail message of an exception should not be confused with a user-level error message, which must be intelligible to end users. Unlike a user-level error message, the detail message is primarily for the benefit of programmers or site reliability engineers, when analyzing a failure. Therefore, information content is far more important than readability. User-level error messages are often *localized*, whereas exception detail messages rarely are.

异常的细节信息不应该和用户级的错误信息混到一起，用户级的错误信息对于终端用户来说，必须是可理解的。不想用户级的错误信息，细节信息主要是给程序员或者网站可靠性维护工程师用来分析问题的。因此，信息内容比可读性更重要一些。用户级别的错误信息经常被本地化，而异常的细节信息就很少被本地化。

> One way to ensure that exceptions contain adequate failure-capture information in their detail messages is to require this information in their constructors instead of a string detail message. The detail message can then be generated automatically to include the information. For example, instead of a String constructor, IndexOutOfBoundsException could have had a constructor that looks like this:

保证异常的细节信息中包含足够的失败信息的一个方法，是在异常的构造器中包含这些信息，而不是在字符串的细节信息中。这些细节信息可以自动生成，并包含这些信息。比如，IndexOutOfBoundsException使用了一个下面这样的构造器来代替字符串构造器：

```java
/**
    * Constructs an IndexOutOfBoundsException.
    *
    * @param lowerBound the lowest legal index value
    * @param upperBound the highest legal index value plus one
    * @param index      the actual index value
    */
public IndexOutOfBoundsException(int lowerBound, int upperBound, int index) {
       // Generate a detail message that captures the failure
       super(String.format(
               "Lower bound: %d, Upper bound: %d, Index: %d",
               lowerBound, upperBound, index));
       // Save failure information for programmatic access
       this.lowerBound = lowerBound;
       this.upperBound = upperBound;
       this.index = index;
}
```

> As of Java 9, IndexOutOfBoundsException finally acquired a constructor that takes an int valued index parameter, but sadly it omits the lowerBound and upperBound parameters. More generally, the Java libraries don’t make heavy use of this idiom, but it is highly recommended. It makes it easy for the programmer throwing an exception to capture the failure. In fact, it makes it hard for the programmer not to capture the failure! In effect, the idiom centralizes the code to generate a high-quality detail message in the exception class, rather than requiring each user of the class to generate the detail message redundantly.

从Java9开始，IndexOutOfBoundsException终于有了一个带int值的索引参数的构造器，但遗憾地是，它忽略了最低边界和最高边界参数。更通俗地说，这种方法在Java库中目前还没有得到广泛的使用，但是这种做法还是强烈推荐的，这使得程序员在捕获到失败时，抛出异常比较简单。事实上，这种做法使得程序员想不捕获异常都难。这种做法可以把代码都集中到异常类中，用于生成高质量的细节信息，而不是让这个类的用户自己来生成多余的细节信息。

> As suggested in Item 70, it may be appropriate for an exception to provide accessor methods for its failure-capture information (lowerBound, upperBound, and index in the above example). It is more important to provide such accessor methods on checked exceptions than unchecked, because the failure-capture information could be useful in recovering from the failure. It is rare (although not inconceivable) that a programmer might want programmatic access to the details of an unchecked exception. Even for unchecked exceptions, however, it seems advisable to provide these accessors on general principle (Item 12, page 57).

正如Item70中建议的那样，为异常的失败信息（就是前面例子中的最低边界，最高边界和索引值）提供访问方法是非常合适的，相对于非受检异常，对于受检异常来说，提供这些访问器更重要一些，因为在恢复失败的时候，失败信息是非常有用的。程序员很少（尽管也是可以想象的）想要通过编程的方法来访问非受检异常的细节。然而，即使是非受检异常，作为一般的原则，提供这些访问其也是明智的（Item12）。