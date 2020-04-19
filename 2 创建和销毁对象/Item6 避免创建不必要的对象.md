### Item6 避免创建不必要的对象

> It is often appropriate to reuse a single object instead of creating a new function- ally equivalent object each time it is needed. Reuse can be both faster and more stylish. An object can always be reused if it is immutable (Item 17).
>
> As an extreme example of what not to do, consider this statement:

当我们每次需要使用一个功能上一样的对象时，相对于创建一个新的对象，重复使用某个对象是更合适的。复用对象除了性能上更快以外，也更加优雅。不可变对象总是可以复用的（Item17）。

下面是一个极端的反面示例。

```java
String s = new String("bikini"); // DON'T DO THIS!
```

>The statement creates a new String instance each time it is executed, and none of those object creations is necessary. The argument to the String constructor ("bikini") is itself a String instance, functionally identical to all of the objects created by the constructor. If this usage occurs in a loop or in a frequently invoked method, millions of String instances can be created needlessly.
>
>The improved version is simply the following:

以上这段代码在每次调用的时候都会创建一个新的String对象实例，且这些对象都是没有必要创建的。String构造器的参数"bikini"本身就是一个String实例，在功能上和通过构造器创建的所有的对象相同。如果这种用法写在一个循环或者一个频繁调用的方法里，就会导致程序创建无数个不必要的String实例。

改进的方法很简单，如下所示：

```java
String s = "bikini";
```

> This version uses a single String instance, rather than creating a new one each time it is executed. Furthermore, it is guaranteed that the object will be reused by any other code running in the same virtual machine that happens to contain the same string literal [JLS, 3.10.5].

这个版本就只使用了一个String实例，而不是每次调用的时候都创建了一个新的实例。并且，在同一个虚拟机里，在代码中，当我们使用内容相同的字符串时，这个字符创实例都将被复用[JLS, 3.10.5]。

> You can often avoid creating unnecessary objects by using *static factory methods* (Item 1) in preference to constructors on immutable classes that provide both. For example, the factory method Boolean.valueOf(String) is preferable to the constructor Boolean(String), which was deprecated in Java 9. The constructor *must* create a new object each time it’s called, while the factory method is never required to do so and won’t in practice. In addition to reusing immutable objects, you can also reuse mutable objects if you know they won’t be modified.
>
> Some object creations are much more expensive than others. If you’re going to need such an “expensive object” repeatedly, it may be advisable to cache it for reuse. Unfortunately, it’s not always obvious when you’re creating such an object. Suppose you want to write a method to determine whether a string is a valid Roman numeral. Here’s the easiest way to do this using a regular expression:

对于同时提供了静态工厂方法（Item1）和构造器的类，我们应该优先使用静态工厂方法来避免创建不必要的对象。比如，工厂方法Boolean.valueOf(String) 优先于构造器Boolean(String) 。构造器每次被调用的时候都会创建新的对象，而工厂方法就没有这样的限制，并且实际应用中我们也不会这样做。除了复用不可变对象以外，也可以复用那些你知道不会被修改的可变对象。

有些对象的创建代价相当昂贵。当需要重复使用一个“昂贵的对象”时，建议将它缓存下来以复用。遗憾的是，在创建一个这种对象的时候，很多时候不是很明显。假如你想写一个方法去判断一个字符串是否是一个有效的罗马数字，最建档的方法是使用正则表达式，如下：

```java
// Performance can be greatly improved!
static boolean isRomanNumeral(String s) {
    return s.matches("^(?=.)M*(C[MD]|D?C{0,3})" + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$"); 
}
```

> The problem with this implementation is that it relies on the String.matches method. **While** **String.matches** **is the easiest way to check if a string matches a regular expression, it’s not suitable for repeated use in performance-critical situations.** The problem is that it internally creates a Pattern instance for the regular expression and uses it only once, after which it becomes eligible for garbage collection. Creating a Pattern instance is expensive because it requires compiling the regular expression into a finite state machine.
>
> To improve the performance, explicitly compile the regular expression into a Pattern instance (which is immutable) as part of class initialization, cache it, and reuse the same instance for every invocation of the isRomanNumeral method:

以上这种事项依赖String.matches方法。**虽然 String.matches方法是判断某个字符是否符合一个正则表达式的最简单的方法，但是它不适合在注重性能的情况中反复使用。** 原因在于String.matches方法在内部会为正则表达式创建一个 Pattern 实例，并且该实例只使用一次，然后就会被垃圾回收掉。而创建一个Pattern实例的成本很高，因为它需要将正则表达式编译成一个有限状态机。

为了提升性能，应该显示的将正则表达式编译成一个Pattern实例（不可变），将它作为类初始化的一部分，缓存起来，然后在每次调用isRomanNumeral方法的时候，重复使用一个实例。如下：

```java
// Reusing expensive object for improved performance
public class RomanNumerals {
    private static final Pattern ROMAN = Pattern.compile("^(?=.)M*(C[MD]|D?C{0,3})"
                + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
    static boolean isRomanNumeral(String s) { 
        return ROMAN.matcher(s).matches();
    } 
}
```

> The improved version of isRomanNumeral provides significant performance gains if invoked frequently. On my machine, the original version takes 1.1 μs on an 8-character input string, while the improved version takes 0.17 μs, which is 6.5 times faster. Not only is the performance improved, but arguably, so is clarity. Making a static final field for the otherwise invisible Pattern instance allows us to give it a name, which is far more readable than the regular expression itself.
>
> If the class containing the improved version of the isRomanNumeral method is initialized but the method is never invoked, the field ROMAN will be initialized needlessly. It would be possible to eliminate the initialization by *lazily initializing* the field (Item 83) the first time the isRomanNumeral method is invoked, but this is *not* recommended. As is often the case with lazy initialization, it would complicate the implementation with no measurable performance improvement (Item 67).

当该方法频繁调用的时候，升级版的isRomanNumeral方法有明显的性能提升。在作者的机器上，对于输入8字符的字符创，原始的版本花费1.1μs，而升级版花费 0.17 μs，快了6.5倍。升级版不仅仅带来了性能提升，还使得代码更加清晰。将不可见的Pattern实例做成静态常量域后，我们可以给它取一个名字，其可读性比正则表达式本身要高得多。

如果包含升级版的isRomanNumeral方法的类被初始化了，但是isRomanNumeral方法没有被调用，那么ROMAN域会被初始化，但是又没有用到。可以使用第一次调用isRomanNumeral方法时，延迟初始化域（Item83)的方法来消除这个不必要的初始化，但是不建议这样做。因为延迟初始化往往会使实现变得复杂，同时也不会带来显著的性能提升（Item67)。

> When an object is immutable, it is obvious it can be reused safely, but there are other situations where it is far less obvious, even counterintuitive. Consider the case of *adapters* [Gamma95]*,* also known as *views*. An adapter is an object that delegates to a backing object, providing an alternative interface. Because an adapter has no state beyond that of its backing object, there’s no need to create more than one instance of a given adapter to a given object.
>
> For example, the keySet method of the Map interface returns a Set view of the Map object, consisting of all the keys in the map. Naively, it would seem that every call to keySet would have to create a new Set instance, but every call to keySet on a given Map object may return the same Set instance. Although the returned Set instance is typically mutable, all of the returned objects are functionally identical: when one of the returned objects changes, so do all the others, because they’re all backed by the same Map instance. While it is largely harmless to create multiple instances of the keySet view object, it is unnecessary and has no benefits.
>
> Another way to create unnecessary objects is *autoboxing*, which allows the programmer to mix primitive and boxed primitive types, boxing and unboxing automatically as needed. **Autoboxing blurs but does not erase the distinction between primitive and boxed primitive types.** There are subtle semantic distinctions and not-so-subtle performance differences (Item 61). Consider the following method, which calculates the sum of all the positive int values. To do this, the program has to use long arithmetic because an int is not big enough to hold the sum of all the positive int values:

对于不可变对象而言，很明显可以安全的复用，但在一些场景里，就不是那么明显，甚至有些违反直觉。比如适配器（adapter) [Gamma95],也被称为视图（view）。适配器是指这样一个对象：将功能的具体实现委托给后备对象（backing object），并提供一个可选择的接口。因为适配器除了其后备对象以外，没有其他的状态信息，因此对于给定对象的特定适配器，没有必要和创建多个适配器实例。

