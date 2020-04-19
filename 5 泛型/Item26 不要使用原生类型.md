## 5 泛型

> **S**INCE Java 5, generics have been a part of the language. Before generics, you had to cast every object you read from a collection. If someone accidentally inserted an object of the wrong type, casts could fail at runtime. With generics, you tell the compiler what types of objects are permitted in each collection. The compiler inserts casts for you automatically and tells you *at compile time* if you try to insert an object of the wrong type. This results in programs that are both safer and clearer, but these benefits, which are not limited to collections, come at a price. This chapter tells you how to maximize the benefits and minimize the complications.

从Java5开始，泛型就成为了Java语言中的一部分。在拥有泛型之前，你必须对每个从集合中读出来的对象进行类型转换。如果有人不小心插入了一个错误类型的对象，这个类型转换就会再运行时失败。使用泛型，你就可以告诉编译器，那些类型的对象允许加入这个集合。编译器会自动地为你进行转换，然后当你要插入一个错误类型的时候，在编译地时候就会报错。这样会使得程序更加安全和清晰，但是要享有这些好处（不仅仅限于集合），是有一定的代价的。本章将告诉你如何将这些好处最大化，同时降低复杂度。

### Item26 不要使用原生类型

> First, a few terms. A class or interface whose declaration has one or more *type parameters* is a *generic* class or interface [JLS, 8.1.2, 9.1.2]. For example, the List interface has a single type parameter, E, representing its element type. The full name of the interface is List<E> (read “list of E”), but people often call it List for short. Generic classes and interfaces are collectively known as *generic types*.

首先介绍一些使用到的术语，一个类或者接口，如果它的声明中带有一个或者多个类型参数，那么它就是要一个泛型类或接口 [JLS, 8.1.2, 9.1.2]。比如，List接口就有一个类型参数E，用来表示其中元素的类型。这个接口的全称是List<E>（读作E的列表），人们通常将其简称为List。泛型类和接口也被统称为泛型。

> Each generic type defines a set of *parameterized types*, which consist of the class or interface name followed by an angle-bracketed list of *actual type parameters* corresponding to the generic type’s formal type parameters [JLS, 4.4, 4.5]. For example, List<String> (read “list of string”) is a parameterized type representing a list whose elements are of type String. (String is the actual type parameter corresponding to the formal type parameter E.)

每一个泛型都包括一组参数类型，其构成方式如下：首先是类或者接口的名字，然后紧跟着用<>括起来的实际类型参数，实际类型参数和泛型中的形式类型参数对应。比如List<String>（读作String的列表），就是一个表示列表的元素类型为String的参数类型，其中String就是一个对应形式类型参数E的实际类型参数。

> Finally, each generic type defines a *raw type*, which is the name of the generic type used without any accompanying type parameters [JLS, 4.8]. For example, the raw type corresponding to List<E> is List. Raw types behave as if all of the generic type information were erased from the type declaration. They exist primarily for compatibility with pre-generics code.

最后，每个泛型类型都定义了一个原生类型，其名字就是没有任何类型参数的泛型类型的名称。比如List<E>的原生类型就是List。原生类型就像是把类型信息从类型声明中擦除了样。它们存在的主要目的是为了保持泛型出现之前的代码的兼容性。

> Before generics were added to Java, this would have been an exemplary collection declaration. As of Java 9, it is still legal, but far from exemplary:

在泛型被加入到Java中之前，下面这段集合声明是可以说是值得参考的。在Java9中，这段代码也是合法的，但却没什么参考价值了。

```java
// Raw collection type - don't do this!
// My stamp collection. Contains only Stamp instances.
private final Collection stamps = ... ;
```

> If you use this declaration today and then accidentally put a coin into your stamp collection, the erroneous insertion compiles and runs without error (though the compiler does emit a vague warning):

如果你现在使用这样的声明，然后不小心往stamp集合里插入了一个coin，这个错误的插入可以正常的编译和运行（虽然编译器会报一个模糊的警告）。

```java
// Erroneous insertion of coin into stamp collection
stamps.add(new Coin( ... )); // Emits "unchecked call" warning
```

> You don’t get an error until you try to retrieve the coin from the stamp collection:

直到你要从stamp集合里取出这个coin的时候，才会报错。

```java
// Raw iterator type - don't do this!
for (Iterator i = stamps.iterator(); i.hasNext(); )
Stamp stamp = (Stamp) i.next(); // Throws ClassCastException stamp.cancel();
```

> As mentioned throughout this book, it pays to discover errors as soon as possible after they are made, ideally at compile time. In this case, you don’t discover the error until runtime, long after it has happened, and in code that may be distant from the code containing the error. Once you see the ClassCastException, you have to search through the codebase looking for the method invocation that put the coin into the stamp collection. The compiler can’t help you, because it can’t understand the comment that says, “Contains only Stamp instances.”
>
> With generics, the type declaration contains the information, not the comment:

