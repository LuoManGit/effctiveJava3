### Item24 静态成员类由于非静态成员类

> A *nested class* is a class defined within another class. A nested class should exist only to serve its enclosing class. If a nested class would be useful in some other context, then it should be a top-level class. There are four kinds of nested classes: *static member classes*, *nonstatic member classes*, *anonymous classes*, and *local classes*. All but the first kind are known as *inner classes*. This item tells you when to use which kind of nested class and why.

嵌套类就是定义在其他类里面的类。嵌套类存在的目的只是为其外围类提供服务。如果一个嵌套类在其他的环境下也有用，那么它就应该是一个顶级类。一共有四种内部类：静态成员类、非静态成员类、匿名类、和局部类。除了第一种以外，其他的类都称为“内部类”。本节将告诉你应该在什么情况下使用什么类型的嵌套类，以及为什么这么做。

> A static member class is the simplest kind of nested class. It is best thought of as an ordinary class that happens to be declared inside another class and has access to all of the enclosing class’s members, even those declared private. A static member class is a static member of its enclosing class and obeys the same accessibility rules as other static members. If it is declared private, it is accessible only within the enclosing class, and so forth.

静态成员类是最简单的嵌套类。最好是把它看做普通的类，只是碰巧声明在了另一个类内部，同时可以访问外围类的所有成员，包括哪些声明为私有的成员。静态成员类也是其外围类的一个静态成员，和其他的静态成员遵守相同的访问规则。如果它被声明为私有的，那么他就只能被外围类访问，等等。

> One common use of a static member class is as a public helper class, useful only in conjunction with its outer class. For example, consider an enum describing the operations supported by a calculator (Item 34). The Operation enum should be a public static member class of the Calculator class. Clients of Calculator could then refer to operations using names like Calculator.Operation.PLUS and Calculator.Operation.MINUS.

一个静态成员类的常见用法是作为公有的辅助类，只有和其外围类一起合作才有用。以一个描述了计算器（calculator）支持的操作（Operation）的枚举为例(Item34)，这个Operation枚举应该是Calculator类的一个公有的内部类，Calculator的客户端就可以通过类似 Calculator.Operation.PLUS 和Calculator.Operation.MINUS的名字来引用这些操作。

> Syntactically, the only difference between static and nonstatic member classes is that static member classes have the modifier static in their declarations. Despite the syntactic similarity, these two kinds of nested classes are very different. Each instance of a nonstatic member class is implicitly associated with an *enclosing instance* of its containing class. Within instance methods of a nonstatic member class, you can invoke methods on the enclosing instance or obtain a reference to the enclosing instance using the *qualified this* construct [JLS, 15.8.4]. If an instance of a nested class can exist in isolation from an instance of its enclosing class, then the nested class *must* be a static member class: it is impossible to create an instance of a nonstatic member class without an enclosing instance.

从语法上，静态和非静态成员类之间的区别只在于其声明中是否有static修饰符。虽然语法上很相似，但是这两种内部类是非常不同的。每一个非静态内部类都隐式地和一个指向外围类实例相关联。在非静态成员类的实例方法中，你可以使用修饰过的this来获得外围实例的引用[JLS, 15.8.4]，从而调用外围实例的方法。如果一个嵌套类的实例，可以与其外部类实例分开而独立存在，那么这个嵌套类就必须是静态成员类。在不存在外围类实例的情况下，无法创建非静态成员类的实例。

> The association between a nonstatic member class instance and its enclosing instance is established when the member class instance is created and cannot be modified thereafter. Normally, the association is established automatically by invoking a nonstatic member class constructor from within an instance method of the enclosing class. It is possible, though rare, to establish the association manually using the expression enclosingInstance.new MemberClass(args). As you would expect, the association takes up space in the nonstatic member class instance and adds time to its construction.

当这个非静态成员类的实例被创建的时候，该实例和其外部类实例之间的关联就建立起来了，并且，这种关联关系不能被修改了。通常情况下，这种关联是通过外围类的实例方法调用非静态成员类的构造器，自动创建的。也可以通过表达式“enclosingInstance.new MemberClass(args)”手动地创建这中关系，但实际中很少这么做。正如你所想的那样，这种关联会占用非静态成员类实例的空间，并且会增加其构造的时间。

