### Item31 使用有限制的通配符来提升API的灵活性

> As noted in Item 28, parameterized types are *invariant*. In other words, for any two distinct types Type1 and Type2, List<Type1> is neither a subtype nor a supertype of List<Type2>. Although it is counterintuitive that List<String> is not a subtype of List<Object>, it really does make sense. You can put any object into a List<Object>, but you can put only strings into a List<String>. Since a List<String> can’t do everything a List<Object> can, it isn’t a subtype (by the Liskov substitution principal, Item 10).

正如Item28里所讲的那样，参数化类型是非协变的。也就是说，对于任何两个不同的类型Type1和Type2。List<Type1>永远都不是List<Type2>的子类型或者父类型。虽然List<String>不是List<Object>的子类型是违反直觉的，但确实是有意义的。你可以往List<Object>里放所有的对象，但是只能往 List<String>里面放String对象。由于List<Stirng>不能做所有List<Object>可以做的事情，所有List<String>就不是List<Object>的子类型（里氏替换原则，Item10)。

> Sometimes you need more flexibility than invariant typing can provide. Consider the Stack class from Item 29. To refresh your memory, here is its public API:

有时，你需要非协变类型不能提供的灵活性，比如Item19中的Stack。为了唤醒你的记忆，下面是Stack的公开API：

```java
public class Stack<E> {
       public Stack();
       public void push(E e);
       public E pop();
       public boolean isEmpty();
}
```

> Suppose we want to add a method that takes a sequence of elements and pushes them all onto the stack. Here’s a first attempt:

假如我们需要添加一个方法，可以把一些列元素都放到stack里。下面是第一个尝试的版本：

```java
// pushAll method without wildcard type - deficient!
   public void pushAll(Iterable<E> src) {
       for (E e : src)
			 push(e);
   }
```

> This method compiles cleanly, but it isn’t entirely satisfactory. If the element type of the Iterable src exactly matches that of the stack, it works fine. But suppose you have a Stack<Number> and you invoke push(intVal), where intVal is of type Integer. This works because Integer is a subtype of Number. So logically, it seems that this should work, too:

这个方法可以干干净净地编译，但是还不是完全让人满意的。只有当Iterator实例src的元素类型恰好和stack的元素类型一致的时候，才能正常工作。但是，假设你现在有一个Stack<Number> ，并且你调用了push(intVal)方法，而intVal的类型是Integer。这个方法可以正常工作，因为Integer是Number的子类型。所以逻辑上来说，下面这段代码也应该可以工作。

```java
Stack<Number> numberStack = new Stack<>();
Iterable<Integer> integers = ... ;
numberStack.pushAll(integers);
```

> If you try it, however, you’ll get this error message because parameterized types are invariant:

然而，当你这个做的时候，由于参数化类型是非协变的，你将得到一个error信息如下。

```java
StackTest.java:7: error: incompatible types: Iterable<Integer>
   cannot be converted to Iterable<Number>
           numberStack.pushAll(integers);
                               ^
```

> Luckily, there’s a way out. The language provides a special kind of parameterized type call a *bounded wildcard type* to deal with situations like this. The type of the input parameter to pushAll should not be “Iterable of E” but “Iterable of some subtype of E,” and there is a wildcard type that means precisely that: Iterable<? extends E>. (The use of the keyword extends is slightly misleading: recall from Item 29 that *subtype* is defined so that every type is a subtype of itself, even though it does not extend itself.) Let’s modify pushAll to use this type:

幸运的是，还有一种方法。Java语言提供了一种特殊的参数化类型，被称为”有限制通配符类型“，就是用来解决类似这种问题的。pushAll方法的参数类型不应该是”E的Iterable接口“，而应该是“E的某个子类型的Iterable接口”，然后，这里有一个通配符类型，可以准确的表示这个意思： Iterable<? extends E>（其中extends关键字可能会造成一些误导，重申一下Item29里子类型的定义，每个类型都是自身的子类型，即使它没有继承它自己）。下面是使用这种类型修改后的pushAll方法：

```java
// Wildcard type for a parameter that serves as an E producer 
public void pushAll(Iterable<? extends E> src) {
       for (E e : src)
           push(e);
}
```

