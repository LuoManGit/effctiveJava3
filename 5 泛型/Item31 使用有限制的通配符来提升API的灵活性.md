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

在Java8之前的版本中，类型推导规则还没有聪明到可以处理前面的那段代码，还需要编译器使用上下文指定的返回类型（或者目标类型）来推导出E的类型。前面的union方法调用的目标类型是Set<Number>。当你在早期的Java版本里编译这段代码的时候（也使用Set.of工厂方法对应的替代方法），你将得到一个很长的，错综复杂的error信息如下：

```java
Union.java:14: error: incompatible types
           Set<Number> numbers = union(integers, doubles);
																	^
     required: Set<Number>
     found:    Set<INT#1>
     where INT#1,INT#2 are intersection types:
       INT#1 extends Number,Comparable<? extends INT#2>
       INT#2 extends Number,Comparable<?>
```

> Luckily there is a way to deal with this sort of error. If the compiler doesn’t infer the correct type, you can always tell it what type to use with an *explicit type* *argument* [JLS, 15.12]. Even prior to the introduction of target typing in Java 8, this isn’t something that you had to do often, which is good because explicit type arguments aren’t very pretty. With the addition of an explicit type argument, as shown here, the code fragment compiles cleanly in versions prior to Java 8:

还好，还是有办法来解决这种错误。如果编译器不能推导出正确的类型，你就可以使用显示的类型参数来告诉它应该使用什么样的类型[JLS, 15.12]。即使在Java8引入目标类型之前，这种操作也并不是需要经常做，这是一件好事，因为显式的类型参数很丑。添加了显式的类型参数后，代码如下，这段代码就可以在Java8之前的版本中，干净的编译了。

```java
// Explicit type parameter - required prior to Java 8 
Set<Number> numbers = Union.<Number>union(integers, doubles);
```

> Next let’s turn our attention to the max method in Item 30. Here is the original declaration:

接下来，让我们把注意力放在Item30里的max方法里，下面是它原先的声明：

```java
public static <T extends Comparable<T>> T max(List<T> list)
```

> Here is a revised declaration that uses wildcard types:

下面是其使用通配符类型修改后的声明：

```java
public static <T extends Comparable<? super T>> T max( List<? extends T> list)
```

> To get the revised declaration from the original, we applied the PECS heuristic twice. The straightforward application is to the parameter list. It produces T instances, so we change the type from List<T> to List<? extends T>. The tricky application is to the type parameter T. This is the first time we’ve seen a wildcard applied to a type parameter. Originally, T was specified to extend Comparable<T>, but a comparable of T consumes T instances (and produces integers indicating order relations). Therefore, the parameterized type Comparable<T> is replaced by the bounded wildcard type Comparable<? super T>. Comparables are always consumers, so you should generally **use** **Comparable <? super T> in preference to** **Comparable<T>.** The same is true of comparators; therefore, you should generally **use** **Comparator<? super T>** **in preference to** **Comparator<T>.**

为了将原先的声明修改到现在的版本，我们需要将PECS原则应用两次。对参数list的应用比较简单，它生产了T实例，因此我们将List<T>改为List<? extends T>。对类型参数T的应用比较复杂了，这是我们第一次看将通配符引用到类型参数上。一开始，T是用来扩展Comparable<T>的，但是T的comparable消费了T实例（生成可以表示顺序关系的integer）。因此类型化参数Comparable<T>被替换成了有限制的通配符类型Comparable<? super T>。Comparable都是消费者，所以通常情况下，**都应该优先使用Comparable <? super T>而不是Comparable<T>**。对于Comparator也一样，所以通常情况下，**都应该优先使用Comparator <? super T>而不是Comparator<T>**。

> The revised max declaration is probably the most complex method declaration in this book. Does the added complexity really buy you anything? Again, it does. Here is a simple example of a list that would be excluded by the original declaration but is permitted by the revised one:

这个修改后的max声明大概是本书中最复杂的方法声明了。那么增加的复杂度真的能起到什么作用吗？是的，是有用的。下面是一个list的简单例子，自原始的方法声明中无法使用，但是在修改后的方法中却是允许的。

```java
List<ScheduledFuture<?>> scheduledFutures = ... ;
```

> The reason that you can’t apply the original method declaration to this list is that ScheduledFuture does not implement Comparable<ScheduledFuture>. Instead, it is a subinterface of Delayed, which extends Comparable<Delayed>. In other words, a ScheduledFuture instance isn’t merely comparable to other ScheduledFuture instances; it is comparable to any Delayed instance, and that’s enough to cause the original declaration to reject it. More generally, the wildcard is required to support types that do not implement Comparable (or Comparator) directly but extend a type that does.

不能在原先的声明中使用这个list的原因是，ScheduledFuture并没有实现Comparable<ScheduledFuture>，而ScheduledFuture是Delayed的子接口，而Delayed实现了Comparable<Delayed>。换句话说，ScheduledFuture不仅仅可以和其他的ScheduledFuture实例比较，还可以和任何一个Delayed实例比较，这些就足够使得原始的声明会拒绝它了。更通俗地说，通配符可以用来支持这种类型，它没有直接实现Comparable（或者Comparator），而是继承一个实现了Comparable（或者Comparator）的类型。

