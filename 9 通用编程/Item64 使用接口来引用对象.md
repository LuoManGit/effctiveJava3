### Item64 使用接口来引用对象

> Item 51 says that you should use interfaces rather than classes as parameter types. More generally, you should favor the use of interfaces over classes to refer to objects. **If appropriate interface types exist, then parameters, return values, variables, and fields should all be declared using interface types.** The only time you really need to refer to an object’s class is when you’re creating it with a constructor. To make this concrete, consider the case of LinkedHashSet, which is an implementation of the Set interface. Get in the habit of typing this:

Item51里说应该使用接口而不是类来作为参数类型。说得更加通俗一些，就是在引用对象的时候，应该优先使用接口而不是类。**如果有合适的接口类型存在，那么参数，返回值，变量，和域都应该使用接口类型来声明**。你唯一真正需要对象的类的时候，是使用构造器创建的时候。为了更具体一下，来看看下面这个LinkedHashSet的例子，LinkedHashSet是Set接口的一个实现。在声明变量的时候应该养成这样的习惯：

```java
// Good - uses interface as type 
Set<Son> sonSet = new LinkedHashSet<>();
```

> not this:

而不是这样：

```java
// Bad - uses class as type!
LinkedHashSet<Son> sonSet = new LinkedHashSet<>();
```

> **If you get into the habit of using interfaces as types, your program will be much more flexible.** If you decide that you want to switch implementations, all you have to do is change the class name in the constructor (or use a different static factory). For example, the first declaration could be changed to read:

**如果你养成了使用接口来作为类型的习惯，你的程序就会变得非常灵活。**如果你决定了想要换一个实现，你需要做的事情就只是修改构造器的名字（或者使用不同的静态工厂）。比如，前面的声明就可以变成下面这样：

```java
Set<Son> sonSet = new HashSet<>();
```

> and all of the surrounding code would continue to work. The surrounding code was unaware of the old implementation type, so it would be oblivious to the change.

然后，所有相关的代码也能正常工作。相关代码并不知道之前的实现类型，因此察觉不到这一改变。

> There is one caveat: if the original implementation offered some special functionality not required by the general contract of the interface and the code depended on that functionality, then it is critical that the new implementation provide the same functionality. For example, if the code surrounding the first declaration depended on LinkedHashSet’s ordering policy, then it would be incorrect to substitute HashSet for LinkedHashSet in the declaration, because HashSet makes no guarantee concerning iteration order.

有一点需要注意的是：如果原始的实现提供了一些 接口的通用约定里不需要的特殊的功能，而其他的代码也依赖这个功能，这种时候，新的实现方法也需要提供相同的功能，就很重要了。比如，如果第一个声明的相关的代码依赖LinkedHashSet的顺序机制，那么我们在声明中使用HashSet来代替LinkedHashSet就是不正确的，因为HashSet并没有保证相关的迭代顺序。

> So why would you want to change an implementation type? Because the second implementation offers better performance than the original, or because it offers desirable functionality that the original implementation lacks. For example, suppose a field contains a HashMap instance. Changing it to an EnumMap will provide better performance and iteration order consistent with the natural order of the keys, but you can only use an EnumMap if the key type is an enum type. Changing the HashMap to a LinkedHashMap will provide predictable iteration order with performance comparable to that of HashMap, without making any special demands on the key type.

那么为什么会想要修改实现类型呢？因为第二种实现的性能比原来的更好，或者它能提供一些原来的实现没有的但是又需要的功能。比如，有一个域包含一个HashMap实例。把HashMap改为一个EnumMap的话，可以提供更好性能，以及迭代顺序和键的自然顺序一致。但是只有键类型是枚举类型的时候，才能使用EnumMap。把HashMap改为LinkedHashMap的话，就可以提供可预测的迭代顺序，其性能也能和HashMap媲美，对键类型也没有特别的要求。

> You might think it’s OK to declare a variable using its implementation type, because you can change the declaration type and the implementation type at the same time, but there is no guarantee that this change will result in a program that compiles. If the client code used methods on the original implementation type that are not also present on its replacement or if the client code passed the instance to a method that requires the original implementation type, then the code will no longer compile after making this change. Declaring the variable with the interface type keeps you honest.

你可能会认为使用实现类型来声明变量也还OK，因为你可以同时修改声明类型和实现类型，但是无法保证这样的改变得到的程序可以编译。如果客户端代码使用了原始实现中的方法，而这个方法新的实现里没有，或者客户端将这个实例传给了一个需要原始实现类型的方法，那么这个代码，在修改了声明类型和实现类型后，就无法进行编译了。使用接口类型来声明变量会可以让你”保持诚实“。

> **It is entirely appropriate to refer to an object by a class rather than an interface if no appropriate interface exists.** For example, consider *value classes,* such as String and BigInteger. Value classes are rarely written with multiple implementations in mind. They are often final and rarely have corresponding interfaces. It is perfectly appropriate to use such a value class as a parameter, variable, field, or return type.

**当没有合适的接口存在的时候，使用类而不是接口来引用对象，是非常合适的**。举个例子，值类，比如String和BigInteger。记住，值类很少会写多个实现。他们通常都是fimal的，也很少有对应的接口。因此，使用这种值类来作为参数、变量、域、返回值的类型是非常合适的。

> A second case in which there is no appropriate interface type is that of objects belonging to a framework whose fundamental types are classes rather than interfaces. If an object belongs to such a *class-based framework*, it is preferable to refer to it by the relevant *base class*, which is often abstract, rather than by its implementation class. Many java.io classes such as OutputStream fall into this category.

第二个没有合适的接口类型的情况是，这个对象所属的框架的基础类型是类额不是接口。如果一个对象属于这样的基于类的框架，使用相关的基础类（大部分时候都是抽象的）来引用要更好一些，而不是它的实现类型。java.io 中的很多类比如OutputStream就属于这种情况。

> A final case in which there is no appropriate interface type is that of classes that implement an interface but also provide extra methods not found in the interface—for example, PriorityQueue has a comparator method that is not present on the Queue interface. Such a class should be used to refer to its instances *only* if the program relies on the extra methods, and this should be very rare.

最后一个没有合适接口类型的情况是，这些类实现了接口，但是还提供了接口中没有的额外的方法——比如，PriorityQueue提供了一个Queue接口里没有的comparator方法。只有当程序依赖这些额外的方法的时候，才应该使用这样的类来引用实例，这种情况是比较少的。

> These three cases are not meant to be exhaustive but merely to convey the flavor of situations where it is appropriate to refer to an object by its class. In practice, it should be apparent whether a given object has an appropriate interface. If it does, your program will be more flexible and stylish if you use the interface to refer to the object. **If there is no appropriate interface, just use the least specific class in the class hierarchy that provides the required functionality.**

上面这些例子并不详尽，只是代表了一些“需要使用类来引用对象“的情况。在实际中，一个给定的对象有没有一个合适的接口，是很明显的。如果有的话，当你使用接口来引用对象的时候，你的程序会变得更加灵活优雅。**如果没有合适的接口的话，就在类层次结构中选择能够提供需要的功能的最小的类。**

