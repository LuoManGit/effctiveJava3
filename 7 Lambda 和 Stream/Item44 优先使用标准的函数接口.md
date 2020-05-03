### Item44 优先使用标准的函数接口

> Now that Java has lambdas, best practices for writing APIs have changed considerably. For example, the *Template Method* pattern [Gamma95], wherein a subclass overrides a *primitive method* to specialize the behavior of its superclass, is far less attractive. The modern alternative is to provide a static factory or constructor that accepts a function object to achieve the same effect. More generally, you’ll be writing more constructors and methods that take function objects as parameters. Choosing the right functional parameter type demands care.

Java现在拥有了lambda，编写API的最佳实践也有了很大的变化。比如在模板方法模式里，其子类通过覆盖父类中的基本方法来实现特殊化的行为，就不那么吸引人了。现在的替换方法就是提供一个接受函数对象的静态工厂或者构造器来达到同样的效果。通俗来说，你都会编写更多使用函数对象多为参数的构造器和方法，因此选择一个正确的函数参数类型就值得我们关注了。

> Consider LinkedHashMap. You can use this class as a cache by overriding its protected removeEldestEntry method, which is invoked by put each time a new key is added to the map. When this method returns true, the map removes its eldest entry, which is passed to the method. The following override allows the map to grow to one hundred entries and then deletes the eldest entry each time a new key is added, maintaining the hundred most recent entries:

比如LinkedHashMap，你可以通过覆盖它的受保护的removeEldestEntry方法来把这个类当做缓存使用，removeEldestEntry方法在有新的值添加到map中的时候，就会被调用。当这个方法返回true的时候，这个map剧会移除里面最老的元素。下面的覆盖使得这个map在增加到100个元素后，在每次加入新的键的时候，就会删除里面最老的元素，一致保存着最近的100个元素：

```java
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
      return size() > 100;
}
```

> This technique works fine, but you can do much better with lambdas. If LinkedHashMap were written today, it would have a static factory or constructor that took a function object. Looking at the declaration for removeEldestEntry, you might think that the function object should take a Map.Entry<K,V> and return a boolean, but that wouldn’t quite do it: The removeEldestEntry method calls size() to get the number of entries in the map, which works because removeEldestEntry is an instance method on the map. The function object that you pass to the constructor is not an instance method on the map and can’t capture it because the map doesn’t exist yet when its factory or constructor is invoked. Thus, the map must pass itself to the function object, which must therefore take the map on input as well as its eldest entry. If you were to declare such a functional interface, it would look something like this:

这个方法可以很好的工作，但是你可以用lambda来做得更好。如果LinkedHashMap是今天写的，它就应该有一个使用函数队形的静态工厂或者构造器。看着这个removeEldestEntry方法的声明，你可能会觉得这种歌函数对象应该有一个Map.Entry<K,V> 参数，并返回一个boolean，但是不能这么做。因为removeEldestEntry方法可以直接调用size（）来获取map中的元素数量，是因为removeEldestEntry方法也是一个map上的实例方法。但是你传递给构造器方法的这个函数对象不是实例方法，同时也不能获取到map实例，因为当构造器或者工厂方法调用的时候这个map根本就不还存在。所以，这个map必须把自己传递给这个函数对象，因此它的输入必须既包括map也包括它的最老的元素。如果你要声明这样一个函数接口，它可能长这样：

```java
// Unnecessary functional interface; use a standard one instead.
   @FunctionalInterface interface EldestEntryRemovalFunction<K,V>{
       boolean remove(Map<K,V> map, Map.Entry<K,V> eldest);
}
```

> This interface would work fine, but you shouldn’t use it, because you don’t need to declare a new interface for this purpose. The java.util.function package provides a large collection of standard functional interfaces for your use. **If one of the standard functional interfaces does the job, you should generally use it in preference to a purpose-built functional interface.** This will make your API easier to learn, by reducing its conceptual surface area, and will provide significant interoperability benefits, as many of the standard functional interfaces provide useful default methods. The Predicate interface, for instance, provides methods to combine predicates. In the case of our LinkedHashMap example, the standard BiPredicate<Map<K,V>, Map.Entry<K,V>> interface should be used in preference to a custom EldestEntryRemovalFunction interface.

