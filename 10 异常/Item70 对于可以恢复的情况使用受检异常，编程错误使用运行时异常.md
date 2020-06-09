### Item70 对于可以恢复的情况使用受检异常，编程错误使用运行时异常

> Java provides three kinds of throwables: *checked exceptions*, *runtime exceptions*, and *errors*. There is some confusion among programmers as to when it is appropriate to use each kind of throwable. While the decision is not always clear-cut, there are some general rules that provide strong guidance.

Java 提供了三种可以抛出的结构：受检异常，运行时异常，和错误。在什么情况下适合使用哪种抛出结构，对于和很多程序员来说，很困扰。虽然这些决定总是不那么清晰，但是还是有一些通用的规则可以提供一些强有力的指导。

> The cardinal rule in deciding whether to use a checked or an unchecked exception is this: **use checked exceptions for conditions from which the caller can reasonably be expected to recover.** By throwing a checked exception, you force the caller to handle the exception in a catch clause or to propagate it outward. Each checked exception that a method is declared to throw is therefore a potent indication to the API user that the associated condition is a possible outcome of invoking the method.

选择使用受检异常还是非受检异常，最主要的规则是：**如果期望调用者可以对异常进行恢复，就应该使用受检异常。**抛出一个受检异常，强迫调用者必须在一个catch块里处理这个异常，或者向外抛出这个异常。每一个方法声明会抛出的受检异常，对于API的用户来说，是一种指示：意味着与异常相关的条件是调用这个方法的一种可能的结果。

> By confronting the user with a checked exception, the API designer presents a mandate to recover from the condition. The user can disregard the mandate by catching the exception and ignoring it, but this is usually a bad idea (Item 77).

API设计者让用户面对受检异常，以要求用户恢复这个异常。用户也可以通过捕获这个异常并忽略它，来忽视这种要求，但是这是一个坏主意（Item77）。

> There are two kinds of unchecked throwables: runtime exceptions and errors. They are identical in their behavior: both are throwables that needn’t, and generally shouldn’t, be caught. If a program throws an unchecked exception or an error, it is generally the case that recovery is impossible and continued execution would do more harm than good. If a program does not catch such a throwable, it will cause the current thread to halt with an appropriate error message.

有两种非受检的可抛出结构：运行时异常和错误。它们具有相同的行为：都是可以抛出的，并且不需要，一般情况也不应该，被捕获。如果一个程序抛出了一个非受检异常或者错误，通常都是不可恢复的情况，继续执行下去的话有害无益。如果程序没有不会这样的可抛出结构，当前线程就会抛出一个合适的错误信息并中断。

> **Use runtime exceptions to indicate programming errors.** The great majority of runtime exceptions indicate *precondition violations*. A precondition violation is simply a failure by the client of an API to adhere to the contract established by the API specification. For example, the contract for array access specifies that the array index must be between zero and the array length minus one, inclusive. ArrayIndexOutOfBoundsException indicates that this precondition was violated.

**使用运行时异常来表示程序错误**。运行时异常主要用来表示“违背前提”的情况。“违背前提”就是指使用API的客户端没有很好地遵守API规范建立好的公约。比如，数据建立的公约指明了数组的下标只能在0和数组长度-1之间，ArrayIndexOutOfBoundsException运行时异常就表明这个前提被违背了。

> One problem with this advice is that it is not always clear whether you’re dealing with a recoverable conditions or a programming error. For example, consider the case of resource exhaustion, which can be caused by a programming error such as allocating an unreasonably large array, or by a genuine shortage of resources. If resource exhaustion is caused by a temporary shortage or by temporarily heightened demand, the condition may well be recoverable. It is a matter of judgment on the part of the API designer whether a given instance of resource exhaustion is likely to allow for recovery. If you believe a condition is likely to allow for recovery, use a checked exception; if not, use a runtime exception. If it isn’t clear whether recovery is possible, you’re probably better off using an unchecked exception, for reasons discussed in Item 71.

这个建议的问题在于，在一些情况下，很难弄清楚，是需要使用一个可恢复的条件还是程序错误。比如资源耗尽的情况，可能是因为一些不合理的大数组的创建造成的，也可能是真的资源太少耗尽了。如果这个资源耗尽是暂时的短缺或者短时的高需求造成的，这种情况可能就是可恢复的。对于API设计来说，这个资源耗尽的实例是否可以恢复是很难确定的。如果你相信这个情况可以恢复，那么就使用受检异常。如果不能，就使用运行时异常。如果是否能恢复不好确定，那么最好就使用非受检异常，具体原因详见Item71。

