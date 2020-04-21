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

























