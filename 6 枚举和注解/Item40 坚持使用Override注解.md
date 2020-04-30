### Item40 坚持使用Override注解

> The Java libraries contain several annotation types. For the typical programmer, the most important of these is @Override. This annotation can be used only on method declarations, and it indicates that the annotated method declaration overrides a declaration in a supertype. If you consistently use this annotation, it will protect you from a large class of nefarious bugs. Consider this program, in which the class Bigram represents a *bigram*, or ordered pair of letters:

在Java类库中有几种主机类型。对于传统的程序员，这些注解里最重要的就是@Override。这个注解只能用在方法声明上，它表示被注解的方法覆盖了父类中的一个方法声明。如果你一直使用这个注解，就会保护你远离一大类的恶毒bugs。比如下面这个程序，类Bigram表示一个双字母，或者有序的字母对。代码如下：

```java
// Can you spot the bug?
   public class Bigram {
       private final char first;
       private final char second;
       public Bigram(char first, char second) {
           this.first  = first;
           this.second = second;
       }
       public boolean equals(Bigram b) {
           return b.first == first && b.second == second;
       }
       public int hashCode() {
           return 31 * first + second;
			 }
       public static void main(String[] args) {
           Set<Bigram> s = new HashSet<>();
           for (int i = 0; i < 10; i++)
               for (char ch = 'a'; ch <= 'z'; ch++)
                   s.add(new Bigram(ch, ch));
           System.out.println(s.size());
       }
}
```

> The main program repeatedly adds twenty-six bigrams, each consisting of two identical lowercase letters, to a set. Then it prints the size of the set. You might expect the program to print 26, as sets cannot contain duplicates. If you try running the program, you’ll find that it prints not 26 but 260. What is wrong with it?

这个main程序重复地添加了26个字母对（每个字母对包含两个相同的小写字母）到set里。然后打印了数组的大小，你可能会希望程序打印的是26，因为set不能包含相同的值。但是当你运行这个程序的时候，你会发现打印的是260而不是26，问题出在哪里？

> Clearly, the author of the Bigram class intended to override the equals method (Item 10) and even remembered to override hashCode in tandem (Item 11). Unfortunately, our hapless programmer failed to override equals, overloading it instead (Item 52). To override Object.equals, you must define an equals method whose parameter is of type Object, but the parameter of Bigram’s equals method is not of type Object, so Bigram inherits the equals method from Object. This equals method tests for object *identity*, just like the == operator. Each of the ten copies of each bigram is distinct from the other nine, so they are deemed unequal by Object.equals, which explains why the program prints 260.

很明显这个Bigram类的作者是想覆盖equals方法（Item10），而且还记得随后覆盖了hashCode方法（Item11）。不幸的是，这个运气不好的程序员并没有覆盖equals，而是重载了它。要覆盖Object.equals方法，你必须定义一个参数类型为Object的equals方法，但是Bigramlei 的equals犯法的参数却不是Object类型的，因此Bigram从Object继承了equals方法。这个equals方法用于测试对象是否相等的方法，就和==操作一样。每一个bigram实例的10个复制品都互相不同，因为Object.equals方法会认为他们是不相等的。也就解释了为什么程序会打印260.

> Luckily, the compiler can help you find this error, but only if you help it by telling it that you intend to override Object.equals. To do this, annotate Bigram.equals with @Override, as shown here:

幸运地是，当你告诉它你是想覆盖Object.equals方法时，编译器会帮助你发现这个错误。为了做到这点，需要使用@Override来对Bigram.equals方法进行注解，代码如下：

```java
@Override public boolean equals(Bigram b) {
       return b.first == first && b.second == second;
}
```

> If you insert this annotation and try to recompile the program, the compiler will generate an error message like this:

当你插入这样一条注解后，再重新编译程序的时候，编译器会生成一条错误信息如下：

```java
Bigram.java:10: method does not override or implement a method
   from a supertype
       @Override public boolean equals(Bigram b) {
       ^
```

> You will immediately realize what you did wrong, slap yourself on the forehead, and replace the broken equals implementation with a correct one (Item 10):