这个接口可以正常工作，但是你却不需要使用它，因为为了达到这个目的，你不需要声明新的接口。java.util.function包提供了大量的标准函数接口供你使用。**只要标准函数接口中有一个方法完成了这个工作，你就可以使用这个方法，而不是创建一个新的函数接口。**这会使得你的API更容易学习，减少概念理解，同时也提升互操作优势，因为很多的标准函数接口都提供了有用的默认方法。比如，Predicate接口，提供了方法合并断言的方法。在我们的LinkedHashMap例子中，应该使用标准的BiPredicate<Map<K,V>, Map.Entry<K,V>> 接口，而不是使用自己写的EldestEntryRemovalFunction接口。

> There are forty-three interfaces in java.util.Function. You can’t be expected to remember them all, but if you remember six basic interfaces, you can derive the rest when you need them. The basic interfaces operate on object reference types. The Operator interfaces represent functions whose result and argument types are the same. The Predicate interface represents a function that takes an argument and returns a boolean. The Function interface represents a function whose argument and return types differ. The Supplier interface represents a function that takes no arguments and returns (or “supplies”) a value. Finally, Consumer represents a function that takes an argument and returns nothing, essentially consuming its argument. The six basic functional interfaces are summarized below:

在java.util.Function中有43个接口，也不指望你全部记住他们。但是如果你记住了6个基础接口，当你需要其他的接口的时候，就可以推断出来。基础接口类型作用于对象引用类型。Operator接口表示参数和返回结果类型相同的函数。Predicate接口表示有一个参数，返回boolean的函数。Function接口表示参数和返回结果类型不同的函数。Supplier接口表示没有参数，返回（或者说”提供“）一个值的函数。最后，Consumer接口表示有一个参数，没有返回值的函数，相当于消费了它的参数。这6个基础函数接口总结如下：

| 接口              | 函数签名            | 示例                |
| ----------------- | ------------------- | ------------------- |
| UnaryOperator<T>  | T apply(T t)        | String::toLowerCase |
| BinaryOperator<T> | T apply(T t1, T t2) | BigInteger::add     |
| Predicate<T>      | boolean test(T t)   | Collection::isEmpty |
| Function<T,R>     | R apply(T t)        | Arrays::asList      |
| Supplier<T>       | T get()             | Instant::now        |
| Consumer<T>       | void accept(T t)    | System.out::println |

> There are also three variants of each of the six basic interfaces to operate on the primitive types int, long, and double. Their names are derived from the basic interfaces by prefixing them with a primitive type. So, for example, a predicate that takes an int is an IntPredicate, and a binary operator that takes two long values and returns a long is a LongBinaryOperator. None of these variant types is parameterized except for the Function variants, which are parameterized by return type. For example, LongFunction<int[]> takes a long and returns an int[].

这6个基础接口还各有3个变体，可以分别作用域基本类型int，long和double。他们的名字和基础类型的区别在于添加了基本类型作为前缀。比如作用于int 的Predicate就是IntPredicate，以及作用于两个long值，并返回一个long值的BinaryOperator就是LongBinaryOperator。除了Function的变体以外，其他的变体类型都不是参数化的。而Function的返回类型是参数化的，比如LongFunction<int[]>表示参数是long类型的，返回值为int[]类型。

> There are nine additional variants of the Function interface, for use when the result type is primitive. The source and result types always differ, because a function from a type to itself is a UnaryOperator. If both the source and result types are primitive, prefix Function with *Src*To*Result*, for example LongToIntFunction (six variants). If the source is a primitive and the result is an object reference, prefix Function with <Src>ToObj, for example DoubleToObjFunction (three variants).

对于返回类型是基本类型的Function接口，有9个变体。由于从一个类型转换到同样的类型的函数接口就是UnaryOperator，所有Function的源类型和结果类型始终不一致。如果他的源类型和结果类型都是基本类型，就使用*Src*To*Result*作为Function的前缀，比如LongToIntFunction（有6个变体）。如果其源类型是基本类型，而结果类型确实对象引用，就使用<Src>ToObj作为Function的前缀，比如DoubleToObjFunction（有3个变体）。

