### Item17 使可变性最小化

> An immutable class is simply a class whose instances cannot be modified. All of the information contained in each instance is fixed for the lifetime of the object, so no changes can ever be observed. The Java platform libraries contain many immutable classes, including String, the boxed primitive classes, and BigInteger and BigDecimal. There are many good reasons for this: Immutable classes are easier to design, implement, and use than mutable classes. They are less prone to error and are more secure.

不可变类就是其实例不可以修改的类。在每个实例中的所有的信息在这个对象的生命周期里都是固定的，不会有任何的改变。Java平台类库包含了很多的不可变类，包括String、基本类型封装类、BigInteger、BigDecimal。不可变类之所以存在有以下几个理由：不可变类相对于可变类更容易设计、实现、使用；也更加不容易出错、比较安全。

> To make a class immutable, follow these five rules:
>
> 1. **Don’t provide methods that modify the object’s state** (known as *mutators*).
> 2. **Ensure that the class can’t be extended.** This prevents careless or malicious subclasses from compromising the immutable behavior of the class by behaving as if the object’s state has changed. Preventing subclassing is generally accomplished by making the class final, but there is an alternative that we’ll discuss later.
> 3. **Make all fields final.** This clearly expresses your intent in a manner that is enforced by the system. Also, it is necessary to ensure correct behavior if a reference to a newly created instance is passed from one thread to another without synchronization, as spelled out in the *memory model* [JLS, 17.5; Goetz06, 16].
> 4. **Make all fields private.** This prevents clients from obtaining access to mutable objects referred to by fields and modifying these objects directly. While it is technically permissible for immutable classes to have public final fields containing primitive values or references to immutable objects, it is not recommended because it precludes changing the internal representation in a later release (Items 15 and 16).
> 5. **Ensure exclusive access to any mutable components.** If your class has any fields that refer to mutable objects, ensure that clients of the class cannot obtain references to these objects. Never initialize such a field to a client-provided object reference or return the field from an accessor. Make *defensive copies* (Item 50) in constructors, accessors, and readObject methods (Item 88).

要使一个类不可变，需要遵循以下规则：

1. **不要提供可以修改这个对象的状态的方法**（也称为设值方法）。
2. **保证这个类不可以被继承**。这可以防止一些不小心或者恶意的子类通过假装对象的状态已经被修改来破坏类的不可变性。为防止子类继承，一般来说，可以通过将类设为final来实现。还有一种可选择的方式后面会讨论。
3. **把所有的域设为final**。这可以通过系统强制的方法清除的表达你的意图。并且，这对于在没有同步机制的时候，把一个新创建的实例引用从一个线程传递到另一个线程时，保证正确的行为来说，有很必要。正如[JLS, 17.5; Goetz06, 16]的内存模型所描述的那样。
4. **把所有域都设为private**。这可以防止客户端获得可变对象的域的引用从而直接修改对象。虽然不可变对象还是可以有包含基本数据类型或者不可变对象引用的公有final域，但是还是不推荐这么做，因为在后面的保本中，就不可以改变内部的表示方法了（Item15和Item16）。
5. **确保可变组件的互斥访问**。如果类有一些域引用了可变对象，要确保这个类的客户端无法获得这些对象的引用。永远不要使用客户端提供的对象引用来初始化这种域，也不要通过访问器返回这些域的引用。在构造器、访问方法和readObject方法（Item88)里使用保护性拷贝（Item50）。

> Many of the example classes in previous items are immutable. One such class is PhoneNumber in Item 11, which has accessors for each attribute but no corre- sponding mutators. Here is a slightly more complex example:

在前面提供的示例类中，有很多都是不可变的，比如在Item11里的PhoneNumber，有访问每一域的方法，却没有对应的设值方法。下面是一个稍微复杂一点的例子：

