## 7 Lambda和Stream

> In Java 8, functional interfaces, lambdas, and method references were added to make it easier to create function objects. The streams API was added in tandem with these language changes to provide library support for processing sequences of data elements. In this chapter, we discuss how to make best use of these facilities.

在Java8中，添加了函数接口，lambda，和方法引用，使得创建函数对象更加容易了。与此同时，还添加了Stream API，为数据元素序列的处理提供了类库级别的支持。在本章中，我们将讨论如何最佳的使用这些机制。

### Item42 Lambda优先于匿名类

> Historically, interfaces (or, rarely, abstract classes) with a single abstract method were used as *function types*. Their instances, known as *function objects*, represent functions or actions. Since JDK 1.1 was released in 1997, the primary means of creating a function object was the *anonymous class* (Item 24). Here’s a code snippet to sort a list of strings in order of length, using an anonymous class to create the sort’s comparison function (which imposes the sort order):

按照经验，函数类型即使只有一个抽象方法的接口（或者，很少的情况可能是抽象类）。他们的实例被称为函数对象，用来表示一个函数或者动作。自从1997年，JDK1.1版本发布后，最主要的创建函数对象的方法就是匿名类（Item24）。下面是对字符串列表按照字符串长度进行排序的代码片段，使用匿名类创建了一个排序的比较函数（指明了排序顺序）：

```java
// Anonymous class instance as a function object - obsolete!
   Collections.sort(words, new Comparator<String>() {
       public int compare(String s1, String s2) {
           return Integer.compare(s1.length(), s2.length());
       }
});
```

> Anonymous classes were adequate for the classic objected-oriented design patterns requiring function objects, notably the *Strategy* pattern [Gamma95]. The Comparator interface represents an *abstract strategy* for sorting; the anonymous class above is a *concrete strategy* for sorting strings. The verbosity of anonymous classes, however, made functional programming in Java an unappealing prospect.

匿名类满足了传统的面向对象设计模式对函数对象的需求，尤其是策略模式。这个Comparator接口表示的是一个用于排序的抽象策略，而上面的匿名类就是一个对字符串进行排序的具体的策略。然而，匿名类的冗长使得函数编程，在Java中一直都没有流行起来。

> In Java 8, the language formalized the notion that interfaces with a single abstract method are special and deserve special treatment. These interfaces are now known as *functional interfaces*, and the language allows you to create instances of these interfaces using *lambda expressions*, or *lambdas* for short. Lambdas are similar in function to anonymous classes, but far more concise. Here’s how the code snippet above looks with the anonymous class replaced by a lambda. The boilerplate is gone, and the behavior is clearly evident:

在Java8里面，正式确定了”只有一个抽象方法的接口是特殊的，需要被特殊对待“的观念。这些接口被称为函数式接口，java还允许使用lambda表达式（简称为lambdas）来创建这些接口的实例。Lambdas在功能上和匿名类相似，但是要简洁得多。下面是，前面使用匿名类的代码片段，使用lambda表示的代码。没有了样板代码，其行为也很明确：

```java
// Lambda expression as function object (replaces anonymous class)
Collections.sort(words, (s1, s2) -> Integer.compare(s1.length(), s2.length()));
```

> Note that the types of the lambda (Comparator<String>), of its parameters (s1 and s2, both String), and of its return value (int) are not present in the code. The compiler deduces these types from context, using a process known as *type inference*. In some cases, the compiler won’t be able to determine the types, and you’ll have to specify them. The rules for type inference are complex: they take up an entire chapter in the JLS [JLS, 18]. Few programmers understand these rules in detail, but that’s OK. **Omit the types of all lambda parameters unless their presence makes your program clearer.** If the compiler generates an error telling you it can’t infer the type of a lambda parameter, *then* specify it. Sometimes you may have to cast the return value or the entire lambda expression, but this is rare.

需要注意的是，这个lambda（Comparator<String>）、它的参数（s1和s2，都是String类型）和它的返回值（int类型）的类型在代码中有没有写。编译器可以通过上下文推导出这些类型，使用的过程称为”类型推导“。在一些例子中，编译器无法确定这些类型，你就必须要指定类型。类型推导的规则太复杂了：在 JLS [JLS, 18]里占了整整一个章节。很少有程序员能够了解这些规则的细节，但是没有关系。**除非这些类型的存在能让你的代码更加清晰，否则就直接省略掉lambda中所有的参数类型。**如果编译器生成了一个错误告诉你它不能推导出这个lambda参数类型，再指定它。有时候，你可能必须对返回值挥着整个lambda表达式进行转换，但是这种情况是很少的。

> One caveat should be added concerning type inference. Item 26 tells you not to use raw types, Item 29 tells you to favor generic types, and Item 30 tells you to favor generic methods. This advice is doubly important when you’re using lambdas, because the compiler obtains most of the type information that allows it to perform type inference from generics. If you don’t provide this information, the compiler will be unable to do type inference, and you’ll have to specify types manually in your lambdas, which will greatly increase their verbosity. By way of example, the code snippet above won’t compile if the variable words is declared to be of the raw type List instead of the parameterized type List<String>.

