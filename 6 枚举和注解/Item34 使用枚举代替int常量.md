## 6 枚举和注解

> **J**AVA supports two special-purpose families of reference types: a kind of class called an *enum type,* and a kind of interface called an *annotation type*. This chapter discusses best practices for using these type families.

Java支持两种特殊用途的引用类型：一种是类，被称为枚举类型；一种是接口，被称为注解类型。本章主要介绍这两种新类型的最佳使用方法。

### Item34 使用枚举代替int常量

> An *enumerated type* is a type whose legal values consist of a fixed set of constants, such as the seasons of the year, the planets in the solar system, or the suits in a deck of playing cards. Before enum types were added to the language, a common pattern for representing enumerated types was to declare a group of named int constants, one for each member of the type:

一个枚举类型是指有一组固定的常量组成的合法值的类型，比如一年的季节，太阳系的星星，一副牌的花色。在enum类型被添加到java语言中之前，通常使用一组命名的int常量来表示枚举类型，每一个int值表示一个枚举类型的成员。代码如下：

```java
// The int enum pattern - severely deficient!
   public static final int APPLE_FUJI         = 0;
   public static final int APPLE_PIPPIN       = 1;
   public static final int APPLE_GRANNY_SMITH = 2;

   public static final int ORANGE_NAVEL  = 0;
   public static final int ORANGE_TEMPLE = 1;
   public static final int ORANGE_BLOOD  = 2;
```

> This technique, known as the int *enum pattern,* has many shortcomings. It provides nothing in the way of type safety and little in the way of expressive power. The compiler won’t complain if you pass an apple to a method that expects an orange, compare apples to oranges with the == operator, or worse:

这种技术，称为”int枚举模式“，有跟多的缺点。它完全没有提供类型安全的保证，表达能力也不强。当你把一个apple传递给一个需要orange的方法时，编译器也不会生成警告，还可以使用==来对apple和orange进行比较，甚至更糟糕：

```java
// Tasty citrus flavored applesauce!
int i = (APPLE_FUJI - ORANGE_TEMPLE) / APPLE_PIPPIN;
```

> Note that the name of each apple constant is prefixed with APPLE_ and the name of each orange constant is prefixed with ORANGE_. This is because Java doesn’t provide namespaces for int enum groups. Prefixes prevent name clashes when two int enum groups have identically named constants, for example between ELEMENT_MERCURY and PLANET_MERCURY.

注意，每一个apple常量都有一个APPLE-前缀，每一个orange常量都有一个ORANGE-前缀。这是因为java没有为int枚举组提供命名空间。前缀可以防止两个int枚举组有同样的常量时，出现命名冲突，比如 ELEMENT_MERCURY和PLANET_MERCURY。

> Programs that use int enums are brittle. Because int enums are *constant variables* [JLS, 4.12.4], their int values are compiled into the clients that use them [JLS, 13.1]. If the value associated with an int enum is changed, its clients must be recompiled. If not, the clients will still run, but their behavior will be incorrect.
>
> There is no easy way to translate int enum constants into printable strings. If you print such a constant or display it from a debugger, all you see is a number, which isn’t very helpful. There is no reliable way to iterate over all the int enum constants in a group, or even to obtain the size of an int enum group.

使用这种int枚举的程序是非常脆弱的，因为int枚举是编译时常量，他们的int值会被编译到使用他们的程序中。如果一个int枚举相关的值被修改了，它的客户端也必须重新编译。如果客户端没有重新编译而继续执行的话，客户端的行为就是不正确的.

没有一种容易的方法可以把这些int枚举值转换为可以打印出来的字符串。如果你直接在打印或者在调试器里显示这个常量，你看到的就是一个用处不大的数字。也没有可靠的方法来编译组里的所有int枚举常量，甚至没办法回去到int枚举组的大小。

> You may encounter a variant of this pattern in which String constants are used in place of int constants. This variant, known as the String *enum pattern*, is even less desirable. While it does provide printable strings for its constants, it can lead naive users to hard-code string constants into client code instead of using field names. If such a hard-coded string constant contains a typographical error, it will escape detection at compile time and result in bugs at runtime. Also, it might lead to performance problems, because it relies on string comparisons.

你可能遇到过int枚举模式的一种变体，这种变体使用String来代替int常量。这种变体被称为String枚举模式，更不值得推荐。虽然它为每个常量提供了可打印的字符串，但是它可能会让初级程序员在客户端代码中使用字符串硬编码，而不是使用变量名。一旦这样的硬编码中存在书写错误，就会导致程序在编译时正常，而运行时出错。而且这种模式还存在性能问题，因为它依赖字符串之间的比较。

