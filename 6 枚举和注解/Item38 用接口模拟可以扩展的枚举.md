### Item38 用接口模拟可以扩展的枚举

> In almost all respects, enum types are superior to the typesafe enum pattern described in the first edition of this book [Bloch01]. On the face of it, one exception concerns extensibility, which was possible under the original pattern but is not supported by the language construct. In other words, using the pattern, it was possible to have one enumerated type extend another; using the language feature, it is not. This is no accident. For the most part, extensibility of enums turns out to be a bad idea. It is confusing that elements of an extension type are instances of the base type and not vice versa. There is no good way to enumerate over all of the elements of a base type and its extensions. Finally, extensibility would complicate many aspects of the design and implementation.

从几乎所有的方面来看，Enum类型都比本书第一版中介绍的类型安全模式要好一些。从表面上看，有一点是个例外，这点和扩展相关。在原来的模式中是可以继承的，而在语言结构中却不支持。也就是说，在原来的模式中，枚举类型可以继承另一个枚举类型。但是在语言特性中，就不可以做到了，绝无例外。在大部分情况下，枚举的继承都是一个坏主意。一个扩展类型的元素是基础类型的实例，而反过来就不是了，这点会让人感到很疑惑。也没有很好的方法可以列举出一个基础类型和它的扩展类型的所有的元素。最后，扩展会让设计和实现的很多方法都变得很复杂。

> That said, there is at least one compelling use case for extensible enumerated types, which is *operation codes,* also known as *opcodes.* An opcode is an enumerated type whose elements represent operations on some machine, such as the Operation type in Item 34, which represents the functions on a simple calculator. Sometimes it is desirable to let the users of an API provide their own operations, effectively extending the set of operations provided by the API.

可扩展的枚举类型至少有一个让人难以拒绝的例子，它就是操作码，也被称为opcode。操作码是一个枚举类型，其元素表示在一些机器上的操作，比如Item34里的Operation类型，就表示简单计算的函数。有的时候，确实需要，API的用户通过有效地继承API的操作集来提供自己的操作。

> Luckily, there is a nice way to achieve this effect using enum types. The basic idea is to take advantage of the fact that enum types can implement arbitrary interfaces by defining an interface for the opcode type and an enum that is the standard implementation of the interface. For example, here is an extensible version of the Operation type from Item 34:

幸运的是，确实有一个很好的方法可以利用枚举类型来达到这个效果。其基本思想是利用枚举类型可以实现任意多的接口，为操作类型定义一个接口，和一个实现了接口的枚举。比如，下面是Item34里的Operation类型的可扩展版本：

```java
// Emulated extensible enum using an interface
  public interface Operation {
      double apply(double x, double y);
	}
  public enum BasicOperation implements Operation {
      PLUS("+") {
					public double apply(double x, double y) { return x + y; } },
			MINUS("-") {
					public double apply(double x, double y) { return x - y; }},
      TIMES("*") {
					public double apply(double x, double y) { return x * y; } },
			DIVIDE("/") {
					public double apply(double x, double y) { return x / y; }};
					
			private final String symbol;
      BasicOperation(String symbol) {
          this.symbol = symbol;
			}
       @Override public String toString() {
           return symbol;
			}
  }
```

> While the enum type (BasicOperation) is not extensible, the interface type (Operation) is, and it is the interface type that is used to represent operations in APIs. You can define another enum type that implements this interface and use instances of this new type in place of the base type. For example, suppose you want to define an extension to the operation type shown earlier, consisting of the exponentiation and remainder operations. All you have to do is write an enum type that implements the Operation interface:

虽然枚举类型BasicOperation不是可扩展的，但是这个接口类型Operation是可扩展的，在API中用来表示操作的也正是这个接口类型。你可以定义另外一个实现这个接口的枚举类型，然后就可以使用新类型的实例来代替基本类型。比如，假如你现在想为之前的操作定义一个扩展，包括指数和取余操作。你所有需要做的就是写一个枚举类型，然后实现Operation接口。代码如下：

```java
// Emulated extension enum
   public enum ExtendedOperation implements Operation {
       EXP("^") {
           public double apply(double x, double y) {
               return Math.pow(x, y);
           } },
       REMAINDER("%") {
           public double apply(double x, double y) {
             	 return x % y; 
           }};
       private final String symbol;
       ExtendedOperation(String symbol) {
           this.symbol = symbol;
       }
       @Override public String toString() {
           return symbol;
       } 
   }
```

> You can now use your new operations anywhere you could use the basic operations, provided that APIs are written to take the interface type (Operation), not the implementation (BasicOperation). Note that you don’t have to declare the abstract apply method in the enum as you do in a nonextensible enum with instance-specific method implementations (page 162). This is because the abstract method (apply) is a member of the interface (Operation).

现在，只要是API采用的是接口类型（Operation），而不是实现类型（BasicOperation），在你使用基本操作的任何地方都可以使用新的操作。需要注意的是，在这个方法里，不需要声明抽象的apply方法，和在不可扩展类中的特定于实例的方法实现不一样（Item34）。这是因为这个抽象方法（apply）是接口（Operation）中的成员。