还有一个和类型推导相关的警告需要指出来。Item26告诉你不要使用原生类型，Item29告诉你要优先使用泛型类型，Item30告诉你要优先使用泛型方法，这些建议在你使用lambdas的时候也同样重要，因为编译器获取的用于类型推导的大部分类型信息都来自于泛型。如果你没有提供这些信息，编译器就无法进行类型推导，你就必须要在lambdas里手动指定类型，会极大地增加代码的复杂程度。在前面的例子中，如果这个变量words被声明为原生类型List而不是参数化类型列表List<String>，这个代码片段就会无法编译。

> Incidentally, the comparator in the snippet can be made even more succinct if a *comparator construction method* is used in place of a lambda (Items 14. 43):

当然，如果使用”比较器构造器方法“来代替lambda，这个代码片段中的比较器还可以更加简洁，代码如下：

```java
Collections.sort(words, comparingInt(String::length));
```

> In fact, the snippet can be made still shorter by taking advantage of the sort method that was added to the List interface in Java 8:

事实上，如果利用Java8的List接口中增加的sort方法，这个代码片段还可以变得更短。如下：

```java
words.sort(comparingInt(String::length));
```

> The addition of lambdas to the language makes it practical to use function objects where it would not previously have made sense. For example, consider the Operation enum type in Item 34. Because each enum required different behavior for its apply method, we used constant-specific class bodies and overrode the apply method in each enum constant. To refresh your memory, here is the code:

在Java中，随着lambdas的加入，在以前不愿意使用函数对象的一些地方，使用函数对象也变得有意义了。比如，Item34中的Operation枚举类型，因为每个枚举常量的apply方法的行为不同，所以我们使用了特定于常量的类主体，并在每个枚举常量中覆盖了applay方法。通过下面的代码回顾一下：

```java
// Enum type with constant-specific class bodies & data (Item 34)
   public enum Operation {
       PLUS("+") {
					public double apply(double x, double y) { return x + y; } 
       },
			 MINUS("-") {
					public double apply(double x, double y) { return x - y; }
       },
       TIMES("*") {
					public double apply(double x, double y) { return x * y; } 
       },
			 DIVIDE("/") {
					public double apply(double x, double y) { return x / y; }
       };
       private final String symbol;
     
       Operation(String symbol) { this.symbol = symbol; }
     
       @Override public String toString() { return symbol; }
     
       public abstract double apply(double x, double y);
   }
```

> Item 34 says that enum instance fields are preferable to constant-specific class bodies. Lambdas make it easy to implement constant-specific behavior using the former instead of the latter. Merely pass a lambda implementing each enum constant’s behavior to its constructor. The constructor stores the lambda in an instance field, and the apply method forwards invocations to the lambda. The resulting code is simpler and clearer than the original version:

Item34说，枚举实例域优先于特定于常量的类主体。lambda表达式可以很简单地使用实例域实现特定于常量的行为，而不使用特定于常量的类主体。只需要在构造器中传递一个实现了每个枚举常量的行为的lambda，这个构造器将这个lambda保存在实例域中，然后apply方法将请求转发给lambda。这样得到的的代码相较于之前的版本，又简洁又明了。

```java
// Enum with function object fields & constant-specific behavior
public enum Operation {
		PLUS ("+",(x,y)->x+y), 
  	MINUS ("-", (x, y) -> x - y), 
  	TIMES ("*", (x, y) -> x * y),
  	DIVIDE("/", (x, y) -> x / y);
    private final String symbol;
    private final DoubleBinaryOperator op;
    Operation(String symbol, DoubleBinaryOperator op) {
        this.symbol = symbol;
        this.op = op;
		}
    @Override public String toString() { return symbol; }
    public double apply(double x, double y) {
        return op.applyAsDouble(x, y);
    } 
}
```

> Note that we’re using the DoubleBinaryOperator interface for the lambdas that represent the enum constant’s behavior. This is one of the many predefined functional interfaces in java.util.function (Item 44). It represents a function that takes two double arguments and returns a double result.

需要注意的是，我们在这里给lamdba使用了DoubleBinaryOperator接口，来表示枚举常量的行为。这是在java.util.function（Item44）里定义好了的很多函数式接口中的其中一个。它表示这个函数有两个double类型的参数，并返回一个double的值。

> Looking at the lambda-based Operation enum, you might think constant- specific method bodies have outlived their usefulness, but this is not the case. Unlike methods and classes, **lambdas lack names and documentation; if a computation isn’t self-explanatory, or exceeds a few lines, don’t put it in a lambda.** One line is ideal for a lambda, and three lines is a reasonable maximum. If you violate this rule, it can cause serious harm to the readability of your programs. If a lambda is long or difficult to read, either find a way to simplify it or refactor your program to eliminate it. Also, the arguments passed to enum constructors are evaluated in a static context. Thus, lambdas in enum constructors can’t access instance members of the enum. Constant-specific class bodies are still the way to go if an enum type has constant-specific behavior that is difficult to understand, that can’t be implemented in a few lines, or that requires access to instance fields or methods.

