> There are two-argument versions of the three basic functional interfaces for which it makes sense to have them: BiPredicate<T,U>, BiFunction<T,U,R>, and BiConsumer<T,U>. There are also BiFunction variants returning the three relevant primitive types: ToIntBiFunction<T,U>, ToLongBiFunction<T,U>, and ToDoubleBiFunction<T,U>. There are two-argument variants of Consumer that take one object reference and one primitive type: ObjDoubleConsumer<T>, ObjIntConsumer<T>, and ObjLongConsumer<T>. In total, there are nine two- argument versions of the basic interfaces.

有三个基础接口还有两个参数的版本（两个参数对于他们确实有意义）：BiPredicate<T,U>, BiFunction<T,U,R>, and BiConsumer<T,U>。BiFunction还有3个返回值为基本类型的变体：ToIntBiFunction<T,U>, ToLongBiFunction<T,U>, 和 ToDoubleBiFunction<T,U>。有两个参数的Consumer有三个 以对象引用和基本类型作为参数的变体：ObjDoubleConsumer<T>, ObjIntConsumer<T>, 和ObjLongConsumer<T>。总的来说，一共有9个带两个参数的基本类型的版本。

> Finally, there is the BooleanSupplier interface, a variant of Supplier that returns boolean values. This is the only explicit mention of the boolean type in any of the standard functional interface names, but boolean return values are supported via Predicate and its four variant forms. The BooleanSupplier interface and the forty-two interfaces described in the previous paragraphs account for all forty-three standard functional interfaces. Admittedly, this is a lot to swallow, and not terribly orthogonal. On the other hand, the bulk of the functional interfaces that you’ll need have been written for you and their names are regular enough that you shouldn’t have too much trouble coming up with one when you need it.

最后，有一个BooleanSupplier接口，是Supplier返回boolean值的变体，这是唯一一个在标准函数接口名字里明确提到boolean的情况，其他的返回boolean的情况由Predicate和它的四个变体来支持了。这个BooleanSupplier接口和前面讲到的42个接口一起组成了43个标准函数接口。显然，这是一个大数目，但是并不是纵横交错的。另一方面，大量的函数接口使得你不需要自己写函数接口，他们的名字也很有规律，你在需要的时候，可以很容易找到它们。

> Most of the standard functional interfaces exist only to provide support for primitive types. **Don’t be tempted to use basic functional interfaces with boxed primitives instead of primitive functional interfaces.** While it works, it violates the advice of Item 61, “prefer primitive types to boxed primitives.” The performance consequences of using boxed primitives for bulk operations can be deadly.

大部分已存在的标准函数接口只支持基本类型。**不要使用基本类型封装类的基础函数接口来代替基本函数接口。**虽然，这样可以工作，但是它违反了Item61的建议：“基本那类型优先于封装类”。使用基本类型封装类进行储量处理的时候，会带来致命的性能问题。

> Now you know that you should typically use standard functional interfaces in preference to writing your own. But when *should* you write your own? Of course you need to write your own if none of the standard ones does what you need, for example if you require a predicate that takes three parameters, or one that throws a checked exception. But there are times you should write your own functional interface even when one of the standard ones is structurally identical.

现在你知道了，在通常情况下，你都应该会用标准函数接口，而不是自己写。但是，什么时候应该自己写呢？显然，当你需要的函数接口在标准函数接口中没有的时候，你需要写一个你自己的。比如，你需要一个带3个参数的predicate，或者抛出受检异常的函数接口。但是有的时候，即使有结构相同的标准函数，你还是可以写自己的函数函数接口。

> Consider our old friend Comparator<T>, which is structurally identical to the ToIntBiFunction<T,T> interface. Even if the latter interface had existed when the former was added to the libraries, it would have been wrong to use it. There are several reasons that Comparator deserves its own interface. First, its name provides excellent documentation every time it is used in an API, and it’s used a lot. Second, the Comparator interface has strong requirements on what constitutes a valid instance, which comprise its *general contract*. By implementing the interface, you are pledging to adhere to its contract. Third, the interface is heavily outfitted with useful default methods to transform and combine comparators.

