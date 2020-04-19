### Item20 接口优先于抽象类

> Java has two mechanisms to define a type that permits multiple implementations: interfaces and abstract classes. Since the introduction of *default methods* for interfaces in Java 8 [JLS 9.4.3], both mechanisms allow you to provide implementations for some instance methods. A major difference is that to implement the type defined by an abstract class, a class must be a subclass of the abstract class. Because Java permits only single inheritance, this restriction on abstract classes severely constrains their use as type definitions. Any class that defines all the required methods and obeys the general contract is permitted to implement an interface, regardless of where the class resides in the class hierarchy.

java里有两个机制允许一个类型有多种实现：接口和抽象类。自从在Java8 在接口里添加了默认方法后，这两种机制都可以为一些实例方法提供实现了。他们之间的主要的区别在于，实现抽象类定义的类型的类必须是这个抽象类的子类。因为Java只允许单个继承，这个关于抽象类的限制，严重限制了抽象类作为类型定义的使用。而对于接口而言，只要是定义了所有需要的方法，同时遵守通用公约，不管这个类处在类层次结构中的什么位置，这个类都可以实现接口。

> **Existing classes can easily be retrofitted to implement a new interface.** All you have to do is to add the required methods, if they don’t yet exist, and to add an implements clause to the class declaration. For example, many existing classes were retrofitted to implement the Comparable, Iterable, and Autocloseable interfaces when they were added to the platform. Existing classes cannot, in general, be retrofitted to extend a new abstract class. If you want to have two classes extend the same abstract class, you have to place it high up in the type hierarchy where it is an ancestor of both classes. Unfortunately, this can cause great collateral damage to the type hierarchy, forcing all descendants of the new abstract class to subclass it, whether or not it is appropriate.

**已经存在的类很容易通过修改来实现一个新的接口**。所有你需要做的事情就是，添加那些之前不存在的必要的方法，然后在类声明语句上添加implements语句就可以了。比如，很多类在添加到平台中的时候，都需要修改来实现Comparable、Iterable、AutoCloseable接口。而通常情况下，已经存在的类不能通过修改来继承一个新的抽象类。如果你想让两个类继承相同的抽象类，你必须把这个抽象类在类型层次结构中的位置提高，让它成为这两个类的祖先类。遗憾地是，这样做会给类层次结构带来附加的伤害，强迫这个公共祖先的所有后代都必须要继承这个抽象类，不论它们是否合适。

> **Interfaces are ideal for defining mixins.** Loosely speaking, a *mixin* is a type that a class can implement in addition to its “primary type,” to declare that it provides some optional behavior. For example, Comparable is a mixin interface that allows a class to declare that its instances are ordered with respect to other mutually comparable objects. Such an interface is called a mixin because it allows the optional functionality to be “mixed in” to the type’s primary functionality. Abstract classes can’t be used to define mixins for the same reason that they can’t be retrofitted onto existing classes: a class cannot have more than one parent, and there is no reasonable place in the class hierarchy to insert a mixin.

接口是定义mixin的理想选择。不严格地讲，一个mixin是指：一个类除了实现自己的“原始类型”外，还可以实现这个类型，来声明自己提供了一些可选择的行为。比如，Comparable接口就是一个mixin接口，它用来表明这个类的实例可以和其他的可以互相比较的类进行排序。这样的接口被称为mixin是因为它允许将一些可选择的功能“混入（mixed in）"到类的原始功能里。而抽象类就不能用来定义mixin，其原因和前面相同，抽象类不能直接加装到已经存在的类上面：因为一个类只能拥有一个父类，并且在类层次结构里也没有合理的位置来插入mixin类型。

> **Interfaces allow for the construction of nonhierarchical type frameworks.**Type hierarchies are great for organizing some things, but other things don’t fall neatly into a rigid hierarchy. For example, suppose we have an interface representing a singer and another representing a songwriter:

**接口可以构建非层次结构的类型框架**。类的层次结构可以很好的管理一些类，但是有一些类就是不能整齐地落在这个严格的层次结构里。比如，假设我们有一个接口便是一个歌手（singer），另一个接口表示一个作曲家（songwriter）。如下：

```java
public interface Singer {
       AudioClip sing(Song s);
}
```

```java
public interface Songwriter {
       Song compose(int chartPosition);
}
```

> In real life, some singers are also songwriters. Because we used interfaces rather than abstract classes to define these types, it is perfectly permissible for a single class to implement both Singer and Songwriter. In fact, we can define a third interface that extends both Singer and Songwriter and adds new methods that are appropriate to the combination:

