## 4 类和接口

> **C**LASSES and interfaces lie at the heart of the Java programming language. They are its basic units of abstraction. The language provides many powerful elements that you can use to design classes and interfaces. This chapter contains guidelines to help you make the best use of these elements so that your classes and interfaces are usable, robust, and flexible.

类和接口是Java编程语言的核心。它们是Java语言的基本抽象单元。Java提供了很多用来设计类和接口的强大的基本元素。在本章中包括了一些指导原则，帮助你更好的使用这些元素，设计出有用、强壮、灵活的类和接口。

### Item15 最小化类和成员的可访问性

> The single most important factor that distinguishes a well-designed component from a poorly designed one is the degree to which the component hides its internal data and other implementation details from other components. A well-designed component hides all its implementation details, cleanly separating its API from its implementation. Components then communicate only through their APIs and are oblivious to each others’ inner workings. This concept, known as *information hiding* or *encapsulation*, is a fundamental tenet of software design [Parnas72].

区分一个组件设计的好坏，唯一重要的因素是这个组件对于其他组件而言，是否很好的影藏了内部的数据和实现细节。一个设计优秀的组件隐藏了所有的实现细节，将API和实现很好的分离开来。组件之间只是通过API进行通信，而对其他组件的内部工作情况完全不知情。这个概念被称为“信息隐藏（information hiding）”或者“封装（encapsulation）”，是软件设计的基本原则之一。

> Information hiding is important for many reasons, most of which stem from the fact that it *decouples* the components that comprise a system, allowing them to be developed, tested, optimized, used, understood, and modified in isolation. This speeds up system development because components can be developed in parallel. It eases the burden of maintenance because components can be understood more quickly and debugged or replaced with little fear of harming other components. While information hiding does not, in and of itself, cause good performance, it enables effective performance tuning: once a system is complete and profiling has determined which components are causing performance problems (Item 67), those components can be optimized without affecting the correctness of others. Information hiding increases software reuse because components that aren’t tightly coupled often prove useful in other contexts besides the ones for which they were developed. Finally, information hiding decreases the risk in building large systems because individual components may prove successful even if the system does not.

信息隐藏之所以很重要有很多原因，其中大多是因为它可以让组成一个系统的各个组件之间解耦（decouples），允许他们可以独立的开发、测试、优化、使用、理解和修改。这些可以加快系统的发展，因为各个组件可以并行开发。也使得维护变得容易，因为各个组件可以很容易地理解，在调试的时候，不用担心影响到其他的组件。虽然信息隐藏不管是对内还是对外都不会带来性能优化，但是它可以使性能调节更容易。一旦一个系统完成且通过分析得知哪个组件有性能问题（Item67)，这些组件就可以在不影响其他组件的情况下进行优化。信息隐藏也可以增强软件的重用性，因为那些没要紧密相连的组件，在不是其开发环境的其他环境中也很有用。最后，信息隐藏在构建大型系统的时候可以降低风险，因为即使是系统不能正常工作的时候，这些组件也还是可用的。

> Java has many facilities to aid in information hiding. The *access control* mechanism [JLS, 6.6] specifies the *accessibility* of classes, interfaces, and members. The accessibility of an entity is determined by the location of its declaration and by which, if any, of the access modifiers (private, protected, and public) is present on the declaration. Proper use of these modifiers is essential to information hiding.
>
> The rule of thumb is simple: **make each class or member as inaccessible as possible.** In other words, use the lowest possible access level consistent with the proper functioning of the software that you are writing.

Java中提供了很多机制可以帮助信息隐藏。访问控制机制明确了类、接口、成员(private, protected, 和 public) 的可访问性。一个实体的可访问性由它的声明位置、以及声明前的访问修饰词决定的。正确使用这些修饰词对于信息隐藏很重要。

规则很简单：**尽可能地让类和成员不可访问**。换句话说，在满足编写的软件的功能的前提下，使用最小的访问级别。

> For top-level (non-nested) classes and interfaces, there are only two possible access levels: *package-private* and *public*. If you declare a top-level class or interface with the public modifier, it will be public; otherwise, it will be package-private. If a top-level class or interface can be made package-private, it should be. By making it package-private, you make it part of the implementation rather than the exported API, and you can modify it, replace it, or eliminate it in a subsequent release without fear of harming existing clients. If you make it public, you are obligated to support it forever to maintain compatibility.
>
> If a package-private top-level class or interface is used by only one class, consider making the top-level class a private static nested class of the sole class that uses it (Item 24). This reduces its accessibility from all the classes in its package to the one class that uses it. But it is far more important to reduce the accessibility of a gratuitously public class than of a package-private top-level class: the public class is part of the package’s API, while the package-private top- level class is already part of its implementation.