> Not only is it possible to pass a single instance of an “extension enum” anywhere a “base enum” is expected, but it is possible to pass in an entire extension enum type and use its elements in addition to or instead of those of the base type. For example, here is a version of the test program on page 163 that exercises all of the extended operations defined previously:

不仅仅可以在任何使用单个“基础枚举”的任何地方传入“扩展枚举”的单个实例，也可以传递整个扩展枚举类型，并使用它的元素，来替代或者补充基础枚举类型的元素。比如，下面是Item34中的test程序的一个版本，可以测试前面定义的所有扩展操作。代码如下：

```java
public static void main(String[] args) { 
  double x = Double.parseDouble(args[0]); 
  double y = Double.parseDouble(args[1]); 
  test(ExtendedOperation.class, x, y);
}
private static <T extends Enum<T> & Operation> void test( 
  Class<T> opEnumType, double x, double y) {
       for (Operation op : opEnumType.getEnumConstants())
           System.out.printf("%f %s %f = %f%n",x, op, y, op.apply(x, y));
}
```

> Note that the class literal for the extended operation type (ExtendedOperation.class) is passed from main to test to describe the set of extended operations. The class literal serves as a *bounded type token* (Item 33). The admittedly complex declaration for the opEnumType parameter (<T extends Enum<T> & Operation> Class<T>) ensures that the Class object represents both an enum and a subtype of Operation, which is exactly what is required to iterate over the elements and perform the operation associated with each one.

需要注意的是，这个扩展操作类型的字面量（ExtendedOperation.class）是错那个main程序传递给test函数的，用来描述扩展操作集合。这个字面量充当的是有限制的类型令牌（Item33）。有点复杂的参数opEnumType的声明(<T extends Enum<T> & Operation> Class<T>) ，确保了这个Class对象代表的类型既是枚举，又是Operation的子类型，这也正是遍历元素和针对每个元素执行相关操作所需要的。

> A second alternative is to pass a Collection<? extends Operation>, which is a *bounded wildcard type* (Item 31), instead of passing a class object:

另外一种可选择的方法是传递一个有限制通配符类型Collection<? extends Operation>，来代替类对象。代码如下：

```java
public static void main(String[] args) {
	double x = Double.parseDouble(args[0]);
	double y = Double.parseDouble(args[1]); 
  test(Arrays.asList(ExtendedOperation.values()), x, y);
}
private static void test(Collection<? extends Operation> opSet, double x, double y) {
       for (Operation op : opSet)
           System.out.printf("%f %s %f = %f%n",x, op, y, op.apply(x, y));
}
```

> The resulting code is a bit less complex, and the test method is a bit more flexible: it allows the caller to combine operations from multiple implementation types. On the other hand, you forgo the ability to use EnumSet (Item 36) and EnumMap (Item 37) on the specified operations.

这样得到的方法要简单一些，test方法也要灵活一些：允许调用者将多个实现类型中操作合并到一起。另一方面，也放弃了在指定操作上使用EnumSet (Item 36) 和 EnumMap (Item 37)的能力。

> Both programs shown previously will produce this output when run with command line arguments 4 and 2:

上面这两个程序在使用命令行参数4和2运行时，都将产生下面这样的结果：

```java
4.000000 ^ 2.000000 = 16.000000
4.000000 % 2.000000 = 0.000000
```

> A minor disadvantage of the use of interfaces to emulate extensible enums is that implementations cannot be inherited from one enum type to another. If the implementation code does not rely on any state, it can be placed in the interface, using default implementations (Item 20). In the case of our Operation example, the logic to store and retrieve the symbol associated with an operation must be duplicated in BasicOperation and ExtendedOperation. In this case it doesn’t matter because very little code is duplicated. If there were a larger amount of shared functionality, you could encapsulate it in a helper class or a static helper method to eliminate the code duplication.

使用接口来模拟可扩展的枚举有一个小小的缺点，就是这些实现没有办法从一个枚举继承到另一个枚举。如果一个方法的实现代码不依赖任何的状态，该方法就应该被放在接口里，并且提供默认实现（Item20）。在我们的Operation的例子中，保存和获取操作相关的符号的逻辑，在BasicOperation和ExtendedOperation中是重复的。在这个例子中，问题不大，因为重复的代码并不多。如果有很多需要共享的功能，你可以把这些功能写在辅助类或者静态辅助方法里来减少代码重复。

> The pattern described in this item is used in the Java libraries. For example, the java.nio.file.LinkOption enum type implements the CopyOption and OpenOption interfaces.

本节中介绍的模式在Java类库中也有使用。比如，枚举类型java.nio.file.LinkOption就实现了CopyOption和OpenOption接口。

> In summary, **while you cannot write an extensible enum type, you can emulate it by writing an interface to accompany a basic enum type that implements the interface.** This allows clients to write their own enums (or other types) that implement the interface. Instances of these types can then be used wherever instances of the basic enum type can be used, assuming APIs are written in terms of the interface.

总结一下，**虽然你不能写一个可以扩展的枚举类型，但是你可以通过写一个接口和一个实现该接口的基础枚举类型来模拟它。**这使得客户端可以写自己的枚举（或者其他类型）来实现这个接口。只要API是根据接口编写的，那么在使用基础枚举类型的地方，都可以使用自己编写的类型的实例。