> Luckily, Java provides an alternative that avoids all the shortcomings of the int and string enum patterns and provides many added benefits. It is the *enum type* [JLS, 8.9]. Here’s how it looks in its simplest form:

幸运的是，Java提供了一种另一种可替代的解决方案，克服了int和String枚举模式的所有问题，还提供了很多新增的好处，它就是枚举类型。下面是它最简单的一种形式：

```java
public enum Apple  { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD }
```

> On the surface, these enum types may appear similar to those of other languages, such as C, C++, and C#, but appearances are deceiving. Java’s enum types are full-fledged classes, far more powerful than their counterparts in these other languages, where enums are essentially int values.

从表面上看，枚举类型和其他语言比如C，C++和C#，好像有点类似。但实际上并非如此，Java中的Enum类型是功能齐全的类，比其他语言中的枚举要强大得多，其他语言的枚举本质上是int值（*此处，中文版书籍翻译为”java的枚举本质是int值“，我不认可，但又不确定*）。

> The basic idea behind Java’s enum types is simple: they are classes that export one instance for each enumeration constant via a public static final field. Enum types are effectively final, by virtue of having no accessible constructors. Because clients can neither create instances of an enum type nor extend it, there can be no instances but the declared enum constants. In other words, enum types are instance-controlled (page 6). They are a generalization of singletons (Item 3), which are essentially single-element enums.

Java的Enum类型的基本想法很简单：他们就是使用公有静态final域为每一个枚举常量导出一个实例的类。Enum类型由于没有可以访问的构造器，Enum类型是真正的final类。由于客户端既不能穿件enum类型的实例，也不能继承enum类，因为除了它声明的枚举实例外，不会有其他的实例。也就是说，枚举类型是实例受控的（详见Item1）。它们是一组单例的集合（Item3），单例本质上是每个元素的枚举。

> Enums provide compile-time type safety. If you declare a parameter to be of type Apple, you are guaranteed that any non-null object reference passed to the parameter is one of the three valid Apple values. Attempts to pass values of the wrong type will result in compile-time errors, as will attempts to assign an expression of one enum type to a variable of another, or to use the == operator to compare values of different enum types.

Enum类型提供了编译时的类型安全。只要你声明了一个参数的类型是Apple，那么久可以保证传递给这个参数的任何非空的对象引用，一定是这三个有效的Apple值之一。试图传递不同错误类型的值，编译时会出现error，试图把一个类型的表达式赋值给另外一个类型的变量，或者使用==操作符来比较两个不同的enum类型的时候，都会出现编译时错误。

> Enum types with identically named constants coexist peacefully because each type has its own namespace. You can add or reorder constants in an enum type without recompiling its clients because the fields that export the constants provide a layer of insulation between an enum type and its clients: constant values are not compiled into the clients as they are in the int enum patterns. Finally, you can translate enums into printable strings by calling their toString method.

包含同名常量的多个Enum类型也可以在系统的和平共处，因为每个Enum类型都有自己的命名空间。你可以新增或者打乱枚举类型中的常量，而不需要重新编译其客户端，因为导出的常量的域为客户端和枚举类型提供了一个隔绝层：其常量值，不再像int枚举模式一样，编译在客户端代码里。最后，你可以通过toString方法将枚举类型转换成可打印的字符串。

> In addition to rectifying the deficiencies of int enums, enum types let you add arbitrary methods and fields and implement arbitrary interfaces. They provide high-quality implementations of all the Object methods (Chapter 3), they implement Comparable (Item 14) and Serializable (Chapter 12), and their serialized form is designed to withstand most changes to the enum type.

除了完善了enums的不足之处以外，enum类型还允许添加任意多的方法和域，允许实现任意多的接口。Enum还提供了所有的Object方法的高效实现（第三章），还实现了Comparable接口（Item14）和Serializable接口（第12章），以及它的序列化形式可以承受大多数的enum类型的转换形式。

> So why would you want to add methods or fields to an enum type? For starters, you might want to associate data with its constants. Our Apple and Orange types, for example, might benefit from a method that returns the color of the fruit, or one that returns an image of it. You can augment an enum type with any method that seems appropriate. An enum type can start life as a simple collection of enum constants and evolve over time into a full-featured abstraction.