正如这本书中经常提到的那样，在错误出现后，应该尽快发现，最好是在编译期发现。在这个例子中，要在运行时才能发现错误，而且错误发生了很久了才会报错，报错的代码和实际出错的代码相隔很远。一旦你看见了ClassCastException，你就必须要搜索所有的代码来找到把coin放入stamp集合的方法调用。编译器帮不了你，因为它不能理解“只包含Stamp实例”这个注解。

使用泛型的话，这个类型声明就会包含这个信息，而不是写注解里。如下：

```java
// Parameterized collection type - typesafe
private final Collection<Stamp> stamps = ... ;
```

> From this declaration, the compiler knows that stamps should contain only Stamp instances and *guarantees* it to be true, assuming your entire codebase compiles without emitting (or suppressing; see Item 27) any warnings. When stamps is declared with a parameterized type declaration, the erroneous insertion generates a compile-time error message that tells you *exactly* what is wrong:

从这个声明中，假设你的整个代码库在编译器都没有抛出（或者隐藏，详见Item27）任何警告，编译器就可以知道stamps只应该包含Stamp实例，并且做出保证。当stamp列表使用参数类型声明的时候，错误的插入在编译器就会生成一个错误信息，告诉你哪里出错了：

```java
Test.java:9: error: incompatible types: Coin cannot be converted to Stamp
       c.add(new Coin());
                 ^
```

> The compiler inserts invisible casts for you when retrieving elements from collections and guarantees that they won’t fail (assuming, again, that all of your code did not generate or suppress any compiler warnings). While the prospect of accidentally inserting a coin into a stamp collection may appear far-fetched, the problem is real. For example, it is easy to imagine putting a BigInteger into a collection that is supposed to contain only BigDecimal instances.

当从集合中取出元素的时候，编译器会进行隐式地转换，并保证不会失败（同样地，其前提还是，你的所有的代码在编译时没有生成或隐藏任何的警告）。虽然不小心把一个coin插入到stamp集合中，有点牵强，但是这类问题却是真实存在的。比如，就很容易想象，有人会把BigInteger插入到只支持BigDecimal实例的集合中去。

> As noted earlier, it is legal to use raw types (generic types without their type parameters), but you should never do it. **If you use raw types, you lose all the safety and expressiveness benefits of generics.** Given that you shouldn’t use them, why did the language designers permit raw types in the first place? For compatibility. Java was about to enter its second decade when generics were added, and there was an enormous amount of code in existence that did not use generics. It was deemed critical that all of this code remain legal and interoperate with newer code that does use generics. It had to be legal to pass instances of parameterized types to methods that were designed for use with raw types, and vice versa. This requirement, known as *migration compatibility*, drove the decisions to support raw types and to implement generics using *erasure* (Item 28).

正如前面说的那样，虽然使用原生类型是合法的，但是你永远都不应该这样做。**如果你使用了原生类型，你就失去了泛型带来的安全性和所有的描述性方面的优势。**既然我们不应该用它们，那为什么语言的设计者还要允许使用它们呢？是为了保持兼容性。当泛型加入的时候，Java即将进入它的第二个十年，已经有大量的没有使用泛型的代码存在了。保证已经存在的代码合法，并且可以和使用泛型的代码互相调用，这是很重要。而且将参数化类型的实例传递给为原生类型设计的方法必须是合法的。这个要求，称为“移植兼容性”，促成了支持原生类型和使用擦除来实现泛型的决定（Item28）。

> While you shouldn’t use raw types such as List, it is fine to use types that are parameterized to allow insertion of arbitrary objects, such as List<Object>. Just what is the difference between the raw type List and the parameterized type List<Object>? Loosely speaking, the former has opted out of the generic type system, while the latter has explicitly told the compiler that it is capable of holding objects of any type. While you can pass a List<String> to a parameter of type List, you can’t pass it to a parameter of type List<Object>. There are subtyping rules for generics, and List<String> is a subtype of the raw type List, but not of the parameterized type List<Object> (Item 28). As a consequence, **you lose type safety if you use a raw type such as** **List, but not if you use a param- eterized type such as List<Object>.**
>
> To make this concrete, consider the following program:

虽然你不应该使用原生类型，比如List，但是使用参数化类型来允许插入任意的对象（比如List<Object>）却是可行的。那么，原生类型List和参数化类型List<Object>之间有什么区别呢？不严格地说，前者不属于泛型系统，而后者明确地告诉了编译器，可持有任意类型的对象。你可以把一个List<String>传递给一个类型为List的参数，但是却不能把它传给一个类型为List<Object>的参数。在泛型中也有子类规则。List<String>是List的子类，却不是参数化类型List<Object>的子类（Item28）。因此，当你使用原生类型的时候，比如List，你就失去了类型安全性，但当你使用参数化类型，比如List<Object>的时候就不会。

