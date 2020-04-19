### Item18 组合优先于继承

> Inheritance is a powerful way to achieve code reuse, but it is not always the best tool for the job. Used inappropriately, it leads to fragile software. It is safe to use inheritance within a package, where the subclass and the superclass implementations are under the control of the same programmers. It is also safe to use inheritance when extending classes specifically designed and documented for extension (Item 19). Inheriting from ordinary concrete classes across package boundaries, however, is dangerous. As a reminder, this book uses the word “inheritance” to mean *implementation inheritance* (when one class extends another). The problems discussed in this item do not apply to *interface inheritance* (when a class implements an interface or when one interface extends another).

继承是一种很有效的重用代码的方式，但是在实际工作中，并不总是最好的方式，它会让软件变得脆弱。在同一个包里使用继承是安全的，因为其子类和父类都在一个程序员的控制下。同时，继承一个为了继承而设计的，有文档说明的类，也是安全的（Item19）。然而，跨包继承一个普通具体的类，就是很危险的。需要提示一下的是，在本书 中使用”继承“来表示”实现继承“（也就是一个类继承另外一个类）。在本章中讨论的问题不能应用于”接口继承“（也就是一个类实现一个接口，或者一个接口继承另一个接口）。

> **Unlike method invocation, inheritance violates encapsulation** [Snyder86]. In other words, a subclass depends on the implementation details of its superclass for its proper function. The superclass’s implementation may change from release to release, and if it does, the subclass may break, even though its code has not been touched. As a consequence, a subclass must evolve in tandem with its superclass, unless the superclass’s authors have designed and documented it specifically for the purpose of being extended.

**和方法调用不一样，继承打破了封装性[Snyder86]**。换句话说，子类会依赖于其父类中具体功能的实现细节。而父类中的实现方式可能会随着版本迭代发生变化，一旦它变了，子类就被破坏了，即使子类的代码没有发生变化。因此，除非父类的设计者为了被继承，专门设计好了，且有相关的文档，否则子类必须随着其父类进行修改。

> To make this concrete, let’s suppose we have a program that uses a HashSet. To tune the performance of our program, we need to query the HashSet as to how many elements have been added since it was created (not to be confused with its current size, which goes down when an element is removed). To provide this functionality, we write a HashSet variant that keeps count of the number of attempted element insertions and exports an accessor for this count. The HashSet class contains two methods capable of adding elements, add and addAll, so we override both of these methods:

为了说明得更加具体一些，假设有一个使用HashSet的程序。为了要调节程序的性能，我们需要查询这个hashSet自创建以来，一共添加了多少个元素（不要和当前大小弄混了，当元素被移除后，当前大小会变少）。为了提供这样的功能，我们写了一个HashSet的变体，记录了试图插入的元素的个数，并为这个数值提供了一个访问方法。这个HashSet类有两个可以添加元素的方法，add和addAll。所以我们需要将这两个方法都重写，如下：

```java
// Broken - Inappropriate use of inheritance!
   public class InstrumentedHashSet<E> extends HashSet<E> {
       // The number of attempted element insertions
       private int addCount = 0;
       public InstrumentedHashSet() {
       }
       public InstrumentedHashSet(int initCap, float loadFactor) {
           super(initCap, loadFactor);
			 }
     @Override 
     public boolean add(E e) {
    		addCount++;
    		return super.add(e);
		 }
		 @Override 
     public boolean addAll(Collection<? extends E> c) { 
       addCount += c.size();
			 return super.addAll(c);
		 }
		 public int getAddCount() {
       return addCount;
		 }
}
```

> This class looks reasonable, but it doesn’t work. Suppose we create an instance and add three elements using the addAll method. Incidentally, note that we create a list using the static factory method List.of, which was added in Java 9; if you’re using an earlier release, use Arrays.asList instead:

这个类看起来很合理，但是却不能正常地工作。假定我们创建一个实例，并且使用addAll方法添加了3个元素。顺便提一下，在java9中，我们可以使用静态工厂方法List.of来创建一个列表。如果你使用更早的版本的话，可以使用Arrays.asList来代替。代码如下：

```java
InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
s.addAll(List.of("Snap", "Crackle", "Pop"));
```