> There is one more wildcard-related topic that bears discussing. There is a duality between type parameters and wildcards, and many methods can be declared using one or the other. For example, here are two possible declarations for a static method to swap two indexed items in a list. The first uses an unbounded type parameter (Item 30) and the second an unbounded wildcard:

还有一个和通配符相关的话题需要讨论。类型参数和通配符之间具有双重性，很多方法，都使用其中一个或者另一个进行声明。比如，下面这个用来交换list中两个被索引元素的静态方法，有两种可能的声明如下：第一个使用的是无限制的类型参数（Item30），而第二个使用的是无限制的通配符。

```java
// Two possible declarations for the swap method
public static <E> void swap(List<E> list, int i, int j); 
public static void swap(List<?> list, int i, int j);
```

> Which of these two declarations is preferable, and why? In a public API, the second is better because it’s simpler. You pass in a list—any list—and the method swaps the indexed elements. There is no type parameter to worry about. As a rule, **if a type parameter appears only once in a method declaration, replace it with a wildcard.** If it’s an unbounded type parameter, replace it with an unbounded wildcard; if it’s a bounded type parameter, replace it with a bounded wildcard.

这两种声明哪一个更好呢？为什么呢？在一个公开的API中，第二个要好一些，因为要简单一些。你可以直接传一个List，任意一个List，这个方法就会交换这指定索引位置上的元素。完全不需要去考虑类型参数。有一个规则，**如果类型参数只在方法的声明中出现了一次，就可以用通配符来替代它**。如果是一个无限制的类型参数，就用无限制的通配符，如果是一个有限制的类型参数，就用有限制的通配符。

> There’s one problem with the second declaration for swap. The straightforward implementation won’t compile:

在使用第二种方法声明进行交换的时候，有一个问题。下面这种直接的实现无法编译：

```java
public static void swap(List<?> list, int i, int j) {
       list.set(i, list.set(j, list.get(i)));
}
```

> Trying to compile it produces this less-than-helpful error message:

当你编译的时候，会生成这样一个没什么用的错误信息：

```java
Swap.java:5: error: incompatible types: Object cannot be
   converted to CAP#1
           list.set(i, list.set(j, list.get(i)));
                                           ^
     where CAP#1 is a fresh type-variable:
       CAP#1 extends Object from capture of ?
```

> It doesn’t seem right that we can’t put an element back into the list that we just took it out of. The problem is that the type of list is List<?>, and you can’t put any value except null into a List< ？>. Fortunately, there is a way to implement this method without resorting to an unsafe cast or a raw type. The idea is to write a private helper method to *capture* the wildcard type. The helper method must be a generic method in order to capture the type. Here’s how it looks:

我们无法把一个从列表里拿出来的元素再放回去，这看起来有些不正常。这里的问题在于这个列表的类型是List<?>，不能往List< ？>里放除了null以外的任何值。幸运的是，在不需要不安全的转换和原生类型的前提下，还是有方法实现这个方法。最好的方法是写一个私有的辅助方法来捕获通配符类型。为了捕获类型，这个辅助方法必须是泛型方法。下面是其代码：

```java
public static void swap(List<?> list, int i, int j) {
       swapHelper(list, i, j);
}
// Private helper method for wildcard capture
private static <E> void swapHelper(List<E> list, int i, int j) { 
			 list.set(i, list.set(j, list.get(i)));
}
```

> The swapHelper method knows that list is a List<E>. Therefore, it knows that any value it gets out of this list is of type E and that it’s safe to put any value of type E into the list. This slightly convoluted implementation of swap compiles cleanly. It allows us to export the nice wildcard-based declaration, while taking advantage of the more complex generic method internally. Clients of the swap method don’t have to confront the more complex swapHelper declaration, but they do benefit from it. It is worth noting that the helper method has precisely the signature that we dismissed as too complex for the public method.

这个swapHepler方法就知道这个列表是List<E>了。因此它知道从这个列表中拿出来的类型都是E，可以安全地放回到这个List里去。这个有点复杂的swap实现可以干干净净地编译。它使得我们可以导出一个基于通配符的好的声明，并在其内部使用了比较复杂的泛型方法。swap方法的客户端并不需要面对这个复杂的swapHelper声明，就可以从中受益。值得一提的是，这个辅助方法的精确的方法签名，正是我们因为太复杂而在公有方法中所放弃的签名。

> In summary, using wildcard types in your APIs, while tricky, makes the APIs far more flexible. If you write a library that will be widely used, the proper use of wildcard types should be considered mandatory. Remember the basic rule: producer-extends, consumer-super (PECS). Also remember that all comparables and comparators are consumers.

总结一下，在API中使用通配符，虽然有点复杂，但是会让API变得很灵活。如果你在写一个需要广泛使用的类库，就必须要使用合适的通配符类型。记住这个基本规则：producer-extends，comsumer-super（PECS)。同时也记住所有的comparable和comparator都是消费者。

