```java
// Immutable complex number class 
public final class Complex { 
    private final double re; 
    private final double im;
    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }
    public double realPart()      { return re; }
    public double imaginaryPart() { return im; }
    public Complex plus(Complex c) {
        return new Complex(re + c.re, im + c.im);
		}
    public Complex minus(Complex c) {
        return new Complex(re - c.re, im - c.im);
		}
    public Complex times(Complex c) {
        return new Complex(re * c.re - im * c.im,
                           re * c.im + im * c.re);
		}
    public Complex dividedBy(Complex c) {
        double tmp = c.re * c.re + c.im * c.im;
        return new Complex((re * c.re + im * c.im) / tmp,
                           (im * c.re - re * c.im) / tmp);
		}
    @Override public boolean equals(Object o) {
       if (o == this)
           return true;
       if (!(o instanceof Complex))
           return false;
       Complex c = (Complex) o;
       // See page 47 to find out why we use compare instead of == 
       return Double.compare(c.re, re) == 0 && Double.compare(c.im, im) == 0;
    }
    @Override public int hashCode() {
        return 31 * Double.hashCode(re) + Double.hashCode(im);
    }
    @Override public String toString() {
        return "(" + re + " + " + im + "i)";
	  }
}
```

> This class represents a *complex number* (a number with both real and imaginary parts). In addition to the standard Object methods, it provides accessors for the real and imaginary parts and provides the four basic arithmetic operations: addition, subtraction, multiplication, and division. Notice how the arithmetic operations create and return a new Complex instance rather than modifying this instance. This pattern is known as the *functional* approach because methods return the result of applying a function to their operand, without modifying it. Contrast it to the *procedural* or *imperative* approach in which methods apply a procedure to their operand, causing its state to change. Note that the method names are prepositions (such as plus) rather than verbs (such as add). This emphasizes the fact that methods don’t change the values of the objects. The BigInteger and BigDecimal classes did *not* obey this naming convention, and it led to many usage errors.

这个类表示一个复数（一个包含实部和虚部的数）。除了标准的Object方法以外，还提供了对实部和虚部的访问方法，以及四个基本的算数操作：加、减、乘、除。需要注意的是，这些算数操作是创建并返回一个新的Complex实例的，而不是修改原有实例。它被称为函数（functional）方法，这种方法在不修改操作数的条件下，返回一个对操作数应用了某个函数的结果。与之对应的是函数（procedural）和过程（imperative）方法，这两种方法在操作数上执行函数的时候，会改变其状态。需要注意的是，这些方法的名字都是介词（比如plus）而不是动词（比如add）。这进一步表明了这个方法不会修改对象的值。BigInteger和BigDecimal类并没有遵循这样的命名约定，因此导致了很多用法有问题。

> The functional approach may appear unnatural if you’re not familiar with it, but it enables immutability, which has many advantages. **Immutable objects are simple.** An immutable object can be in exactly one state, the state in which it was created. If you make sure that all constructors establish class invariants, then it is guaranteed that these invariants will remain true for all time, with no further effort on your part or on the part of the programmer who uses the class. Mutable objects, on the other hand, can have arbitrarily complex state spaces. If the documentation does not provide a precise description of the state transitions performed by mutator methods, it can be difficult or impossible to use a mutable class reliably.

如果你不熟悉这些函数方法的话，可能看起来会有点不自然，但这些方法带来了有很多优点的不可变性。**不可变的对象很简单**。一个不可变的对象始终处在一个确定的状态里，也就是创建时的状态。如果你保证了所有的构造器都建立了类的约束关系，那么就可以保证这些约束关系会一直存在，不需要你或者使用这个类的程序员在做其他的努力。另一方面，不可变对象可以有任意多的状态空间，如果其文档没有对设值方法带来的状态转换提供一个准确的描述，要可靠的使用一个可变对象就会很困难，甚至不可能。

> **Immutable objects are inherently thread-safe; they require no synchronization.** They cannot be corrupted by multiple threads accessing them concurrently. This is far and away the easiest approach to achieve thread safety. Since no thread can ever observe any effect of another thread on an immutable object, **immutable objects can be shared freely.** Immutable classes should therefore encourage clients to reuse existing instances wherever possible. One easy way to do this is to provide public static final constants for commonly used values. For example, the Complex class might provide these constants:

**不可变对象本质上是线程安全的，不需要任何同步操作**。在多个线程并发访问的时候，也不会被破坏。毫无疑问这是实现线程安全的最简单的方法。由于没有一个线程可以察觉到其他线程对不可变对象的影响，**那么不可变对象就可以自由地共享**。因此不可变类应该鼓励其客户端尽可能的重用已经存在的对象。一个很简单的方法对于常用的值，提供公有静态final常量。如下，Complex类可能提供如下几个实例：

