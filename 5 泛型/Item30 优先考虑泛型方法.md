### Item30 优先考虑泛型方法

> Just as classes can be generic, so can methods. Static utility methods that operate on parameterized types are usually generic. All of the “algorithm” methods in Collections (such as binarySearch and sort) are generic.
>
> Writing generic methods is similar to writing generic types. Consider this deficient method, which returns the union of two sets:

就像类可以是泛型一样，方法也可以。基于参数化类型进行计算的静态工具方法通常来说都是泛型的。在Collections里的所有”算法“方法（比如binarySearch和sort方法）都是泛型的。

写泛型方法和泛型类型差不多。先看下面这个有缺陷的方法，它返回两个set的并集。

```java
// Uses raw types - unacceptable! (Item 26)
   public static Set union(Set s1, Set s2) {
       Set result = new HashSet(s1);
       result.addAll(s2);
       return result;
}
```

> This method compiles but with two warnings:

这个方法可以编译，但是会有两个warning，如下：

```java
Union.java:5: warning: [unchecked] unchecked call to HashSet(Collection<? extends E>) as a member of raw type HashSet
           Set result = new HashSet(s1);
                        ^
   Union.java:6: warning: [unchecked] unchecked call to
   addAll(Collection<? extends E>) as a member of raw type Set
           result.addAll(s2);
                        ^
```

> To fix these warnings and make the method typesafe, modify its declaration to declare a *type parameter* representing the element type for the three sets (the two arguments and the return value) and use this type parameter throughout the method. **The type parameter list, which declares the type parameters, goes between a method’s modifiers and its return type.** In this example, the type parameter list is <E>, and the return type is Set<E>. The naming conventions for type parameters are the same for generic methods and generic types (Items 29, 68):

为了解决掉这写warning，并使得方法是类型安全的，首先要修改它的声明，以声明一个类型参数表示这三个set（两个参数，和一个返回值）的元素类型；然后在整个方法中都使用这个类型参数。**这个声明类型参数的类型参数列表，应该写在方法修饰符和返回类型之间。**在这个例子中，方法参数列表就是<E>，返回类型就是Set<E>。在泛型方法中的类型参数命名习惯和在泛型类型中一样（Item29，68）。泛型方法如下：

```java
// Generic method
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
		Set<E> result = new HashSet<>(s1); 
  	result.addAll(s2);
		return result;
}
```

> At least for simple generic methods, that’s all there is to it. This method compiles without generating any warnings and provides type safety as well as ease of use. Here’s a simple program to exercise the method. This program contains no casts and compiles without errors or warnings:

至少对于简单泛型方法，就是这么回事了。这个方法在编译时不会生成任何的warning，能保证类型安全，使用还方便。下面是一个使用这个方法的一个简单的程序，这个程序不包含任何的类型转换， 在编译时，也不会生成error和warning:

```java
// Simple program to exercise generic method
 public static void main(String[] args) {
       Set<String> guys = Set.of("Tom", "Dick", "Harry");
       Set<String> stooges = Set.of("Larry", "Moe", "Curly");
       Set<String> aflCio = union(guys, stooges);
       System.out.println(aflCio);
}
```

> When you run the program, it prints [Moe, Tom, Harry, Larry, Curly, Dick]. (The order of the elements in the output is implementation-dependent.)

当你运行这个程序的时候，它会打印[Moe, Tom, Harry, Larry, Curly, Dick]。（这个输出的元素的顺序和set的实现有关）。

> A limitation of the union method is that the types of all three sets (both input parameters and the return value) have to be exactly the same. You can make the method more flexible by using *bounded wildcard types* (Item 31).

这个union方法有一个限制，这三个set（包括输入参数和返回值）的类型都必须是完全一样的。你可以使用”有限制的通配符类型“来使得这个方法更加灵活（Item31）

