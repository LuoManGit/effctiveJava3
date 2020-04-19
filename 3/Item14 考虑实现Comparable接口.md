### Item14 考虑实现Comparable接口

> Unlike the other methods discussed in this chapter, the compareTo method is not declared in Object. Rather, it is the sole method in the Comparable interface. It is similar in character to Object’s equals method, except that it permits order comparisons in addition to simple equality comparisons, and it is generic. By implementing Comparable, a class indicates that its instances have a *natural ordering.* Sorting an array of objects that implement Comparable is as simple as this:

和本章中讨论的其他方法不一样，compareTo方法没有在Object里声明。compareTo是Comparable接口中唯一的方法，conpareTo方法不仅可以进行简单的相等比较，还可以进行顺序比较，并且是泛型的，除这些以外，它和Object的equals方法有类似的特征。一个类实现了Comparable就意味着它的实例们有一个自然的排序关系。将一个实现了Comparable的对象的数组进行排序，就像下面这么简单：

```java
Arrays.sort(a);
```

> It is similarly easy to search, compute extreme values, and maintain automatically sorted collections of Comparable objects. For example, the following program, which relies on the fact that String implements Comparable, prints an alphabetized list of its command-line arguments with duplicates eliminated:

同样地，对于实现了Comparable的对象的集合，进行查询、计算极值、自动维持排序都是一个很容易的事情。比如，下面的程序，依赖String实现了Comparable这一事实，打印了去除重复值的命令参数的按字母顺序排序。

```java
public class WordList {
       public static void main(String[] args) {
           Set<String> s = new TreeSet<>();
           Collections.addAll(s, args);
           System.out.println(s);
		 	} 
}
```

By implementing Comparable, you allow your class to interoperate with all of the many generic algorithms and collection implementations that depend on this interface. You gain a tremendous amount of power for a small amount of effort. Virtually all of the value classes in the Java platform libraries, as well as all enum types (Item 34), implement Comparable. If you are writing a value class with an obvious natural ordering, such as alphabetical order, numerical order, or chronological order, you should implement the Comparable interface:

一旦你的类实现了Comparable接口，你的类就可以和所有依赖这个接口的泛型算法以及集合实现进行协作了。只需要付出一点点努力，就可以收获非常强大的功能。实际上，在Java平台类库里，所有的值类以及Enum类（Item34）都实现了Comparable。如果你要写一个有明显的自然顺序的值类，比如按字母顺序、数字顺序或者时间顺序，那你就应该实现Comparable接口。Comparable接口如下：

```java
public interface Comparable<T> {
       int compareTo(T t);
}
```

> The general contract of the compareTo method is similar to that of equals:
>
> Compares this object with the specified object for order. Returns a negative integer, zero, or a positive integer as this object is less than, equal to, or greater than the specified object. Throws ClassCastException if the specified object’s type prevents it from being compared to this object.

compareTo方法的通用约定和equals方法类似：

将这个对象和指定对象进行排序比较。返回一个负数、0、正数分别代表这个对象小于、等于、大于指定对象。当指定对象的类型不能用来和这个对象比较时，抛出ClassCastException。

> In the following description, the notation sgn(*expression*) designates the mathematical *signum* function, which is defined to return -1, 0, or 1, according to whether the value of *expression* is negative, zero, or positive.
>
> - The implementor must ensure that sgn(x.compareTo(y))==-sgn(y. compareTo(x)) for all x and y. (This implies that x.compareTo(y) must throw an exception if and only if y.compareTo(x) throws an exception.)
> - The implementor must also ensure that the relation is transitive: (x. compareTo(y) > 0 && y.compareTo(z) > 0) implies x.compareTo(z) > 0.
> - Finally,the implementor must ensure that x.compareTo(y)==0 implies that sgn(x.compareTo(z)) == sgn(y.compareTo(z)), for all z.
> - It is strongly recommended, but not required, that (x.compareTo(y) == 0) == (x.equals(y)). Generally speaking, any class that implements the Comparable interface and violates this condition should clearly indicate this fact. The recommended language is “Note: This class has a natural ordering that is inconsistent with equals.”