```java
public static final Complex ZERO = new Complex(0, 0);
public static final Complex ONE  = new Complex(1, 0);
public static final Complex I    = new Complex(0, 1);
```

> This approach can be taken one step further. An immutable class can provide static factories (Item 1) that cache frequently requested instances to avoid creating new instances when existing ones would do. All the boxed primitive classes and BigInteger do this. Using such static factories causes clients to share instances instead of creating new ones, reducing memory footprint and garbage collection costs. Opting for static factories in place of public constructors when designing a new class gives you the flexibility to add caching later, without modifying clients.

这种方法可以进一步扩展，一个不可变类可以提供一个静态工厂方法（Item1）,用于缓存常用的实例，当已经存在这个实例的时候，可以避免创建一个新的实例。所有的基本类型的封装类和BigInteger都是这么做的。使用静态工厂方法可以让客户端共用实例，而不是创建一个新的，可以减少内存的使用以及降低垃圾回收的成本。在创建一个新的类的时候就用静态工厂方法来代替公有构造器还有一个好处：在后面要添加缓存的时候，可以不用影响到客户端。

> A consequence of the fact that immutable objects can be shared freely is that you never have to make *defensive copies* of them (Item 50). In fact, you never have to make any copies at all because the copies would be forever equivalent to the originals. Therefore, you need not and should not provide a clone method or *copy constructor* (Item 13) on an immutable class. This was not well understood in the early days of the Java platform, so the String class does have a copy constructor, but it should rarely, if ever, be used (Item 6).

“不可变对象可以自由共享”带来的一个事实是，就不在需要使用保护性拷贝了（Item50）。事实上，你不需要做任何的复制，因为这些复制品和原始对象一致都是一样的。因此，你不需要也不应该为不可变对象提供一个复制构造器或者clone方法。在早期的Java平台中，并没有很好的理解这一点，因此String类还是有一个复制构造器，但是应该尽量少用它（Item6）。

> **Not only can you share immutable objects, but they can share their internals.** For example, the BigInteger class uses a sign-magnitude representation internally. The sign is represented by an int, and the magnitude is represented by an int array. The negate method produces a new BigInteger of like magnitude and opposite sign. It does not need to copy the array even though it is mutable; the newly created BigInteger points to the same internal array as the original.

不仅仅可以共享不可变对象，也可以共享其内部状态。比如，BigInteger类内部使用了符号-数值的表示方法。符号用一个int来表示，数值用一个int数组来表示。negate（相反数）的方法创建了一个拥有相同的数值和相反的符号的新的BigInteger对象。在这里，即使数组是可变的，也不需要复制这个数组，在新的BigInteger对象和原始对象中指向同一个数组。

> **Immutable objects make great building blocks for other objects,** whether mutable or immutable. It’s much easier to maintain the invariants of a complex object if you know that its component objects will not change underneath it. A special case of this principle is that immutable objects make great map keys and set elements: you don’t have to worry about their values changing once they’re in the map or set, which would destroy the map or set’s invariants.
>
> **Immutable objects provide failure atomicity for free** (Item 76). Their state never changes, so there is no possibility of a temporary inconsistency.

**不可变对象为其他的对象（不论是可变对象还是不可变对象）提供了大量的组件**。如果你知道复杂对象内部的组件对象不会发生变化的话，要维护其约束就要容易得多。这种原则的一个特殊的例子就是使用不可变对象作为map的key和set中的元素：你不用担心这些在map和set中的值会发生变化，而导致破坏了map和set的约束。

**不可变对象也无偿地提供了失败的原子性**。他们的状态永远都不会变化，因此不可能出现临时不一致的情况。

> **The major disadvantage of immutable classes is that they require a separate object for each distinct value.** Creating these objects can be costly, especially if they are large. For example, suppose that you have a million-bit BigInteger and you want to change its low-order bit:

不可变对象的主要的缺点是，对于每个不同的值，需要一个单独的对象。创建这些对象需要付出代价，尤其是这些对象很大的时候。比如，假定你有一个位数上百万的BigInteger，然后，你想改变其低位的值。

