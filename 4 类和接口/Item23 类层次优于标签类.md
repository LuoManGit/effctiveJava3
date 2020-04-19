### Item23 类层次优于标签类

> Occasionally you may run across a class whose instances come in two or more flavors and contain a *tag* field indicating the flavor of the instance. For example, consider this class, which is capable of representing a circle or a rectangle:

你可能会遇到这样的类，它有两种或多种不同风格的实例，并在实例中有一个标签域（tag field)来表明该实例的风格，比如，下面这个类，可以表示圆形和长方性两种风格：

```java
// Tagged class - vastly inferior to a class hierarchy!
   class Figure {
       enum Shape { RECTANGLE, CIRCLE };
       // Tag field - the shape of this figure
       final Shape shape;
       // These fields are used only if shape is RECTANGLE
       double length;
       double width;
       // This field is used only if shape is CIRCLE
       double radius;
       // Constructor for circle
       Figure(double radius) {
           shape = Shape.CIRCLE;
           this.radius = radius;
       }
       // Constructor for rectangle
       Figure(double length, double width) {
           shape = Shape.RECTANGLE;
           this.length = length;
           this.width = width;
			}
       double area() {
           switch(shape) {
             case RECTANGLE:
               return length * width;
             case CIRCLE:
               return Math.PI * (radius * radius);
             default:
               throw new AssertionError(shape);
					} 
       }
}
```

> Such *tagged classes* have numerous shortcomings. They are cluttered with boilerplate, including enum declarations, tag fields, and switch statements. Readability is further harmed because multiple implementations are jumbled together in a single class. Memory footprint is increased because instances are burdened with irrelevant fields belonging to other flavors. Fields can’t be made final unless constructors initialize irrelevant fields, resulting in more boilerplate. Constructors must set the tag field and initialize the right data fields with no help from the compiler: if you initialize the wrong fields, the program will fail at runtime. You can’t add a flavor to a tagged class unless you can modify its source file. If you do add a flavor, you must remember to add a case to every switch statement, or the class will fail at runtime. Finally, the data type of an instance gives no clue as to its flavor. In short, **tagged classes are verbose, error-prone, and inefficient.**

这种标签类有很多的缺点。它们充斥着样板代码，包括枚举声明、标签域、以及switch语句。把很多个实现杂乱地放在一个类里，可读性非常的差。并且由于每个实例都包含属于其他风格实例的，与该实例不相关的域，导致内存占用也增加了。如果构造器不初始化不相关的域，域就不能做成final的，但若构造器初始化了不相关的域，又会产生更多的样板代码。构造器必须设置标签域以及初始化正确的数据域，编译器帮不上任何的忙，如果你将错误的域进行了初始化，程序就会在运行的时候出错。你还不能给标签类添加一个新的风格，除非你可以修改它的源文件。如果你确实需要添加一个新的风格，你也必须记住要给每一个switch语句中都添加一个case，否则这个类在运行的时候，就会出错。最后，其实例的数据类型也不能提供任何关于其风格的信息。**简而言之，标签类啰嗦冗长、容易出错、还效率低下。

> Luckily, object-oriented languages such as Java offer a far better alternative for defining a single data type capable of representing objects of multiple flavors: subtyping. **A tagged class is just a pallid imitation of a class hierarchy.**

幸运地是，面向对象的语言，比如java都提供了一个更好的方案，来定义能表示多种风格对象的单个数据类型，它就是子类化。**标签类只是类层次的一个简单的模仿**。

> To transform a tagged class into a class hierarchy, first define an abstract class containing an abstract method for each method in the tagged class whose behavior depends on the tag value. In the Figure class, there is only one such method, which is area. This abstract class is the root of the class hierarchy. If there are any methods whose behavior does not depend on the value of the tag, put them in this class. Similarly, if there are any data fields used by all the flavors, put them in this class. There are no such flavor-independent methods or fields in the Figure class.

为了将一个标签类转化为类层次，首先要定义一个抽象类，在抽象类中，对于标签类中的每个行为依赖于标签值的方法都提供一个抽象方法。在前面的Figure类中，只有一个这样的方法，那就是area（）。这个抽象类是类层次结构的根。如果标签类中有一些其行为不依赖标签值的方法，也把它们放在抽象类里。类似地，还有所有风格的实例都会使用的数据域，也应该放在抽象类里，在Figure类中没有这种“风格无关”的方法和域。