在下面的描述中，符号sgn(expression)表示数学函数signum，signum函数当expression值为负数、0、正数的时候分别返回-1、0、1。

- 实现者必须保证，对于所有的x，y满足sgn(x.compareTo(y))==-sgn(y. compareTo(x)) 。（这也就意味着当且仅当y.compareTo(x)抛出异常的时候，x.compareTo(y)也必须抛出异常）。
- 实现者还必须保证这种关系具有可传递性，即若(x. compareTo(y) > 0 && y.compareTo(z) > 0) 则x.compareTo(z) > 0。
- 最后，实现着还必须保证，若x.compareTo(y)==0，则对于所有z，有sgn(x.compareTo(z)) == sgn(y.compareTo(z))。
- 还有一点强烈推荐，但是不强求：(x.compareTo(y) == 0) == (x.equals(y))。一般来说，如果一个类实现了Comparable接口，但是违反了这个条件的话，就应该做明确的声明。推荐使用这样的话术：“注意，该类的内在的排序功能和equals方法不一致。”

> Don’t be put off by the mathematical nature of this contract. Like the equals contract (Item 10), this contract isn’t as complicated as it looks. Unlike the equals method, which imposes a global equivalence relation on all objects, compareTo doesn’t have to work across objects of different types: when confronted with objects of different types, compareTo is permitted to throw ClassCastException. Usually, that is exactly what it does. The contract does *permit* intertype comparisons, which are typically defined in an interface implemented by the objects being compared.

不要被约定里的数学关系迷惑了。和equals约定（Item10）一样，这个约定也没有看起来那么复杂。和equals方法不一样的是，equals方法是对所有的对象强制进行的全局对等比较，而comparaTo没有必要进行不同类型之间的比较。当面对不同类型的对象的时候，compaeTo方法可以直接抛出ClassCastException。通常情况下，也是这么做的。但是约定还是允许跨类型比较的，这一般是在被比较的对象中实现的接口中进行定义的。

> Just as a class that violates the hashCode contract can break other classes that depend on hashing, a class that violates the compareTo contract can break other classes that depend on comparison. Classes that depend on comparison include the sorted collections TreeSet and TreeMap and the utility classes Collections and Arrays, which contain searching and sorting algorithms.

就像违反了hashCode约定会破坏其他依赖hash值的类一样，违反了compareTo约定的类也会破坏那些依赖比较的类。依赖比较的类包括有序集合TreeSet、TreeMap，以及包含查询、排序算法的工具类Collections和Arrays。

> Let’s go over the provisions of the compareTo contract. The first provision says that if you reverse the direction of a comparison between two object references, the expected thing happens: if the first object is less than the second, then the second must be greater than the first; if the first object is equal to the second, then the second must be equal to the first; and if the first object is greater than the second, then the second must be less than the first. The second provision says that if one object is greater than a second and the second is greater than a third, then the first must be greater than the third. The final provision says that all objects that compare as equal must yield the same results when compared to any other object.

让我们来回顾一下compareTo约定的条款。第一条说如果换一下两个对象引用的比较方向，期望发生下面这些事情：如果第一个对象小于第二个对象，则第二个对象就一定要大于第一个对象；若第一个对象等于第二个对象，则第二个对象就一定也要等于第一个对象；若第一个对象大于第二个对象，则第二个对象就一定要小于第一个对象。第二条是说，如果第一个对象大于第二个对象，第二个对象大于第三个对象，则对一个对象必须要大于第三个对象。最后一个约定是说所有比较结果是相等的对象，在和其他对象比较时，结果应该一致。

