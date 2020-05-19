### Item65 接口优先于反射

> The *core reflection facility*, java.lang.reflect, offers programmatic access to arbitrary classes. Given a Class object, you can obtain Constructor, Method, and Field instances representing the constructors, methods, and fields of the class represented by the Class instance. These objects provide programmatic access to the class’s member names, field types, method signatures, and so on.

核心反射机制，java.lang.reflect，提供了通过编程方式来访问任意类的能力。给定一个Class对象，可以获取 Constructor, Method, 和 Field 实例，分别代表这个Class实例的类的构造器，方法，和域。这些对象提供了编程的方式来访问类的成员名称，域类型，方法签名等等。

> Moreover, Constructor, Method, and Field instances let you manipulate their underlying counterparts *reflectively*: you can construct instances, invoke methods, and access fields of the underlying class by invoking methods on the Constructor, Method, and Field instances. For example, Method.invoke lets you invoke any method on any object of any class (subject to the usual security constraints). Reflection allows one class to use another, even if the latter class did not exist when the former was compiled. This power, however, comes at a price:

此外， Constructor, Method, 和 Field域让你可以操作它们的底层对等体：你可以通过调用Constructor, Method和Field实例的方法来构造实例、调用方法、访问底层类的域。比如Method.invoke让你可以调用类的任意对象的任意方法（遵从常规的安全限制）。反射还允许一个类访问另一个类，即使当前面的类编译时，第二个类还不存在。然而，这种能力是要付出代价的：

> • **You lose all the benefits of compile-time type checking,** including exception checking. If a program attempts to invoke a nonexistent or inaccessible method reflectively, it will fail at runtime unless you’ve taken special precautions.
>
> • **The code required to perform reflective access is clumsy and verbose.**It is tedious to write and difficult to read.
>
> • **Performance suffers.** Reflective method invocation is much slower than normal method invocation. Exactly how much slower is hard to say, as there are many factors at work. On my machine, invoking a method with no input parameters and an int return was eleven times slower when done reflectively.

- **你失去了所有的编译时类型检查带来的所有的好处**，包括异常检查。如果你的程序反射地调用了一个不存在或者不可达的方法时，在运行的时候，就会失败，除非你提前采取了措施。
- **需要执行反射访问的代码非常的笨拙冗长**。写起来单调乏味，读起来也很麻烦。
- **性能损失**。反射方法调用比正常的方法调用要慢得多。具体慢多少很难说，因为影响因素太多了。在作者的机器上，对于一个没有输入参数，返回值为int的方法，反射调用的时候速度慢了11倍。

> There are a few sophisticated applications that require reflection. Examples include code analysis tools and dependency injection frameworks. Even such tools have been moving away from reflection of late, as its disadvantages become clearer. If you have any doubts as to whether your application requires reflection, it probably doesn’t.

有一些特别复杂的程序确实需要反射。比如代码分析工具和依赖注入框架。即使是这些工具，他们也渐渐开始不再使用反射了，因为反射的缺点越来越明显了。如果你在犹豫你的程序是否需要反射，那么它很可能不需要。

> **You can obtain many of the benefits of reflection while incurring few of its costs by using it only in a very limited form.** For many programs that must use a class that is unavailable at compile time, there exists at compile time an appropriate interface or superclass by which to refer to the class (Item 64). If this is the case, you can **create instances reflectively and access them normally via their interface or superclass.**

**以有限的形式使用反射，虽然需要付出一些代价，但是还是可以获得一些好处的**。对于有时候程序员必须在编译期使用一个无法获取的类，而在编译期存在合适的接口或者超类来引用这个类（Item64）。**在这种情况下，你可以通过反射创建对象，然后通过他们的接口或者超类来访问它们。**

> For example, here is a program that creates a Set<String> instance whose class is specified by the first command line argument. The program inserts the remaining command line arguments into the set and prints it. Regardless of the first argument, the program prints the remaining arguments with duplicates eliminated. The order in which these arguments are printed, however, depends on the class specified in the first argument. If you specify java.util.HashSet, they’re printed in apparently random order; if you specify java.util.TreeSet, they’re printed in alphabetical order because the elements in a TreeSet are sorted:

比如，下面这个程序，创建了一个Set<String>实例，这个实例的类是通过第一个命令行参数来指定的。这个程序把剩余的命令行参数插入到了这个set中，然后打印了出来。不管第一个参数是什么，程序都会打印出剩下参数的去重的结果，这些参数打印的顺序取决域第一个参数指定的类。如果你指定为java.util.HashSet，它们打印出来的顺序就是随机的。如果你指定为java.util.TreeSet，他们就会按字母顺序打印出来，因为TreeSet里面的元素是排序的。代码如下：