在现实中，有些歌手也是作曲家。因为我们使用了接口而不是抽象类来定义这些类型，所以可以完美的让一个类同时实现Singer和Songwriter接口。事实上，我们还可以定义第三个接口继承Singer和SongWriter接口，并添加一些适合这个结合体的新的方法，如下：

```java
public interface SingerSongwriter extends Singer, Songwriter {
       AudioClip strum();
       void actSensitive();
}
```

> You don’t always need this level of flexibility, but when you do, interfaces are a lifesaver. The alternative is a bloated class hierarchy containing a separate class for every supported combination of attributes. If there are *n* attributes in the type system, there are 2^*n* possible combinations that you might have to support. This is what’s known as a *combinatorial explosion*. Bloated class hierarchies can lead to bloated classes with many methods that differ only in the type of their arguments because there are no types in the class hierarchy to capture common behaviors.

我们并不总是需要这种灵活性，但是一旦这么做了，接口就会如同救世主一般。接口的另一个替代方法是，为每一种需要支持的属性的组合编写一个单独的类，这样就会组成一个臃肿的类层次。如果在类型系统中有n个属性，然后就会有2^n种可能需要支持的组合。这也被称为”组合爆炸“。臃肿的类层次可能导致类也变得臃肿，这些类里有很多方法只有参数的类型不同，因为类层次结构中没有能客户公共行为的类型。

> **Interfaces enable safe, powerful functionality enhancements** via the *wrapper class* idiom (Item 18). If you use abstract classes to define types, you leave the programmer who wants to add functionality with no alternative but inheritance. The resulting classes are less powerful and more fragile than wrapper classes.

**接口可以通过包装类模式来实现安全、强大的功能增强**。如果你使用的抽象类来定义类型，那么想要添加新的功能的程序员处理继承以外，别无选择。这样得到的类和包装类相比，功能不够强大，也更加脆弱。

> When there is an obvious implementation of an interface method in terms of other interface methods, consider providing implementation assistance to programmers in the form of a default method. For an example of this technique, see the removeIf method on page 104. If you provide default methods, be sure to document them for inheritance using the @implSpec Javadoc tag (Item 19).

当一个接口方法根据其他的接口方法有了明显的实现的时候，就可以考虑以默认方法的形式，给程序员提供一些实现帮助。这个技术的实例详见Item21中的removeIf方法。如果你提供了默认方法，请确定使用过javadoc标签来为继承提供文档说明(Item19)。

> There are limits on how much implementation assistance you can provide with default methods. Although many interfaces specify the behavior of Object methods such as equals and hashCode, you are not permitted to provide default methods for them. Also, interfaces are not permitted to contain instance fields or nonpublic static members (with the exception of private static methods). Finally, you can’t add default methods to an interface that you don’t control.

你可以通过默认方法提供的实现帮助是有一定限制的。虽然很多接口都明确了Object的方法比如equals、hashCode的行为，但是还是不准为这些方法提供默认实现。同时，接口也不在允许包含实例域或者非公有静态成员（私有静态方法除外）。最后，你也不能给不受你控制的接口添加默认方法。

> You can, however, combine the advantages of interfaces and abstract classes by providing an abstract *skeletal implementation class* to go with an interface. The interface defines the type, perhaps providing some default methods, while the skeletal implementation class implements the remaining non-primitive interface methods atop the primitive interface methods. Extending a skeletal implementa- tion takes most of the work out of implementing an interface. This is the *Template Method* pattern [Gamma95].

但是，你可以通过对接口实现一个抽象的骨架实现类（skeletal implementation class）来综合抽象类和接口的优点。接口定义了类型，还可能提供了一些默认方法。骨架实现类实现了除了基本接口方法外的其他的非基本接口方法。扩展骨架实现做了实现一个接口之外的大部分工作。这个被称为模板方法模式（Template Method ）[Gamma95].

> By convention, skeletal implementation classes are called Abstract*Interface*, where *Interface* is the name of the interface they implement. For example, the Collections Framework provides a skeletal implementation to go along with each main collection interface: AbstractCollection, AbstractSet, AbstractList, and AbstractMap. Arguably it would have made sense to call them SkeletalCollection, SkeletalSet, SkeletalList, and SkeletalMap, but the Abstract convention is now firmly established. When properly designed, skeletal implementations (whether a separate abstract class, or consisting solely of default methods on an interface) can make it *very* easy for programmers to provide their own implementations of an interface. For example, here’s a static factory method containing a complete, fully functional List implementation atop AbstractList:

按照习惯，骨架实现类一般被都命名为Abstract*Interface*，其中*Interface*就是其实现的接口的名字。比兔，在集合框架里，为所有的重要的集合接口都提供了一个骨架实现：AbstractCollection, AbstractSet, AbstractList, 和AbstractMap。可以说将其命名为SkeletalCollection, SkeletalSet, SkeletalList, 和 SkeletalMap也是有道理的，但是Abstract的约定用法已经根深蒂固了。如果设计合适，骨架实现（不管是单独的抽象类还是包含默认方法的接口）都会让程序员为接口提供自己的实现变得非常容易。如下，便是一个静态工厂方法，包含了一个基于AbstractList的完整、功能齐全的列表实现。

```java
// Concrete implementation built atop skeletal implementation
   static List<Integer> intArrayAsList(int[] a) {
       Objects.requireNonNull(a);
		 // The diamond operator is only legal here in Java 9 and later 
     // If you're using an earlier release, specify <Integer> 
     	 return new AbstractList<>() {
           @Override public Integer get(int i) {
               return a[i];  // Autoboxing (Item 6)
					 }
           @Override public Integer set(int i, Integer val) {
               int oldVal = a[i];
               a[i] = val;     // Auto-unboxing
               return oldVal;  // Autoboxing
					 }
           @Override public int size() {
               return a.length;
					 } 
   		 };
	 }
```

> When you consider all that a List implementation does for you, this example is an impressive demonstration of the power of skeletal implementations. Incidentally, this example is an *Adapter* [Gamma95] that allows an int array to be viewed as a list of Integer instances. Because of all the translation back and forth between int values and Integer instances (boxing and unboxing), its perfor- mance is not terribly good. Note that the implementation takes the form of an *anonymous class* (Item 24).

当你想知道一个列表实现为你做了哪些事情的时候，这个例子就充分展示了骨架实现类的全部功能。顺便提一下，这个例子也是一个适配器（Adapter）[Gamma95] ，它可以将一个int数组看做是一个Integer实例的列表。由于存在int值和Integer实例之间的来回转换（自动装箱和自动拆箱），这个例子的性能不太好。还需要注意的是，这个实现采用了匿名类的形式（Item24）。

> The beauty of skeletal implementation classes is that they provide all of the implementation assistance of abstract classes without imposing the severe constraints that abstract classes impose when they serve as type definitions. For most implementors of an interface with a skeletal implementation class, extending this class is the obvious choice, but it is strictly optional. If a class cannot be made to extend the skeletal implementation, the class can always implement the interface directly. The class still benefits from any default methods present on the interface itself. Furthermore, the skeletal implementation can still aid the implementor’s task. The class implementing the interface can forward invocations of interface methods to a contained instance of a private inner class that extends the skeletal implementation. This technique, known as *simulated multiple inheritance*, is closely related to the wrapper class idiom discussed in Item 18. It provides many of the benefits of multiple inheritance, while avoiding the pitfalls.

骨架实现类的优美之处在于它提供了抽象类有的所有的实现帮助，但是又没有”抽象类作为类型定义“所带来的限制。在大多数情况下，我们直接通过骨架实现类来实现一个接口是很明智的，但也不是必须的。如果一个类不能继承骨架实现类，这个类可以直接实现接口，这个类还是可以利用接口提供的所有的默认实现。此外，骨架实现仍然可以帮助接口的实习。实现接口的类中可以包含一个继承了骨架实现的私有内部类，然后把接口方法的调用转发到这个私有内部类的实例上。这种技术被称为”模拟多重继承“，和Item18 里讨论的包装类模型密切相关。它提供了很多多重继承的好处，同时还避免了相应的缺陷。

> Writing a skeletal implementation is a relatively simple, if somewhat tedious, process. First, study the interface and decide which methods are the primitives in terms of which the others can be implemented. These primitives will be the abstract methods in your skeletal implementation. Next, provide default methods in the interface for all of the methods that can be implemented directly atop the primitives, but recall that you may not provide default methods for Object methods such as equals and hashCode. If the primitives and default methods cover the interface, you’re done, and have no need for a skeletal implementation class. Otherwise, write a class declared to implement the interface, with implementations of all of the remaining interface methods. The class may contain any nonpublic fields ands methods appropriate to the task.