> We would expect the getAddCount method to return three at this point, but it returns six. What went wrong? Internally, HashSet’s addAll method is implemented on top of its add method, although HashSet, quite reasonably, does not document this implementation detail. The addAll method in InstrumentedHashSet added three to addCount and then invoked HashSet’s addAll implementation using super.addAll. This in turn invoked the add method, as overridden in InstrumentedHashSet, once for each element. Each of these three invocations added one more to addCount, for a total increase of six: each element added with the addAll method is double-counted.

在这个时候，我们会期望getAddCount方法返回3，但是它却返回了6。是哪里出了问题？在HashSet的addAll方法的内部实现是基于add方法的，即使HashSet并没有提供文档说明它的实现细节，但这也是合理的。在InstrumentedHashSet的addAll方法里，先给addCount加了3，然后使用super.addAll调用了HashSet的addAll方法的实现。而对于每一个元素，HashSet中的addAll方法又调用了在InstrumentedHashSet覆盖后的add方法。这三次调用都给addCount多加了一次，就导致总的值变成6了，每一个通过addAll方法添加的元素都被计了两次数。

> We could “fix” the subclass by eliminating its override of the addAll method. While the resulting class would work, it would depend for its proper function on the fact that HashSet’s addAll method is implemented on top of its add method. This “self-use” is an implementation detail, not guaranteed to hold in all implementations of the Java platform and subject to change from release to release. Therefore, the resulting InstrumentedHashSet class would be fragile.

我们可以通过去掉这个覆盖的addAll方法来修复这个子类。虽然这样的类可以工作了，但它的功能还是依赖于HashSet的addAll方法是通过调用add方法来实现的这一事实。这种“自用性”是实现的细节，并不保证在所有的Java平台的实现中都一样，同时随着版本迭代，也可能会修改。因此，这种InstrumentedHashSet类是很脆弱的。

> It would be slightly better to override the addAll method to iterate over the specified collection, calling the add method once for each element. This would guarantee the correct result whether or not HashSet’s addAll method were implemented atop its add method because HashSet’s addAll implementation would no longer be invoked. This technique, however, does not solve all our problems. It amounts to reimplementing superclass methods that may or may not result in self-use, which is difficult, time-consuming, error-prone, and may reduce performance. Additionally, it isn’t always possible because some methods cannot be implemented without access to private fields inaccessible to the subclass.

有一个稍微好一点的方法是，覆盖addAll方法，遍历给定的集合，为每一个元素都调用一次add方法。这样就可以保证，不管HashSet的addAll方法的实现是否依赖于add方法，结果都是正确的，因为HashSet的addAll方法根本就没有被调用到。然而这种方式也不能解决所有的问题。它相当于重新实现了父类的方法，这些方法可能是自用的，也可能不是，重写的方法很复杂、耗时、容易出错、还可能产生性能问题。并且，这样做也并不总是可行的，因为子类无法访问其私有域，有些方法就可能无法实现。

> A related cause of fragility in subclasses is that their superclass can acquire new methods in subsequent releases. Suppose a program depends for its security on the fact that all elements inserted into some collection satisfy some predicate. This can be guaranteed by subclassing the collection and overriding each method capable of adding an element to ensure that the predicate is satisfied before adding the element. This works fine until a new method capable of inserting an element is added to the superclass in a subsequent release. Once this happens, it becomes possible to add an “illegal” element merely by invoking the new method, which is not overridden in the subclass. This is not a purely theoretical problem. Several security holes of this nature had to be fixed when Hashtable and Vector were retrofitted to participate in the Collections Framework.

导致子类脆弱的另一个相关的原因是，父类在后续的版本中，可能会添加一些新的方法。假定一个程序的安全性依赖于添加到集合里的每一个元素都满足一定的条件。这可以通过继承一个集合，然后覆盖所有可以添加元素的方法，在添加元素之前，必须保证满足条件。在其父类在后续版本中添加了新的可以插入元素的方法之前，这个子类都能正常地工作。一旦父类发生了这样的变化，通过调用还没有在子类中覆盖的新的方法，就可以很容易地添加一个“非法”的元素了。这不仅仅是一个理论问题，当HashTable和Vector要加入到Collections框架中的时候，就有几个这种的安全漏洞必须要被修复。