```java
BigInteger moby = ...;
moby = moby.flipBit(0);
```

> The flipBit method creates a new BigInteger instance, also a million bits long, that differs from the original in only one bit. The operation requires time and space proportional to the size of the BigInteger. Contrast this to java.util.BitSet. Like BigInteger, BitSet represents an arbitrarily long sequence of bits, but unlike BigInteger, BitSet is mutable. The BitSet class provides a method that allows you to change the state of a single bit of a million- bit instance in constant time:

flipBit方法创建了一个新的BigInteger实例，也有上百万的位数，但和原始的对象，只有一位不同。这个操作需要的时间和空间和BIgInteger的大小成正比。与之对应的是 java.util.BitSet，和BigInteger类似，BigSet也可以表示任意位数长的序列，但是和BigInteger不同的是，BigSet是可变的。BigSet类提供了一个可以在常数时间内修改上百万位数的实例的其中一位，如下：

```java
BitSet moby = ...;
moby.flip(0);
```

> The performance problem is magnified if you perform a multistep operation that generates a new object at every step, eventually discarding all objects except the final result. There are two approaches to coping with this problem. The first is to guess which multistep operations will be commonly required and to provide them as primitives. If a multistep operation is provided as a primitive, the immutable class does not have to create a separate object at each step. Internally, the immutable class can be arbitrarily clever. For example, BigInteger has a package-private mutable “companion class” that it uses to speed up multistep operations such as modular exponentiation. It is much harder to use the mutable companion class than to use BigInteger, for all of the reasons outlined earlier. Luckily, you don’t have to use it: the implementors of BigInteger did the hard work for you.

当执行每步都会生成一个新对象的多步操作，且除了最后的结果，其他的对象都会被抛弃时，这个性能问题就更加明显了。这里有两种方法可以觉得这个问题，第一个方法是，先猜测通常情况下需要哪些多步操作，然后将这些多步方法作为基本方法提供。如果一个多步操作已经有了对应的基本方法，这个不可变类就不用每一步都创建一个单独的对象了。在其内部，不可变对象可以很灵活。比如，BigInteger有一个包级私有的可变“配套类（companion class），专门用来对例如模指数（modular exponentiation）这样的多步操作进行加速。由于一些前面提到的原因，直接使用其可变的配套类相对于BigInteger，要困难得多。幸运的是，你也不需要直接使用它，BigInteger已经替你做了这复杂的实现工作。

> The package-private mutable companion class approach works fine if you can accurately predict which complex operations clients will want to perform on your immutable class. If not, then your best bet is to provide a *public* mutable companion class. The main example of this approach in the Java platform libraries is the String class, whose mutable companion is StringBuilder (and its obsolete predecessor, StringBuffer).

当你能准确地预测到客户端将对你的不可变类采取的复杂操作的时候，包级私有的可变配套类方法可以工作得很好。如果做不到，最好的方法就是提供一个公有的可变的配套类。这种方法在Java平台中最典型的例子就是String类，它的可变的伙伴类是StringBuilder（及其被废弃的前任StringBuffer）。

> Now that you know how to make an immutable class and you understand the pros and cons of immutability, let’s discuss a few design alternatives. Recall that to guarantee immutability, a class must not permit itself to be subclassed. This can be done by making the class final, but there is another, more flexible alternative. Instead of making an immutable class final, you can make all of its constructors private or package-private and add public static factories in place of the public constructors (Item 1). To make this concrete, here’s how Complex would look if you took this approach:

现在，你已经知道了如何去实现一个不可变类，也理解了不可变性的优点和缺点。让我们再来讨论一些可选择的设计方式。为了确保不可变性，这个类绝对不能允许自己被子类化。这个可以通过将类设为final来实现，按时还有另外一个更灵活的选择。你可以把这个类的构造器设为私有的或者包级私有的，并且提供一个静态的工厂方法来代替公有的构造器（Item1）。以Complex为例，来看一下如何使用这种方法：

```java
// Immutable class with static factories instead of constructors
   public class Complex {
       private final double re;
       private final double im;
			 private Complex(double re, double im) { 
			 		 this.re = re;
           this.im = im;
       }
			 public static Complex valueOf(double re, double im) { 
					 return new Complex(re, im);
			 }
       ... // Remainder unchanged
   }
```