> One consequence of these three provisions is that the equality test imposed by a compareTo method must obey the same restrictions imposed by the equals contract: reflexivity, symmetry, and transitivity. Therefore, the same caveat applies: there is no way to extend an instantiable class with a new value component while preserving the compareTo contract, unless you are willing to forgo the benefits of object-oriented abstraction (Item 10). The same workaround applies, too. If you want to add a value component to a class that implements Comparable, don’t extend it; write an unrelated class containing an instance of the first class. Then provide a “view” method that returns the contained instance. This frees you to implement whatever compareTo method you like on the containing class, while allowing its client to view an instance of the containing class as an instance of the contained class when needed.

这三个条款带来的一个结果是通过compareTo方法进行的相等性测试，也必须遵守和equals约定相同的限制：自反性、对称性、传递性。因此，这个警告也同样适用：无法在继承一个可实例化的类并添加一个新值组件的同时，不违反compareTo约定，除非你愿意放弃一些面向对象的抽象带来的好处（Item10）。这个变通方法也同样适用，如果你想往一个实现了Comparable的类里添加一个值，不要继承这个类，直接写一个不相关的类包含第一个类的一个实例。然后写一个view方法来返回这个被包含的实例。这就使得你可以自由地在第二个类上实现你想要的compareTo方法，同时也允许客户端在需要的时候，将第二个类实例看做是第一个类的实例。

> The final paragraph of the compareTo contract, which is a strong suggestion rather than a true requirement, simply states that the equality test imposed by the compareTo method should generally return the same results as the equals method. If this provision is obeyed, the ordering imposed by the compareTo method is said to be *consistent with* equals. If it’s violated, the ordering is said to be *inconsistent with* equals. A class whose compareTo method imposes an order that is inconsistent with equals will still work, but sorted collections containing elements of the class may not obey the general contract of the appropriate collection interfaces (Collection, Set, or Map). This is because the general contracts for these interfaces are defined in terms of the equals method, but sorted collections use the equality test imposed by compareTo in place of equals. It is not a catastrophe if this happens, but it’s something to be aware of.

compareTo的最后一个自然段是一条强烈的建议而不是一条要求，简单地说明了compareTo进行的相等性测试的结果通常来说应该和equals方法的结果一致。如果遵守了这条建议，那么通过compareTo获得的排序就被认为和equals一致；如果违反了，就认为排序和equals不一致。排序和equals不一致的类也能工作，但是包含该类实例的排序集合就可能违反相应集合（Collection,Set,Map)的通用约定。因为这些约定的通用约定是根据equals方法来定义的，但是排序集合使用compareTo代替equals进行相等性测试。虽然就算这种情况出现也不会造成很大的灾难，但是我们还是应该有所了解。

> For example, consider the BigDecimal class, whose compareTo method is inconsistent with equals. If you create an empty HashSet instance and then add new BigDecimal("1.0") and new BigDecimal("1.00"), the set will contain two elements because the two BigDecimal instances added to the set are unequal when compared using the equals method. If, however, you perform the same procedure using a TreeSet instead of a HashSet, the set will contain only one element because the two BigDecimal instances are equal when compared using the compareTo method. (See the BigDecimal documentation for details.)

比如，BigDecimal类的compareTo方法就和equals方法不一致。如果你创建一个空的HashSet实例，然后添加new BigDecimal("1.0") 和new BigDecimal("1.00")。hashSet中将包含两个实例，因为添加到set中的两个BigDecimal实例在使用equals方法进行比较的时候，是不相等的。但是，当你使用TreeSet代替HashSet执行相同的操作的时候，treeSet中将只包含一个值，因为当这两个BigDecimal实例在使用compareTo方法进行比较的时候，是相等的。（详情参见BIgDecimal文档）

> Writing a compareTo method is similar to writing an equals method, but there are a few key differences. Because the Comparable interface is parameterized, the compareTo method is statically typed, so you don’t need to type check or cast its argument. If the argument is of the wrong type, the invocation won’t even compile. If the argument is null, the invocation should throw a NullPointer- Exception, and it will, as soon as the method attempts to access its members.