写一个骨架实现是一个简单，甚至有点单调乏味的过程。首先，认真研究这个接口，确定哪些方法是基本方法，剩下的就是可以根据他们来实现的方法。首选，这些基本方法在你的骨架实现中应该是抽象方法。其次为接口中其他可以通过基本方法实现的方法提供默认方法实现，在重申一下，不能为Object方法比如equals和hashCode提供默认实现。如果基本方法和默认方法已经完全覆盖了你的接口，那你的工作就完成了，不需要再写一个骨架实现类了。否则，还要写一个类声明实现这个接口，然后实现接口中剩余的方法。这个类还可以包括一些有助于任务实现的非公有的域和方法。

> As a simple example, consider the Map.Entry interface. The obvious primitives are getKey, getValue, and (optionally) setValue. The interface specifies the behavior of equals and hashCode, and there is an obvious implementation of toString in terms of the primitives. Since you are not allowed to provide default implementations for the Object methods, all implementations are placed in the skeletal implementation class:

来看一个简单的例子，Map.Entry接口。很明显，它的基本方法就是getKey, getValue, 和 (可选的) setValue。接口还定义了equals和hashCode的行为，还有一个有明显实现的toString方法，它们都可以基于基本方法来实现。由于不能为Object方法提供默认实现，所有的实现都被放在骨架实现类里了。如下：

```java
// Skeletal implementation class
   public abstract class AbstractMapEntry<K,V>
           implements Map.Entry<K,V> {
       // Entries in a modifiable map must override this method
       @Override public V setValue(V value) {
           throw new UnsupportedOperationException();
       }
     // Implements the general contract of Map.Entry.equals
       @Override public boolean equals(Object o) {
           if (o == this)
               return true;
           if (!(o instanceof Map.Entry))
               return false;
         Map.Entry<?,?> e = (Map.Entry) o;
				 return Objects.equals(e.getKey(),getKey())
							&& Objects.equals(e.getValue(), getValue());
       }
       // Implements the general contract of Map.Entry.hashCode
       @Override public int hashCode() {
           return Objects.hashCode(getKey())
                ^Objects.hashCode(getValue());
			 }
       @Override public String toString() {
           return getKey() + "=" + getValue();
			 } 
}
```

> Note that this skeletal implementation could not be implemented in the Map.Entry interface or as a subinterface because default methods are not permit- ted to override Object methods such as equals, hashCode, and toString.
>
> Because skeletal implementations are designed for inheritance, you should follow all of the design and documentation guidelines in Item 19. For brevity’s sake, the documentation comments were omitted from the previous example, but **good documentation is absolutely essential in a skeletal implementation,** whether it consists of default methods on an interface or a separate abstract class.

需要注意的是，这个骨架实现不能在Map.Entry接口或者其子接口里实现，因为默认方法不能覆盖覆盖Object方法，比如equals，hashCode，toString。

由于骨架实现就是为了继承设计的，所以也应该遵守Item19中介绍的设计和文档说明的指导方针。简洁起见，在上面的例子中省略了文档说明，但是一个好的文档说明在骨架实现中是不可或缺的。不管在接口或者单独的抽象类中是否包含了默认实现。

> A minor variant on the skeletal implementation is the *simple implementation,* exemplified by AbstractMap.SimpleEntry. A simple implementation is like a skeletal implementation in that it implements an interface and is designed for inheritance, but it differs in that it isn’t abstract: it is the simplest possible working implementation. You can use it as it stands or subclass it as circumstances warrant.
>
> To summarize, an interface is generally the best way to define a type that permits multiple implementations. If you export a nontrivial interface, you should strongly consider providing a skeletal implementation to go with it. To the extent possible, you should provide the skeletal implementation via default methods on the interface so that all implementors of the interface can make use of it. That said, restrictions on interfaces typically mandate that a skeletal implementation take the form of an abstract class.

在骨架实现中发生一点小小的变化就是简单实现（simple implementation）了，AbstractMap.SimpleEntry就是一个很好的简单实现的例子。简单实现也实现了一个接口，也是为了继承设计的，这两点和骨架实现一致。不同的是它不是抽象的，它是一个可以工作的最简单的实现。你可以根据情况直接使用，或者将它子类化。

总结一下，接口通常来说是定义允许多个实现的类型的最好的方式。当你要导出一个重要的接口的时候，你应该坚决地考虑为这个接口提供一个骨架实现。也应该尽可能地通过接口中的默认方法来提供骨架实现，这样这个接口的所有实现者都可以使用这些骨架实现。但由于接口的相关限制，使得有些骨架实现必须采用抽象类的形式。

### 