> This approach is often the best alternative. It is the most flexible because it allows the use of multiple package-private implementation classes. To its clients that reside outside its package, the immutable class is effectively final because it is impossible to extend a class that comes from another package and that lacks a public or protected constructor. Besides allowing the flexibility of multiple implementation classes, this approach makes it possible to tune the performance of the class in subsequent releases by improving the object-caching capabilities of the static factories.

这个方法通常是最好的选择。这是最灵活的，因为它可以有多个包级私有的实现类。对于包外的客户端而言，这个不可变类实际上是final的，因为不可以从另外一个包继承寄一个缺乏公开或受保护的构造器的类。除了可以带来多个实现类的灵活性以外，这种方法还可以在以后的版本中通过提升静态工厂方法的对象缓存能力，来调节类的性能。

> It was not widely understood that immutable classes had to be effectively final when BigInteger and BigDecimal were written, so all of their methods may be overridden. Unfortunately, this could not be corrected after the fact while preserving backward compatibility. If you write a class whose security depends on the immutability of a BigInteger or BigDecimal argument from an untrusted client, you must check to see that the argument is a “real” BigInteger or BigDecimal, rather than an instance of an untrusted subclass. If it is the latter, you must defensively copy it under the assumption that it might be mutable (Item 50):

在编写BigDecimal和BigInteger的时候，对于”不可变类必须是final“这个规定还没有被广泛理解，因此其所有的方法都可以被重写。遗憾的是，为了保持后向兼容，这个问题一直没有被解决。如果你编写了一个类，这个类的安全性依赖于来自不可信任的客户端的BigInteger和BigDecimal域的不可变性，你就必须要检查这个域是真正的BigInteger或者BigDecimal，还是其不可信任的子类的实例。如果是后者，在假定这个是可变的前提下，必须进行保护性拷贝。

```java
 public static BigInteger safeInstance(BigInteger val) {
       return val.getClass() == BigInteger.class ?
               val : new BigInteger(val.toByteArray());
}
```

> The list of rules for immutable classes at the beginning of this item says that no methods may modify the object and that all its fields must be final. In fact these rules are a bit stronger than necessary and can be relaxed to improve performance. In truth, no method may produce an *externally visible* change in the object’s state. However, some immutable classes have one or more nonfinal fields in which they cache the results of expensive computations the first time they are needed. If the same value is requested again, the cached value is returned, saving the cost of recalculation. This trick works precisely because the object is immutable, which guarantees that the computation would yield the same result if it were repeated.

在本节开头提到的不可变对象需遵循的规则列表里说，没有方法会修改对象，且所有的域都必须是final的。事实上，这些规则有点过分强硬，但是没有那么有必要，为了提升性能可以放宽一些。事实上，应该说成”没有方法可以对对象的状态产生“外部可见”的改变“。在很多的不可见类中都有一个或者多个非final域，当第一次执行计算结果的时候，用来存储一些计算消耗大的结果。如果再次有相同的请求，就将返回缓存的值，就可以减少计算的消耗。这个方式可以很好地工作，因为对象是不可变的，就保证了每次重复计算都会产生相同的结果。

> For example, PhoneNumber’s hashCode method (Item 11, page 53) computes the hash code the first time it’s invoked and caches it in case it’s invoked again. This technique, an example of *lazy initialization* (Item 83), is also used by String.

比如，PhoneNumber的hashCode方法在第一次调用的时候会计算hash值，然后会缓存起来，以备后来再次调用。这种技术是”延迟初始化“的一个例子，String里面也用了。

> One caveat should be added concerning serializability. If you choose to have your immutable class implement Serializable and it contains one or more fields that refer to mutable objects, you must provide an explicit readObject or readResolve method, or use the ObjectOutputStream.writeUnshared and ObjectInputStream.readUnshared methods, even if the default serialized form is acceptable. Otherwise an attacker could create a mutable instance of your class. This topic is covered in detail in Item 88.

