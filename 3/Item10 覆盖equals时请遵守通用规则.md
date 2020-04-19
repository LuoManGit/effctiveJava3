## 3 对所有的对象都通用的方法

> **A**LTHOUGH Object is a concrete class, it is designed primarily for extension. All of its nonfinal methods (equals, hashCode, toString, clone, and finalize) have explicit *general contracts* because they are designed to be overridden. It is the responsibility of any class overriding these methods to obey their general contracts; failure to do so will prevent other classes that depend on the contracts (such as HashMap and HashSet) from functioning properly in conjunction with the class.
>
> This chapter tells you when and how to override the nonfinal Object methods. The finalize method is omitted from this chapter because it was discussed in Item 8. While not an Object method, Comparable.compareTo is discussed in this chapter because it has a similar character.

虽然Object是一个具体的类，但是这个类就是设计来扩展的。为了更好的被覆盖（overridden），它的所有非final的方法（equals, hashCode, toString, clone, 和 finalize）都有明确的通用约定（general contract）。任何类要覆盖这些方法的时候，都有责任遵守这些通用约定。否则这些类将无法和基于这个约定时限的类（不如HashMap和HashSet）一起正常的运作。

本章告诉你什么时候以及如何去覆盖Object的非final方法。其中finalize方法并不在本章中讨论，因为Item8里面已经讨论过了。虽然Comparable.compareTo不是Object的方法，但由于它有类似的特征，所以本章中也对它进行了讨论。

### Item10 覆盖equals时请遵守通用规则

> Overriding the equals method seems simple, but there are many ways to get it wrong, and consequences can be dire. The easiest way to avoid problems is not to override the equals method, in which case each instance of the class is equal only to itself. This is the right thing to do if any of the following conditions apply:

覆盖equals方法看起来很简单，但是很容易出错，并且后果十分严重。最简单的避免问题的方法就是不覆盖equals方法，在这种情况下，每个对象都只等于它自己。只要满足以下任何一种情况都没有必要覆盖equals方法。

> • **Each instance of the class is inherently unique.** This is true for classes such as Thread that represent active entities rather than values. The equals implementation provided by Object has exactly the right behavior for these classes.
>
> • **There is no need for the class to provide a “logical equality” test.** For example, java.util.regex.Pattern could have overridden equals to check whether two Pattern instances represented exactly the same regular expression, but the designers didn’t think that clients would need or want this functionality. Under these circumstances, the equals implementation inherited from Object is ideal.

**每个实例本质上都是唯一的类**。对于那些代表活动实体而不是值的类（比如 Thread）正是如此。Object提供的equals实现对于这些类来说，就是正确的行为。

**并不需要提供”逻辑相等“的测试的类”**。比如java.util.regex.Pattern可以覆盖equals方法来检测两个Pattern实例是否代表同一个正则表达式，但是设计者人为客户端并不需要或者说不想要这样的功能。在这种情况下，直接继承Object类的equals实现是最理想的。

> - **A super class has already over ridden equals,and the super class behavior is appropriate for this class**.  For example, most Set implementations inherit their equals implementation from AbstractSet, List implementations from AbstractList, and Map implementations from AbstractMap.
> - **The classis private or package-private,and you are certain that its equals method will never be invoked.** If you are extremely risk-averse, you can override the equals method to ensure that it isn’t invoked accidentally.

**当其父类已经覆盖了equals方法，并且父类的行为也适用的类**。比如，大部分的Set实现类都从AbstractSet那里继承了equals实现，List实现类从AbstractList继承equals实现，Map实现类从AbstractMap继承equals实现。

**私有或者包级私有，你确定它的equals方法不会被调用的类**。如果你非常不想貌相，你可以像下面这样覆盖equals方法，以防止被意外的调用。

```java
@Override public boolean equals(Object o) {
         throw new AssertionError(); // Method is never called
}

```

> So when is it appropriate to override equals? It is when a class has a notion of *logical equality* that differs from mere object identity and a superclass has not already overridden equals. This is generally the case for *value classes.* A value class is simply a class that represents a value, such as Integer or String. A programmer who compares references to value objects using the equals method expects to find out whether they are logically equivalent, not whether they refer to the same object. Not only is overriding the equals method necessary to satisfy programmer expectations, it enables instances to serve as map keys or set elements with predictable, desirable behavior.
>
> One kind of value class that does *not* require the equals method to be overridden is a class that uses instance control (Item 1) to ensure that at most one object exists with each value. Enum types (Item 34) fall into this category. For these classes, logical equality is the same as object identity, so Object’s equals method functions as a logical equals method.

那什么是时候需要覆盖equals方法呢？当这个类有逻辑相等的概念（不等同于对象相等的概念），并且父类没有覆盖equals方法。通常是指一些值类。值类是指这个类就是代表一个值，比如Integer或者String类。当程序员使用equals方法来比较两个值对象的引用时，期望知道他们是不是逻辑相等，而不是知道他们是不是指向同一个对象。重新equals方法不仅仅是为了满足程序员的需求，也使得这些实例在作为map的key或者set的元素的时候，出现我们预期的、满意的行为。

