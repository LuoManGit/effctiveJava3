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

























