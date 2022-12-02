## 2 创建和销毁对象

> This chapter concerns creating and destroying objects: when and how to create them, when and how to avoid creating them, how to ensure they are destroyed in a timely manner, and how to manage any cleanup actions that must precede their destruction.

本章关心的是对象的创建和销毁过程，主要涉及的问题有：什么时候需要创建对象以及如何去创建，什么时候需要避免创建对象以及如何避免，如何保证对象被及时销毁，和怎么去管理那些必须在对象被销毁前进行的清理动作。

### Item1 考虑使用静态方法替代构造器

> The traditional way for a class to allow a client to obtain an instance is to provide a public constructor. There is another technique that should be a part of every programmer’s toolkit. A class can provide a public static factory method, which is simply a static method that returns an instance of the class. Here’s a simple example from Boolean(the boxed primitive class for boolean). This method translates a boolean primitive value into a Boolean object reference:

当我们想获得一个类的实例对象时，一种传统的方法就是让该类提供一个公用的构造器。还有另外一种每个程序员都应该知道的方法是让该类提供一个共有的静态的工厂方法，该静态工厂方法，返回一个该类的实例对象。比如Boolean中便提供了一个这样的静态工厂方法，传入一个boolean的基本类型，返回一个Boolean对象引用。（Boolean是基本类型boolean的包装类)

```java
public static Boolean valueOf(boolean b) { 
    return b ? Boolean.TRUE : Boolean.FALSE;
}
```

> Note that a static factory method is not the same as the Factory Method pattern from *Design Patterns*[Gamma95]. The static factory method described in this item has no direct equivalent in Design Patterns.

需要注意的是，这里的静态工厂方法不直接等于设计模式里的工厂方法模式。

> A class can provide its clients with static factory methods instead of, or in addition to, public constructors. Providing a static factory method instead of a public constructor has both advantages and disadvantages.

总的来说就是，当我们需要为某个类创建实例对象的时候，除了让该类提供公用的构造器以外，还可以使用静态工厂方法。相对于公用构造器，静态工厂方法有以下几点优势和不足：

> **One advantage of static factory methods is that, unlike constructors, they have names**. If the parameters to a constructor do not, in and of themselves, describe the object being returned, a static factory with a well-chosen name is easier to use and the resulting client code easier to read. For example, the constructor BigInteger(int, int, Random), which returns a BigInteger that is probably prime, would have been better expressed as a static factory method named BigInteger.probablePrime. (This method was added in Java4.)

**第一个优势就是，静态工厂方法可以有方法名字，而构造器没有。** 如果一个构造器自己或者其参数无法准确的描述其返回的对象，那么一个名字取得比较好的静态工厂方法就是一个更好的选择，也会使得调用该方法的客户端代码更清晰易懂。比如，构造器 BigInteger(int, int, Random) 可能会返回一个素数，这种情况，使用一个名为BigInteger.probablePrime的静态工厂方法，就可以表达更清晰一些（该方法在Java4里已经有了）。

> A class can have only a single constructor with a given signature. Programmers have been known to get around this restriction by providing two constructors whose parameter lists differ only in the order of their parameter types. This is a really bad idea. The user of such an API will never be able to remember which constructor is which and will end up calling the wrong one by mistake. People reading code that uses these constructors will not know what the code does without referring to the class documentation.

当构造器方法的参数固定的时候，一个类里只能有一个这样的构造器。为了避开这个限制，有的程序员会采用交换不同类型参数的位置来提供两个这样的构造器。但这种方式是相当糟糕的，因为在这种情况下（两个构造器的参数个数和类型相同，只是位置不同），构造器的使用者，很容易就用错，导致调用了错误的方法。在阅读代码的时候，如果不去查看该类的说明文档的话。也很难弄清楚代码究竟干了些啥。

> **A second advantage of static factory methods is that, unlike constructors, they are not required to create a new object each time they’re invoked.** This allows immutable classes (Item 17) to use preconstructed instances, or to cache instances as they’re constructed, and dispense them repeatedly to avoid creating unnecessary duplicate objects.

**静态工厂方法的第二个优势就在于没有必要在每次调用的时候都返回一个新的对象，而构造器就不行。** 这个优点使得那些不可变类可以使用之前创建好的对象，或者缓缓存号已经创建好的对象，以重复的利用这些对象，避免创建没有必要的重复的对象。

> The Boolean.valueOf(boolean) method illustrates this technique: it never creates an object. This technique is similar to the Flyweight pattern [Gamma95]. It can greatly improve performance if equivalent objects are requested often, especially if they are expensive to create.