对于顶层（非嵌套）的类和接口而言，有两种可能的访问级别：包级私有（package-private）和公有（public）。如果你使用public修饰符合声明一个顶级类或者接口，它就是公有的；如果你没有，它就是包级私有的。如果这个类或者接口是包级私有的，那么它就是这部分的实现中的一部分，而不是导出的API，在未来的版本中，你可以修改它，替换它，或者删除它，也不会影响到已经存在的客户端，如果你把它设为公有的，为了保持兼容，你就有责任一致支持它。

如果一个包级私有的顶层类或接口只被一个类使用，可以考虑将这个顶级类变成使用它的唯一类的一个私有静态嵌套类（Item24）。将这个类的可见性从包里的所有类减少到了使用它的类。但是相对而言，还是将一个不必要公有的类的可见性降低更重要。因为公有类是包的API中的一部分，而包级私有的顶层类是包的实现的一部分。

> For members (fields, methods, nested classes, and nested interfaces), there are four possible access levels, listed here in order of increasing accessibility:
>
> - **private**—The member is accessible only from the top-level class where it is declared.
> - **package-private**—The member is accessible from any class in the package where it is declared. Technically known as *default* access, this is the access level you get if no access modifier is specified (except for interface members, which are public by default).
> - **protected**—The member is accessible from subclasses of the class where it is declared (subject to a few restrictions [JLS, 6.6.2]) and from any class in the package where it is declared.
> - **public**—The member is accessible from anywhere.

对于成员（域，方法，嵌套类，和嵌套接口）而言，有四种可能的访问级别，按可访问性递增排序如下：

- **私有的（private）**— 成员只能被其声明的顶层类访问。
- **包级私有（package-private）**—成员可以被所声明的类所在包的类访问。从技术上，被称为“默认”的访问级别，也就是你没有指定访问修饰符时的默认访问级别。（接口成员除外，它们的默认访问级别是公有的）
- **受保护的（protected）**—成员可以被其声明的类的子类所访问（有一些限制 [JLS, 6.6.2]），还可以被所声明的类所在包的类访问。
- **公有的（public）**—成员在任何地方都可以被访问。

> After carefully designing your class’s public API, your reflex should be to make all other members private. Only if another class in the same package really needs to access a member should you remove the private modifier, making the member package-private. If you find yourself doing this often, you should reexamine the design of your system to see if another decomposition might yield classes that are better decoupled from one another. That said, both private and package-private members are part of a class’s implementation and do not normally impact its exported API. These fields can, however, “leak” into the exported API if the class implements Serializable (Items 86 and 87).

在仔细设计了类的公有api以后，你应该本能地把所有的其他成员都设为私有的。只有当同一个包里的其他类确实需要访问这个类的成员的时候，你猜应该删除private修饰符，使这个成员变成包级私有的。如果你发现你总是在做这样的事情，你就需要反思一下你的系统的设计，看看是不是换另一种分解方案获得的类们，他们之间的耦合度会不会低一些。也就是说，私有和包级私有的成员都属于类的实现中的一部分不会影响到它的导出的API。但是，当这个类实现了Serializable接口时（Item86和Item87），这些域可能会被泄露到导出的API里去。

> For members of public classes, a huge increase in accessibility occurs when the access level goes from package-private to protected. A protected member is part of the class’s exported API and must be supported forever. Also, a protected member of an exported class represents a public commitment to an implementation detail (Item 19). The need for protected members should be relatively rare.
>
> There is a key rule that restricts your ability to reduce the accessibility of methods. If a method overrides a superclass method, it cannot have a more restrictive access level in the subclass than in the superclass [JLS, 8.4.8.3]. This is necessary to ensure that an instance of the subclass is usable anywhere that an instance of the superclass is usable (the *Liskov substitution principle*, see Item 15). If you violate this rule, the compiler will generate an error message when you try to compile the subclass. A special case of this rule is that if a class implements an interface, all of the class methods that are in the interface must be declared public in the class.

对于公有类的成员而言，访问级别从包级私有到被保护的，这个成员的可访问性会有一个巨大的提升。一个被保护的成员也是这个类的导出API中的一部分，必须被永久的支持。同样地，一个导出类的被保护的成员也代表这个类对实现细节的公开承诺。需要使用被保护的成员的情况很少见。

