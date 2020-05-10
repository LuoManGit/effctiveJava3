### Item52 谨慎使用重载

> The following program is a well-intentioned attempt to classify collections according to whether they are sets, lists, or some other kind of collection:

下面这个程序的设计意图是很好的，它企图根据集合是Set、List 或者Collection来进行分类：

```java
// Broken! - What does this program print?
   public class CollectionClassifier {
       public static String classify(Set<?> s) {
           return "Set";
       }
       public static String classify(List<?> lst) {
           return "List";
       }
       public static String classify(Collection<?> c) {
           return "Unknown Collection";
       }
       public static void main(String[] args) {
           Collection<?>[] collections = {
               new HashSet<String>(),
               new ArrayList<BigInteger>(),
               new HashMap<String, String>().values()
           };
           for (Collection<?> c : collections)
               System.out.println(classify(c));
       } 
   }
```

> You might expect this program to print Set, followed by List and Unknown Collection, but it doesn’t. It prints Unknown Collection three times. Why does this happen? Because the classify method is *overloaded*, and **the choice of which overloading to invoke is made at compile time.** For all three iterations of the loop, the compile-time type of the parameter is the same: Collection<？>. The runtime type is different in each iteration, but this does not affect the choice of overloading. Because the compile-time type of the parameter is Collection<?>, the only applicable overloading is the third one, classify(Collection<？>), and this overloading is invoked in each iteration of the loop.

你可能希望这个程序户会打印Set，然后紧跟着List和Unknown Collection，但是它不是这样的。它打印了三次Unknown Collection。为什么会这样呢？因为这个classify方法是重载的，**重载方法的调用选择是在编译器生成的**。对于循环中的每一个迭代，这些参数的编译器类型都是一样的，都是Collection<？>。在每次迭代中，运行参数类型是不一样的，但是已经不能影响重载方法的选择了。因为编译时，参数类型是Collection<？>，唯一合适的重载方法就是第三个classify(Collection<？>)，因此在循环中的每次迭代中调用的都是这个方法。

> The behavior of this program is counterintuitive because **selection among overloaded methods is static, while selection among overridden methods is dynamic.** The correct version of an *overridden* method is chosen at runtime, based on the runtime type of the object on which the method is invoked. As a reminder, a method is overridden when a subclass contains a method declaration with the same signature as a method declaration in an ancestor. If an instance method is overridden in a subclass and this method is invoked on an instance of the subclass, the subclass’s *overriding method* executes, regardless of the compile- time type of the subclass instance. To make this concrete, consider the following program:

这个程序的行为违反直觉，是因为**重载方法的选择就静态的，而被覆盖的方法的选择是动态的。在运行时，可以选择正确的覆盖方法版本，是因为方法的调用是基于对象的运行时类型的。还需要提醒一下，覆盖方法是指，子类中有一个方法的方法声明的签名和其祖先中的方法声明一样。如果这个实例方法在子类中覆盖了，当在这个子类实例上调用这个方法的时候，子类覆盖的方法就会执行，不管这个子类实例编译时的类型是什么。为了进行具体地说明，看看下面这个程序：

```java
class Wine {
       String name() { return "wine"; }
}
class SparklingWine extends Wine {
       @Override String name() { return "sparkling wine"; }
}
class Champagne extends SparklingWine {
       @Override String name() { return "champagne"; }
}
public class Overriding {
       public static void main(String[] args) {
         List<Wine> wineList = List.of(
           new Wine(), new SparklingWine(), new Champagne());
         for (Wine wine : wineList)
           System.out.println(wine.name());
       } 
}
```

> The name method is declared in class Wine and overridden in subclasses SparklingWine and Champagne. As you would expect, this program prints out wine, sparkling wine, and champagne, even though the compile-time type of the instance is Wine in each iteration of the loop. The compile-time type of an object has no effect on which method is executed when an overridden method is invoked; the “most specific” overriding method always gets executed. Compare this to overloading, where the runtime type of an object has no effect on which overloading is executed; the selection is made at compile time, based entirely on the compile-time types of the parameters.

name这个方法声明在Wine类里，然后在子类SparklingWine和Champagne中都进行了覆盖。正如你所想的那样，这个程序会打印” wine, sparkling wine,  champagne“，即使在循环的每个迭代中，每个实例的编译期类型都是Wine。当被覆盖的方法被调用的时候，编译期类型并不会影响到具体执行的方法，最具体的覆盖方法总是会被调用。而在重载中，运行时的类型不会影响重载方法的调用，这个决定是在编译期进行的，完全取决于参数的编译期类型。

> In the CollectionClassifier example, the intent of the program was to discern the type of the parameter by dispatching automatically to the appropriate method overloading based on the runtime type of the parameter, just as the name method did in the Wine example. Method overloading simply does not provide this functionality. Assuming a static method is required, the best way to fix the CollectionClassifier program is to replace all three overloadings of classify with a single method that does explicit instanceof tests:

在CollectionClassifier例子里，程序希望，基于参数运行时类型，自动识别根据参数的类型来分配到合适的重载函数上，就和Wine例子里的name方法一样。但是方法重载不能提供这样的功能。假如需要一个静态方法，最好的修复CollectionClassifier程序的方法是使用一个包含明确的instanceof测试的单个方法，来替代这3个重载方法。单个方法代码如下：

```java
public static String classify(Collection<?> c) {
       return c instanceof Set  ? "Set" :
              c instanceof List ? "List" : "Unknown Collection";
}
```

> Because overriding is the norm and overloading is the exception, overriding sets people’s expectations for the behavior of method invocation. As demonstrated by the CollectionClassifier example, overloading can easily confound these expectations. It is bad practice to write code whose behavior is likely to confuse programmers. This is especially true for APIs. If the typical user of an API does not know which of several method overloadings will get invoked for a given set of parameters, use of the API is likely to result in errors. These errors will likely manifest themselves as erratic behavior at runtime, and many programmers will have a hard time diagnosing them. Therefore you should **avoid confusing uses of overloading.**

因为覆盖方法的结果是合乎情理的，而重载方法结果却出人意料，所以覆盖方法满足了人们对于方法调用行为的期望。正如CollectionClassifier例子所展示的那样，重载很容易出现一些意料之外的结果，让人困惑。在实际应用中，编写一些让程序员困惑的代码，是很不好的。尤其是对于API而言，如果一个API的大部分用户都不知道给定的一组参数，会调用几个重载方法中的哪一个，那么这个API的使用很可能出错，这些错误，在运行时，很可能表现为不正确的行为，很多程序员都需要花很长的时间来诊断这些问题。因此，**应该避免一些让人困惑的重载的使用。”

> Exactly what constitutes a confusing use of overloading is open to some debate. **A safe, conservative policy is never to export two overloadings with the same number of parameters.** If a method uses varargs, a conservative policy is not to overload it at all, except as described in Item 53. If you adhere to these restrictions, programmers will never be in doubt as to which overloading applies to any set of actual parameters. These restrictions are not terribly onerous because **you can always give methods different names instead of overloading them.**

是什么造成的重载的让人疑惑的使用的呢？现在还是存在争议。**一个安全，保守的策略是永远不要导出两个参数个数相同的重载方法。**如果一个方法使用了可变参数，一个保守的策略是除了Item53里介绍的情况外，不要重载它。如果你遵守了这些规则，程序员就永远不会，在应用一组实际参数到重载方法的时候，存在疑惑。这些限制也不怎么麻烦，因为**你可以通过给一些方法起不同的名字来替代重载他们**。

> For example, consider the ObjectOutputStream class. It has a variant of its write method for every primitive type and for several reference types. Rather than overloading the write method, these variants all have different names, such as writeBoolean(boolean), writeInt(int), and writeLong(long). An added benefit of this naming pattern, when compared to overloading, is that it is possible to provide read methods with corresponding names, for example, readBoolean(), readInt(), and readLong(). The ObjectInputStream class does, in fact, provide such read methods.

比如，ObjectOutputStream类。它的write方法针对每一个基本类型和几个引用类型都有对应的变体。这些变体并没有重载write方法，而是有不同的名字，比如writeBoolean(boolean), writeInt(int), 和 writeLong(long)。和重载相比，这种命名模式还有一个优点，就是还可以为read方法提供对应的名字，比如，readBoolean(), readInt(), 和 readLong()。实际上，ObjectInputStream类也确实提供了这些读方法。

> For constructors, you don’t have the option of using different names: multiple constructors for a class are *always* overloaded. You do, in many cases, have the option of exporting static factories instead of constructors (Item 1). Also, with constructors you don’t have to worry about interactions between overloading and overriding, because constructors can’t be overridden. You will probably have occasion to export multiple constructors with the same number of parameters, so it pays to know how to do it safely.

对于构造器，你没有办法选择不同的名字，因此一个构造器的多个构造器总是重载的。你也确实可以选择使用静态工厂方法来替代构造器（Item1）。并且，使用构造器，你不用担心覆盖方法和重载之间的相互作用，因为构造器不能被覆盖。你可能有是有导出了一个构造器，它的参数格式想用，这个时候就需要了解一下如何能安全地使用它。

> Exporting multiple overloadings with the same number of parameters is unlikely to confuse programmers *if* it is always clear which overloading will apply to any given set of actual parameters. This is the case when at least one corresponding formal parameter in each pair of overloadings has a “radically different” type in the two overloadings. Two types are radically different if it is clearly impossible to cast any non-null expression to both types. Under these circumstances, which overloading applies to a given set of actual parameters is fully determined by the runtime types of the parameters and cannot be affected by their compile-time types, so a major source of confusion goes away. For example, ArrayList has one constructor that takes an int and a second constructor that takes a Collection. It is hard to imagine any confusion over which of these two constructors will be invoked under any circumstances.