> One common use of a nonstatic member class is to define an *Adapter* [Gamma95] that allows an instance of the outer class to be viewed as an instance of some unrelated class. For example, implementations of the Map interface typically use nonstatic member classes to implement their *collection views*, which are returned by Map’s keySet, entrySet, and values methods. Similarly, implementations of the collection interfaces, such as Set and List, typically use nonstatic member classes to implement their iterators:

非静态成员类的一个常见用法是定义一个Adapter（适配器）[Gamma95] 。允许一个外围类的实例被看做是一些不相关的实例类的实例。比如，在Map接口的实现中，通常都使用非静态成员类来实现他们的集合视图，集合视图通过Map的keySet、entrySet、values方法返回。同样地，在集合接口的视线中，比如Set和List，通常也使用非静态成员类来实现他们的迭代器。如下：

```java
// Typical use of a nonstatic member class
   public class MySet<E> extends AbstractSet<E> {
       ... // Bulk of the class omitted
       @Override public Iterator<E> iterator() {
           return new MyIterator();
			 }
			 private class MyIterator implements Iterator<E> { ...
			 }
   }
```

> **If you declare a member class that does not require access to an enclosing instance,** **always** **put the** **static** **modifier in its declaration,** making it a static rather than a nonstatic member class. If you omit this modifier, each instance will have a hidden extraneous reference to its enclosing instance. As previously mentioned, storing this reference takes time and space. More seriously, it can result in the enclosing instance being retained when it would otherwise be eligible for garbage collection (Item 7). The resulting memory leak can be catastrophic. It is often difficult to detect because the reference is invisible.

**如果你声明的成员类不需要访问其外围类实例，那么就应该在声明中添加static修饰符**，让这个成员类是静态的而不是非静态的。如果你省略了这个修饰符，每一个内部类实例都会有一个额外的隐藏的指向外部类实例的引用。正如前面提到的那样，保存这个引用会花费时间和空间。更严重地是，它可能导致一个本该被垃圾回收的外部类实例被保留下来，这个问题导致的内存泄露问题是灾难性的。但是却难以发现，因为这个引用时不可见的。

> A common use of private static member classes is to represent components of the object represented by their enclosing class. For example, consider a Map instance, which associates keys with values. Many Map implementations have an internal Entry object for each key-value pair in the map. While each entry is associated with a map, the methods on an entry (getKey, getValue, and setValue) do not need access to the map. Therefore, it would be wasteful to use a nonstatic member class to represent entries: a private static member class is best. If you accidentally omit the static modifier in the entry declaration, the map will still work, but each entry will contain a superfluous reference to the map, which wastes space and time.

私有静态成员类的常见的用法是代表外围类代表的对象组件。以Map实例为例，它把键值关联起来，在很多的Map的实现中，都有一个内部Entry对象来表示map中的每一个键值对。虽然每一个entry都和map相关联，但是其方法（getKey, getValue, 和setValue）都不需要访问map。因此，使用非静态成员类来表示entry是比较浪费的，私有静态成员类就是最好的选择。如果你不小心漏掉了entry声明前的stati修饰符，这个map也能工作，但是它的每一个entry都包含一个指向map的多余的引用，既浪费时间又浪费空间。

> It is doubly important to choose correctly between a static and a nonstatic member class if the class in question is a public or protected member of an exported class. In this case, the member class is an exported API element and cannot be changed from a nonstatic to a static member class in a subsequent release without violating backward compatibility.

如果这个成员类是导出类中的公有或者受保护的成员，那么毫无疑问，在静态和非静态成员类中做出正确的选择是相当重要的。在这种情况下，这个成员类也是导出API元素，在后续的版本中，如果不打破后向兼容性的话，是不可能将一个非静态成员类改为静态的。