> Next, define a concrete subclass of the root class for each flavor of the original tagged class. In our example, there are two: circle and rectangle. Include in each subclass the data fields particular to its flavor. In our example, radius is particular to circle, and length and width are particular to rectangle. Also include in each subclass the appropriate implementation of each abstract method in the root class. Here is the class hierarchy corresponding to the original Figure class:

然后，为原始标签类的每一种风格，定义一个根类的子类。在我们的例子中，有两个：circle和rectangle。在每个子类中，都要包含其对应的数据域。在我们的例子中，radius是属于circle的，length和width是属于rectangle的。在每个子类中，也要包含根类中抽象方法的对应的实现。下面是原始的Figure类对应的类层次结构代码：

```java
// Class hierarchy replacement for a tagged class
   abstract class Figure {
       abstract double area();
	}
```

```java
class Circle extends Figure {
       final double radius;
       Circle(double radius) { 
         this.radius = radius; 
       }
			 @Override double area() { 
         return Math.PI * (radius * radius); 
       } 
}
```

```java
class Rectangle extends Figure {
       final double length;
       final double width;
       Rectangle(double length, double width) {
           this.length = length;
           this.width  = width;
			 }
       @Override double area() { return length * width; }
   }
```

> This class hierarchy corrects every shortcoming of tagged classes noted previously. The code is simple and clear, containing none of the boilerplate found in the original. The implementation of each flavor is allotted its own class, and none of these classes is encumbered by irrelevant data fields. All fields are final. The compiler ensures that each class’s constructor initializes its data fields and that each class has an implementation for every abstract method declared in the root class. This eliminates the possibility of a runtime failure due to a missing switch case. Multiple programmers can extend the hierarchy independently and interoperably without access to the source for the root class. There is a separate data type associated with each flavor, allowing programmers to indicate the flavor of a variable and to restrict variables and input parameters to a particular flavor.

这个类层次解决了前面提到的便签类的所有的问题。这个代码看起来简洁明了，也不包含之前的那些样板代码。每种风格的实现都写在自己的类里，没有哪个类会被不相关的数据域拖累。所有的域都是final的。编译器也可以保证每个类的构造器都初始化自己的数据域，以及每个类都对根类中的抽象方法有具体的实现。也消除了因为switch语句缺失case语句而在运行时报错的可能性。程序员们，在不访问根类的源代码的情况下，也能独立地扩展类层次。针对每个不同的风格，都有单独的数据类型，因此程序员可以指明变量的风格，并且把变量和输入参数限制到一个特定的风格里。

> Another advantage of class hierarchies is that they can be made to reflect natural hierarchical relationships among types, allowing for increased flexibility and better compile-time type checking. Suppose the tagged class in the original example also allowed for squares. The class hierarchy could be made to reflect the fact that a square is a special kind of rectangle (assuming both are immutable):

类层次的另一个优点是，他们可以用来反映类型之间本质上中的层次关系，增加了灵活性，也有助于编译时进行更好的类型检测。假如例子中的便签类还支持square（正方形）。这个类层级结构就可以反映正方形是一种特殊的长方形（假定他们都是不可变的）。代码如下：

```java
class Square extends Rectangle {
       Square(double side) {
           super(side, side);
       }
}
```

> Note that the fields in the above hierarchy are accessed directly rather than by accessor methods. This was done for brevity and would be a poor design if the hierarchy were public (Item 16).
>
> In summary, tagged classes are seldom appropriate. If you’re tempted to write a class with an explicit tag field, think about whether the tag could be eliminated and the class replaced by a hierarchy. When you encounter an existing class with a tag field, consider refactoring it into a hierarchy.

需要注意的是，上面这些类层次代码中的所有的域都是直接访问的，而没有提供访问方法。这是为了代码看起来更加简洁，如果这些类层次是公有的，这就是一个非常差的设计（Item16）。

总结一下，标签类很少有适用的时候。如果你要写一个有明确的标签域的类，就要考虑这个标签是否可以被消除，并用类层次设计来代替原有类。当你遇到一个已经存在的有标签域的类，就应该考虑使用类层次来重构它。

### 