为了说得更明确一些，看下面这个程序：

```java
// Fails at runtime - unsafeAdd method uses a raw type (List)!
public static void main(String[] args) {
		List<String> strings = new ArrayList<>(); 
  	unsafeAdd(strings, Integer.valueOf(42));
		String s = strings.get(0); // Has compiler-generated cast
}
private static void unsafeAdd(List list, Object o) { 
  list.add(o);
}
```

> This program compiles, but because it uses the raw type List, you get a warning:

由于程序使用的是原生类型，所有可以编译，会收到一条警告如下：

```java
Test.java:10: warning: [unchecked] unchecked call to add(E) as a member of the raw type List
             list.add(o);
                     ^
```

> And indeed, if you run the program, you get a ClassCastException when the program tries to cast the result of the invocation strings.get(0), which is an Integer, to a String. This is a compiler-generated cast, so it’s normally guaranteed to succeed, but in this case we ignored a compiler warning and paid the price.

确实是这样的，如果你运行这个程序的话，当程序试图把strings.get(0)调用的结果从Integer转换到String时候，就会抛出一个ClassCastException。这是一个编译器生成的转换，通常来说是会成功的，但是由于我们忽略了编译器警告，因此便付出了代价。

> If you replace the raw type List with the parameterized type List<Object> in the unsafeAdd declaration and try to recompile the program, you’ll find that it no longer compiles but emits the error message:

如果你在unsafeAdd的声明中，使用参数化类型List<Object>来代替原生类型List，然后重新编译程序，你就会发现，如果不解决下面这个错误信息的话，是无法进行编译的：

```java
Test.java:5: error: incompatible types: List<String> cannot be
         converted to List<Object>
             unsafeAdd(strings, Integer.valueOf(42));
                 ^
```

> You might be tempted to use a raw type for a collection whose element type is unknown and doesn’t matter. For example, suppose you want to write a method that takes two sets and returns the number of elements they have in common. Here’s how you might write such a method if you were new to generics:

在不确定或者不在乎集合内的元素类型的时候，你可能会使用原生类型。比如，假设你想写一个方法，从两个set中返回其中相同的元素的个数。如果你会泛型不了解的话，你可能会写出下面这样的方法：

```java
// Use of raw type for unknown element type - don't do this! 
static int numElementsInCommon(Set s1, Set s2) {
             int result = 0;
             for (Object o1 : s1)
                 if (s2.contains(o1))
                     result++;
             return result;
     }
```

> This method works but it uses raw types, which are dangerous. The safe alternative is to use *unbounded wildcard types*. If you want to use a generic type but you don’t know or care what the actual type parameter is, you can use a question mark instead. For example, the unbounded wildcard type for the generic type Set<E> is Set<?> (read “set of some type”). It is the most general parameterized Set type, capable of holding *any* set. Here is how the numElementsInCommon declaration looks with unbounded wildcard types:

这个方法可以工作，但是使用了危险的原生类型。安全的替代方法是使用无限制通配符类型（unbounded wildcard types）。当你想使用泛型，却又不知道也不关系其真正的类型参数是什么的时候你就可以使用一个问号来代替。比如泛型Set<E>的无限制通配符类型就是Set<?>(读作，某个类型的集合)。这是一个最普通的参数化Set类型，可以持有任意的Set。下面是numElementsInCommon使用无限制通配符类型进行声明的代码：

```java
// Uses unbounded wildcard type - typesafe and flexible 
static int numElementsInCommon(Set<?> s1, Set<?> s2) { ... }
```

> What is the difference between the unbounded wildcard type Set<?> and the raw type Set? Does the question mark really buy you anything? Not to belabor the point, but the wildcard type is safe and the raw type isn’t. You can put *any* element into a collection with a raw type, easily corrupting the collection’s type invariant (as demonstrated by the unsafeAdd method on page 119); **you can’t put any element (other than null) into a Collection< ？>**. Attempting to do so will generate a compile-time error message like this:

那么，无限制通配符类型Set<?>和原生类型Set之间的区别是什么呢？这个问号确实能起到作用吗？通配符类型是安全的，但是原生类型却不是，这一点是毋庸置疑的。你可以往一个原生类型的集合里，添加任何一个元素，可以很容易打破集合的类型约束（就像前面的unsafeAdd方法所示范的那样）；**但是你却不能往一个Collection<？>里添加除了null以外的任何元素**。当你企图这么做的时候，就会在编译的时候，生成一个如下所示的错误信息：

```java
WildCard.java:13: error: incompatible types: String cannot be
   converted to CAP#1
       c.add("verboten");
             ^
     where CAP#1 is a fresh type-variable:
       CAP#1 extends Object from capture of ?
```