> Both of these problems stem from overriding methods. You might think that it is safe to extend a class if you merely add new methods and refrain from overriding existing methods. While this sort of extension is much safer, it is not without risk. If the superclass acquires a new method in a subsequent release and you have the bad luck to have given the subclass a method with the same signature and a different return type, your subclass will no longer compile [JLS, 8.4.8.3]. If you’ve given the subclass a method with the same signature and return type as the new superclass method, then you’re now overriding it, so you’re subject to the problems described earlier. Furthermore, it is doubtful that your method will fulfill the contract of the new superclass method, because that contract had not yet been written when you wrote the subclass method.

上面这些问题都是覆盖方法带来的。你可能就会想，如果我在继承一个类的时候只是添加一些新的方法，而不覆盖存在的方法，是不是就安全了。虽然这种继承是要安全一些，但是也不是没有风险的。如果父类在后续版本中需要一个新的方法，然后你又很倒霉，你已经在子类中写了一个和它签名相同但是返回类型不同的方法，这个时候，你的子类就无法编译了。如果你子类中方法签名和返回类型都和父类新的方法一致，那就相当于覆盖了它，你就有了前面讨论过的那些问题了。此外，你写的方法很难满足父类新的方法的约定，因为当你写这个方法的时候，约定都还没有。

> Luckily, there is a way to avoid all of the problems described above. Instead of extending an existing class, give your new class a private field that references an instance of the existing class. This design is called *composition* because the existing class becomes a component of the new one. Each instance method in the new class invokes the corresponding method on the contained instance of the existing class and returns the results. This is known as *forwarding*, and the methods in the new class are known as *forwarding methods*. The resulting class will be rock solid, with no dependencies on the implementation details of the existing class. Even adding new methods to the existing class will have no impact on the new class. To make this concrete, here’s a replacement for InstrumentedHashSet that uses the composition-and-forwarding approach. Note that the implementation is broken into two pieces, the class itself and a reusable *forwarding class,* which contains all of the forwarding methods and nothing else:

幸运地是，还有一种方法可以避免前面提到的所有的问题。在你的新类中添加一个私有域来引用一个已经存在的类的实例。这种设计被称为“组合”，因为已存在类变成了新类的一个组件。每一个新类的实例方法都调用其包含的已存在类实例的对应的方法，然后返回结果。这种方式被称为”转发“，新类里的方法被称为”转发方法“。这样得到的类就非常稳固了，因为它完全不依赖已存在类的实现细节。即使往已存在类里添加新的方法也不会影响到新类。为了进行更加明确地说明，下面是使用组合-转发方式的实现来替代InstrumentedHashSet。需要注意的是，这种实现分为两部分，这个类本身和可重用的”转发类“，转发类中只包含转发方法，没有其他的方法：

```java
// Wrapper class - uses composition in place of inheritance
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;
    public InstrumentedSet(Set<E> s) {
        super(s);
    }
    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }
		@Override public boolean addAll(Collection<? extends E> c) { addCount += c.size();
				return super.addAll(c);
    }
    public int getAddCount() {
        return addCount;
    }
}
```

```java
// Reusable forwarding class
public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;
    public ForwardingSet(Set<E> s)    { this.s = s; }
    public void clear()               { s.clear();            }
    public boolean contains(Object o) { return s.contains(o); }
		public boolean isEmpty()          { return s.isEmpty();}
		public int size()                 { return s.size();}
		public Iterator<E> iterator()     { return s.iterator();}
		public boolean add(E e)           { return s.add(e)}
		public boolean remove(Object o)   { return s.remove(o)}
		public boolean containsAll(Collection<?> c)
                                   { return s.containsAll(c); }
    public boolean addAll(Collection<? extends E> c)
                                   { return s.addAll(c);      }
    public boolean removeAll(Collection<?> c)
                                   { return s.removeAll(c);   }
    public boolean retainAll(Collection<?> c)
                                   { return s.retainAll(c);   }
    public Object[] toArray()          { return s.toArray();  }
    public <T> T[] toArray(T[] a)      { return s.toArray(a); }
    @Override public boolean equals(Object o)
                                       { return s.equals(o);  }
    @Override public int hashCode()    { return s.hashCode(); }
    @Override public String toString() { return s.toString(); }
}
```