> As you would expect, an anonymous class has no name. It is not a member of its enclosing class. Rather than being declared along with other members, it is simultaneously declared and instantiated at the point of use. Anonymous classes are permitted at any point in the code where an expression is legal. Anonymous classes have enclosing instances if and only if they occur in a nonstatic context. But even if they occur in a static context, they cannot have any static members other than *constant variables*, which are final primitive or string fields initialized to constant expressions [JLS, 4.12.4].

顾名思义，匿名类是没有名字的。它也不是其外围类的一个成员。在使用的时候同时声明和实例化，而不是和其他成员一起被声明。匿名类可以在代码中任何地方出现，只要其表达式是正确的。只有当匿名类出现在非静态的环境中的时候，它才拥有其外围类实例的引用。但是即是他们出现在静态环境中，也不会拥有除了常数变量以外的静态成员，常数变量是被初始化为常量表达式的final基本类型或String域。

> There are many limitations on the applicability of anonymous classes. You can’t instantiate them except at the point they’re declared. You can’t perform instanceof tests or do anything else that requires you to name the class. You can’t declare an anonymous class to implement multiple interfaces or to extend a class and implement an interface at the same time. Clients of an anonymous class can’t invoke any members except those it inherits from its supertype. Because anonymous classes occur in the midst of expressions, they must be kept short— about ten lines or fewer—or readability will suffer.

匿名类的使用有很多的限制。除了匿名类声明的时候，其他时候无法进行实例化。也不能对其进行instanceof测试，所有需要类的名字的事情都不能做。也不能声明一个匿名类 同时 “实现多个接口或者继承一个类”和实现一个接口。匿名类的客户端不能调用任何成员，从父类继承来的除外。因为匿名类是写在表达式中间的，所以他们必须很短--大概10行或者更短，否则就会破坏代码的可读性。

> Before lambdas were added to Java (Chapter 6), anonymous classes were the preferred means of creating small *function objects* and *process objects* on the fly, but lambdas are now preferred (Item 42). Another common use of anonymous classes is in the implementation of static factory methods (see intArrayAsList in Item 20).

在lambda被加入到Java之前（Chapter6），匿名类是用来创建小的函数对象和过程对象的最佳方法，但是现在lambda表达式更受欢迎了（Item42）。另外一个匿名类的常见用法是在静态工厂方法的实现中（见Item20里的intArrayAsList）。

> Local classes are the least frequently used of the four kinds of nested classes. A local class can be declared practically anywhere a local variable can be declared and obeys the same scoping rules. Local classes have attributes in common with each of the other kinds of nested classes. Like member classes, they have names and can be used repeatedly. Like anonymous classes, they have enclosing instances only if they are defined in a nonstatic context, and they cannot contain static members. And like anonymous classes, they should be kept short so as not to harm readability.

局部类在四种嵌套类中使用频率最低。一个局部类几乎可以在任何本地变量可以声明的地方进行声明，并且遵守和本地变量相同的作用域规则。局部类和其他嵌套类有都一点共同的属性。和成员类一样，局部类有名字，也可以被重复使用；和匿名类样，局部类也只有定义在非静态环境中才拥有外围实例对象，也不能包含静态成员；和匿名类一样，局部类为了不破坏代码可读性，也应该很短。

> To recap, there are four different kinds of nested classes, and each has its place. If a nested class needs to be visible outside of a single method or is too long to fit comfortably inside a method, use a member class. If each instance of a member class needs a reference to its enclosing instance, make it nonstatic; otherwise, make it static. Assuming the class belongs inside a method, if you need to create instances from only one location and there is a preexisting type that characterizes the class, make it an anonymous class; otherwise, make it a local class.

总结一下，嵌套类一共有四种类型，每一种都有其适用的场景。如果一个嵌套类需要在单个方法外部可见，或者太长了不适合写在方法内部，就使用成员类。如果成员类的每个实例都需要外围实例的引用，就让这个成员类是非静态的；否则，就设为非静态的。假如这个类属于某个方法内部，只需要在一个地方使用，并且其原始类型就可以说明其特征，就把它做成匿名类；否则做成局部类。