```java
// Reflective instantiation with interface access
   public static void main(String[] args) {
       // Translate the class name into a Class object
       Class<? extends Set<String>> cl = null;
       try {
           cl = (Class<? extends Set<String>>)  // Unchecked cast!
                   Class.forName(args[0]);
       } catch (ClassNotFoundException e) {
           fatalError("Class not found.");
       }
       // Get the constructor
       Constructor<? extends Set<String>> cons = null;
       try {
           cons = cl.getDeclaredConstructor();
       } catch (NoSuchMethodException e) {
           fatalError("No parameterless constructor");
       }
       // Instantiate the set
       Set<String> s = null;
       try {
           s = cons.newInstance();
       } catch (IllegalAccessException e) {
           fatalError("Constructor not accessible");
       } catch (InstantiationException e) {
           fatalError("Class not instantiable.");
       } catch (InvocationTargetException e) {
           fatalError("Constructor threw " + e.getCause());
       } catch (ClassCastException e) {
           fatalError("Class doesn't implement Set");
       }
       // Exercise the set
       s.addAll(Arrays.asList(args).subList(1, args.length));
       System.out.println(s);
   }
   private static void fatalError(String msg) {
       System.err.println(msg);
       System.exit(1);
   }
```

> While this program is just a toy, the technique it demonstrates is quite powerful. The toy program could easily be turned into a generic set tester that validates the specified Set implementation by aggressively manipulating one or more instances and checking that they obey the Set contract. Similarly, it could be turned into a generic set performance analysis tool. In fact, this technique is sufficiently powerful to implement a full-blown *service provider framework* (Item 1). Usually, this technique is all that you need in the way of reflection.

虽然这个程序只是一个小玩具，但是它展示的技术是非常强大的。这个小程序可以容易地改成一个通用的set测试器，通过侵入式地操作一个或者多个实例并检查他们是否遵守了Set约定，以对指定的Set实现进行验证。同样地，它还可以改成一个通用的set性能分析工具。实际上，这些技术强大到可以实现一个成熟的服务提供框架（Item1）。实际上，这个技术也是你正式你需要的反射的所有了。

> This example demonstrates two disadvantages of reflection. First, the example can generate six different exceptions at runtime, all of which would have been compile-time errors if reflective instantiation were not used. (For fun, you can cause the program to generate each of the six exceptions by passing in appropriate command line arguments.) The second disadvantage is that it takes twenty-five lines of tedious code to generate an instance of the class from its name, whereas a constructor invocation would fit neatly on a single line. The length of the program could be reduced by catching ReflectiveOperationException, a superclass of the various reflective exceptions that was introduced in Java 7. Both disadvantages are restricted to the part of the program that instantiates the object. Once instantiated, the set is indistinguishable from any other Set instance. In a real program, the great bulk of the code is thus unaffected by this limited use of reflection.

上面这个例子也展示了反射的两个缺点。第一个是这个程序会在运行时会生成6个不同的异常，如果不使用反射的话这些就是编译时错误。（感兴趣的话，你可以给这个程序传入不同的命令行参数，让它生成这6个不同的异常）。第二个缺点是这段代码花了25行冗长的代码来根据一个类的名字来创建其实例，而调用构造器，只需要一行就够了。在Java7中新增了一个ReflectiveOperationException，是很多种反射异常的父类，直接捕获这个异常可以减少程序的长度。这两个缺点都在程序将这个类实例化的时候。一旦完成实例化，这个set就和其他的set实例没什么差别了。在实际的程序中，绝大部分代码都不会受这种限定使用反射的方法影响。

> If you compile this program, you’ll get an unchecked cast warning. This warning is legitimate, in that the cast to Class<? extends Set<String>> will succeed even if the named class is not a Set implementation, in which case the program with throw a ClassCastException when it instantiates the class. To learn about suppressing the warning, read Item 27.

如果你编译这段程序，你会得到一个非受检转换警告。这个警告是合法的，即使这个class的名字并不是一个Set实现，将其转换成Class<? extends Set<String>> 的转换也会成功，在这种情况下，实例化类的时候，会抛出ClassCastException异常。可以阅读Item27来了解如何禁止这个警告。

> A legitimate, if rare, use of reflection is to manage a class’s dependencies on other classes, methods, or fields that may be absent at runtime. This can be useful if you are writing a package that must run against multiple versions of some other package. The technique is to compile your package against the minimal environment required to support it, typically the oldest version, and to access any newer classes or methods reflectively. To make this work, you have to take appropriate action if a newer class or method that you are attempting to access does not exist at runtime. Appropriate action might consist of using some alternate means to accomplish the same goal or operating with reduced functionality.

反射的一个少见的合法的使用是用来管理一个类在运行时对其他可能不存在的类，方法或者域的依赖。如果你在写一个包必须支持在其他包的多个版本下运行，反射就会很有用。这个技术是在需要支持最小的环境下编译，最小的环境通常是最老的版本，并且通过反射访问一些新的类和方法。为了使得它可工作，当运行时不存在新的类和方法的时候，你必须采取合适的措施，这里的合适的措施可能包括使用其他的方法来完成同样的目的，或者使用简化的功能。

> In summary, reflection is a powerful facility that is required for certain sophisticated system programming tasks, but it has many disadvantages. If you are writing a program that has to work with classes unknown at compile time, you should, if at all possible, use reflection only to instantiate objects, and access the objects using some interface or superclass that is known at compile time.

总结一下，反射是一些复杂系统程序任务必须有的强大的机制，但是它有很多的缺点。如果你要写一个程序必须和一些编译时不确定的类一起工作，如果可以的话，你应该只使用反射来实例化对象，然后通过编译器知道的合适的接口和父类来访问对象。