但有一种值类确实不需要覆盖equals方法，这种值类实例受控（Item1），保证了每个值只有一个对象。Enum类型是属于这种值类（Item34）。对于这种类，逻辑相等和对象相等的等价的，因此Object的equals方法也就是一个逻辑相等的方法。

> When you override the equals method, you must adhere to its general contract. Here is the contract, from the specification for Object :
>
> The equals method implements an *equivalence relation.* It has these properties:
>
> - *Reflexive*: For any non-null reference value x , x.equals(x) must return true.
> - *Symmetric*: For any non-null reference values x and y , x.equals(y) must return true if and only if y.equals(x) returns true.
> - *Transitive* : For any non-null reference values x, y, z, if x.equals(y) returns true and y.equals(z) returns true, then x.equals(z) must return true.
> - *Consistent*: For any non-null reference values x and y, multiple invocations of x.equals(y) must consistently return true or consistently return false, provided no information used in equals comparisons is modified.
> - For any non-null reference value x, x.equals(null) must return false.

当你想要覆盖equals方法的时候，必须要遵守它的通用约定，下面就是这些约定，来自Object的规范：

equals方法实现了一种等价关系，它的属性如下：

- 自反性：对于任何非null的引用值x，x.equals(x)必须返回true。
- 对称性：对于任何非null的引用值x，y，当且仅当x.equals(y)返回为true时，y.equals(x)必须返回true。
- 传递性：对于任意非null的引用值x，y，z，如果 x.equals(y)返回true，并且y.equals(z)返回true，那么x.equals(z)就必须返回true。
- 一致性：对于任意非空的引用值x，y，只要对象中equals需要的信息没有被修改，那么多次调用 x.equals(y) 的返回值必须一致地返回true，或者一致地返回false。
- 对于任意非空的引用值x， x.equals(null)必须返回false。

> Unless you are mathematically inclined, this might look a bit scary, but do not ignore it! If you violate it, you may well find that your program behaves erratically or crashes, and it can be very difficult to pin down the source of the failure. To paraphrase John Donne, no class is an island. Instances of one class are frequently passed to another. Many classes, including all collections classes, depend on the objects passed to them obeying the equals contract.
>
> Now that you are aware of the dangers of violating the equals contract, let’s go over the contract in detail. The good news is that, appearances notwithstanding, it really isn’t very complicated. Once you understand it, it’s not hard to adhere to it.

除非你对数学很感兴趣，否则这些看起来有点可怕，但是千万不要因此忽略这些。如果你违反这些约定，你的程序可能就会行为异常甚至崩溃，而且要定位要问题的源头非常的困难。正如John Donne所说的那样，没有哪个类是一座孤岛，一个类的实例需要频繁地传递给其他的类实例。有很多的类，包括所有的集合类，都需要传给他的对象遵守了equals的约定。

现在已经意识到违反equals约定的危险性。就让我们来详细地看看这些约定。好消息是这些约定表面上看起来复杂，但实际上并不是那么复杂。一旦理解了，要遵守也不难。

> So what is an equivalence relation? Loosely speaking, it’s an operator that partitions a set of elements into subsets whose elements are deemed equal to one another. These subsets are known as *equivalence classes*. For an equals method to be useful, all of the elements in each equivalence class must be interchangeable from the perspective of the user. Now let’s examine the five requirements in turn:
>
> **Reflexivity**—The first requirement says merely that an object must be equal to itself. It’s hard to imagine violating this one unintentionally. If you were to violate it and then add an instance of your class to a collection, the contains method might well say that the collection didn’t contain the instance that you just added.

什么是对等关系呢？不严格地说，对等关系就是一个将一组元素划分成几个小组的操作，并认为每个分组里的元素是相互相等的，这些分组被称为”等价类（equivalence class）“*这个名字好有歧义*。从用户的角度来看，对于一个可用的equals方法，一个等价类里的所有的元素都是可交换的。现在我们挨着挨着看看这5个要求：

**自反性** — 第一个要求就是说一个对象必须和自己相等。很难想象要怎么才能违背这条约定。如果你违背了这条约定，然后将一个类的实例添加到集合里，然后contains方法可能会告诉你集合里没有你刚刚添加的这个实例。

> **Symmetry**—The second requirement says that any two objects must agree on whether they are equal. Unlike the first requirement, it’s not hard to imagine violating this one unintentionally. For example, consider the following class, which implements a case-insensitive string. The case of the string is preserved by toString but ignored in equals comparisons:

**对称性** — 第二个要求是说两个对象进行equals比较的时候结果应该一致。和第一个要求不一样，这条还是比较容易无意间违反的。比如看看下面这个类，它实现了一个大小写不敏感的String。字符串被toString保存，但是在equals比较中被忽略了（？？？）。

```java
// Broken - violates symmetry!
   public final class CaseInsensitiveString {
       private final String s;
       public CaseInsensitiveString(String s) {
           this.s = Objects.requireNonNull(s);
			 }
       // Broken - violates symmetry!
       @Override public boolean equals(Object o) {
           if (o instanceof CaseInsensitiveString)
               return s.equalsIgnoreCase(
                   ((CaseInsensitiveString) o).s);
           if (o instanceof String)  // One-way interoperability!
               return s.equalsIgnoreCase((String) o);
           return false;
       }
       ...  // Remainder omitted
   }
```