那么我们为什么妖王enum类型里添加方法或者域呢？一开始，可能是想把数据和这些实例关联起来。比如我们的Apple和Orange类型，添加一个可以返回水果颜色的方法，或者返回图片的方法，就很有必要。你可以给enum类型增加任何一个看起来合适的方法。一个Enum类型一开始可能就是一组枚举常量的集合，随着时间推进，就进化成了一个功能齐全的抽象了。

> For a nice example of a rich enum type, consider the eight planets of our solar system. Each planet has a mass and a radius, and from these two attributes you can compute its surface gravity. This in turn lets you compute the weight of an object on the planet’s surface, given the mass of the object. Here’s how this enum looks. The numbers in parentheses after each enum constant are parameters that are passed to its constructor. In this case, they are the planet’s mass and radius:

举一个功能齐全的enum类型的例子，比如太阳系里的八大行星。每一个行星都质量和半径，然后根据质量和半径可以计算其表面重力值。然后给定一个对象额质量，就可以计算其在该行星表面所受的重力。下面是这个Enum的代码。每个枚举后面的圆括号内的数字，是要传递给构造器的参数。在这个例子中，就是行星的质量和半径。

```java
// Enum type with data and behavior
   public enum Planet {
       MERCURY(3.302e+23, 2.439e6),
       VENUS  (4.869e+24, 6.052e6),
       EARTH  (5.975e+24, 6.378e6),
       MARS   (6.419e+23, 3.393e6),
       JUPITER(1.899e+27, 7.149e7),
       SATURN (5.685e+26, 6.027e7),
       URANUS (8.683e+25, 2.556e7),
       NEPTUNE(1.024e+26, 2.477e7);
       private final double mass;           // In kilograms
       private final double radius;         // In meters
       private final double surfaceGravity; // In m / s^2
     	 // Universal gravitational constant in m^3 / kg s^2
       private static final double G = 6.67300E-11;
       // Constructor
       Planet(double mass, double radius) {
           this.mass = mass;
           this.radius = radius;
           surfaceGravity = G * mass / (radius * radius);
			 }
       public double mass()           { return mass; }
       public double radius()         { return radius; }
       public double surfaceGravity() { return surfaceGravity; }
       public double surfaceWeight(double mass) {
           return mass * surfaceGravity;  // F = ma
			 } 
  }
```

> It is easy to write a rich enum type such as Planet. **To associate data with enum constants, declare instance fields and write a constructor that takes the data and stores it in the fields.** Enums are by their nature immutable, so all fields should be final (Item 17). Fields can be public, but it is better to make them private and provide public accessors (Item 16). In the case of Planet, the constructor also computes and stores the surface gravity, but this is just an optimization. The gravity could be recomputed from the mass and radius each time it was used by the surfaceWeight method, which takes an object’s mass and returns its weight on the planet represented by the constant.

要编写一个像Planet这样丰富的Enum类型并不难。**为了将枚举常量和数据关联起来，需要声明实例数据域，然后写一个构造器，使用这些数据，并把他们保存在数据域里**。Enum实例天生就是不可变的，因此所有的域都应该是final的（Item17）。域可以是公开的，但是最好还是将域设为私有的，然后提供公有的访问方法（Item16）。在Planet这个例子中，构造器还计算并保存了surfaceGravity的值，这仅仅是一个优化。这个surfaceGravity的值也可以在，每次调用surfaceWeight方法时，使用mass和radius重新计算。surfaceWeight方法参数为一个对象的质量，返回这个对象在这个常量代表的星星上的重量。

> While the Planet enum is simple, it is surprisingly powerful. Here is a short program that takes the earth weight of an object (in any unit) and prints a nice table of the object’s weight on all eight planets (in the same unit):

虽然这个Planet枚举的代码很简单，但是它的功能强大到让人出奇。下面这段简单的程序，根据某个物体在地球上的重量（任何单位），答应出一个超级棒的表格，显示该物体在8个行星上的重量（相同单位）。代码如下：

```java
public class WeightTable {
  public static void main(String[] args) {
		double earthWeight = Double.parseDouble(args[0]);
		double mass = earthWeight / Planet.EARTH.surfaceGravity(); 
		for (Planet p : Planet.values())
       System.out.printf("Weight on %s is %f%n", p, p.surfaceWeight(mass));
		} 
}
```

> Note that Planet, like all enums, has a static values method that returns an array of its values in the order they were declared. Note also that the toString method returns the declared name of each enum value, enabling easy printing by println and printf. If you’re dissatisfied with this string representation, you can change it by overriding the toString method. Here is the result of running our WeightTable program (which doesn’t override toString) with the command line argument 185:

