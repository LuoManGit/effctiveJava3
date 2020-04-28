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