> The well-intentioned equals method in this class naively attempts to interoperate with ordinary strings. Let’s suppose that we have one case-insensitive string and one ordinary one:

这个类里的精心设计的equals方法，企图将CaseInsensitiveString和原始String对象进行互操作，让我们假设我们有一个CaseInsensitiveString对象和一个普通的字符串，如下：

```java
CaseInsensitiveString cis = new CaseInsensitiveString("Polish"); 
String s = "polish";
```

> As expected, cis.equals(s) returns true. The problem is that while the equals method in CaseInsensitiveString knows about ordinary strings, the equals method in String is oblivious to case-insensitive strings. Therefore, s.equals(cis) returns false, a clear violation of symmetry. Suppose you put a case-insensitive string into a collection:

cis.equals(s)会如期返回true。但是问题是CaseInsensitiveString里面的类会判断原始的类，但是String里面的equals方法不会考虑大小写不敏感的字符串，因此s.equals(cis)会返回false，显然这就违反了对称性。假如你把一个CaseInsensitiveString对象放在一个集合里如下：

```java
   List<CaseInsensitiveString> list = new ArrayList<>();
   list.add(cis);
```

> What does list.contains(s) return at this point? Who knows? In the current OpenJDK implementation, it happens to return false, but that’s just an implementation artifact. In another implementation, it could just as easily return true or throw a runtime exception. **Once you’ve violated the** **equals** **contract, you simply don’t know how other objects will behave when confronted with your object.**
>
> To eliminate the problem, merely remove the ill-conceived attempt to interoperate with String from the equals method. Once you do this, you can refactor the method into a single return statement:

在这种情况下，list.contains(s)会返回什么呢？没有知道，在OpenJDK的实现中，它会返回false，但是这只是特定的实现返回的结果而已。在其他的实现中，可能返回true，也可能抛出一个runtime Exception。当你违反了equals的预订时，很难知道其他的对象和你的对象合作的时候会出现什么行为。

为了解决这个问题，只需要在equals方法中将企图和String对象互操作的代码删掉就好了。一旦删掉后，我们就可以用一条单独的语句重构这个方法，如下：

```java
@Override public boolean equals(Object o) {
       return o instanceof CaseInsensitiveString &&
           ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
}
```

> **Transitivity**—The third requirement of the equals contract says that if one object is equal to a second and the second object is equal to a third, then the first object must be equal to the third. Again, it’s not hard to imagine violating this requirement unintentionally. Consider the case of a subclass that adds a new *value component* to its superclass. In other words, the subclass adds a piece of information that affects equals comparisons. Let’s start with a simple immutable two-dimensional integer point class:

**传递性** — 第三个要求是说如果一个对象equal to第二个对象，第二个对象equal to第三个对象，那么第一个对象也必须要 equal to 第三个对象。同样地，这条也很容易无意间违背。考虑这样一种情况，一个子类添加一个新的值组件（value component）到其父类中。换句话说就是子类添加的信息影响了equals的比较结果。让我们以一个简单的不可变的二维整形Point类为例：

```java
public class Point {
       private final int x;
       private final int y;
       public Point(int x, int y) {
           this.x = x;
					 this.y = y; 
			 }
       @Override public boolean equals(Object o) {
           if (!(o instanceof Point))
               return false;
           Point p = (Point)o;
           return p.x == x && p.y == y;
			}
       ...  // Remainder omitted
   }
```

> Suppose you want to extend this class, adding the notion of color to a point:

假如你想继承这个类，给这个点加上颜色信息,如下：

```java
public class ColorPoint extends Point {
       private final Color color;
       public ColorPoint(int x, int y, Color color) {
           super(x, y);
           this.color = color;
       }
       ...  // Remainder omitted
   }
```

> How should the equals method look? If you leave it out entirely, the implementation is inherited from Point and color information is ignored in equals comparisons. While this does not violate the equals contract, it is clearly unacceptable. Suppose you write an equals method that returns true only if its argument is another color point with the same position and color:

这里的equals方法应该怎么写呢？如果你完全不管的话，就将会从Point继承equals实现，那么equals比较中就完全没有考虑到color信息。虽然这样做并没有违反equals的约定，但这显然是不接受的。假如写了一个下面这样的equals方法，只有当参数也是一个ColorPoint对象，同时color和point信息完全一致时，返回true：

```java
 // Broken - violates symmetry!
   @Override public boolean equals(Object o) {
       if (!(o instanceof ColorPoint))
          return false;
       return super.equals(o) && ((ColorPoint) o).color == color;
}
```

> The problem with this method is that you might get different results when comparing a point to a color point and vice versa. The former comparison ignores color, while the latter comparison always returns false because the type of the argument is incorrect. To make this concrete, let’s create one point and one color point:

这个方法的问题是当我们比较一个Point对象和一个ColorPoint对象时，违反了对称性。Point对象的equals方法忽略了color信息，可能会返回true；但是ColorPoint对象的equals方法会因为参数不正确总是返回false。为了具体说明，创建一个Point对象和一个ColorPoint对象入下：

```java
Point p = new Point(1, 2);
ColorPoint cp = new ColorPoint(1, 2, Color.RED);
```