> The design of the InstrumentedSet class is enabled by the existence of the Set interface, which captures the functionality of the HashSet class. Besides being robust, this design is extremely flexible. The InstrumentedSet class implements the Set interface and has a single constructor whose argument is also of type Set. In essence, the class transforms one Set into another, adding the instrumentation functionality. Unlike the inheritance-based approach, which works only for a single concrete class and requires a separate constructor for each supported constructor in the superclass, the wrapper class can be used to instrument any Set implementation and will work in conjunction with any preexisting constructor:

Set接口的存在使得InstrumentedSet类的设计成为可能，Set接口描述了HashSet类的功能。这个设计除了带来了鲁棒性以外，还更加灵活。InstrumentedSet类实现了Set接口，有一个唯一的参数为Set类型构造器，本质上来说，这个类把一个set转换为了另一个set，添加了计数的功能。和基于继承的方法不一样的是，基于继承的方法只适用于单个确定的类，并且需要为父类中支持的每一个的构造器分别写一个构造器；而这里的包装类可以用来包装任何的Set实现类，并且可以和任何一个已经存在的构造器合作：

```java
Set<Instant> times = new InstrumentedSet<>(new TreeSet<>(cmp));
Set<E> s = new InstrumentedSet<>(new HashSet<>(INIT_CAPACITY));
```

> The InstrumentedSet class can even be used to temporarily instrument a set instance that has already been used without instrumentation:

这个InstrumentedSet类也可以用来临时替换一个原本没有技术功能的set实例：

```java
static void walk(Set<Dog> dogs) {
       InstrumentedSet<Dog> iDogs = new InstrumentedSet<>(dogs);
       ... // Within this method use iDogs instead of dogs
}
```

> The InstrumentedSet class is known as a *wrapper* class because each InstrumentedSet instance contains (“wraps”) another Set instance. This is also known as the *Decorator* pattern [Gamma95] because the InstrumentedSet class “decorates” a set by adding instrumentation. Sometimes the combination of composition and forwarding is loosely referred to as *delegation.* Technically it’s not delegation unless the wrapper object passes itself to the wrapped object [Lieber- man86; Gamma95].

由于每个InstrumentedSet实例都包含（包装）了另一个Set实例，因此被称为包装类。这也正是装饰者模式（ Decorator pattern ）[Gamma95] ，因为InstrumentedSet通过新增了计数功能，对set实例进行了装饰。有时候，不严格地说，也把这种组合和转发的结合称为”委托“。通常来说，这并不是委托，除非包装对象把自己传递给了被包装对象[Lieber- man86; Gamma95]。

> The disadvantages of wrapper classes are few. One caveat is that wrapper classes are not suited for use in *callback frameworks*, wherein objects pass self-references to other objects for subsequent invocations (“callbacks”). Because a wrapped object doesn’t know of its wrapper, it passes a reference to itself (this) and callbacks elude the wrapper. This is known as the *SELF problem* [Lieberman86]. Some people worry about the performance impact of forwarding method invocations or the memory footprint impact of wrapper objects. Neither turn out to have much impact in practice. It’s tedious to write forwarding methods, but you have to write the reusable forwarding class for each interface only once, and forwarding classes may be provided for you. For example, Guava provides forwarding classes for all of the collection interfaces [Guava].

这种包装类的缺点很少。有个需要注意的点是，在回调框架中，不适合使用包装类。在回调系统中，对象会将自己的引用传递给其他的对象，以便后面的调用（即回调）。因为被包装的对象并不知道它外面的包装对象，它就会传递一个自己的引用，在回调的时候也会因此避开了包装类。这个问题被称为SELF问题[Lieberman86]。有些人会担心转发方法带来的性能影响或者包装对象占用内存问题，这些在实际应用中都没有太大的影响。写转发方法的过程比较单调无聊，但是对于每一个接口，你只需要写一次可重用的转发类就可以了，而且有的包也可能已经为你提供了这些转发类。比如，Guava就为所有的集合接口都提供了转发类。

> Inheritance is appropriate only in circumstances where the subclass really is a *subtype* of the superclass. In other words, a class *B* should extend a class *A* only if an “is-a” relationship exists between the two classes. If you are tempted to have a class *B* extend a class *A*, ask yourself the question: Is every *B* really an *A*? If you cannot truthfully answer yes to this question, *B* should not extend *A*. If the answer is no, it is often the case that *B* should contain a private instance of *A* and expose a different API: *A* is not an essential part of *B*, merely a detail of its implementation.

