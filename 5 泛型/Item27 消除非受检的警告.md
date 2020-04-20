### Item27 消除非受检的警告

> When you program with generics, you will see many compiler warnings: unchecked cast warnings, unchecked method invocation warnings, unchecked parameterized vararg type warnings, and unchecked conversion warnings. The more experience you acquire with generics, the fewer warnings you’ll get, but don’t expect newly written code to compile cleanly.
>
> Many unchecked warnings are easy to eliminate. For example, suppose you accidentally write this declaration:

当你在使用泛型进行编程的时候，你可能会看到很多的编译器警告：非受检转换警告、非受检方法调用警告、非受检参数化可变参数类型警告、和非受检转变警告。随着你使用泛型经验的增加，代码生生的警告就越少，不要奢望一开始编写泛型就可以干干净净地进行编译。

很多非受检警告都可以很容易地被清除，比如，假如你写了这个声明：

```java
Set<Lark> exaltation = new HashSet();
```

> The compiler will gently remind you what you did wrong:

编译器会细致地提醒你哪里做错了，如下：

```java
Venery.java:4: warning: [unchecked] unchecked conversion
           Set<Lark> exaltation = new HashSet();
                                      ^
     required: Set<Lark>
     found:    HashSet
```

> You can then make the indicated correction, causing the warning to disappear. Note that you don’t actually have to specify the type parameter, merely to indicate that it’s present with the *diamond operator* (<>), introduced in Java 7. The compiler will then *infer* the correct actual type parameter (in this case, Lark):

然后，你就可以根据指示进行修改，以消除这个警告。需要注意的是，你并不需要真正指明参数类型，只需要用Java7中介绍的<>来表示就好了。编译器可以推测出实际正确的类型参数（在这个例子里，就是Lark）。修改如下：

```java
Set<Lark> exaltation = new HashSet<>();
```

> Some warnings will be *much* more difficult to eliminate. This chapter is filled with examples of such warnings. When you get warnings that require some thought, persevere! **Eliminate every unchecked warning that you can.** If you eliminate all warnings, you are assured that your code is typesafe, which is a very good thing. It means that you won’t get a ClassCastException at runtime, and it increases your confidence that your program will behave as you intended.

而另一些警告就很难消除了，本章的例子中就有很多这些警告。当你碰到一些需要深入思考的警告时，一定要坚持！**消除所有你能消除的非受检警告**。如果你把所有的警告都消除了，你就可以保证你的代码是类型安全的，这是一个非常好的事情。这意味着在运行时，不会出现ClassCastException，你也可以更加相信你的程序会表现得和你期望的一致。

> **If you can’t eliminate a warning, but you can prove that the code that provoked the warning is typesafe, then (and only then) suppress the warning with an** **@SuppressWarnings("unchecked")** **annotation.** If you suppress warnings without first proving that the code is typesafe, you are giving yourself a false sense of security. The code may compile without emitting any warnings, but it can still throw a ClassCastException at runtime. If, however, you ignore unchecked warnings that you know to be safe (instead of suppressing them), you won’t notice when a new warning crops up that represents a real problem. The new warning will get lost amidst all the false alarms that you didn’t silence.

如果你实在不能消除一个警告，但你可以证明生成这个警告的代码是类型安全的，（只有在这种情况下）你就可以使用@SuppressWarnings("unchecked")注解来禁止这个警告。如果你并没有证明这段代码是类型安全的，就禁止了警告，你就是给了自己一种错误的安全感。代码在编译的时候不会出现错误，但是在运行时，还是可能抛出ClassCastException。然而，如果你忽略了那些你认为安全的非受检警告，没有禁止他们，当出现其他真正的问题的警告时，你就不会注意到。因为这个新的警告会被淹没在你没有禁止的那些警告中。

> The SuppressWarnings annotation can be used on any declaration, from an individual local variable declaration to an entire class. **Always use the** **SuppressWarnings** **annotation on the smallest scope possible.** Typically this will be a variable declaration or a very short method or constructor. Never use SuppressWarnings on an entire class. Doing so could mask critical warnings.
>
> If you find yourself using the SuppressWarnings annotation on a method or constructor that’s more than one line long, you may be able to move it onto a local variable declaration. You may have to declare a new local variable, but it’s worth it. For example, consider this toArray method, which comes from ArrayList:

SuppressWarnings注解可以被用在所有的声明上，从单独的本地变量声明到整个类声明。**总是在尽可能小的作用域范围内使用SuppressWarnings注解**。通常，SuppressWarnings注解被用在变量声明，或者，是很短的方法或构造器上。永远不要把SuppressWarnings注解用在整个类上。这样做的话，会掩盖掉一些重要的警告。

如果你发现你在一个超过一行的方法或者构造器上使用SuppressWarnings注解，你可能需要把它移到本地变量声明上。你可能必须要声明一个新的本地变量，但这是很值得的。比如，考虑下面这个来自ArrayList的toArray方法：

```java
public <T> T[] toArray(T[] a) {
       if (a.length < size)
			 		return (T[]) Arrays.copyOf(elements, size, a.getClass()); 
  		 System.arraycopy(elements, 0, a, 0, size);
			 if (a.length > size)
          a[size] = null;
       return a;
}
```

> If you compile ArrayList, the method generates this warning:

如果你编译这个ArrayList，这个方法会生成下面这个警告：

```java
ArrayList.java:305: warning: [unchecked] unchecked cast
return (T[]) Arrays.copyOf(elements, size, a.getClass());
                           ^
     required: T[]
     found:    Object[]

```

> It is illegal to put a SuppressWarnings annotation on the return statement, because it isn’t a declaration [JLS, 9.7]. You might be tempted to put the annota- tion on the entire method, but don’t. Instead, declare a local variable to hold the return value and annotate its declaration, like so:

由于返回语句不是一个声明，因此不能在返回语句上方SuppressWarnings注解[JLS, 9.7]。你可能会想把这个注解放在整个方法声明上，不要这么做。而是应该声明一个本地变量持有返回值，然后把注解放在这个声明上，如下：

```java

   // Adding local variable to reduce scope of @SuppressWarnings
   public <T> T[] toArray(T[] a) {
       if (a.length < size) {
						// This cast is correct because the array we're creating 
         		// is of the same type as the one passed in, which is T[]. 
         		@SuppressWarnings("unchecked") T[] result =
              (T[]) Arrays.copyOf(elements, size, a.getClass());
            return result;
       }
       System.arraycopy(elements, 0, a, 0, size);
       if (a.length > size)
           a[size] = null;
       return a;
}
```

> The resulting method compiles cleanly and minimizes the scope in which unchecked warnings are suppressed.

这样得到的方法，在编译时不会生成警告，同时也最小化了非受检警告被禁止的作用域。

> **Every time you use a** **@SuppressWarnings("unchecked")** **annotation, add a comment saying why it is safe to do so.** This will help others understand the code, and more importantly, it will decrease the odds that someone will modify the code so as to make the computation unsafe. If you find it hard to write such a comment, keep thinking. You may end up figuring out that the unchecked operation isn’t safe after all.
>
> In summary, unchecked warnings are important. Don’t ignore them. Every unchecked warning represents the potential for a ClassCastException at runtime. Do your best to eliminate these warnings. If you can’t eliminate an unchecked warning and you can prove that the code that provoked it is typesafe, suppress the warning with a @SuppressWarnings("unchecked") annotation in the narrowest possible scope. Record the rationale for your decision to suppress the warning in a comment.

当你每次用@SuppressWarnings("unchecked")注解的时候，都应该添加一个注解说明这么做为什么是安全的。这样做可以帮助别人理解代码，更重要的是，这样可以减少其他人修改代码后导致计算不安全的可能性。如果你发现要写一个这样的注解很难，就需要好好思考一下了。最终你可能会返现这个未受检操作是不安全的。

总结一下，非受检警告很重要，不要忽略它们。每一个非受检警告都代表着运行时潜在的ClassCastException。尽可能消除这些警告。如果你实在不能消除这个受检警告，并且你能证明这段代码是类型安全的，可以在尽可能小的作用域范围上使用@SuppressWarnings("unchecked") 注解来禁止警告。当你决定禁止警告的时候，必须在注解中说明禁止原因。