> Then p.equals(cp) returns true, while cp.equals(p) returns false. You might try to fix the problem by having ColorPoint.equals ignore color when doing “mixed comparisons”:

此时p.equals(cp)会返回true，而cp.equals(p)会返回false。为了解决这个问题，你可能会修改ColorPoint的equals方法，当在做混合比较的时候，忽略掉color属性。如下：

```java
// Broken - violates transitivity!
   @Override public boolean equals(Object o) {
       if (!(o instanceof Point))
           return false;
       // If o is a normal Point, do a color-blind comparison
       if (!(o instanceof ColorPoint))
           return o.equals(this);
       // o is a ColorPoint; do a full comparison
       return super.equals(o) && ((ColorPoint) o).color == color;
   }
```

> This approach does provide symmetry, but at the expense of transitivity:

这个方法确实满足了对称性，但是付出了违背传递性的代价。考虑下面这几个对象：

```java
ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
Point p2 = new Point(1, 2);
ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
```

> Now p1.equals(p2) and p2.equals(p3) return true, while p1.equals(p3) returns false, a clear violation of transitivity. The first two comparisons are “color-blind,” while the third takes color into account.
>
> Also, this approach can cause infinite recursion: Suppose there are two subclasses of Point, say ColorPoint and SmellPoint, each with this sort of equals method. Then a call to myColorPoint.equals(mySmellPoint) will throw a StackOverflowError.

现在就出现了 p1.equals(p2) 和 p2.equals(p3) 都返回true, 但是 p1.equals(p3) 返回 false，显然违背了传递性，这是因为，前两个比较忽略了color，但第三个比较又把color算进来了。

同时，这个方法还可能出现无限递归问题：假如Point有两个子类ColorPoint和SmellPont,这两个子类都有前面这样的equals方法。当调用myColorPoint.equals(mySmellPoint)方法时就会因为无限递归问题抛出StackOverflowError。

> So what’s the solution? It turns out that this is a fundamental problem of equivalence relations in object-oriented languages. **There is no way to extend an instantiable class and add a value component while preserving the** **equals** **contract**, unless you’re willing to forgo the benefits of object-oriented abstraction.

要怎么去解决呢？这是一个面向对象的语言在等价关系上的一个基本问题，没有办法可以在继承一个可实例化的类时，添加一个值组件，同时还遵守equals约定。除非你愿意放弃一些面向对象的抽象带来的好处。

> You may hear it said that you can extend an instantiable class and add a value component while preserving the equals contract by using a getClass test in place of the instanceof test in the equals method:

你可能听过在equals方法在中使用getClass测试来代替instanceOf测试，可以实现继承一个可实例化的类并添加一个纸组件，同时遵守equals约定。如下：

```java
// Broken - violates Liskov substitution principle (page 43)
   @Override public boolean equals(Object o) {
       if (o == null || o.getClass() != getClass())
           return false;
       Point p = (Point) o;
       return p.x == x && p.y == y;
   }
```

> This has the effect of equating objects only if they have the same implementation class. This may not seem so bad, but the consequences are unacceptable: An instance of a subclass of Point is still a Point, and it still needs to function as one, but it fails to do so if you take this approach! Let’s suppose we want to write a method to tell whether a point is on the unit circle. Here is one way we could do it:

这种方法只有当对象具有相同的实现类的时候，才可能相等。这个看起来好像不那么糟糕，但是这个结果确是无法接受的，一个Point的子类仍然是一个Point，他们需要拥有这些功能，但是当我们使用这种方法的时候，就不是这样了。假设我们想写一个方法来判断一个point是不是在单位圆内，我们可以这么做：

```java
// Initialize unitCircle to contain all Points on the unit circle
   private static final Set<Point> unitCircle = Set.of(
           new Point( 1,  0), new Point( 0,  1),
           new Point(-1,  0), new Point( 0, -1));
   public static boolean onUnitCircle(Point p) {
       return unitCircle.contains(p);
   }
```

> While this may not be the fastest way to implement the functionality, it works fine. Suppose you extend Point in some trivial way that doesn’t add a value component, say, by having its constructor keep track of how many instances have been created:

虽然这种方式不是实现这个功能的最快的方法，但是它也能很好的工作。假如你为了一些不重要的功能继承了Point类，没有添加值组件，不如在构造器中，记录一共创建了多少个实例，如下：

```java
public class CounterPoint extends Point {
       private static final AtomicInteger counter =
              new AtomicInteger();
       public CounterPoint(int x, int y) {
           super(x, y);
           counter.incrementAndGet();
       }
       public static int numberCreated() { 
         return counter.get(); 
       }
}
```

> The *Liskov substitution principle* says that any important property of a type should also hold for all its subtypes so that any method written for the type should work equally well on its subtypes [Liskov87]. This is the formal statement of our earlier claim that a subclass of Point (such as CounterPoint) is still a Point and must act as one. But suppose we pass a CounterPoint to the onUnitCircle method. If the Point class uses a getClass-based equals method, the onUnitCircle method will return false regardless of the CounterPoint instance’s *x* and *y* coordinates. This is so because most collections, including the HashSet used by the onUnitCircle method, use the equals method to test for containment, and no CounterPoint instance is equal to any Point. If, however, you use a proper instanceof-based equals method on Point, the same onUnitCircle method works fine when presented with a CounterPoint instance.