你很快就会意识到是哪里搞错了，打一下你的头，然后把错误的equals实现换成正确的（Item10）。正确代码如下：

```java
@Override public boolean equals(Object o) { 
  	if (!(o instanceof Bigram))
				return false;
		Bigram b = (Bigram) o;
		return b.first == first && b.second == second;
}
```

> Therefore, you should **use the** **Override** **annotation on every method declaration that you believe to override a superclass declaration.** There is one minor exception to this rule. If you are writing a class that is not labeled abstract and you believe that it overrides an abstract method in its superclass, you needn’t bother putting the Override annotation on that method. In a class that is not declared abstract, the compiler will emit an error message if you fail to override an abstract superclass method. However, you might wish to draw attention to all of the methods in your class that override superclass methods, in which case you should feel free to annotate these methods too. Most IDEs can be set to insert Override annotations automatically when you elect to override a method.

因此你应该在**每一个打算用来覆盖超类声明的方法声明上使用Override注解。**这个规则有一点小小的例外。如果你要写一个非抽象类，并且确信它覆盖了超类中的抽象方法，你就不需要再这个方法上放一个Override注解了。在一个非抽象类中，如果你没有正确覆盖超类中的抽闲方法，编译器会生成一个错误信息。然而，你可能会想把注意力放在你的类里所有覆盖了超类的方法上，在这种情况下，你也可以放心的标注这些方法。当你选择覆盖一个方法的时候，大部分的IDE会自动地插入Override注解。

> Most IDEs provide another reason to use the Override annotation consistently. If you enable the appropriate check, the IDE will generate a warning if you have a method that doesn’t have an Override annotation but does override a superclass method. If you use the Override annotation consistently, these warnings will alert you to unintentional overriding. They complement the compiler’s error messages, which alert you to unintentional failure to override. Between the IDE and the compiler, you can be sure that you’re overriding methods everywhere you want to and nowhere else.

大部分的IDE都提供了另外一种使用Override注解的理由。如果你启用了相应的代码检查功能，当你的方法确实覆盖了超类的方法，但是又没有使用Override注解的时候，IDE会生成一个警告。如果你始终使用Override注解，这些警告就会提醒你当心无意识的覆盖。这是编译器的错误信息（提醒你当心无意识地覆盖失败）的一种补充。在IDE和编译器的帮助下，你可以确定你覆盖了所有想覆盖的方法，无一遗漏。

> The Override annotation may be used on method declarations that override declarations from interfaces as well as classes*.* With the advent of default methods, it is good practice to use Override on concrete implementations of interface methods to ensure that the signature is correct. If you know that an interface does not have default methods, you may choose to omit Override annotations on concrete implementations of interface methods to reduce clutter.

Override注解可以用在覆盖了接口和方法的方法声明上。随着默认方法的出现，在接口方法的具体实现上使用Override注解是一个很好的实践，可以保证方法签名是正确的。如果你知道这个方法没有默认方法，你可以选择在接口方法的具体实现上省略Override注解，以减少混乱。

> In an abstract class or an interface, however, it *is* worth annotating *all* methods that you believe to override superclass or superinterface methods, whether concrete or abstract. For example, the Set interface adds no new methods to the Collection interface, so it should include Override annotations on all of its method declarations to ensure that it does not accidentally add any new methods to the Collection interface.

然而，在抽象类和接口中，还是很有必要标注所有你想覆盖的超类或者超接口的方法，不管他们是抽象的还是具体的。比如，Set接口没有往Collection接口中添加任何的方法，因此，它应该在所有的方法声明上添加Override注解，以保证它没有不小心往Collection接口中添加任何新的方法。

> In summary, the compiler can protect you from a great many errors if you use the Override annotation on every method declaration that you believe to override a supertype declaration, with one exception. In concrete classes, you need not annotate methods that you believe to override abstract method declarations (though it is not harmful to do so).

总结一下，如果你在每一个用来覆盖超类声明的方法声明上使用Override注解，编译器会替你防止大量的错误。这个规则有一个小小的例外，在具体类中，你不需要给那些用来覆盖抽象方法声明的方法添加注解（虽然这么做了也没什么坏处）。