关于序列化，有一条告诫必须要在这里提出来。如果你的不可变类实现了序列化，并且它包含了一个或者多个的域引用了可变对象，你必须要提供一个明确的readObject 或者 readResolve 方法 ，或者使用ObjectOutputStream.writeUnshared 和 ObjectInputStream.readUnshared 方法，即使这个默认的序列化形式是可以接受的。否则攻击者可能会写一个你的类的可变实例，这个话题在Item88里还会详细讨论到。

> To summarize, resist the urge to write a setter for every getter. **Classes should be immutable unless there’s a very good reason to make them mutable.** Immutable classes provide many advantages, and their only disadvantage is the potential for performance problems under certain circumstances. You should always make small value objects, such as PhoneNumber and Complex, immutable. (There are several classes in the Java platform libraries, such as java.util.Date and java.awt.Point, that should have been immutable but aren’t.) You should seriously consider making larger value objects, such as String and BigInteger, immutable as well. You should provide a public mutable companion class for your immutable class *only* once you’ve confirmed that it’s necessary to achieve satisfactory performance (Item 67).

总结一下，坚决不要为每一个getter方法提供一个setter方法。**除非有很恰当的原因让类是可变的，否则应该做成不可变类。**不可变类有很多的有点，其唯一的缺陷是在某些特定的情况下会造成潜在的性能问题。对于一些小的值对象，比如PhoneNumber和Complex，总是需要做成不可变的（但在Java平台类库中，有几个类应该是不可变的但却不是，比如 java.util.Date 和 java.awt.Point）。你需要谨慎的考虑将大的值对象做成不可变的，比如String和BIgInteger。当你确定有必要实现让人满意的性能的时候，你应该为你的不可变类提供一个公有的可变的配套类（Item67）。

> There are some classes for which immutability is impractical. **If a class cannot be made immutable, limit its mutability as much as possible.** Reducing the number of states in which an object can exist makes it easier to reason about the object and reduces the likelihood of errors. Therefore, make every field final unless there is a compelling reason to make it nonfinal. Combining the advice of this item with that of Item 15, your natural inclination should be to **declare every field** **private final** **unless there’s a good reason to do otherwise.**

有一些类，要做成不可变的是不现实的。**如果一个类不能做成不可变的，就应该尽可能的限制其可变性**。减少对象中存在的状态，可以让这个对象更容易分析，也不容易出错。除非有不可抗拒的理由需要将域设为非final，否则每个域都应该为final。结合这条和Item15的建议，你应该自然而然地倾向于：**把所有的域都声明为private final的，除非也特别好的理由要用其他的方式去做**。

> **Constructors should create fully initialized objects with all of their invariants established.** Don’t provide a public initialization method separate from the constructor or static factory unless there is a *compelling* reason to do so. Similarly, don’t provide a “reinitialize” method that enables an object to be reused as if it had been constructed with a different initial state. Such methods generally provide little if any performance benefit at the expense of increased complexity.

**构造器应该建立所有的约束关系，并创建完全初始化的对象**。除了构造器和静态工厂方法以外，除非有难以抗拒的理由，否则不要提供其他的公有初始化方法。同样地，不要提供一个”重新初始化“的方法，来允许一个对象被重用，就像这个对象是用另一个不同的初始化状态创建出来的一样。这类方法只能带来一点点的性能提升，却极大地增加了复杂性。

> The CountDownLatch class exemplifies these principles. It is mutable, but its state space is kept intentionally small. You create an instance, use it once, and it’s done: once the countdown latch’s count has reached zero, you may not reuse it.
>
> A final note should be added concerning the Complex class in this item. This example was meant only to illustrate immutability. It is not an industrial-strength complex number implementation. It uses the standard formulas for complex multiplication and division, which are not correctly rounded and provide poor semantics for complex NaNs and infinities [Kahan91, Smith62, Thomas94].

CounDownLatch是这些原则的一个典型的例子。它是可变的，而且其状态空间被有意地设计得很小，你可以创建一个实例，使用一次，其任务就完成了。一但这个countDownLatch的计数变成了0 ，就不可以再重用了。

关于本节中的Complex类，还有最后一个需要注意的点是，这个例子只是用来说明不可变性的。它不是一个具有生产使用强度的负数实现。对于乘法和除法，使用了标准的计算公式，进行了不准确的四舍五入，对于负数NaNs和无穷大的数都没有提供很好的表示方法。

### 