里氏替换原则（liskov substitution principle)认为，一个类的重要的属性，也将适用于它的子类型，因袭该类型中编写的方法就同样适用于其子类。里氏替换原则是我们前面写的内容（Point的子类（比如CounterPoint）仍然是一个Point类，并且拥有Point的功能）的一种比较正式的表达。假设我们给onUnitCircle方法传递一个CounterPoint对象，如果Point类使用的是基于getClass的equals方法，无论我们的CounterPoint对象的x和y的值是什么，onUnitCircle方法总是会返回false，这是因为大部分集合，包括onUnitCircle使用的HashSet在内，都使用equals方法还检测是否包含某对象，而任何的CounterPoint和Pont实例都不相等。但是，如果你在Point中使用基于instanceOf的合适的equals方法，这个onUnitCircle方法在传入一个CounterPoint对象的时候也能正常工作。

> While there is no satisfactory way to extend an instantiable class and add a value component, there is a fine workaround: Follow the advice of Item 18, “Favor composition over inheritance.” Instead of having ColorPoint extend Point, give ColorPoint a private Point field and a public *view* method (Item 6) that returns the point at the same position as this color point:

虽然面对继承可实例化类，并添加一个值组件时，没有令人满意的方法来重写equals，但是又一个不错的权宜之计：按照Item18的建议，”组合优先于继承“。给ColorPoint添加一个私有的Point域，设置一个公有的视图方法放回和这个colorPoint具有相同位置的Point对象，来代替继承Point对象。如下：

```java
// Adds a value component without violating the equals contract
   public class ColorPoint {
      private final Point point;
      private final Color color;
      public ColorPoint(int x, int y, Color color) {
         point = new Point(x, y);
         this.color = Objects.requireNonNull(color);
		}
      /**
       * Returns the point-view of this color point.
       */
		public Point asPoint() { 
			return point;
		}
      @Override public boolean equals(Object o) {
         if (!(o instanceof ColorPoint))
            return false;
         ColorPoint cp = (ColorPoint) o;
         return cp.point.equals(point) && cp.color.equals(color);
		}
      ...  // Remainder omitted
}
```

> There are some classes in the Java platform libraries that do extend an instantiable class and add a value component. For example, java.sql.Timestamp extends java.util.Date and adds a nanoseconds field. The equals implementation for Timestamp does violate symmetry and can cause erratic behavior if Timestamp and Date objects are used in the same collection or are otherwise intermixed. The Timestamp class has a disclaimer cautioning programmers against mixing dates and timestamps. While you won’t get into trouble as long as you keep them separate, there’s nothing to prevent you from mixing them, and the resulting errors can be hard to debug. This behavior of the Timestamp class was a mistake and should not be emulated.
>
> Note that you *can* add a value component to a subclass of an *abstract* class without violating the equals contract. This is important for the sort of class hierarchies that you get by following the advice in Item 23, “Prefer class hierarchies to tagged classes.” For example, you could have an abstract class Shape with no value components, a subclass Circle that adds a radius field, and a subclass Rectangle that adds length and width fields. Problems of the sort shown earlier won’t occur so long as it is impossible to create a superclass instance directly.

在java平台类库里，有一些类继承了可实例化的类还添加了值组件。比如java.sql.Timestamp继承了java.util.Date，还添加了一个nanoseconds 域。Timestamp的equals实现就确实违反了对称性，如果Timestamp对象和Date对象用在同一个集合里，或者以其他的方式混合到一起，就会出现错误的行为。Timestamp类有一个免责声明，告诫程序员不要将Date和Timestamp混用。只要你想他们分开，你就不要遇到任何问题。虽然没有任何东西阻碍你将他们混到一起用，但是出现的错误很难调试。Timestamp类的这种做法不错误的，不要去效仿。

当然，我们继承一个抽象类时，可以添加一个值组件，也不用违反equals的约定。对于根据Item23（”类层次优先于标签类）获得的类层次结构来说，很重要。比如，你可以有一个抽象类Shape，没有任何的值组件，然后有一个子类Circle，添加了一个radius域，另一个子类Rectangle，添加了length和width域。前面提到的问题都会应为无法直接创建父类的实例而不会出现。

> **Consistency**—The fourth requirement of the equals contract says that if two objects are equal, they must remain equal for all time unless one (or both) of them is modified. In other words, mutable objects can be equal to different objects at different times while immutable objects can’t. When you write a class, think hard about whether it should be immutable (Item 17). If you conclude that it should, make sure that your equals method enforces the restriction that equal objects remain equal and unequal objects remain unequal for all time.
>
> Whether or not a class is immutable, **do not write an** **equals** **method that depends on unreliable resources.** It’s extremely difficult to satisfy the consistency requirement if you violate this prohibition. For example, java.net.URL’s equals method relies on comparison of the IP addresses of the hosts associated with the URLs. Translating a host name to an IP address can require network access, and it isn’t guaranteed to yield the same results over time. This can cause the URL equals method to violate the equals contract and has caused problems in practice. The behavior of URL’s equals method was a big mistake and should not be emulated. Unfortunately, it cannot be changed due to compatibility requirements. To avoid this sort of problem, equals methods should perform only deterministic computations on memory-resident objects.

