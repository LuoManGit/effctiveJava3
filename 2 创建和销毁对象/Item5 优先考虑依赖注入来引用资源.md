### Item5 优先考虑依赖注入来引用资源

> Many classes depend on one or more underlying resources. For example, a spell checker depends on a dictionary. It is not uncommon to see such classes implemented as static utility classes(Item 4):

一些类依赖一个或者更多的底层资源，比如，一个拼写的检查器依赖字典。我们常常见到这种类被做成静态工具类(Item4)，如下。

```java
// Inappropriate use of static utility - inflexible & untestable!
public class SpellChecker {
    private static final Lexicon dictionary = ...;
    private SpellChecker() {} // Noninstantiable
    public static boolean isValid(String word) { ... }
    public static List<String> suggestions(String typo) { ... }
}
```

>Similarly, it’s not uncommon to see them implemented as singletons (Item 3):

类似地，这种类也常常被做成了单例，如下：

```java
// Inappropriate use of singleton - inflexible & untestable!
public class SpellChecker {
    private final Lexicon dictionary = ...;
    private SpellChecker(...) {}
    public static INSTANCE = new SpellChecker(...);
    public boolean isValid(String word) { ... }
    public List<String> suggestions(String typo) { ... }
}
```

> Neither of these approaches is satisfactory, because they assume that there is only one dictionary worth using. In practice, each language has its own dictionary, and special dictionaries are used for special vocabularies. Also, it may be desirable to use a special dictionary for testing. It is wishful thinking to assume that a single dictionary will suffice for all time.
>
> You could try to have *SpellChecker* support multiple dictionaries by making the *dictionary* field nonfinal and adding a method to change the dictionary in an existing spell checker, but this would be awkward, error-prone, and unworkable in a concurrent setting. **Static utility classes and singletons are inappropriate for classes whose behavior is parameterized by an underlying resource.**
>
> What is required is the ability to support multiple instances of the class (in our example, SpellChecker), each of which uses the resource desired by the client (in our example, the dictionary). A simple pattern that satisfies this requirement is to **pass the resource into the constructor when creating a new instance**. This is one form of *dependency injection*: the dictionary is a *dependency* of the spell checker and is *injected* into the spell checker when it is created.

以上两种方式都不理想，因为他们都假定只有一个词典可用，而在实际应用中，每种语言都有自己的词典，而且一些特殊的词汇还有特殊的词典。同时，测试的时候可能还需要一个特殊的词典，因此假设只用一个词典就满足所有的需求是不现实的。

你可能会尝试将SpellChecker类的dictionary域设置为非final（nonfinal)，并添加一个方法来改变词典，以达到支持多词典的目标。但是这种方式具有使用不方便，容易出错，并行状态下不可行的问题。因此，静态工具类和单例都不适用于需要引用底层资源的类。

我们需要的是具有支持多个实例能力的类（在本例中指SpellChecker），每个实例使用客户端指定的资源（在本例中指dictionary）。一个简单的能满足需求的方法是**创建实例的时候，将资源传入构造器中**。这是*依赖注入*的一种形式：dictionary是SpellChecker的一个依赖，在SpellChecker实例创建的时候通过构造器注入。如下所示：

```java
// Dependency injection provides flexibility and testability
public class SpellChecker {
		private final Lexicon dictionary;
		public SpellChecker(Lexicon dictionary) { 
      this.dictionary = Objects.requireNonNull(dictionary);
		}
    public boolean isValid(String word) { ... }
    public List<String> suggestions(String typo) { ... }
}
```

> The dependency injection pattern is so simple that many programmers use it for years without knowing it has a name. While our spell checker example had only a single resource (the dictionary), dependency injection works with an arbitrary number of resources and arbitrary dependency graphs. It preserves immutability (Item 17), so multiple clients can share dependent objects (assuming the clients desire the same underlying resources). Dependency injection is equally applicable to constructors, static factories (Item 1), and builders (Item 2).

这种依赖注入的方式很简单，以至于很多程序员用了很多年也不知道它还有个名字。虽然SpellChecker这个例子只有一个资源（dictionary），但是依赖注入也适用于任意数量的资源以及任意的依赖形式，依赖注入保证了注入资源的不可变性，因此多个客户端可以共享依赖对象（假定这些客户端需要相同的底层资源），依赖注入也同样适用于构造器和静态工厂(Item 1)和Builder（Item 2).

> A useful variant of the pattern is to pass a resource *factory* to the constructor. A factory is an object that can be called repeatedly to create instances of a type. Such factories embody the *Factory Method* pattern [Gamma95]. The Supplier<T> interface, introduced in Java 8, is perfect for representing factories. Methods that take a Supplier<T> on input should typically constrain the factory’s type parameter using a *bounded wildcard type* (Item 31) to allow the client to pass in a factory that creates any subtype of a specified type. For example, here is a method that makes a mosaic using a client-provided factory to produce each tile:

这种模式的一个有用的变体是，将资源工厂传给构造器。工厂是可以重复调用来创建实例的对象。这类工厂具体表现为工厂方法模式，在java8里的Supplier<T> 接口，最适用于表示工厂。使用Supplier<T>作为输入的方法通常应该限制工厂的类型参数为带限制的通配符类型（bounded wildcard type）（Item 31）,以便客户端可以传入一个可以创建其子类的工厂。以下是一个创建Mosaic对象的方法，该方法使用客户端提供的生成各种Tile的工厂。

```java
Mosaic create(Supplier<? extends Tile> tileFactory) { ... }
```

> Although dependency injection greatly improves flexibility and testability, it can clutter up large projects, which typically contain thousands of dependencies. This clutter can be all but eliminated by using a *dependency injection framework*, such as Dagger [Dagger], Guice [Guice], or Spring [Spring]. The use of these frameworks is beyond the scope of this book, but note that APIs designed for manual dependency injection are trivially adapted for use by these frameworks.
>
> In summary, do not use a singleton or static utility class to implement a class that depends on one or more underlying resources whose behavior affects that of the class, and do not have the class create these resources directly. Instead, pass the resources, or factories to create them, into the constructor (or static factory or builder). This practice, known as dependency injection, will greatly enhance the flexibility, reusability, and testability of a class.

虽然依赖注入极大地提高了灵活性和可测试性，但是它可能导致大型项目产生混乱，因为大型项目中通常包含成千上万的依赖。这种混乱通过使用一些框架就可以很好地清楚，比如Dagger，Fuice，或者Spring。这些框架的使用不在本书的介绍范围内，但是需要注意的是，那些设计成手动依赖注入的api一般都可以用这些框架来实现。

总的来说，一个需要依赖底层资源并且其行为会受底层资源影响的类，不要使用单例或者静态工厂方法来实现，也不要由该类直接创建这些资源。应该将资源或者创建资源的工厂传给构造器（静态工厂方法、Builder）。依赖注入极大的提高了类的灵活性、可重用性、可测试性。

### 