比如Boolean.valueOf(boolean)方法就是利用了这点，它永远不会创建对象。这种方式和[享元模式 (Flyweight pattern )](https://www.runoob.com/design-pattern/flyweight-pattern.html)类似，当总是频繁的创建相同的对象，且创建开销较大时，能很好的提升性能。

> The ability of static factory methods to return the same object from repeated invocations allows classes to maintain strict control over what instances exist at any time. Classes that do this are said to be instance-controlled.There are several reasons to write instance-controlled classes. Instance control allows a class to guarantee that it is a singleton (Item 3) or noninstantiable (Item 4). Also, it allows an immutable value class (Item 17) to make the guarantee that no two equal instances exist: a.equals(b) if and only if a == b*.* This is the basis of the Flyweight pattern [Gamma95]. Enum types (Item 34) provide this guarantee.

静态工厂方法的这种 当重复调用时 返回同一对象 的能力，使得该类可以严格的控制任何时刻可以存在的实例对象。我们将这种类称为实体控制类。一般在以下情况下编写实体控制类： 第一种情况是通过实体控制保证该类是单例(Item3)的，或者不可初始化的(Item4)；第二种情况是通过实体控制保证不可变类不会同时存在两个相同的实例，即只有当a==b为true的时候，a.equals(b) 为true。这些也是享元模式的基础。枚举类型也能提供这种保证。

> **A third advantage of static factory methods is that, unlike constructors, they can return an object of any subtype of their return type.** This gives you great flexibility in choosing the class of the returned object.
>
> One application of this flexibility is that an API can return objects without making their classes public. Hiding implementation classes in this fashion leads to a very compact API. This technique lends itself to interface-based frameworks(Item 20), where interfaces provide natural return types for static factory methods.
>
> Prior to Java 8, interfaces couldn’t have static methods. By convention, static factory methods for an interface named *Type* were put in a noninstantiable companion class(Item 4) named *Types*. For example, the Java Collections Framework has forty-five utility implementations of its interfaces, providing unmodifiable collections, synchronized collections, and the like. Nearly all of these implementations are exported via static factory methods in one noninstantiable class (java.util.Collections). The classes of the returned objects are all nonpublic.

静态工厂方法的第三个优点是可以返回原返回类型的任意子类型对象，而构造器不行。这个优点让我们可以更加灵活的选择类的返回对象。

这种灵活性的一个应用场景是，API可以返回一个对象，而该对象的类可以是非公有的。在这种情况下，影藏具体的实现类可以使得API更加简洁。这种技术适用于基于接口的框架，静态工厂方法的返回类型为接口类型，而实际返回对象为具体的实现类。

在Java8之前，接口里不能有静态方法。按照之前的传统，接口Type的静态工厂方法都被放在该接口的一个不可实例化的伙伴类Types里。比如，java集合框架里有45个接口的实现，包括不可修改集合，同步集合等等。几乎这些实现都可以在一个不可实例化类java.util.Collections里通过静态工厂方法返回。这些返回的对象所属类都是非公有的。[非公有类与内部类的区别](https://blog.csdn.net/yangfeisc/article/details/44492975)

> The Collections Framework API is much smaller than it would have been had it exported forty-five separate public classes, one for each convenience implementation. It is not just the bulk of the API that is reduced but the conceptual weight:the number and difficulty of the concepts that programmers must master in order to use the API. The programmer knows that the returned object has precisely the API specified by its interface, so there is no need to read additional class documentation for the implementation class. Furthermore, using such a static factory method requires the client to refer to the returned object by interface rather than implementation class, which is generally good practice (Item 64).
>
> As of Java 8, the restriction that interfaces cannot contain static methods was eliminated, so there is typically little reason to provide a noninstantiable companion class for an interface. Many public static members that would have been at home in such a class should instead be put in the interface itself. Note, however, that it may still be necessary to put the bulk of the implementation code behind these static methods in a separate package-private class. This is because Java 8 requires all static members of an interface to be public. Java 9 allows private static methods, but static fields and static member classes are still required to be public.

这种集合框架的api设计 比为45种实现分别返回公有类的设计 要简洁得多。这种减少不仅仅是数量上减少，也是概念上的减少。程序员为了使用这个api小掌握的概念的数量和复杂度都减少了。且程序员知道返回的对象是由他的接口精确确定的，不需要去阅读具体实现类的参考文档了。除此之外，当调用返回类型为接口类型的静态工厂方法时，要求客户端必须通过接口来引用返回的对象，而不是具体的实现类，这是一个非常好的编程习惯（Item64）。

在Java8里， 去除了接口不能包含静态方法的限制，因此没有理由再为一个接口提供一个不可实例化的伙伴类了。很多应该放在伙伴类里的静态变量，现在都应该放在接口里了。然而，由于Java8 中要求所有的静态成员必须是公有的，因此还是有必要将一部分代码放在一个单独的包私有类的静态方法里。在Java9里允许接口包含私有的静态方法，但是静态域和静态类成员还是必须是公有的。

>**A fourth advantage of static factories is that the class of the returned object can vary from call to call as a function of the input parameters.** Any subtype of the declared return type is permissible. The class of the returned object can also vary from release to release.
>
>The *EnumSet* class (Item 36) has no public constructors, only static factories. In the OpenJDK implementation, they return an instance of one of two subclasses, depending on the size of the underlying enum type: if it has sixty-four or fewer elements, as most enum types do, the static factories return a *RegularEnumSet* instance, which is backed by a single *long*; if the enum type has sixty-five or more elements, the factories return *a JumboEnumSet* instance, backed by a *long* array.
>
>The existence of these two implementation classes is invisible to clients. If *RegularEnumSet* ceased to offer performance advantages for small enum types, it could be eliminated from a future release with no ill effects. Similarly, a future release could add a third or fourth implementation of *EnumSet* if it proved beneficial for performance. Clients neither know nor care about the class of the object they get back from the factory; they care only that it is some subclass of *EnumSet*.

**静态工厂方法的第四个优势是，可以根据传入的参数返回不同类型的对象。**返回对象类型可以是声明的返回类型的任意子类型。返回对象的类型也可以根据发行版本的不同而不同。

EnumSet类(Item36)没有公有的构造器，只有静态工厂方法，在OpenJDK的实现中，其静态工厂方法会根据底层枚举类型的大小来返回两个子类中的一个实例。如果该枚举类和大多数实例一样，包含的元素少于或等于64个，那么其静态工厂方法将会返回一个 由单个long支持的RegularEnumSet实例；否则，将返回一个有long数组支持的JumboEnumSet实例。

这两个具体的实现类对于客户端来说是不可见的。如果RegularEnumSet对于小的枚举类型不在具有性能优势，那么在未来的版本里，将其移除，对于客户端，也不会有什么坏影响。同样地，在未来的版本中，也可以新增第三个或者第四个EnumSet的实现类用于提供更好的性能。客户端不用关心从工厂获取的对象的具体实现，只需要知道该对象是Enum的子类实例即可。

> **A fifth advantage of static factories is that the class of the returned object need not exist when the class containing the method is written.** Such flexible static factory methods form the basis of service provider frameworks, like the Java Database Connectivity API (JDBC). A service provider framework is a system in which providers implement a service, and the system makes the implementations available to clients, decoupling the clients from the implementations.
>
> There are three essential components in a service provider framework: a service interface, which represents an implementation; a provider registration API, which providers use to register implementations; and a service access API, which clients use to obtain instances of the service. The service access API may allow clients to specify criteria for choosing an implementation. In the absence of such criteria, the API returns an instance of a default implementation, or allows the client to cycle through all available implementations. The service access API is the flexible static factory that forms the basis of the service provider framework.

静态工厂方法的第五个优势在于，在编写包含该方法的类的时候，返回对象的类可以不存在。这种灵活的静态工厂方法组成了服务提供框架的基础，比如java数据库连接API (JDBC)，服务提供框架是指在一个系统中，提供者实现了一个服务，系统使得客户端可以使用这些实现，将客户端从具体实现中解耦出来。

一个服务提供框架包括三个基本的组件：用于展示实现的服务接口，用于提供者注册实现的提供者注册api，用于客户端获取服务实例的服务访问api。服务访问api根据客户端的指定内容选择具体的实现，如果没有指定，则返回一个默认的实现，或者是允许客户端循环使用所有可用的实现。服务访问api就是灵活的静态工厂方法，它们组成了服务提供者框架的基础。

>An optional fourth component of a service provider framework is a service provider interface, which describes a factory object that produce instances of the service interface. In the absence of a service provider interface, implementations must be instantiated reflectively (Item 65). In the case of JDBC, *Connection* plays the part of the service interface, *DriverManager.registerDriver* is the provider registration API, *DriverManager.getConnection* is the service access API, and *Driver* is the service provider interface.
>
>There are many variants of the service provider framework pattern. For example, the service access API can return a richer service interface to clients than the one furnished by providers. This is the *Bridge* pattern [Gamma95]. Dependency injection frameworks (Item 5) can be viewed as powerful service providers. Since Java 6, the platform includes a general-purpose service provider framework, *java.util.ServiceLoader*, so you needn’t, and generally shouldn’t, write your own (Item 59). JDBC doesn’t use *ServiceLoader*, as the former predates the latter.

除了以上三个组件外，另外一个可选择的组件是服务提供接口。服务提供接口描述了生产服务实例的工厂对象。在缺少服务提供接口是，具体的实现必须通过反着进行初始化。以JDBC为例，  Connection是服务接口，DriverManager.registerDriver是提供者注册api，DriverManager.getConnection是服务访问api，Driver是服务提供接口。

服务提供框架有很多的变种。比如服务访问api可以给客服端提供一个比提供者提供的服务更为丰富的服务，也就是桥接模式。依赖注入框架(Item5)可以看做是一个更强大的服务提供者。从Java6 开始，jdk中包含了一个通用的服务提供框架 java.util.ServiceLoader，所以你不需要也不应该写一个自己的框架(Item59）。JDBC没有使用ServiceLoader，因为JDBC在Java6之前就存在了。

> **The main limitation of providing only static factory methods is that classes without public or protected constructors cannot be subclassed.** For example, it is impossible to subclass any of the convenience implementation classes in the Collections Framework. Arguably this can be a blessing in disguise because it encourages programmers to use composition instead of inheritance (Item 18), and is required for immutable types (Item 17).
>
> **A second shortcoming of static factory methods is that they are hard for programmers to find.** They do not stand out in API documentation in the way that constructors do, so it can be difficult to figure out how to instantiate a class that provides static factory methods instead of constructors. The Javadoc tool may someday draw attention to static factory methods. In the meantime, you can reduce this problem by drawing attention to static factories in class or interface documentation and by adhering to common naming conventions. Here are some common names for static factory methods. This list is far from exhaustive:

**静态工厂方法主要的不足在于没有公有或者受保护的构造器的类不能子类化。**比如，在集合框架中，所有的便利实现都不能拥有子类。但这也鼓励程序员使用组合代替继承(Item18)，并且这也是不可变类型所需要的，因此也算是因祸得福了。

**静态工厂方法的另一个不足在于程序员很难找到他们。** 因为静态工厂方法不会像狗再起一样在API文档中标注出来。导致难以知道如何去初始化一个只提供静态工厂方法的类。Javadoc工具在未来可能会加标注在静态工厂方法上。同时，你可以多注意类和接口文档里的静态工厂方法，并且遵守通用的命名规则来减少这种问题。下面列举了一部分通用的静态工厂方法的名字。

>  • *from*—A type-conversion method that takes a single parameter and returns a corresponding instance of this type, for example:

from—一种类型转换方法，通过传入单个参数，返回一个该类型的实例。比如：

```java
Date d = Date.from(instant);
```

> **•** *of*—An aggregation method that takes multiple parameters and returns an instance of this type that incorporates them, for
> example:

of—一种聚合方法，通过传入多个参数，返回一个包含这些参数的类型实例。比如

```java
Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);
```

> • *valueOf*—A more verbose alternative to *from* and *of*, for example:

valueOf—用于替换from和of的一种更为详细的方法名。比如

```java
BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);
```

> • *instance or getInstance*—Returns an instance that is described by its parameters (if any) but cannot be said to have the same value, for example:

instance or getInstance—返回一个有参数（如果有的话）描述的实例，但参数一致的时候，实例也可能不一样。比如

```java
StackWalker luke = StackWalker.getInstance(options);
```

> • *create or newInstance*—Like instance or getInstance, except that the method guarantees that each call returns a new instance, for example:

create or newInstance— 和instance or getInstance类似，但是保证每次调用都返回新的实例。比如

```java
Object newArray = Array.newInstance(classObject, arrayLen);
```

>  • *getType*—Like getInstance, but used if the factory method is in a different class. Type is the type of object returned by the factory method, for example:

getType—和getInstance类似，在工厂方法包含在其他类里时使用，Type就是其返回实例的Type。比如。

```java
FileStore fs = Files.getFileStore(path);
```

>  •*newType*—Like newInstance, but used if the factory method is in a different class. Type is the type of object returned by the factory method, for example:

getType—和newInstance类似，在工厂方法包含在其他类里时使用，Type就是其返回实例的Type。比如。

```java
BufferedReader br = Files.newBufferedReader(path);
```

> •*type*—A concise alternative to *getType* and *newType*, for example:

type—getType和newType的更加简洁的表达，比如。

```java
List<Complaint> litany = Collections.list(legacyLitany);
```

> In summary, static factory methods and public constructors both have their uses, and it pays to understand their relative merits. Often static factories are preferable, so avoid the reflex to provide public constructors without first considering static factories.

总的说来，静态工厂方法和公有构造器各有各的用途，我们需要了解他们各自的优点。但是，通常情况下，静态工厂方法比公有构造器更适合，因此，我们要避免第一反应就是公有构造器，而不是静态工厂方法。