**一致性**—equals约定的第四个要求是说如果两个对象相等，那么只要这两个对象都没有被修改，它们就应该一致相等。话句话说就是，可变对象在不同的时候，可以和不同的对象相等，而不可变对象就不可以。当你在写一个类的时候，要好好思考这个类是不是该写成不可变的（Item17）。如果你确定这个类应该是不可变的，你就要确保你的equals方法满足这个限制：相等的对象们始终相等，不相等的对象们始终不相等。

不管这个类是不是不可变类，**都不要取决于不可靠的资源的equals方法**。如果你违背了这个禁令，就很难去满足一致性要求。比如java.net.URL里的equals方法依赖于URL里的主机的IP地址的比较，把主机名转换成一个IP地址需要网络服务，而且也不能保证不同时间会返回相同的结果，因为这个IP是会变的。这就使得URL的equals方法违反了equals约定，在实际应用中，也会有问题。所以URL的equals方法的行为也是个大大的错误，不要去模仿。遗憾地是，为了兼容性，这个方法不能修改。为了避免这种问题，equals方法应该根据驻留在内存里的对象进行确定性计算。

> **Non-nullity—**The final requirement lacks an official name, so I have taken the liberty of calling it “non-nullity.” It says that all objects must be unequal to null. While it is hard to imagine accidentally returning true in response to the invocation o.equals(null), it isn’t hard to imagine accidentally throwing a NullPointerException. The general contract prohibits this. Many classes have equals methods that guard against it with an explicit test for null:

**非空性**—最后一个要求没有官方的名字，作者就给它起了个名字叫“非空性（non-nullity）”。这个要求是说所有的对象和null都必须不相等，很难想象要怎么才能意外地在调用o.equals(null)的时候返回true，但是意外地抛出NullPointerException还是很好想象的。但是约定不允许抛异常，必须返回false。有很多类为了不抛出异常，为null的情况，使用了一个专门的测试如下：

```java
@Override public boolean equals(Object o) {
              if (o == null)
                  return false;
              ...
}
```

> This test is unnecessary. To test its argument for equality, the equals method must first cast its argument to an appropriate type so its accessors can be invoked or its fields accessed. Before doing the cast, the method must use the instanceof operator to check that its argument is of the correct type:

这种测试时没有必要的。因为equals方法在测试相等的时候，需要就是把这个参数转换为特定的类型，以便调用对象的访问方法或者访问它的域。在做类型转换之前，equals方法必须使用instanceof操作来检查这个参数的类型正不正确，如下：

```java
@Override public boolean equals(Object o) {
             if (!(o instanceof MyType))
                 return false;
             MyType mt = (MyType) o;
             ...
}
```

> If this type check were missing and the equals method were passed an argument of the wrong type, the equals method would throw a ClassCastException, which violates the equals contract. But the instanceof operator is specified to return false if its first operand is null, regardless of what type appears in the second operand [JLS, 15.20.2]. Therefore, the type check will return false if null is passed in, so you don’t need an explicit null check.

如果equals方法中没有这个类型检查的话，当传入一个错误的类型的时候，equals方法就会抛出ClassCastException，这也违反了equals约定。而当instanceof操作的第一个操作数是null的时候，不管第二个操作数是什么，都会直接返回false。因此当参数为null的时候，类型检查会直接返回false，就没有必要专门写个null检查。

> Putting it all together, here’s a recipe for a high-quality equals method:
>
> 1. **Use the** **==** **operator to check if the argument is a reference to this object.** If so, return true. This is just a performance optimization but one that is worth doing if the comparison is potentially expensive.
> 2. **Use the** **instanceof** **operator to check if the argument has the correct type.** If not, return false. Typically, the correct type is the class in which the method occurs. Occasionally, it is some interface implemented by this class. Use an interface if the class implements an interface that refines the equals contract to permit comparisons across classes that implement the interface. Collection interfaces such as Set, List, Map, and Map.Entry have this property.
>
> 3. **Cast the argument to the correct type.** Because this cast was preceded by an instanceof test, it is guaranteed to succeed.
> 4. **For each “significant” field in the class, check if that field of the argument matches the corresponding field of this object.** If all these tests succeed, return true; otherwise, return false. If the type in Step 2 is an interface, you must access the argument’s fields via interface methods; if the type is a class, you may be able to access the fields directly, depending on their accessibility.

把前面的那些需求都写到一起，可以得到一些编写高质量的equals方法的诀窍：

1. **使用==来检查参数是否是这个对象的引用**。如果是的话，就会返回true。如果比较的代价比较昂贵的话没使用这种方法就可以带来性能的优化。
2. **使用instanceof操作来检查参数的类型是否正确**。如果类型不正确的话，返回false。通常情况下，equals方法所在类的类型就是正确的类型。极个别的情况，这个类实现了几个接口，如果类实现的接口里的equals方法，允许在实现了该接口的类之间进行比较，就使用接口类型作为正确的类型。集合接口里的Set，List，Map，Map.Entry都有这一特性。
3. **将参数转化为正确的类型**。因为在转换前进行了instanceof检查，所以这个操作可以确保成功。
4. **对于类里的重要域，检查参数的该域和该对象对应的域是否相等**。如果相等的话，返回true；否则，返回false。如果第二步里的类型是接口，就必须使用接口方法访问参数的域；如果类型是类的话，有可能可以直接访问这些域，这取决于他们的可访问性。