> On occasion, you will need to create an object that is immutable but applicable to many different types. Because generics are implemented by erasure (Item 28), you can use a single object for all required type parameterizations, but you need to write a static factory method to repeatedly dole out the object for each requested type parameterization. This pattern, called the *generic singleton factory*, is used for function objects (Item 42) such as Collections.reverseOrder, and occasionally for collections such as Collections.emptySet.

有时候，你可能需要创建一个不可变对象，但是又需要同时满足很多不同类型。由于泛型是基于擦除实现的（Item28），所以你可以使用一个单个的对象来表示所有不同的类型参数的对象，但是还需要写一个静态工厂方法，来重复地给每一个需要的类型参数返回这个对象。这种模式被称为”泛型单例工厂“，常被用在函数对象上，比如Collections.reverseOrder，有时候也用在集合上，比如Collections.emptySet。

> Suppose that you want to write an identity function dispenser. The libraries provide Function.identity, so there’s no reason to write your own (Item 59), but it is instructive. It would be wasteful to create a new identity function object time one is requested, because it’s stateless. If Java’s generics were reified, you would need one identity function per type, but since they’re erased a generic singleton will suffice. Here’s how it looks:

假如你现在想写一个恒等函数分发器，虽然类库中已经提供了Function.identity，因此不需要再自己编写了（Item59），但是自己编写是很有教育意义的。在每次请求的时候，都创建一个新的恒等函数对象是非常浪费的，因为恒等函数对象是无状态的。如果Java的泛型是具体化的，你就必须要为每一个类型提供一个恒等函数，但是由于泛型是类型擦除的，所以一个泛型对象就够了。下面是恒等函数分发器的代码：

```java
// Generic singleton factory pattern
   private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;
   @SuppressWarnings("unchecked")
	 public static <T> UnaryOperator<T> identityFunction() { 
      return (UnaryOperator<T>) IDENTITY_FN;
	 }
```

> The cast of IDENTITY_FN to (UnaryFunction<T>) generates an unchecked cast warning, as UnaryOperator<Object> is not a UnaryOperator<T> for every T. But the identity function is special: it returns its argument unmodified, so we know that it is typesafe to use it as a UnaryFunction<T>, whatever the value of T. Therefore, we can confidently suppress the unchecked cast warning generated by this cast. Once we’ve done this, the code compiles without error or warning.
>
> Here is a sample program that uses our generic singleton as a UnaryOperator<String> and a UnaryOperator<Number>. As usual, it contains no casts and compiles without errors or warnings:

把IDENTITY_FN转换为UnaryFunction<T>会产生一个非受检转换警告，因为对于每个T，UnaryOperator<Object> 并不都是一个UnaryOperator<T>。但是恒等函数比较特殊：它直接返回了它的参数，没有做任何的修改，因此我们知道，不管T是什么，把UnaryOperator<Object>作为UnaryFunction<T>来使用，都是类型安全的。因此我们可以自信地禁止这个转换生成的非受检转换警告。一旦我们这么做了，代码在编译的时候就不会产生任何的error和warning。

这里有一个简单的程序，使用我们的泛型单例作为UnaryOperator<String>和UnaryOperator<Number>。像往常一样，它没有包含转换，在编译的时候也不会生成error和warning。代码如下：

```java
// Sample program to exercise generic singleton
   public static void main(String[] args) {
       String[] strings = { "jute", "hemp", "nylon" };
       UnaryOperator<String> sameString = identityFunction();
       for (String s : strings)
           System.out.println(sameString.apply(s));
       Number[] numbers = { 1, 2.0, 3L };
       UnaryOperator<Number> sameNumber = identityFunction();
       for (Number n : numbers)
           System.out.println(sameNumber.apply(n));
}
```

> It is permissible, though relatively rare, for a type parameter to be bounded by some expression involving that type parameter itself. This is what’s known as a *recursive type bound*. A common use of recursive type bounds is in connection with the Comparable interface, which defines a type’s natural ordering (Item 14). This interface is shown here:

类型参数受包含自身的表达式的限制是许可的，虽然相对比较少见。这就是”递归类型限制“。一个最常见的递归类型限制和Comparable接口有关，Comparable接口定义了这个类型的自然顺序（Item14）。下面是Comparable接口的代码：

```java
public interface Comparable<T> {
       int compareTo(T o);
}
```

> The type parameter T defines the type to which elements of the type implementing Comparable<T> can be compared. In practice, nearly all types can be compared only to elements of their own type. So, for example, String implements Comparable<String>, Integer implements Comparable<Integer>, and so on.
>
> Many methods take a collection of elements implementing Comparable to sort it, search within it, calculate its minimum or maximum, and the like. To do these things, it is required that every element in the collection be comparable to every other element in it, in other words, that the elements of the list be *mutually comparable*. Here is how to express that constraint:

这个类型参数T定义的类型，可以和实现Comparable<T>的类型的元素进行比较。在实际中，基本所有的类型都只能和自己类型的元素进行比较，因此，比如，String实现了Comparable<String>，Integer实现了Comparable<integer>，等等。

有很方法参数都是一个元素实现了Comparable接口的集合，可以对集合进行排序、查找、计算最大值、最小值、或者类似的。为了要做到这些，需要集合中的每个元素都可以和集合里其他元素进行比较。也就是说，集合中的元素必须可以互相比较。下面是这个限制的表示方式：

```java
// Using a recursive type bound to express mutual comparability 
public static <E extends Comparable<E>> E max(Collection<E> c);
```

> The type bound <E extends Comparable<E>> may be read as “any type E that can be compared to itself,” which corresponds more or less precisely to the notion of mutual comparability.
>
> Here is a method to go with the previous declaration. It calculates the maximum value in a collection according to its elements’ natural order, and it compiles without errors or warnings:

<E extends Comparable<E>>这个类型限制可以读作”每一个可以和自身比较的类型E“，这与互相比较这一概念，不多不少有点一致。

下面前面的这个声明方法的具体实现，根据元素的自然顺序，计算了集合中的最大值。这个方法编译时不会生成任何error和warning。方法代码如下：

```java
// Returns max value in a collection - uses recursive type bound
public static <E extends Comparable<E>> E max(Collection<E> c) { 
			 if (c.isEmpty())
           throw new IllegalArgumentException("Empty collection");
       E result = null;
       for (E e : c)
           if (result == null || e.compareTo(result) > 0)
               result = Objects.requireNonNull(e);
       return result;
}
```

> Note that this method throws IllegalArgumentException if the list is empty. A better alternative would be to return an Optional<E> (Item 55).
>
> Recursive type bounds can get much more complex, but luckily they rarely do. If you understand this idiom, its wildcard variant (Item 31), and the *simulated self-type* idiom (Item 2), you’ll be able to deal with most of the recursive type bounds you encounter in practice.
>
> In summary, generic methods, like generic types, are safer and easier to use than methods requiring their clients to put explicit casts on input parameters and return values. Like types, you should make sure that your methods can be used without casts, which often means making them generic. And like types, you should generify existing methods whose use requires casts. This makes life easier for new users without breaking existing clients (Item 26).

需要注意的是，如果集合是空的，这个方法会抛出IllegalArgumentException，更好的替代方法是返回一个Optional<E>(Item55)。

递归类型限制，可以变得超级复杂，还好人们很少这样去做。如果你理解了这种习惯用法，它的通配符变体，和模拟自类型的习惯用法，你就可以很好地处理你实际中遇到的绝大部分递归类型限制。

总的来说，泛型方法，和泛型类型一样，比那些需要客户端对输入参数和返回值进行显示类型转换的方法，要安全简单得多。和泛型类型相似，你应该要确定你的方法可以不转换就使用，这通常意味着需要将它泛型化。和泛型类型相似，你应该把所有已经存在的需要类型转换的方法泛型化，这样可以给新用户带来方便，也不会影响那些已经存在的客户端（Item26）。