如果，对于给定的参数的组合，总是可以很清楚地知道会使用哪一种重载，那么导出几个参数数量相同的参数也可以不给程序员带来疑惑。这种情况就是，至少有一个对应的参数在两个重载中有”完全不同“的类型。如果这两个类型的非空表示之间不能进行任何转换，那么这两个类就是完全不同的。在这种情况下，给定一组确定的参数，可以根据它的运行时参数类型完全决定该使用哪个重载，而不会受编译器类型的影响（*？？？选择使用哪个重载，应该还是在编译期决定的，怎么又运行期了。*），因此造成困惑的主要的来源就没有了。比如，ArrayList有一个参数为int的构造器，还有一个参数为Collection的构造器，因此很难想象在什么情况下，会对这两个构造器的调用产生疑惑。

> Prior to Java 5, all primitive types were radically different from all reference types, but this is not true in the presence of autoboxing, and it has caused real trouble. Consider the following program:

在Java5之前，所有的基本类型和引用类型都是完全不同的类型，但是自动装箱出现以后，就不是这样的了，并且会造成一些真实的问题。考虑下面这个程序：

```java
public class SetList {
       public static void main(String[] args) {
           Set<Integer> set = new TreeSet<>();
           List<Integer> list = new ArrayList<>();
           for (int i = -3; i < 3; i++) {
               set.add(i);
               list.add(i);
           }
           for (int i = 0; i < 3; i++) {
               set.remove(i);
               list.remove(i);
           }
           System.out.println(set + " " + list);
       }
}
```

> First, the program adds the integers from −3 to 2, inclusive, to a sorted set and a list. Then, it makes three identical calls to remove on the set and the list. If you’re like most people, you’d expect the program to remove the non-negative values (0, 1, and 2) from the set and the list and to print [-3, -2, -1] [-3, -2, -1]. In fact, the program removes the non-negative values from the set and the odd values from the list and prints [-3, -2, -1] [-2, 0, 2]. It is an understatement to call this behavior confusing.

首先，程序分别往一个TreeSet和List实例里添加了从-3到2（全包括）的integer值。然后，对set和list调用了3次相同的remove方法。如果你和大多数人一样，你可能会希望这个程序会从set和list里删除非负数值（0，1，2），然后打印 [-3, -2, -1]  [-3, -2, -1]。实际上，这个程序删除了set里面的非负值，list里的奇数，打印了[-3, -2, -1] [-2, 0, 2]。我们把这个称之为混乱，已经算是很保守的说法了。

> Here’s what’s happening: The call to set.remove(i) selects the overloading remove(E), where E is the element type of the set (Integer), and autoboxes i from int to Integer. This is the behavior you’d expect, so the program ends up removing the positive values from the set. The call to list.remove(i), on the other hand, selects the overloading remove(int i), which removes the element at the specified *position* in the list. If you start with the list [-3, -2, -1, 0, 1, 2] and remove the zeroth element, then the first, and then the second, you’re left with [-2, 0, 2], and the mystery is solved. To fix the problem, cast list.remove’s argument to Integer, forcing the correct overloading to be selected. Alternatively, you could invoke Integer.valueOf on i and pass the result to list.remove. Either way, the program prints [-3, -2, -1] [-3, -2, -1], as expected:

实际发生的是这样的：这个set.remove(i)调用选择的是重载remove(E)，这里的E表示的是set<Integer>里面的元素的类型，然后把i自动装箱成Integer。这个行为就正是你所期望的，因此这个程序也删除了set里的正值。另一方面， list.remove(i)方法的调用，选择的重载是remove(int i)，删除的是列表中指定位置的元素。如果你一开始列表是 [-3, -2, -1, 0, 1, 2] ，然后删除第0个元素，然后删除第一个元素，然后删除第二个，剩下的就是 [-2, 0, 2]，这个莫名其妙的事情就是这样的。为了解决这个问题，可以把list.remove方法的参数转换为Integer，强制选择中确的重载。你也可以选择调用Integer.valueOf(i)然后把返回结果传给list.remove方法。不管是哪一种方法，这个程序都会像期待的那样，打印[-3, -2, -1] [-3, -2, -1]。

```java
for (int i = 0; i < 3; i++) {
  set.remove(i);
  list.remove((Integer) i); // or remove(Integer.valueOf(i)) 
}
```

> The confusing behavior demonstrated by the previous example came about because the List<E> interface has two overloadings of the remove method: remove(E) and remove(int). Prior to Java 5 when the List interface was “generified,” it had a remove(Object) method in place of remove(E), and the corresponding parameter types, Object and int, were radically different. But in the presence of generics and autoboxing, the two parameter types are no longer radically different. In other words, adding generics and autoboxing to the language damaged the List interface. Luckily, few if any other APIs in the Java libraries were similarly damaged, but this tale makes it clear that autoboxing and generics increased the importance of caution when overloading.





















