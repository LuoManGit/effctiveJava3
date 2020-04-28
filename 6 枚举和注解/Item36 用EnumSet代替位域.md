### Item36 用EnumSet代替位域	

> If the elements of an enumerated type are used primarily in sets, it is traditional to use the int enum pattern (Item 34), assigning a different power of 2 to each constant:

如果一个枚举类型的元素主要是用在集合中，一般会用int枚举模式（Item34)，比如将2的不同指数赋值给每一个常量，代码如下：

```
// Bit field enumeration constants - OBSOLETE!
   public class Text {
       public static final int STYLE_BOLD          = 1 << 0;  //1
       public static final int STYLE_ITALIC        = 1 << 1;  //2
       public static final int STYLE_UNDERLINE     = 1 << 2;  //4
       public static final int STYLE_STRIKETHROUGH = 1 << 3;  // 8
       // Parameter is bitwise OR of zero or more STYLE_ constants
       public void applyStyles(int styles) { ... }
   }
```

> This representation lets you use the bitwise OR operation to combine several constants into a set, known as a *bit field*:

这种表示方法让你可以使用OR操作把几个常量合并到一个集合中，被称为位域：

```java
text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
```

> The bit field representation also lets you perform set operations such as union and intersection efficiently using bitwise arithmetic. But bit fields have all the disadvantages of int enum constants and more. It is even harder to interpret a bit field than a simple int enum constant when it is printed as a number. There is no easy way to iterate over all of the elements represented by a bit field. Finally, you have to predict the maximum number of bits you’ll ever need at the time you’re writing the API and choose a type for the bit field (typically int or long) accordingly. Once you’ve picked a type, you can’t exceed its width (32 or 64 bits) without changing the API.

这种位域表示法，可以通过位操作，使得比如并集和交集这样的集合操作更加高效。但是这种位域表示有int枚举常量的所有的缺点，甚至更多。当他们作为一个数字打印出来的时候，位域比int枚举常量还要难以解释。也没有简单的方法可以遍历位域表示的所有元素。最后，你在写API的时候，还必须要预测你可能遇到的最大的值的位数，同时还要给位域选择对于的数据类型（对应int或者long）。一旦你选定了类型，在不修改api的前提下，就无法超出它的宽度（32或者64位）。

> Some programmers who use enums in preference to int constants still cling to the use of bit fields when they need to pass around sets of constants. There is no reason to do this, because a better alternative exists. The java.util package provides the EnumSet class to efficiently represent sets of values drawn from a single enum type. This class implements the Set interface, providing all of the richness, type safety, and interoperability you get with any other Set implementation. But internally, each EnumSet is represented as a bit vector. If the underlying enum type has sixty-four or fewer elements—and most do—the entire EnumSet is represented with a single long, so its performance is comparable to that of a bit field. Bulk operations, such as removeAll and retainAll, are implemented using bitwise arithmetic, just as you’d do manually for bit fields. But you are insulated from the ugliness and error-proneness of manual bit twiddling: the EnumSet does the hard work for you.

一些程序员虽然更倾向于使用枚举而不是int常量，但是当需要传递多组常量集的时候，还是会倾向于使用位域。其实没有理由这么去做，因为还有更好的选择，java,util包里提供了EnumSet类来高效的表示从单个枚举类型中取出的一组值的集合。这个类实现了Set接口，提供了丰富的功能、类型安全、以及可以和任何其他Set实现的互用性。但是在内部是线上，每一个EnumSet都是一个位向量，如果底层的枚举类型只有小于等于64个元素（大部分情况都是这样的），整个EnumSet就会用一个单个的long来表示，因此其性能是可以和位域媲美的。大部分操作，比如removeAll和retainAll，都是基于位运算来实现的，就像你手工操作位域一样。EnumSet为你做了很多艰巨的工作，让你可以远离手动进行位域操作带来的丑陋的代码和容易出现的错误。

> Here is how the previous example looks when modified to use enums and enum sets instead of bit fields. It is shorter, clearer, and safer:

下面是前面的例子使用Enum和EnumSer表示的代码。它更短，更简洁，更安全：

```java
// EnumSet - a modern replacement for bit fields
public class Text {
public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }
       // Any Set could be passed in, but EnumSet is clearly best
       public void applyStyles(Set<Style> styles) { ... }
}
```

> Here is client code that passes an EnumSet instance to the applyStyles method. The EnumSet class provides a rich set of static factories for easy set creation, one of which is illustrated in this code:

下面是将EnumSet实例传递给applyStyles方法的客户端代码。EnumSet类提供了很多的静态工厂方法来简化集合的创建，下面代码中展示的便是其中一个：

```jva
text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```

> Note that the applyStyles method takes a Set<Style> rather than an EnumSet<Style>. While it seems likely that all clients would pass an EnumSet to the method, it is generally good practice to accept the interface type rather than the implementation type (Item 64). This allows for the possibility of an unusual client to pass in some other Set implementation.

需要注意的是applyStyles方法的声明中的参数类型是Set<Style>而不是EnumSet<Style>。虽然好像所有的客户端都会传递EnumSet给这个方法，但是通常来说， 在实际中，接受接口类型还是比实现类型要好一些（Item64）。这也允许那些不同寻常的客户端会传递一些其他的Set实现。

> In summary, **just because an enumerated type will be used in sets, there is no reason to represent it with bit fields.** The EnumSet class combines the conciseness and performance of bit fields with all the many advantages of enum types described in Item 34. The one real disadvantage of EnumSet is that it is not, as of Java 9, possible to create an immutable EnumSet, but this will likely be remedied in an upcoming release. In the meantime, you can wrap an EnumSet with Collections.unmodifiableSet, but conciseness and performance will suffer.

总结一下，**如果仅仅因为枚举类型需要被用在集合中，完全没有理解使用位域。**EnumSet类集简洁、位域的高性能、和Item34里介绍的枚举的很多优点于一身。这个EnumSet的一个缺点是，截止到Java9，都不可能创建一个不可变的EnumSet，但是这个问题可能会在后面的版本中修复。同时你也可以使用Collections.unmodifiableSet来包装EnumSet，但是简洁性和高性能可能会受到影响。