> Admittedly this error message leaves something to be desired, but the compiler has done its job, preventing you from corrupting the collection’s type invariant, whatever its element type may be. Not only can’t you put any element (other than null) into a Collection<?>, but you can’t assume anything about the type of the objects that you get out. If these restrictions are unacceptable, you can use *generic methods* (Item 30) or *bounded wildcard types* (Item 31).

虽然这个错误信息还缺少一些想看到的东西， 但是编译器已经完成了它的任务，阻止了我们破坏集合的类型约束，不论这个元素类型是什么。你不仅仅不能把任何元素（除了null）放到Collection<?>里去，还不能对从里面取出来的元素的类型做任何的假设。如果这些限制无法接受，你可以使用泛型方法（Item30）或者有限制通配符类型（Item31）。

> There are a few minor exceptions to the rule that you should not use raw types. **You must use raw types in class literals.** The specification does not permit the use of parameterized types (though it does permit array types and primitive types) [JLS, 15.8.2]. In other words, List.class, String[].class, and int.class are all legal, but List<String>.class and List<?>.class are not.

对于你不应该使用原生类型这一规则，有几种小小的例外情况。**在类字面量中，你必须使用原生类型**。规范不允许使用参数化类型（但是又允许使用数组类型和基本类型）[JLS, 15.8.2]。换句话说，List.class, String[].class, 和int.class都是合法的，但是 List<String>.class 和 List<?>.class却是不合法的。

> A second exception to the rule concerns the instanceof operator. Because generic type information is erased at runtime, it is illegal to use the instanceof operator on parameterized types other than unbounded wildcard types. The use of unbounded wildcard types in place of raw types does not affect the behavior of the instanceof operator in any way. In this case, the angle brackets and question marks are just noise. **This is the preferred way to use the** **instanceof** **operator with generic types:**

这个规则的第二个例外，是关于instanceof操作符的，因为在运行中，泛型类型信息是被擦除了，使用instanceof操作符在参数化类型上时非法的，无限制通配符类型除外。使用无限制通配符来代替原生类型不会对instanceof操作符的结果产生任何的影响，在这种情况下，<> 和 ? 就仅仅只是多余的了。下面是泛型使用instanceof操作符的首选的方法：

```java
// Legitimate use of raw type - instanceof operator 
if (o instanceof Set) { // Raw type
	Set<?> s = (Set<?>) o; // Wildcard type
	... 
}
```

> Note that once you’ve determined that o is a Set, you must cast it to the wildcard type Set<?>, not the raw type Set. This is a checked cast, so it will not cause a compiler warning.

需要注意的是， 一旦你确定了o是一个Set，就不必须讲题转化为通配符类型Set<?>，而不是原生类型。这是一个受检的转化，这样就不会出现编译器警告了。

> In summary, using raw types can lead to exceptions at runtime, so don’t use them. They are provided only for compatibility and interoperability with legacy code that predates the introduction of generics. As a quick review, Set<Object> is a parameterized type representing a set that can contain objects of any type, Set<?> is a wildcard type representing a set that can contain only objects of some unknown type, and Set is a raw type, which opts out of the generic type system. The first two are safe, and the last is not.
>
> For quick reference, the terms introduced in this item (and a few introduced later in this chapter) are summarized in the following table:

总结一下，使用原生类型可能会在运行时出现异常，因此不要使用它们。它们只是用来保证泛型发布之前的代码的兼容性和互用性的。来做一个快速的回顾，Set<object>是一个参数化类型，表示这个set可以持有任意类型的对象；Set<?>是一个通配符类型，表示这个set只能包含某个未知类型的对象；Set是一个原生类型，不适于泛型类型系统的一部分。前两种是安全的，而后一种是不安全的。

为了便于参考，将本节中介绍的术语（还有一些在本章中后面会用到）总结如下表：

|                    术语                     |               范例               |      条目      |
| :-----------------------------------------: | :------------------------------: | :------------: |
|      参数化类型（Parameterized type）       |           List<String>           |     Item26     |
|    实际类型参数（Actual type parameter）    |              String              |     Item26     |
|            泛型（Generic type）             |             List<E>              | Item26和Iten29 |
|    形式类型参数（Formal type parameter）    |                E                 |     Item26     |
| 无限制通配符类型（Unbounded wildcard type） |             List<?>              |     Item26     |
|            原生类型（Raw type）             |               List               |     Item26     |
|   有限制类型参数（Bounded type parameter)   |        <E extends Number>        |     Item29     |
|    递归类型限制（Recursive type bound）     |    <T extends Comparable<T>>     |     Item30     |
|  有限制通配符类型（Bounded wildcard type）  |      List<? extends Number>      |     Item31     |
|         泛型方法（Generic method）          | static <E> List<E> asList(E[] a) |     Item30     |
|           类型令牌（Type token）            |           String.class           |     Item33     |