> With this change, not only does Stack compile cleanly, but so does the client code that wouldn’t compile with the original pushAll declaration. Because Stack and its client compile cleanly, you know that everything is typesafe.
>
> Now suppose you want to write a popAll method to go with pushAll. The popAll method pops each element off the stack and adds the elements to the given collection. Here’s how a first attempt at writing the popAll method might look:

通过这个修改后，不仅仅Stack可以干净地编译，那些没有和原始pushAll声明一起编译过的客户端代码也能干干净净地编译。由于Stack和客户端代码都可以干净地编译，你就知道所有的东西都是类型安全的。

现在假设你要写一个pushAll对应的方法popAll。popAll方法把stack中的每一个元素都弹出来，然后添加到一个给定的集合里。下面是第一次尝试写得popAll方法的代码：

```java
// popAll method without wildcard type - deficient!
   public void popAll(Collection<E> dst) {
       while (!isEmpty())
           dst.add(pop());
}
```

> Again, this compiles cleanly and works fine if the element type of the destination collection exactly matches that of the stack. But again, it isn’t entirely satisfactory. Suppose you have a Stack<Number> and variable of type Object. If you pop an element from the stack and store it in the variable, it compiles and runs without error. So shouldn’t you be able to do this, too?

同样地，这个代码也能干净地编译，当目标集合的元素类型和stack的一致的时候，也能正常工作。但是同样地，这并不能让人满意。假如你有一个Stack<Number>和一个类型为Object的变量。当你把一个元素从stack中弹出来并存储到这变量中时，可以正常编译运行，不会有error。那么，你为什么不能像下面这么做呢？

```java
Stack<Number> numberStack = new Stack<Number>();
Collection<Object> objects = ... ;
numberStack.popAll(objects);
```

> If you try to compile this client code against the version of popAll shown earlier, you’ll get an error very similar to the one that we got with our first version of pushAll: Collection<Object> is not a subtype of Collection<Number>. Once again, wildcard types provide a way out. The type of the input parameter to popAll should not be “collection of E” but “collection of some supertype of E” (where supertype is defined such that E is a supertype of itself [JLS, 4.10]). Again, there is a wildcard type that means precisely that: Collection<? super E>. Let’s modify popAll to use it:

当你企图基于前面的popAll版本来编译这个客户端代码的时候，你将会得到一个error，和前面写的第一版的pushAll方法差不多：Collection<Object> 不是Collection<Number>的子类型。还是同样地，通配符类型也提供了一个方法。popAll方法的输入参数类型不应该是“E的集合”应该是“E的一些超类型的集合”（这里的超类型定义是正确的，因为E是它自己的超类。同样地，也有一个通配符类型能准确表达这个意思：Collection<? super E>。下面是使用它修改的popAll方法：

```java
// Wildcard type for parameter that serves as an E consumer 
public void popAll(Collection<? super E> dst) {
       while (!isEmpty())
           dst.add(pop());
}
```

> With this change, both Stack and the client code compile cleanly.
>  The lesson is clear. **For maximum flexibility, use wildcard types on input parameters that represent producers or consumers.** If an input parameter is both a producer and a consumer, then wildcard types will do you no good: you need an exact type match, which is what you get without any wildcards.

使用这个修改后的版本，Stack和客户端代码都可以干干净净地编译。

这个结论很明显：为了最大化灵活度，对于代表生产者或者消费者的输入参数，使用通配符类型。如果这个输入参数即是生产者又是消费者，这个通配符类型就没什么用了。你需要的是准确的类型匹配，不需要使用任何的通配符。

> Here is a mnemonic to help you remember which wildcard type to use:
>
> **PECS stands for producer-extends, consumer-super.**
>
> In other words, if a parameterized type represents a T producer, use <? extends T>; if it represents a T consumer, use <? super T>. In our Stack example, pushAll’s src parameter produces E instances for use by the Stack, so the appropriate type for src is Iterable<? extends E>; popAll’s dst parameter consumes E instances from the Stack, so the appropriate type for dst is Collection<? super E>. The PECS mnemonic captures the fundamental principle that guides the use of wildcard types. Naftalin and Wadler call it the *Get and Put Principle* [Naftalin07, 2.4].