有一个关键的规则限制了降低方法的可见性的能力：如果一个方法是覆盖的父类的方法，那么这个方法在子类中的可见性就不能比父类中低[JLS, 8.4.8.3]，这对于确保在父类实例可用的任何地方，都可以使用其子类的实例，很重要（里氏替换原则，详见Item10）。如果你违反了这个负责，在你试图辨析子类的时候，编译器就会生成一个错误信息。这条规则的一个特殊的情况是，当一个类实现一个接口时，这个类中所有来自接口的方法都必须要声明为公有的。

> To facilitate testing your code, you may be tempted to make a class, interface, or member more accessible than otherwise necessary. This is fine up to a point. It is acceptable to make a private member of a public class package-private in order to test it, but it is not acceptable to raise the accessibility any higher. In other words, it is not acceptable to make a class, interface, or member a part of a package’s exported API to facilitate testing. Luckily, it isn’t necessary either because tests can be made to run as part of the package being tested, thus gaining access to its package-private elements.

为了方便测试，你可能会试图将类，接口，或者成员的可访问性提高一些，超过其本身需要的可访问性。一定程度来说，可以这么做。为了测试，把一个共有类的私有成员提升到包级私有，是可以接受的。但是如果可访问性提高更多就不可以接受了。换句话说，为了测试，将类、接口、或者成员变成一个包的导出api中的一部分是不能接受的。幸运的是，也没有必要那么做，测试可以作为被测试包的一部分运行，也就能访问包级私有的成员了。

> **Instance fields of public classes should rarely be public** (Item 16). If an instance field is nonfinal or is a reference to a mutable object, then by making it public, you give up the ability to limit the values that can be stored in the field. This means you give up the ability to enforce invariants involving the field. Also, you give up the ability to take any action when the field is modified, so **classes with public mutable fields are not generally thread-safe.** Even if a field is final and refers to an immutable object, by making it public you give up the flexibility to switch to a new internal data representation in which the field does not exist.

公有类的实体域（非静态域）一般情况都不应该是公有的（Item16）。如果一个实体域是非final的，或者是一个可变对象的final引用，如果设置成公有的话，就相当于放弃了对保存在这个域的值的限制权力，也意味着你放弃了强制要求这个域不可变的能力，也放弃了当这个域发生改变时，采取对应措施的能力。因此包含公有可变域的类一般来说都不是线程安全的。即使这个域是指向不可变对象的final域，如果设置成public的话，也相当于放弃了“切换到另一种新的内部数据表示”的灵活性。

> The same advice applies to static fields, with one exception. You can expose constants via public static final fields, assuming the constants form an integral part of the abstraction provided by the class. By convention, such fields have names consisting of capital letters, with words separated by underscores (Item 68). It is critical that these fields contain either primitive values or references to immutable objects (Item 17). a field containing a reference to a mutable object has all the disadvantages of a nonfinal field. While the reference cannot be modified, the referenced object can be modified—with disastrous results.

这种建议也适用于静态域，但有一个例外。假设一些常量构成了类提供的抽象中的必不可少的一部分，就可以通过公有静态域来暴露这些常量。通常情况下，这些域都使用大写字母里命名，使用下划线来进行分割（Item68)。很重要的是，这种域包含的值应该是基本类型或者是不可变对象的引用。包含可变对象的应用的静态final域有所有非final域的缺点，因为即使这个引用无法修改，但是引用的对象可以被修改，会带来灾难性的后果。

> Note that a nonzero-length array is always mutable, so **it is wrong for a class to have a public static final array field, or an accessor that returns such a field.** If a class has such a field or accessor, clients will be able to modify the contents of the array. This is a frequent source of security holes:

需要注意的是长度非0的数组始终是可变的，因此**一个类拥有一个公有静态final数组域、或者提供一个方法返回这样的域都是错误的**。如果一个类有这样一个域或者访问方法，那么客户端就可以修改数组中的内容。这是一个安全漏洞的常见根源：

```java
// Potential security hole!
   public static final Thing[] VALUES =  { ... };
```

> Beware of the fact that some IDEs generate accessors that return references to private array fields, resulting in exactly this problem. There are two ways to fix the problem. You can make the public array private and add a public immutable list:

还需要注意到，一些IDE自动生成的访问方法会返回私有数组域的引用，也会导致这种问题。这里有两种方法可以解决这个问题，你可以将这个公有数组改为私有，然后提供一个公有的不可变列表，如下：

```java
private static final Thing[] PRIVATE_VALUES = { ... };
public static final List<Thing> VALUES =
    Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
```