需要注意的是，这个Planet，以及所有的枚举，都有一个静态的values方法，可以返回一个枚举值的数组，其舜玉和声明的顺序一致。还需要注意的是，Enum的toString方法会返回每个枚举值声明的名字，这样println和printf方法就可以很容易的打印它们。如果你不满意这个字符串表达方式，你可以通过覆盖toString方法来覆盖它。下面是使用命令行参数185，来运行WeightTable程序打印的结果（没有覆盖toString方法）：

```java
	 Weight on MERCURY is 69.912739
   Weight on VENUS is 167.434436
   Weight on EARTH is 185.000000
   Weight on MARS is 70.226739
   Weight on JUPITER is 467.990696
   Weight on SATURN is 197.120111
   Weight on URANUS is 167.398264
   Weight on NEPTUNE is 210.208751

```

> Until 2006, two years after enums were added to Java, Pluto was a planet. This raises the question “what happens when you remove an element from an enum type?” The answer is that any client program that doesn’t refer to the removed element will continue to work fine. So, for example, our WeightTable program would simply print a table with one fewer row. And what of a client program that refers to the removed element (in this case, Planet.Pluto)? If you recompile the client program, the compilation will fail with a helpful error message at the line that refers to the erstwhile planet; if you fail to recompile the client, it will throw a helpful exception from this line at runtime. This is the best behavior you could hope for, far better than what you’d get with the int enum pattern.

知道2006年，也就是枚举被添加到java中两年后，Pluto还是一个行星。这个时候，问题就来了，”当你把一个枚举类型中的一个元素删除后，会发生什么？“答案就是，没有引用那个被删除的元素的客户端都可以正常工作。比如，我们的WeightTable程序就只是打印的表格少了一行而已。那么引用了被删除的元素（在这个例子里就是Planet.Pluto)的客户端会发生什么呢？如果你重新编译了这个客户端程序，在编译时就会失败，并且抛出很有用的异常信息，告诉你是哪一行引用了之前的planet；如果没有重新编译客户端，那么运行时就会抛出一个有用的异常。这就是大家最希望的样子了，比之前int枚举模式时要好得多得多。

> Some behaviors associated with enum constants may need to be used only from within the class or package in which the enum is defined. Such behaviors are best implemented as private or package-private methods. Each constant then carries with it a hidden collection of behaviors that allows the class or package containing the enum to react appropriately when presented with the constant. Just as with other classes, unless you have a compelling reason to expose an enum method to its clients, declare it private or, if need be, package-private (Item 15).

枚举常量的一些行为可能只需要在定义该枚举的类或者方法中使用，这种行为最好是用私有或者包级私有的方法来实现。每个常量就会有一组隐藏的行为，使得包含该枚举的类型或包在使用常量的时候，可以运作得很好。和其他的类一样，除非有让人难以拒绝的原因，不要把枚举方法暴露给它的客户端，将其声明为私有的，或者有需要的话，声明为包级私有的（Item15）。

> If an enum is generally useful, it should be a top-level class; if its use is tied to a specific top-level class, it should be a member class of that top-level class (Item 24). For example, the java.math.RoundingMode enum represents a rounding mode for decimal fractions. These rounding modes are used by the BigDecimal class, but they provide a useful abstraction that is not fundamentally tied to BigDecimal. By making RoundingMode a top-level enum, the library designers encourage any programmer who needs rounding modes to reuse this enum, leading to increased consistency across APIs.

如果一个枚举类型有普遍适用性，它就应该是一个顶级类；如果它只是在某个顶级类中使用，那么它应该是这个顶级类的成员类。比如，java.math.RoundingMode枚举类表示十进制小数的舍入模式（rounding mode）。这个舍入模式只在Bigdecimal类里使用，但是RoundingMode提供了一个有用的抽象，不仅仅局限于BigDecimal类。因此类库设计者通过把RoundingMode做成一个顶级类，以鼓励其他需要舍入模式的程序员重用这个枚举，从而增加API之间的一致性。

> The techniques demonstrated in the Planet example are sufficient for most enum types, but sometimes you need more. There is different data associated with each Planet constant, but sometimes you need to associate fundamentally different *behavior* with each constant. For example, suppose you are writing an enum type to represent the operations on a basic four-function calculator and you want to provide a method to perform the arithmetic operation represented by each con- stant. One way to achieve this is to switch on the value of the enum:

Planet示例中的技术对于对于大部分的枚举类型来说，已经足够了，但是有时候，你可能需要更多。在Planet的每个实例上关联了不同的数据，有时候，你可能还需要每个实例关联不同的行为。比如，假如你在写一个表示基本四则运算的枚举类型，你想提供一个方法来执行每个实例表示的算数操作。下面是一种通过在枚举的值上使用switch来实现的代码：

```java
// Enum type that switches on its own value - questionable
public enum Operation {
       PLUS, MINUS, TIMES, DIVIDE;
       // Do the arithmetic operation represented by this constant
       public double apply(double x, double y) {
           switch(this) {
               case PLUS:   return x + y;
               case MINUS:  return x - y;
               case TIMES:  return x * y;
               case DIVIDE: return x / y;
					 }
           throw new AssertionError("Unknown op: " + this);
      }
}
```

> This code works, but it isn’t very pretty. It won’t compile without the throw statement because the end of the method is technically reachable, even though it will never be reached [JLS, 14.21]. Worse, the code is fragile. If you add a new enum constant but forget to add a corresponding case to the switch, the enum will still compile, but it will fail at runtime when you try to apply the new operation.
>
> Luckily, there is a better way to associate a different behavior with each enum constant: declare an abstract apply method in the enum type, and override it with a concrete method for each constant in a *constant-specific class body*. Such methods are known as *constant-specific method implementations*:

这个代码可以工作，但是不是很完美。如果没有throw语句，代码将无法编译，因为从技术的角度上来说，这个方法的末尾是可达的，即使它实际上并不会执行到这段代码[JLS, 14.21]。更糟糕的是，这段代码是易碎的。当你添加了一个新的枚举常量，但是又忘记了在switch语句中添加对应的case的时候，这个enum还是可以编译，但是在运行时，当你想应用这个新的操作的时候，就会失败。

幸运的是，这里有一个很好的方法可以给每个枚举常量关联一个不同的行为：在enum类中声明一个抽象的apply方法，然后在“特定于常量的类主体”中，使用具体的方法来覆盖每个常量的抽象方法。这种方法被称为“特定于常量的方法实现”。代码如下：

```java
// Enum type with constant-specific method implementations
public enum Operation {
	PLUS {public double apply(double x,double y){return x + y;}}, 
  MINUS {public double apply(double x, double y){return x - y;}}, 
  TIMES {public double apply(double x, double y){return x * y;}}, 
  DIVIDE {public double apply(double x, double y){return x / y;}};
  public abstract double apply(double x, double y);
}
```

> If you add a new constant to the second version of Operation, it is unlikely that you’ll forget to provide an apply method, because the method immediately follows each constant declaration. In the unlikely event that you do forget, the compiler will remind you because abstract methods in an enum type must be over- ridden with concrete methods in all of its constants.
>
> Constant-specific method implementations can be combined with constant- specific data. For example, here is a version of Operation that overrides the toString method to return the symbol commonly associated with the operation:

如果你想给这种版本的Operation添加一个新的常量，基本上不可能忘记提供一个apply方法，因为这个方法就直接跟在每个常量声明后面。就算你不小心忘了，编译器也会提醒你，因为枚举类型中的抽象方法，在所有的常量中，必须用具体的方法来覆盖。

特定于常量的方法实现可以和特定于常量的数据一起使用，比如，下面这个版本的Operation，覆盖了toString方法来返回每个operation关联的常用符号，代码如下：

```java
// Enum type with constant-specific class bodies and data
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

> The toString implementation shown makes it easy to print arithmetic expressions, as demonstrated by this little program:

这个toString方法的实现使得答应数学表达式变得更容易，比如下面这一小段代码：

```java
public static void main(String[] args) {
       double x = Double.parseDouble(args[0]);
       double y = Double.parseDouble(args[1]);
       for (Operation op : Operation.values())
           System.out.printf("%f %s %f = %f%n",x, op, y, op.apply(x, y));
}
```

> Running this program with 2 and 4 as command line arguments produces the following output:

使用命令行参数2和4来运行这个程序，会生成如下的输出结果：

```java
	 2.000000 + 4.000000 = 6.000000
   2.000000 - 4.000000 = -2.000000
   2.000000 * 4.000000 = 8.000000
   2.000000 / 4.000000 = 0.500000	
```

> Enum types have an automatically generated valueOf(String) method that translates a constant’s name into the constant itself. If you override the toString method in an enum type, consider writing a fromString method to translate the custom string representation back to the corresponding enum. The following code (with the type name changed appropriately) will do the trick for any enum, so long as each constant has a unique string representation:



