这个助记符可以帮助你记住应该使用哪些通配符：

**PECS 表示producer-extends, consumer-super。**

换句话说，如果这个参数化类型表示一个T生产者，使用<? extends T>；如果表示的是T消费者，使用 <? super T>。在我们的Stack的例子里，pushAll的src参数产生了E实例供Stack使用；所以src的合适类型是Iterable<? extends T>；popAll的dst参数消费每一个来自Stack的E实例，因此合适的类型是Collection<? super E>。PECS助记符刻画了直到通配符使用的基本原则。Naftalin和Wadler把它称之为“Get and Put Principle”。

> With this mnemonic in mind, let’s take a look at some method and constructor declarations from previous items in this chapter. The Chooser constructor in Item 28 has this declaration:

记住这个助记符，然后我们看一下本章中前面的章节中写的一些方法和构造器声明。Item28中的Chooser构造器有如下的声明：

```java
public Chooser(Collection<T> choices)
```

> This constructor uses the collection choices only to **produce** values of type T (and stores them for later use), so its declaration should use a wildcard type that **extends** T. Here’s the resulting constructor declaration:

这个构造器使用的集合choices只是用来生产类型为T的值（然后保存起来为了后面的使用），因此它的声明应该使用extends T这一通配符。这是修改后的构造器声明：

```java
// Wildcard type for parameter that serves as an T producer 
public Chooser(Collection<? extends T> choices)
```

> And would this change make any difference in practice? Yes, it would. Suppose you have a List<Integer>, and you want to pass it in to the constructor for a Chooser<Number>. This would not compile with the original declaration, but it does once you add the bounded wildcard type to the declaration.

这个会在实际应用中会有区别吗？答案是肯定的。假如你有一个List<Integer>，然后你想把它传给一个Chooser<Number>的构造器（*我想应该是这样 Chooser<Number> chooser= new Chooser(listInteger)*)。你使用原来的声明，将无法编译，但是如果你在声明中使用有限制的通配符类型，就可以进行编译了。

> Now let’s look at the union method from Item 30. Here is the declaration:

现在让我们来看看Item30中的union方法，下面是声明：

```java
public static <E> Set<E> union(Set<E> s1, Set<E> s2)
```

> Both parameters, s1 and s2, are E producers, so the PECS mnemonic tells us that the declaration should be as follows:

这两个参数s1和s2都是E的生产者，因此PECS助记符告诉我们这个声明应该是这样的：

```java
public static <E> Set<E> union(Set<? extends E> s1, Set<? extends E> s2)
```

> Note that the return type is still Set<E>. **Do not use bounded wildcard types as return types.** Rather than providing additional flexibility for your users, it would force them to use wildcard types in client code. With the revised declaration, this code will compile cleanly:

需要注意的是，这个返回类型还是Set<E>。**不要使用有限制的通配符类型作为返回类型。**这样做既不能给用户提供额外的灵活性，还会强迫他们在客户端代码中使用通配符类型。使用前面修改后的声明，下面这个代码可以很干净地编译：

```java
Set<Integer> integers = Set.of(1, 3, 5);
Set<Double>  doubles  = Set.of(2.0, 4.0, 6.0);
Set<Number>  numbers  = union(integers, doubles);
```

> Properly used, wildcard types are nearly invisible to the users of a class. They cause methods to accept the parameters they should accept and reject those they should reject. **If the user of a class has to think about wildcard types, there is probably something wrong with its API.**

如果使用得到，这个通配符类型对于类的使用者来说几乎是不可见的。他们可以让方法接收他们应该接收的参数，并拒绝他们应该拒绝的参数。**如果类的使用者必须要考虑通配符类型，那么这个API可能就有点问题。**

> Prior to Java 8, the type inference rules were not clever enough to handle the previous code fragment, which requires the compiler to use the contextually specified return type (or *target type*) to infer the type of E. The target type of the union invocation shown earlier is Set<Number>. If you try to compile the fragment in an earlier version of Java (with an appropriate replacement for the Set.of factory), you’ll get a long, convoluted error message like this:

