编写compareTo方法和编写equals方法类似，但是有一些关键点不同，由于Comparable接口是参数化的，compareTo方法的类型是静态的，因此不必进行类型检查或者转换参数类型。如果参数的类型有问题的话，编译都无法通过。如果参数是null的话，在compareTo方法试图访问其成员的时候就会抛出NullPointerException。

> In a compareTo method, fields are compared for order rather than equality. To compare object reference fields, invoke the compareTo method recursively. If a field does not implement Comparable or you need a nonstandard ordering, use a Comparator instead. You can write your own comparator or use an existing one, as in this compareTo method for CaseInsensitiveString in Item 10:

compareTo方法中对域的比较是为了进行顺序的比较，而不是相等性的比较。在比较对象引用域的时候，可以递归地调用compareTo方法。如果这个域没有实现Comparable接口或者你需要一个不标准的顺序，可以使用Comparator替换。你可以写一个自己的comparator或者使用一个已经存在的。例如Item10中的CaseInsensitiveString的compareTo方法如下：

```java
// Single-field Comparable with object reference field
   public final class CaseInsensitiveString
           implements Comparable<CaseInsensitiveString> {
       public int compareTo(CaseInsensitiveString cis) {
           return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
			 }
       ... // Remainder omitted
   }
```

> Note that CaseInsensitiveString implements Comparable<CaseInsensitiveString>. This means that a CaseInsensitiveString reference can be compared only to another CaseInsensitiveString reference. This is the normal pattern to follow when declaring a class to implement Comparable.

需要注意的是，CaseInsensitiveString实现了Comparable<CaseInsensitiveString>。这就意味着一个CaseInsensitiveString引用只能和另一个CaseInsensitiveString引用进行比较。这是一个类声明实现Comparable的常用的方法。

> Prior editions of this book recommended that compareTo methods compare integral primitive fields using the relational operators < and >, and floating point primitive fields using the static methods Double.compare and Float.compare. In Java 7, static compare methods were added to all of Java’s boxed primitive classes. **Use of the relational operators** **<** **and** **>** **in** **compareTo** **methods is verbose and error-prone and no longer recommended.**

在本书的前两个版本中，在compareTo方法里，对于整数基本数据类型的比较推荐使用相关操作符>和<，小数的基本数据类型使用静态方法Double.compare 和 Float.compare。在java7中，在所有的java的基本类型的封装类中都添加了静态比较方法。**在compareTo方法中，使用相关操作符>和<很啰嗦，也容易出错，因此不再推荐了**。

> If a class has multiple significant fields, the order in which you compare them is critical. Start with the most significant field and work your way down. If a comparison results in anything other than zero (which represents equality), you’re done; just return the result. If the most significant field is equal, compare the next- most-significant field, and so on, until you find an unequal field or compare the least significant field. Here is a compareTo method for the PhoneNumber class in Item 11 demonstrating this technique:

如果一个类有多个重要的域，比较这些域的顺序就很重要了。应该从最关键的域开始比较，如果某个域的比较结果是非0 (0表示相等)，那就已经完成比较了，可以直接返回结果。如果最重要的域是相等的，就比较下一个最重要的域，以此类推，知道你找到了一个不相等的域，或者比较到了最后一个域。下面是Item11中的PhoneNumber类的compareTo方法，说明了这个方法的具体实现：

```java
// Multiple-field Comparable with primitive fields
   public int compareTo(PhoneNumber pn) {
       int result = Short.compare(areaCode, pn.areaCode);
       if (result == 0)  {
           result = Short.compare(prefix, pn.prefix);
           if (result == 0)
               result = Short.compare(lineNum, pn.lineNum);
			 }
       return result;
   }
```

