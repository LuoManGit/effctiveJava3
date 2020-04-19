### Item16 在公有类中使用访问方法，而不是公有域

> Occasionally, you may be tempted to write degenerate classes that serve no purpose other than to group instance fields:

有时候，你可能会编写一些退化类，除了用来组织实例域以外，没有其他的作用，如下：

```java
// Degenerate classes like this should not be public!
   class Point {
       public double x;
       public double y;
   }
```

> Because the data fields of such classes are accessed directly, these classes do not offer the benefits of *encapsulation* (Item 15). You can’t change the representation without changing the API, you can’t enforce invariants, and you can’t take auxiliary action when a field is accessed. Hard-line object-oriented programmers feel that such classes are anathema and should always be replaced by classes with private fields and public *accessor methods* (getters) and, for mutable classes, *mutators* (setters):

因为这种类的数据域可以被直接访问，这些类没有提供“封装”（Item15）的优势。你在不改变API的情况下，就不能改变这种表示方法，也不能强加任何约束，当这些域被访问的时候，也不能采取任何的辅助行为。坚持面向对象的程序员对这种类深恶痛绝，应该使用包含私有域和公有访问方法（getters）的类来替代，对于可变对象而言，还应该添加设值方法（setter），如下：

```java
// Encapsulation of data by accessor methods and mutators
   class Point {
       private double x;
       private double y;
       public Point(double x, double y) {
           this.x = x;
					 this.y = y; }
       public double getX() { return x; }
       public double getY() { return y; }
       public void setX(double x) { this.x = x; }
       public void setY(double y) { this.y = y; }
}
```

> Certainly, the hard-liners are correct when it comes to public classes: **if a class is accessible outside its package, provide accessor methods** to preserve the flexibility to change the class’s internal representation. If a public class exposes its data fields, all hope of changing its representation is lost because client code can be distributed far and wide.

毫无疑问，对于公有类而言，面向那个对象的编程是正确的：**如果一个类在其所在包外可达，提供访问方法**可以拥有改变类的内部表示形式的灵活性。如果一个共有类导出了它的数据域，那么就无法改变其表示形式了，因为客户端的代码，可能到处都是了。

> However, **if a class is package-private or is a private nested class, there is nothing inherently wrong with exposing its data fields**—assuming they do an adequate job of describing the abstraction provided by the class. This approach generates less visual clutter than the accessor-method approach, both in the class definition and in the client code that uses it. While the client code is tied to the class’s internal representation, this code is confined to the package containing the class. If a change in representation becomes desirable, you can make the change without touching any code outside the package. In the case of a private nested class, the scope of the change is further restricted to the enclosing class.

然而，如果一个类是包级私有的或者是一个私有嵌套类，暴露其数据域就没有什么本质上的问题了--假定这些数据域确实是这个类的抽象中的一部分。相对于访问方法模式，当类的定义和客户端都使用的时候，这种方式不容易带来视觉混乱。当客户端代码被绑定到一个类的内部表示上时，客户端代码也只能在这个类的包内。如果需要改变其表达方式的时候，就可以在不影响包外代码的情况下做修改。对于私有嵌套类，这个需要改变的范围就进一步限制到这个外围类里了。

> Several classes in the Java platform libraries violate the advice that public classes should not expose fields directly. Prominent examples include the Point and Dimension classes in the java.awt package. Rather than examples to be emulated, these classes should be regarded as cautionary tales. As described in Item 67, the decision to expose the internals of the Dimension class resulted in a serious performance problem that is still with us today.

Java平台中的几个类违反了共有类不要直接暴露数据域的建立，有名的例子是 java.awt包内的Point 和Dimension类。这些例子是不应该被效仿的，而且他们还应该被作为反面的警示例子。正如Item67介绍的那样，在Dimension类中暴露内部数据的决定导致了一些严重的性能问题，这个问题现在还存在。

> While it’s never a good idea for a public class to expose fields directly, it is less harmful if the fields are immutable. You can’t change the representation of such a class without changing its API, and you can’t take auxiliary actions when a field is read, but you can enforce invariants. For example, this class guarantees that each instance represents a valid time:

虽然对于公有类而言，直接暴露其域从来都不是一个好方法，但如果这些域是不可变的，危害就会小一些。除非修改这个类的api，就无法改变这个类的表示形式，且当这个域被读取的时候，也不能采取辅助措施，但是你可以强加约束条件。如下，这个类保证了每个类都表示一个合法的时间：

```java
// Public class with exposed immutable fields - questionable
   public final class Time {
       private static final int HOURS_PER_DAY    = 24;
       private static final int MINUTES_PER_HOUR = 60;
       public final int hour;
       public final int minute;
       public Time(int hour, int minute) {
           if (hour < 0 || hour >= HOURS_PER_DAY)
              throw new IllegalArgumentException("Hour: " + hour);
           if (minute < 0 || minute >= MINUTES_PER_HOUR)
							throw new IllegalArgumentException("Min: " + minute); 
					 this.hour = hour;
					 this.minute = minute;
			}
       ... // Remainder omitted
   }
```

> In summary, public classes should never expose mutable fields. It is less harmful, though still questionable, for public classes to expose immutable fields. It is, however, sometimes desirable for package-private or private nested classes to expose fields, whether mutable or immutable.

总结一下，公有类应该永远都不暴露其可变域。对于共有类暴露不可变域，虽然危害不大，但还是有问题。但有时候还是需要包级私有和私有内部类暴露其域的，不管是可变域还是不可变域。

### 