比如我们的老朋友Comparator<T>，它的结构和ToIntBiFunction<T,T>接口一样。即使当Comparator<T>被添加到类库中的时候，ToIntBiFunction<T,T>已经存在了，但是如果用ToIntBiFunction<T,T>的话，就会有点问题。Comparator之所以值得拥有自己的接口，有几个原因。第一是当每次在API中使用的时候，Comparator接口的名字提供了很好的文档说明，而且它用的很多。第二是，Comparator接口，在生成一个合法的对象上，有很多强制要求，也就是通用约定。实现这个接口，就意味着你保证遵守这些约定。第三，这个接口还提供了一整套的用来转换和合并比较器的默认方法。

> You should seriously consider writing a purpose-built functional interface in preference to using a standard one if you need a functional interface that shares one or more of the following characteristics with Comparator:
>
> • It will be commonly used and could benefit from a descriptive name. 
>
> • It has a strong contract associated with it.
>
> • It would benefit from custom default methods.
>
> If you elect to write your own functional interface, remember that it’s an interface and hence should be designed with great care (Item 21).

当你需要一个函数接口的时候，该函数接口和Comparator有一个或几个相同的如下特征的时候，你就应该谨慎地考虑写一个自己的函数接口，而不是用标准函数接口。

- 接口常用，且将受益于描述性的名字。
- 接口与严格的约定相关。
- 接口受益于定制的缺省方法

当你选择要写自己的函数接口的时候，要记住这是一个接口，因此需要特别细心地设计。

> Notice that the EldestEntryRemovalFunction interface (page 199) is labeled with the @FunctionalInterface annotation. This annotation type is similar in spirit to @Override. It is a statement of programmer intent that serves three purposes: it tells readers of the class and its documentation that the interface was designed to enable lambdas; it keeps you honest because the interface won’t compile unless it has exactly one abstract method; and it prevents maintainers from accidentally adding abstract methods to the interface as it evolves. **Always annotate your functional interfaces with the** **@FunctionalInterface** **annotation.**

注意，这个前面的EldestEntryRemovalFunction接口，使用了@FunctionalInterface注解进行标记。这个注解类型本质上和@Override一样。这是一个表明了程序员意图的语句，有三个目的：告诉阅读这个类的人，并在文档说明中表示这个接口是允许lambdas的；只有当接口中只有一个抽象方法的时候，这个接口才能编译；也防止了维护者在迭代时，不小心添加了抽象方法在接口中。**总应该使用@FunctionalInterface注解来标注你的接口。**

> A final point should be made concerning the use of functional interfaces in APIs. Do not provide a method with multiple overloadings that take different functional interfaces in the same argument position if it could create a possible ambiguity in the client. This is not just a theoretical problem. The submit method of ExecutorService can take either a Callable<T> or a Runnable, and it is possible to write a client program that requires a cast to indicate the correct overloading (Item 52). The easiest way to avoid this problem is not to write overloadings that take different functional interfaces in the same argument position. This is a special case of the advice in Item 52, “use overloading judiciously.”

最后一个点是在API中使用函数接口必须小心。不要给一个方法 提供 多个 在同一个参数位置是不同的函数接口的重载，因为在客户端中可能会造成歧义。这不仅仅是一个理论问题，ExecutorService中的submit方法就即可以以Callable<T> 为参数，也可以以Runnable为参数。为了表明正确的重载方法，可能会需要客户端进行一个转换（Item52）。避免这个问题的最简单的方法就是不要写在同一个参数位置有不同函数接口的重载。这是Item52“谨慎使用重载”建议的一个特殊的情况。

> In summary, now that Java has lambdas, it is imperative that you design your APIs with lambdas in mind. Accept functional interface types on input and return them on output. It is generally best to use the standard interfaces provided in java.util.function.Function, but keep your eyes open for the relatively rare cases where you would be better off writing your own functional interface.

总的来说，现在Java有了lambda，在设计API的时候，就需要时刻谨记使用lambda。输入的时候接收函数接口类型，并在输出是返回。一般来说，最好的使用java.util.function.Function里提供的标准函数接口，但是也还是要注意在相对比较少的几种情况下，最好还是自己设计函数接口。