> In Java 8, the Comparator interface was outfitted with a set of *comparator construction methods*, which enable fluent construction of comparators. These comparators can then be used to implement a compareTo method, as required by the Comparable interface. Many programmers prefer the conciseness of this approach, though it does come at a modest performance cost: sorting arrays of PhoneNumber instances is about 10% slower on my machine. When using this approach, consider using Java’s *static import* facility so you can refer to static comparator construction methods by their simple names for clarity and brevity. Here’s how the compareTo method for PhoneNumber looks using this approach:

在java8里，Comparator接口中提供了全套的比较构造器方法，使得比较器的构造工作变得非常流畅。这些比较器会被用来实现compareTo方法。很多程序员都喜欢这种方法带来的简洁，但这种方法也确实带来了一些性能损耗：在作者的机器上，给PhoneNumber数组排序的速度慢了10%。当使用这种方式的时候，为了让代码就会变得简洁清晰，可以考虑使用java的静态导入技术，然后你就可以通过其简单的名字来调用静态比较器构造方法。下面是PhoneNumber类的compareTo方法使用这种方式的实现代码：

```java
// Comparable with comparator construction methods
   private static final Comparator<PhoneNumber> COMPARATOR =
           comparingInt((PhoneNumber pn) -> pn.areaCode)
             .thenComparingInt(pn -> pn.prefix)
             .thenComparingInt(pn -> pn.lineNum);
   public int compareTo(PhoneNumber pn) {
       return COMPARATOR.compare(this, pn);
}
```

> This implementation builds a comparator at class initialization time, using two comparator construction methods. The first is comparingInt. It is a static method that takes a *key extractor function* that maps an object reference to a key of type int and returns a comparator that orders instances according to that key. In the previous example, comparingInt takes a *lambda* () that extracts the area code from a PhoneNumber and returns a Comparator<PhoneNumber> that orders phone numbers according to their area codes. Note that the lambda explicitly specifies the type of its input parameter (PhoneNumber pn). It turns out that in this situation, Java’s type inference isn’t powerful enough to figure the type out for itself, so we’re forced to help it in order to make the program compile.

在这种实现中，在类进行初始化的时候，就通过调用两个比较构造器方法创建了比较器Comparator。第一个方法是静态方法comparingInt，其参数为键提取函数，将对象引用映射到类型为int的键上，返回一个根据这个键进行实例顺序比较的比较器。在前面的例子中，comparingInt使用一个lambda表达式从PhoneNumber中提取areaCode，然后返回一个根据areaCode对PhoneNumber实例进行顺序比较的比较器。需要注意的是，这里的lambda表达式明确指定了输入参数的类型(PhoneNumber pn)。事实证明，在这种情况下，java的类型推导还不够强大到可以找到自己的类型，因此我们必须要直接指定，否则程序无法通过编译。

> If two phone numbers have the same area code, we need to further refine the comparison, and that’s exactly what the second comparator construction method, thenComparingInt, does. It is an instance method on Comparator that takes an int key extractor function, and returns a comparator that first applies the original comparator and then uses the extracted key to break ties. You can stack up as many calls to thenComparingInt as you like, resulting in a *lexicographic ordering*. In the example above, we stack up two calls to thenComparingInt, resulting in an ordering whose secondary key is the prefix and whose tertiary key is the line number. Note that we did *not* have to specify the parameter type of the key extractor function passed to either of the calls to thenComparingInt: Java’s type inference was smart enough to figure this one out for itself.

如果两个phonNumber实例拥有相同的areaCode，我们就需要进行进一步的比较了，这正是第二个比较构造器thenComparingInt所做的事情。这是一个Comparator的实例方法，其参数为一个int键提取器，返回一个比较器，该比较器首先应用原始的比较器，然后使用第二个提取出来的键进行比较。你可以叠加任意多次的thenComparingInt调用，并按照字典序排序。在前面的例子中，我们调用了两次thenComparingInt方法，因此按照第二个键为prefix，第三个键为lineNum进行排序。需要注意的是，在thenComparingInt的两次调用中，我们都不需要为键提取函数指定参数类型，java的类型推导自己推导出类型了。