比如，Map接口的keSet方法将返回一个map对象的Set视图，该Set视图中包括了map对象中的所有的key。我们坑会天真地以为每次调用keySet方法将会创建一个新的Set实例，但事情不是这样的，针对一个给定的map对象，每次调用keySet方法将会返回同一个Set实例。虽然返回的Set通常是可变的，但是每个返回的对象功能是一样的，当其中一个返回对象改变的时候，其他所有的对象也会改变，因为他们是由同一个map对象支撑的。虽然创建对个keySet视图并没什么大的危害，但是也没什么必要和好处。

另外一个创建不必要对象的情况是自动装箱（autoboxing)，自动装箱和拆箱（unboxing）使得程序员可以将基本类型与其对应的封装类混用。**自动装箱模糊了基本类型和其封装类之间的差别，但是并没有完全消除**。有着微妙的语义上的差别，和明显的性能上的差别。考虑一个计算所有int正整数的和的方法，如下。程序中使用了long，因为int装不下。

```java
// Hideously slow! Can you spot the object creation?
private static long sum() {
    Long sum = 0L;
    for (long i = 0; i <= Integer.MAX_VALUE; i++)
        sum += i;
    return sum; 
}
```

> This program gets the right answer, but it is *much* slower than it should be, due to a one-character typographical error. The variable sum is declared as a Long instead of a long, which means that the program constructs about 2^31 unnecessary Long instances (roughly one for each time the long i is added to the Long sum). Changing the declaration of sum from Long to long reduces the runtime from 6.3 seconds to 0.59 seconds on my machine. The lesson is clear: **prefer primitives to boxed primitives, and watch out for unintentional autoboxing.**
>
> This item should not be misconstrued to imply that object creation is expensive and should be avoided. On the contrary, the creation and reclamation of small objects whose constructors do little explicit work is cheap, especially on modern JVM implementations. Creating additional objects to enhance the clarity, simplicity, or power of a program is generally a good thing.
>
> Conversely, avoiding object creation by maintaining your own *object pool* is a bad idea unless the objects in the pool are extremely heavyweight. The classic example of an object that *does* justify an object pool is a database connection. The cost of establishing the connection is sufficiently high that it makes sense to reuse these objects. Generally speaking, however, maintaining your own object pools clutters your code, increases memory footprint, and harms performance. Modern JVM implementations have highly optimized garbage collectors that easily outperform such object pools on lightweight objects.

这个程序可以计算得到正确的值，但是程序确比实际情况更加慢一些，这是因为程序中打错了一个字符。变量sum被声明成了Long而不是long，这使得程序创建了2^31不需要的Long实例（大约每次long和Long相加都创建一个)。将Long改成long以后，在作者的机器上，程序耗时从6.3秒变成了0.59秒。结论很明显：**优先使用基本类型而不是其封装类，且当心无意识的自动装箱**。

不要误以为本条目的意思是对象创建成本高，需要避免创建对象。相反地，对于构造器只做少量明确工作的小对象，其创建和回收成本是很低很低的，尤其是对于现代JVM而言，更是如此。创建一些额外的对象有助于提升程序的清晰性、简洁性、功能性，这也是通常一件好事情。

相反，通过维护自己的对象池（object pool）来避免对象创建是一个不好的做法，除非对象池里的对象都是非常重量级的。最经典的适合使用对象池管理的对象是数据库连接，因为数据库连接创建的代价非常高，因此重用这些对象非常有必要。总的来说，维护自己的对象池会让代码变得复杂，同时增加内存使用，还可能损害性能。现代JVM实现对垃圾回收做了相当好的优化，其性能很容易比一些轻量对象的对象池好。

> The counterpoint to this item is Item 50 on *defensive copying*. The present item says, “Don’t create a new object when you should reuse an existing one,” while Item 50 says, “Don’t reuse an existing object when you should create a new one.” Note that the penalty for reusing an object when defensive copying is called for is far greater than the penalty for needlessly creating a duplicate object. Failing to make defensive copies where required can lead to insidious bugs and security holes; creating objects unnecessarily merely affects style and performance.

与本节相对应的是第50节的“保护性拷贝”，本节说：“当你应该重用一个已经存在的对象时，不要去创建一个新的对象“。而第50节说：”当你需要创建一个新的对象的时候，不要使用已经存在的”。注意，在推荐拷贝性保护的时候，重用对象付出的代价远远大于创建一个不必要的重复对象。在必要时不使用拷贝性保护可能会导致一些影藏的bug和安全漏洞，而创建一个不必要的对象只是会影响代码风格和性能而已。