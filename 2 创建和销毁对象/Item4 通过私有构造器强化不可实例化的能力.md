### Item4 通过私有构造器强化不可实例化的能力



> Occasionally you’ll want to write a class that is just a grouping of static methods and static fields. Such classes have acquired a bad reputation because some people abuse them to avoid thinking in terms of objects, but they do have valid uses. They can be used to group related methods on primitive values or arrays, in the manner of *java.lang.Math* or *java.util.Arrays*. They can also be used to group static methods, including factories (Item 1), for objects that implement some interface, in the manner of *java.util.Collections*. (As of Java 8, you can also put such methods in the interface, assuming it’s yours to modify.) Lastly, such classes can be used to group methods on a final class, since you can’t put them in a subclass.

有时，你可能回想编写一个只包含静态方法和静态域的类。这些类的名声不太好，因为有人会滥用这种类来逃避思考如何面向对象编程。但是这些类又确实有用，比如java.lang.Math和java.util.Arrays类，它们把基本类型值和数组上的一些相关的方法放到一起；再比如java.util.Collections，把实现了某些接口的对象上的静态方法比如工厂方法放在一起（在java8里，你也可以把这些方法放在接口里)。最后，这种类也可以用来把final类上的方法组织起来，因为不能把它们放在子类里。

> Such utility classes were not designed to be instantiated: an instance would be nonsensical. In the absence of explicit constructors, however, the compiler provides a public, parameterless default constructor. To a user, this constructor is indistinguishable from any other. It is not uncommon to see unintentionally instantiable classes in published APIs.

这种工具类不应该被(utility class)实例化，因为实例对它而言毫无意义。虽然没有显示的构造器，但是编译器会默认提供一个公有的无参的默认构造器。对于用户而言，这个构造器和其他的构造器没有什么区别，在一些已经发布了的api里，我们也通常可以看见一些被无意识实例化的类。

> **Attempting to enforce noninstantiability by making a class abstract does not work.**The class can be subclassed and the subclass instantiated. Furthermore, it misleads the user into thinking the class was designed for inheritance (Item 19). There is, however, a simple idiom to ensure noninstantiability. A default constructor is generated only if a class contains no explicit constructors, so a class can be made noninstantiable by including a private constructor:

通过把类设计成抽象类来强制不可实例化是行不通的。因为抽象类也已被继承，而其子类可以被实例化。同时，将类抽象化会误导用户，这让用户误以为这个类就是设计来继承的（item19)。有一个简单的习惯，可以保证非抽象类不可被实例化，因为只有当类不包含显式的构造器的时候，编译器才会生成一个默认的构造器，因此，当一个类有一个private的构造器时，它就不能被实例化了。示例如下：

```java
// Noninstantiable utility class
public class UtilityClass {
// Suppress default constructor for noninstantiability
    private UtilityClass() {
        throw new AssertionError();
    }
... // Remainder omitted 
}
```

> Because the explicit constructor is private, it is inaccessible outside the class. The *AssertionError* isn’t strictly required, but it provides insurance in case the constructor is accidentally invoked from within the class. It guarantees the class will never be instantiated under any circumstances. This idiom is mildly counterintuitive because the constructor is provided expressly so that it cannot be invoked. It is therefore wise to include a comment, as shown earlier.

在例子中，由于显式构造器是private的，因此在类的外部无法被访问。AssertionError虽然并不是一定要有的，但它可以用来避免构造器不小心从类的内部被调用，它保证了类在任何情况下都不被实例化。这种用法有些违反直觉，专门提供一个构造器，却又不能调用，因此像示例一样写一个注释，是很非常明智的。

> As a side effect, this idiom also prevents the class from being subclassed. All constructors must invoke a superclass constructor, explicitly or implicitly, and a subclass would have no accessible superclass constructor to invoke.

这样做的一个副作用就是，这个类不能被继承。因为所有的构造器都必须显式或者隐式地调用父类的构造器，但在前面介绍的方法里，子类就没有可调用的父类构造器。