> The Comparator class has a full complement of construction methods. There are analogues to comparingInt and thenComparingInt for the primitive types long and double. The int versions can also be used for narrower integral types, such as short, as in our PhoneNumber example. The double versions can also be used for float. This provides coverage of all of Java’s numerical primitive types.
>
> There are also comparator construction methods for object reference types. The static method, named comparing, has two overloadings. One takes a key extractor and uses the keys’ natural order. The second takes both a key extractor and a comparator to be used on the extracted keys. There are three overloadings of the instance method, which is named thenComparing. One overloading takes only a comparator and uses it to provide a secondary order. A second overloading takes only a key extractor and uses the key’s natural order as a secondary order. The final overloading takes both a key extractor and a comparator to be used on the extracted keys.

Comparator有全套的构造器方法。针对long和double，也有类似comparingInt和thenComparingInt这样的构造器方法。int版本的也可以用在更小的整数类型上，比如PhoneNumber中的short。double版本同样也可以用在float上。也就是提供的方法覆盖了Java的所有数字型基本类型。

对于对象引用类型，也有比较构造方法。其静态方法名为comparing有两个重载：第一个只有一个键提取器参数，使用键的自然顺序进行比较；第二个有两个参数，一个建提取器和一个作用在这个提取键上的比较器。其实例方法名为thenComparing，有三个重载：第一个只有一个比较器参数，使用它进行第二级排序比较；第二个只有一个键提取器参数，然后使用这个键的自然顺序进行比较；最后一个有两个参数，一个键提取器和一个作用在这个键上的比较器。

> Occasionally you may see compareTo or compare methods that rely on the fact that the difference between two values is negative if the first value is less than the second, zero if the two values are equal, and positive if the first value is greater. Here is an example:

compareTo和compare方法有时候也会依赖两个数据的区别，如果第一个数比第二个数小，则为负值，若第一个数和第二个数相等，则为0，若第一个数比第二个数大，则为正值。如下：

```java
// BROKEN difference-based comparator - violates transitivity!
   static Comparator<Object> hashCodeOrder = new Comparator<>() {
       public int compare(Object o1, Object o2) {
           return o1.hashCode() - o2.hashCode();
       }
};
```

> Do not use this technique. It is fraught with danger from integer overflow and IEEE 754 floating point arithmetic artifacts [JLS 15.20.1, 15.21.1]. Furthermore, the resulting methods are unlikely to be significantly faster than those written using the techniques described in this item. Use either a static compare method:

不要使用上面这种方法，因为它可能会导整数溢出，而且违反了 IEEE 754浮点算数标准。此外，这个方法也不一定比使用介绍前面的方法块。使用静态比较方法如下：

```java
// Comparator based on static compare method
   static Comparator<Object> hashCodeOrder = new Comparator<>() {
       public int compare(Object o1, Object o2) {
           return Integer.compare(o1.hashCode(), o2.hashCode());
       }
};
```

> or a comparator construction method:

或者使用比较器构造方法如下：

```java
// Comparator based on Comparator construction method
   static Comparator<Object> hashCodeOrder =
           Comparator.comparingInt(o -> o.hashCode());
```

> In summary, whenever you implement a value class that has a sensible ordering, you should have the class implement the Comparable interface so that its instances can be easily sorted, searched, and used in comparison-based collections. When comparing field values in the implementations of the compareTo methods, avoid the use of the < and > operators. Instead, use the static compare methods in the boxed primitive classes or the comparator construction methods in the Comparator interface.

总结一下，当你要实现一个有明确顺序的值类的时候，都应该让这个类实现Comparable接口，如此以来，这个类的实例就可以很容易的被排序，查找，以及使用在基于比较的集合中。在comparaTo方法中实现对域的比较的时候，应该避免使用>和<操作符，相反，应该使用基本类型的封装类里的静态比较方法，或者是Comparator接口里的比较器构造方法。