> Alternatively, you can make the array private and add a public method that returns a copy of a private array:

或者你可以将数组设为私有的，然后提供一个公有方法来返回私有数组的一个克隆，如下：

```java
private static final Thing[] PRIVATE_VALUES = { ... };
public static final Thing[] values() {
    return PRIVATE_VALUES.clone();
}
```

> To choose between these alternatives, think about what the client is likely to do with the result. Which return type will be more convenient? Which will give better performance?

至于到底选择用哪一种，就应该考虑客户端需要如何去处理这个结果了，哪一种返回类型更方便，哪一种性能更好？

> As of Java 9, there are two additional, implicit access levels introduced as part of the *module system*. A module is a grouping of packages, like a package is a grouping of classes. A module may explicitly export some of its packages via *export declarations* in its *module declaration* (which is by convention contained in a source file named module-info.java). Public and protected members of unexported packages in a module are inaccessible outside the module; within the module, accessibility is unaffected by export declarations. Using the module system allows you to share classes among packages within a module without making them visible to the entire world. Public and protected members of public classes in unexported packages give rise to the two implicit access levels, which are intramodular analogues of the normal public and protected levels. The need for this kind of sharing is relatively rare and can often be eliminated by rearranging the classes within your packages.
>
> Unlike the four main access levels, the two module-based levels are largely advisory. If you place a module’s JAR file on your application’s class path instead of its module path, the packages in the module revert to their non-modular behavior: all of the public and protected members of the packages’ public classes have their normal accessibility, regardless of whether the packages are exported by the module [Reinhold, 1.2]. The one place where the newly introduced access levels are strictly enforced is the JDK itself: the unexported packages in the Java libraries are truly inaccessible outside of their modules.

在Java9里，作为模块系统（module system）中的一部分，有两种新增的隐式访问级别。一个模块就是一组包，就像包就是一组类一样。一个模块可以显示地通过其模块声明（module declaration）中的导出声明（export declaration）导出它的一些包，模块声明一包都包含在一个名叫module-info.java的源文件里。那些没有被导出的包里的公有和受保护的成员在模块外是无法访问的，在模块内部，其访问不受影响。使用这种模块系统，允许你在一个模块内部的包之间共享类，而不把它们公开给全世界。因此，我们可以看做未导出的包中的公有类中的公有和受保护成员的可访问性提升成了两个隐式访问级别，作为正常的公有和受保护级别的模块内部的对等体（intramodular analogues）。这种类型的共享的需求比较少见，而且可以通过重新组合包中的类来解决。

和其他的四种访问级别不同，这两种基于模块的访问级别只是建议性的。如果你把一个模块的JAR包放在了程序的类路径上，而不是模块路径上。在模块内的包的行为就会变回非模块行为，也就是，其包内的所有的共有类的公有和受保护的成员都会有正常的可访问性，不论这个包是都在模块中被导出。一个严格使用介绍的新的访问级别的地方就是JDK自己，在Java类库中，未导出的包在模块外部是绝对无法访问的。

> Not only is the access protection afforded by modules of limited utility to the typical Java programmer, and largely advisory in nature; in order to take advantage of it, you must group your packages into modules, make all of their dependencies explicit in module declarations, rearrange your source tree, and take special actions to accommodate any access to non-modularized packages from within your modules [Reinhold, 3]. It is too early to say whether modules will achieve widespread use outside of the JDK itself. In the meantime, it seems best to avoid them unless you have a compelling need.
>
> To summarize, you should reduce accessibility of program elements as much as possible (within reason). After carefully designing a minimal public API, you should prevent any stray classes, interfaces, or members from becoming part of the API. With the exception of public static final fields, which serve as constants, public classes should have no public fields. Ensure that objects referenced by public static final fields are immutable.

对于传统的程序员而言，不仅仅由受限工具的模块提供了访问性保护，而且这个本质上也只是建议性的。为了要利用这个优势，你必须将你的包组织成模块，在模块声明中明确指出他们之间的依赖，重新组织代码结构树。从模块内部采取一些特殊的措施来提供对那些非模块包的访问[Reinhold, 3]。除了JDK自身以外，现在说模块会取得广泛应用还为时尚早。也就意味值，除非你确实需要以外，最好还是不要使用他们。

总结一下，你应该尽可能（合理）地减少程序成员的可访问性。在仔细设计了一个最小的公用api以外，应该避免其他的类、接口、成员变成API的一部分。除了保存常量的公有静态final域外，公有类不应该包含公有域。保证一个对象的公有静态常量的引用对象是不可变的。

### 