> While the Java Language Specification does not require it, there is a strong convention that *errors* are reserved for use by the JVM to indicate resource deficiencies, invariant failures, or other conditions that make it impossible to continue execution. Given the almost universal acceptance of this convention, it’s best not to implement any new Error subclasses. Therefore, **all of the unchecked throwables you implement should subclass** **RuntimeException** (directly or indirectly). Not only shouldn’t you define Error subclasses, but with the exception of AssertionError, you shouldn’t throw them either.

虽然Java语言规范没有做相关的要求，但是按照惯例，Error都是JVM用来表示资源缺乏、约束失败、或者其他导致不能继续执行的情况。因为这已经是最普遍接受的约定，那么最好就不要实现新的Error的子类了。因此，**你实现的所有的可抛出结构都应该是RuntimeException（直接或间接）的子类**。你不仅仅不应该定义除了AssertionError以外的Error的子类，也不应该抛出他们。

> It is possible to define a throwable that is not a subclass of Exception, RuntimeException, or Error. The JLS doesn’t address such throwables directly but specifies implicitly that they behave as ordinary checked exceptions (which are subclasses of Exception but not RuntimeException). So when should you use such a beast? In a word, never. They have no benefits over ordinary checked exceptions and would serve merely to confuse the user of your API.

也可以定义一个可抛出结构，既不是Exception，也不是RuntimeException或者Error的子类。JLS确实没有直接指明这样的可抛出结构，但是含蓄地指示了他们的行为和普通的受检异常一样（是Exception的子类而不是RuntimeException的子类）。那么什么时候使用这样的结构呢？永远不要。和普通受检异常相比没有任何的优点，并且还会让你的API的用户困扰。

> API designers often forget that exceptions are full-fledged objects on which arbitrary methods can be defined. The primary use of such methods is to provide code that catches the exception with additional information concerning the condition that caused the exception to be thrown. In the absence of such methods, programmers have been known to parse the string representation of an exception to ferret out additional information. This is extremely bad practice (Item 12). Throwable classes seldom specify the details of their string representations, so string representations can differ from implementation to implementation and release to release. Therefore, code that parses the string representation of an exception is likely to be nonportable and fragile.

API的设计者经常会忘记异常本身也是功能齐全的对象，可以在上面定义任意的方法。这种方法的主要的用途是用在捕获异常的时候，提供更多的造成抛出异常的原因的信息。如果没有这些方法的话，程序员就必须要去分析异常的字符串表达式才能获取到额外的信息了，在实际情况中，这是非常不好的（Item12）。Throwable类几乎都没有指定它们的字符串表达式的细节，因此其字符串表达式也会根据具体实现和版本发生变化。因此，编写代码来分析异常的字符串表达式，很可能是不可移植的，也是非常脆弱的。

> Because checked exceptions generally indicate recoverable conditions, it’s especially important for them to provide methods that furnish information to help the caller recover from the exceptional condition. For example, suppose a checked exception is thrown when an attempt to make a purchase with a gift card fails due to insufficient funds. The exception should provide an accessor method to query the amount of the shortfall. This will enable the caller to relay the amount to the shopper. See Item 75 for more on this topic.

由于受检异常通常来说都意味着可以恢复的情况，因此对于他们来说，提供一些辅助方法尤其重要，调用者就可以通过这些方法来获取一些帮助其恢复异常情况的信息。比如，当尝试购买一个礼物卡片，但是没有足够的余额的时候，抛出了一个异常。那么这个异常就应该有一个可以查询到所缺的钱的数额。因此调用者就可以把这个数值回复给用户。详情参见Item75.

> To summarize, throw checked exceptions for recoverable conditions and unchecked exceptions for programming errors. When in doubt, throw unchecked exceptions. Don’t define any throwables that are neither checked exceptions nor runtime exceptions. Provide methods on your checked exceptions to aid in recovery.

总结一下，对于可以恢复的情况，抛出受检异常，而对于程序错误，抛出非受检的异常。无法确定是否可以恢复的时候，抛出非受检异常。不要定义任何的既不是受检异常也不是运行时异常得的可抛出结构。给你的受检异常提供一些方法来帮助恢复异常。