> For primitive fields whose type is not float or double, use the == operator for comparisons; for object reference fields, call the equals method recursively; for float fields, use the static Float.compare(float, float) method; and for double fields, use Double.compare(double, double). The special treatment of float and double fields is made necessary by the existence of Float.NaN, -0.0f and the analogous double values; see JLS 15.21.1 or the documentation of Float.equals for details. While you could compare float and double fields with the static methods Float.equals and Double.equals, this would entail autoboxing on every comparison, which would have poor performance. For array fields, apply these guidelines to each element. If every element in an array field is significant, use one of the Arrays.equals methods.

对于那些不是float也不是double的原始类型域，使用==来比较；对于对象引用域，递归地调用equals方法；对于float域，使用Float.compare(float,float)方法进行比较；对于double域，使用Double.compare(double,double)。float之所以必须要特殊对待，是因为存在Float.NaN, -0.0f，double也一样。详情可以参看Float.equals的文档，或者JLS 15.21.1 。虽然可以使用静态方法Float.equals 和Double.equals来比较float和double域，但是这会导致每次比较的时候都需要进行自动装箱，性能很差。对于数组类型的域，可以将这些指导原则应用到每个元素上。如果数组中的每个元素都非常的重要的话，可以直接使用Arrays.equals方法。

> Some object reference fields may legitimately contain null. To avoid the possibility of a NullPointerException, check such fields for equality using the static method Objects.equals(Object, Object).

一些对象的引用域为null可能是合法的。为了避免抛出NullPointException，可以使用静态方法Objects.equals(Object,Object)方法来比较这种域。

> For some classes, such as CaseInsensitiveString above, field comparisons are more complex than simple equality tests. If this is the case, you may want to store a *canonical form* of the field so the equals method can do a cheap exact comparison on canonical forms rather than a more costly nonstandard comparison. This technique is most appropriate for immutable classes (Item 17); if the object can change, you must keep the canonical form up to date.

对于一些类，比如前面提到的CaseInsensitiveString，它的域的比较就很复杂，不是简单相等的检查。在这种情况下，可以保存一个这个域的“范式”，然后在equals方法中就可以直接精确地比较这个范式，开销比较低，而不是进行开销高、不精确的比较。这种技术最适合不可变类了（Item10），因为如果这个类可变的话，它的范式也必须跟着变。

> The performance of the equals method may be affected by the order in which fields are compared. For best performance, you should first compare fields that are more likely to differ, less expensive to compare, or, ideally, both. You must not compare fields that are not part of an object’s logical state, such as lock fields used to synchronize operations. You need not compare *derived fields*, which can be calculated from “significant fields,” but doing so may improve the performance of the equals method. If a derived field amounts to a summary description of the entire object, comparing this field will save you the expense of comparing the actual data if the comparison fails. For example, suppose you have a Polygon class, and you cache the area. If two polygons have unequal areas, you needn’t bother comparing their edges and vertices.

对象域的比较顺序会影响equals方法的性能。为了性能最优，应该先比较那些最容易不同的、比较代价小的、或者两者都占的域。千万不要去比较那些不是这个类的逻辑状态的域，比如用来同步计算的锁域。也没必要去比较那些“衍生域（derived fields）”，它们会在计算重要域（significant fields）的时候被计算，但是如果比较这些域可以带来性能提升的时候例外。假如一个衍生域表示的是整个类的总体描述，在比较结果是false的时候，相对于比较原始数据，比较这个衍生域可以减少开销。比如，现在有一个Polygon类，然后你缓存了它的面积area（*这个面积就是衍生域*）。当两个polygon实例的面积area不同的时候就不需要去比较它的边和顶点了。

> **When you are finished writing your** **equals** **method, ask yourself three questions: Is it symmetric? Is it transitive? Is it consistent?** And don’t just ask yourself; write unit tests to check, unless you used AutoValue (page 49) to generate your equals method, in which case you can safely omit the tests. If the properties fail to hold, figure out why, and modify the equals method accordingly. Of course your equals method must also satisfy the other two properties (reflexivity and non-nullity), but these two usually take care of themselves.
>
> An equals method constructed according to the previous recipe is shown in this simplistic PhoneNumber class:

**在编写完equals方法后，问自己三个问题：它满足对称性吗？满足传递性吗？满足一致性吗？**除了问自己以外，还需要写单元测试来检查一下，除非你使用AutoValue（page49）来生成equals方法，这种情况下可以放心地省略测试。当你做这些测试失败后，需要找出来为什么错，然后据此修改equals方法。当然我们的equals方法也需要满足自反性和非空性，但是这两个特性通常自己就满足。

通过前面那些诀窍写的一个简单的PhoneNumber类的equals方法如下：