只有在子类确实是父类的一个子类型（subtype）的时候，才适合使用继承。换句话说，只有当B类和A类之间存在”is-a"关系的时候，类B才应该继承类A。当你企图当类B继承类A的时候，问你自己一个问题：每一个B都真的是A吗？对这个问题，如果你不能坚定地回答“是”的话，B就不应该继承A。如果你的答案是“不是”的话，通常来说，B应该包含一个私有的A的实例，并暴露一个不同的API。A就不是B中不可或缺的一部分了，只是B的实现细节而已。

> There are a number of obvious violations of this principle in the Java platform libraries. For example, a stack is not a vector, so Stack should not extend Vector. Similarly, a property list is not a hash table, so Properties should not extend Hashtable. In both cases, composition would have been preferable.

在Java平台类库中，有很多明显违反了这一原则的例子。比如，堆栈不是向量，因此Stack就不应该继承Vector。同样地，域列表不是散列表，因此Properties不应该继承HashTable。在这些例子中，组合模式都要更适合一些。

> If you use inheritance where composition is appropriate, you needlessly expose implementation details. The resulting API ties you to the original implementation, forever limiting the performance of your class. More seriously, by exposing the internals you let clients access them directly. At the very least, it can lead to confusing semantics. For example, if p refers to a Properties instance, then p.getProperty(key) may yield different results from p.get(key): the former method takes defaults into account, while the latter method, which is inherited from Hashtable, does not. Most seriously, the client may be able to corrupt invariants of the subclass by modifying the superclass directly. In the case of Properties, the designers intended that only strings be allowed as keys and values, but direct access to the underlying Hashtable allows this invariant to be violated. Once violated, it is no longer possible to use other parts of the Properties API (load and store). By the time this problem was discovered, it was too late to correct it because clients depended on the use of non-string keys and values.

如果你在更适合使用组合的地方使用继承，就会暴露一些没有必要暴露的实现细节。这样得到的API将你和原始的实现绑在一起了，会一直限制你的类的性能。更严重的是，暴露这些内部细节会使得客户端可以直接访问他们。往轻了说，这会导致产生语义上的混淆。比如，假如p是一个Properties实例的引用，p.getProperty(key)的结果就可能和p.get(key)不同，因为前一个方法会考虑到默认的属性表，而后一种方法是从HashTable里继承来的，不会考虑默认的属性表。往重了说，客户端还可能通过直接修改父类破坏子类的约束。在Properties的例子中，设计者希望只用String来作为键值，但是直接访问HashTable中的方法会导致这一约束被破坏。一旦破坏了，Properties的其他的API（比如load和store）可能就不能使用了。当这些问题发现的时候，就已经太晚了以至于无法修改了，因为其客户端已经依赖这些非String的键值了。

> There is one last set of questions you should ask yourself before deciding to use inheritance in place of composition. Does the class that you contemplate extending have any flaws in its API? If so, are you comfortable propagating those flaws into your class’s API? Inheritance propagates any flaws in the superclass’s API, while composition lets you design a new API that hides these flaws.

在你决定要使用继承代替组合的时候，你要问自己很多的问题。这个你打算继承的类的API里有没有什么缺陷？如果有的话，你是否能接受这些缺陷传播到你的类的API里吗？继承会传播父类中所有的缺陷，而组合却允许你自己设计一个新的API来隐藏这些缺陷。

> To summarize, inheritance is powerful, but it is problematic because it violates encapsulation. It is appropriate only when a genuine subtype relationship exists between the subclass and the superclass. Even then, inheritance may lead to fragility if the subclass is in a different package from the superclass and the superclass is not designed for inheritance. To avoid this fragility, use composition and forwarding instead of inheritance, especially if an appropriate interface to implement a wrapper class exists. Not only are wrapper classes more robust than subclasses, they are also more powerful.

总结一下，虽然继承的功能很强大，但是由于它破会了封装性，所以有很多的问题。只有当子类和父类之间确实存在真正的子类型关系的时候，继承才使用。即使如此，当子类从另一个包中继承父类，且父类并不是专门设计来继承的时候，继承将导致子类十分脆弱。为了避免这种脆弱，可以使用组合转发来代替继承，尤其是有一个用来实现包装类的合适的接口的时候。包装类不仅比子类更加健壮，其功能也更加强大。

### 