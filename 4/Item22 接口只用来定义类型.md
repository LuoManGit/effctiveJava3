### Item22 接口只用来定义类型

> When a class implements an interface, the interface serves as a *type* that can be used to refer to instances of the class. That a class implements an interface should therefore say something about what a client can do with instances of the class. It is inappropriate to define an interface for any other purpose.
>
> One kind of interface that fails this test is the so-called *constant interface*. Such an interface contains no methods; it consists solely of static final fields, each exporting a constant. Classes using these constants implement the interface to avoid the need to qualify constant names with a class name. Here is an example:

当一个类实现了一个接口后，这个接口就可以充当可以引用这个类实例的类型。因此，一个类实现了一个接口，就意味着表明了客户端可以对这个类的实例做一些事情。为了其他的目的而设计接口都是不合适的。

有一类违反这个建议的接口叫做“常量接口（constant interface）”。这样的接口里不包含方法，只包含一些静态final域，用来导出一个常量。类实现这个接口就可以直接使用这些常量，可以避免通过类名来修饰常量名。下面是个例子：

```java
 // Constant interface antipattern - do not use!
   public interface PhysicalConstants {
       // Avogadro's number (1/mol)
       static final double AVOGADROS_NUMBER   = 6.022_140_857e23;
       // Boltzmann constant (J/K)
       static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
       // Mass of the electron (kg)
       static final double ELECTRON_MASS      = 9.109_383_56e-31;
   }
```

> **The constant interface pattern is a poor use of interfaces.** That a class uses some constants internally is an implementation detail. Implementing a constant interface causes this implementation detail to leak into the class’s exported API. It is of no consequence to the users of a class that the class implements a constant interface. In fact, it may even confuse them. Worse, it represents a commitment: if in a future release the class is modified so that it no longer needs to use the constants, it still must implement the interface to ensure binary compatibility. If a nonfinal class implements a constant interface, all of its subclasses will have their namespaces polluted by the constants in the interface.

常量接口是接口的一种退化的使用，类使用使用这些常量，本质上是一种实现细节。而实现一个常量接口，会将这些实现细节暴露到类的导出API中。类实现常量接口，对于类的用户来说并没有什么用处。实际上，往往会让他们感到困惑。更糟糕的是，它还代表了一个承诺：即使在未来的版本中，类已经修改到不需要这些常量了，为了保持兼容性，这个类还是必须要实现这个接口。如果一个非final的类实现了一个常量接口，那么它的所有的子类的命名空间都被这个接口的常量污染了。

> There are several constant interfaces in the Java platform libraries, such as java.io.ObjectStreamConstants. These interfaces should be regarded as anomalies and should not be emulated.

在Java平台类库中也有几个常量接口，比如java.io.ObjectStreamConstants。这些接口应该被视为反面的例子，不应该去模仿。

> If you want to export constants, there are several reasonable choices. If the constants are strongly tied to an existing class or interface, you should add them to the class or interface. For example, all of the boxed numerical primitive classes, such as Integer and Double, export MIN_VALUE and MAX_VALUE constants. If the constants are best viewed as members of an enumerated type, you should export them with an *enum type* (Item 34). Otherwise, you should export the constants with a noninstantiable *utility class* (Item 4). Here is a utility class version of the PhysicalConstants example shown earlier:

如果你确实需要导出常量，这里有几种合理的选择。如果常量和某个存在的接口或类紧密相关，你就应该把它添加到这个类或接口里去。比如所有的数值型基本类型的封装类，比如Integer和Double都导出了MIN_VALUE和MAX_VALUE常量。如果这些常量可以很好地看做是枚举类型的成员，那就应该用枚举类型（Item34)来导出它们。除这些情况以外，你应该用一个不可实例化的工具类（Item4）来导出这些常量。下面是前面展示的PhysicalConstants类的一个工具类版本：

```java
// Constant utility class
package com.effectivejava.science;
public class PhysicalConstants {
   private PhysicalConstants() { }  // Prevents instantiation
	 public static final double AVOGADROS_NUMBER = 6.022_140_857e23;  					   
   public static final double BOLTZMANN_CONST =1.380_648_52e-23; 
	 public static final double ELECTRON_MASS = 9.109_383_56e-31;
}
```

> Incidentally, note the use of the underscore character (_) in the numeric literals. Underscores, which have been legal since Java 7, have no effect on the values of numeric literals, but can make them much easier to read if used with discretion. Consider adding underscores to numeric literals, whether fixed of floating point, if they contain five or more consecutive digits. For base ten literals, whether integral or floating point, you should use underscores to separate literals into groups of three digits indicating positive and negative powers of one thousand.

顺便提一下，注意例子中的数字字面量的下划线(_)的使用。下划线，在Java7中开始就是合法的了，对数字字面量的值没有任何影响，使用得当，能有效的提高数字的可读性。不管是定点数还是浮点数，如果它包含了5个或以上连续的数字，都应该在字面量中考虑使用下划线。不管是定点数还是浮点数，对于基数为10的字面量而言，你应该每三位使用下划线隔开，以表示一千的正负倍数（*乘1000，除1000*）。

> Normally a utility class requires clients to qualify constant names with a class name, for example, PhysicalConstants.AVOGADROS_NUMBER. If you make heavy use of the constants exported by a utility class, you can avoid the need for qualifying the constants with the class name by making use of the *static import* facility:

通常情况下，工具类需要客户端通过类名来访问常量，比如PhysicalConstants.AVOGADROS_NUMBER。如果你对一个工具类中导出的常量使用很频繁的话，你可以使用静态导入机制，来避免通过类名访问常量。如下：

```java
// Use of static import to avoid qualifying constants
   import static com.effectivejava.science.PhysicalConstants.*;
   public class Test {
       double atoms(double mols) {
           return AVOGADROS_NUMBER * mols;
       }
...
// Many more uses of PhysicalConstants justify static import
   }

```

> In summary, interfaces should be used only to define types. They should not be used merely to export constants.

总而言之，接口只应该用来定义类型，不应该只用来导出常量。

### 