```java
// Class with a typical equals method
   public final class PhoneNumber {
       private final short areaCode, prefix, lineNum;
       public PhoneNumber(int areaCode, int prefix, int lineNum) {
           this.areaCode = rangeCheck(areaCode, 999, "area code");
           this.prefix   = rangeCheck(prefix,   999, "prefix");
           this.lineNum  = rangeCheck(lineNum, 9999, "line num");
			 }
			 private static short rangeCheck(int val, int max, String arg) {
         	 if (val < 0 || val > max)
								throw new IllegalArgumentException(arg + ": " + val); return (short) val;
				}
		 @Override 
     public boolean equals(Object o) { 
       		 if (o == this)
               return true;
           if (!(o instanceof PhoneNumber))
               return false;
           PhoneNumber pn = (PhoneNumber)o;
           return pn.lineNum == lineNum && pn.prefix == prefix
                   && pn.areaCode == areaCode;
					}
       ... // Remainder omitted
   }

```

> Here are a few final caveats:
>
> • **Always override hashCode when you override equals**(Item11).
>
> • **Don’t try to be too clever.** If you simply test fields for equality, it’s not hard to adhere to the equals contract. If you are overly aggressive in searching for equivalence, it’s easy to get into trouble. It is generally a bad idea to take any form of aliasing into account. For example, the File class shouldn’t attempt to equate symbolic links referring to the same file. Thankfully, it doesn’t.
>
> • **Don’t substitute another type for Object in the equals declaration.**It is not uncommon for a programmer to write an equals method that looks like this and then spend hours puzzling over why it doesn’t work properly:

这里还有一些最后的忠告：

- **在覆盖equals方法的时候总要覆盖hashcode方法（Item1）**。
- **不要让equals方法太过智能**。如果你只是简单地检测各个域是否相等，遵守equals的约定一点也不难。但当你想过度地去寻求各种等价关系，就很容易出问题。通常情况下，把任何一种别名形式放在equals考虑的范围内都不是一个好主意，比如File类就不应该把指向同一个文件的符号链接（symbolic links）拿来比较，幸运的是，File没有这样做。
- **在equals方法里不要使用其他的类型替代Object。**程序员写一个下面这样的equals方法很常见，花好几个小时都不知道为啥有问题。

```java
// Broken - parameter type must be Object!
     public boolean equals(MyClass o) {
         ...
}
```

> The problem is that this method does not *override* Object.equals, whose argument is of type Object, but *overloads* it instead (Item 52). It is unacceptable to provide such a “strongly typed” equals method even in addition to the normal one, because it can cause Override annotations in subclasses to generate false positives and provide a false sense of security.
>
> Consistent use of the Override annotation, as illustrated throughout this item, will prevent you from making this mistake (Item 40). This equals method won’t compile, and the error message will tell you exactly what is wrong:

这个问题就是这个方法并没有覆盖（override)参数类型为Object的Object.equals方法，而是重载（overload）（Item52)了。提供这样一个具有“强类型”的equals方法作为原始的方法的补充，是无法接受的，因为当子类使用override注解时，即使用错了也能正常编译，带来一种错误的安全感。

正如本节中所有的例子一样，坚持使用override注解，可以防止出现这种错误（Item40）。下面这个equals方法不会编译，并且错误信息也会告诉你到底哪里错了：

```java
// Still broken, but won’t compile
@Override public boolean equals(MyClass o) {
... }
```

> Writing and testing equals (and hashCode) methods is tedious, and the resulting code is mundane. An excellent alternative to writing and testing these methods manually is to use Google’s open source AutoValue framework, which automatically generates these methods for you, triggered by a single annotation on the class . In most cases, the methods generated by AutoValue are essentially identical to those you’d write yourself.
>
> IDEs, too, have facilities to generate equals and hashCode methods, but the resulting source code is more verbose and less readable than code that uses AutoValue, does not track changes in the class automatically, and therefore requires testing. That said, having IDEs generate equals (and hashCode) methods is generally preferable to implementing them manually because IDEs do not make careless mistakes, and humans do.

编写和测试equals方法和hashCode方法是一件很单调乏味的事情，得到的代码也很乏味。代替编写测试这些方法的最好的方法是使用Google的开源框架AutoValue。只需要再类上添加一个简单的注释，AutoValue就可以自动的为你生成这些方法。在大部分情况下，AutoValue生成的方法和你自己写的基本一致。

一些IDE也有生成equals和hashCode方法的功能，但是它们生成的代码相对于AutoValue而言，比较啰嗦，可读性也不那么高，也不会根据类的修改自动修改，因此还是需要测试。即使如此，使用IDE生成的equals（和hashCode）方法也比手动写的要好一些，因为IDE不会犯一些不小心的错误，但是人会。

> In summary, don’t override the equals method unless you have to: in many cases, the implementation inherited from Object does exactly what you want. If you do override equals, make sure to compare all of the class’s significant fields and to compare them in a manner that preserves all five provisions of the equals contract.

总结一下，若非必要不要覆盖equals方法，因为很多情况下，从Object继承到的equals的实现就刚好是你想要的。如果确实需要覆盖equals方法，确保比较类的所有的重要的域，并且在比较的时候要注意不要违背了equals给的五条约定。

### 