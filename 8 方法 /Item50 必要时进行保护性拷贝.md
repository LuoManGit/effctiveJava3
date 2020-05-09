### Item50 必要时进行保护性拷贝

> One thing that makes Java a pleasure to use is that it is a *safe language*. This means that in the absence of native methods it is immune to buffer overruns, array overruns, wild pointers, and other memory corruption errors that plague unsafe languages such as C and C++. In a safe language, it is possible to write classes and to know with certainty that their invariants will hold, no matter what happens in any other part of the system. This is not possible in languages that treat all of memory as one giant array.

Java使用起来很舒适的一个原因是它是一个“安全的语言”。这意味着，在没有本地方法的情况下，它对缓冲区溢出、数组越界、非法指针以及其他危及不安全语言（比如C和C++）的内存破坏错误，都是免疫的。在一个安全的语言中，我们可以写一个类，并且知道不管系统的其他部分发生什么事情，这个类的约束条件都一致为真。这一点，对于那些把整个内存看做是一个巨大的数组的语言来说，是不可能的。

> Even in a safe language, you aren’t insulated from other classes without some effort on your part. **You must program defensively, with the assumption that clients of your class will do their best to destroy its invariants.** This is increasingly true as people try harder to break the security of systems, but more commonly, your class will have to cope with unexpected behavior resulting from the honest mistakes of well-intentioned programmers. Either way, it is worth taking the time to write classes that are robust in the face of ill-behaved clients.

即使是在安全的语言中，如果不付出一些努力的话，你也不能和其他的类隔绝开来。**假如你的类的客户端会尽全力来破坏整个类的约束，那么你就必须保护性地设计程序。**事实上，只有当人试图破坏系统的安全的时候，才会出现这样的情况，或者更常见的是，你的类必须处理一些意料之外的结果，这些结果来自那些本来是好意，但是使用错误的程序员。不管是哪一种方式，花时间去写一个在面对病态客户端的时候依旧保持健壮性的类，是很值得的。

> While it is impossible for another class to modify an object’s internal state without some assistance from the object, it is surprisingly easy to provide such assistance without meaning to do so. For example, consider the following class, which purports to represent an immutable time period:

虽然，在没有来自对象的帮助的情况下，一个类要修改一个对象的内部状态是不可能的，但是我们很容易在无意间提供了这样的帮助。比如，下面这个类，它声称可以表示段不可变的时间：

```java
// Broken "immutable" time period class
   public final class Period {
       private final Date start;
       private final Date end;
        /**
        * @param start the beginning of the period
        * @param end the end of the period; must not precede start 
        * @throws IllegalArgumentException if start is after end
        * @throws NullPointerException if start or end is null
        */
       public Period(Date start, Date end) {
           if (start.compareTo(end) > 0)
               throw new IllegalArgumentException(
                   start + " after " + end);
           this.start = start;
           this.end   = end;
       }
       public Date start() {
           return start;
       }
     public Date end() {
           return end;
     }
       ...  // Remainder omitted
   }
```

> At first glance, this class may appear to be immutable and to enforce the invariant that the start of a period does not follow its end. It is, however, easy to violate this invariant by exploiting the fact that Date is mutable:

乍一看，这个类好像确实是不可变的，还强制执行了开始时间不得在结束时间之后的限制。然而，利用Date是可变的这点，很容易打破这个约束，如下：

```java
// Attack the internals of a Period instance
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end); 
end.setYear(78); // Modifies internals of p!
```

> As of Java 8, the obvious way to fix this problem is to use Instant (or LocalDateTime or ZonedDateTime) in place of a Date because Instant (and the other java.time classes) are immutable (Item 17). **Date** **is obsolete and should no longer be used in new code.** That said, the problem still exists: there are times when you’ll have to use mutable value types in your APIs and internal representations, and the techniques discussed in this item are appropriate for those times.

在Java8中，最明显的修改这个问题的方法是使用Instant（或者LocalDateTime或者ZonedDateTime）来代替Date，因为Instant它们是不可变的（Item17）。**Date过时了，也不应该用在新的代码中了**。也就是说，这个问题依然存在，很多时候我们在API和内部表示中确实需要可变对象类型，在本节中要介绍的技术就是针对这些情况的。

> To protect the internals of a Period instance from this sort of attack, **it is essential to make a** **defensive copy** **of each mutable parameter to the constructor** and to use the copies as components of the Period instance in place of the originals:

为了保护Period实例的内部信息远离这种攻击，**给传递给构造器的可变参数做一个保护性拷贝是很有必要的，**并且使用这些拷贝来代替原始对象作为Period实例的组件。代码如下：

```java
// Repaired constructor - makes defensive copies of parameters
   public Period(Date start, Date end) {
       this.start = new Date(start.getTime());
       this.end   = new Date(end.getTime());
       if (this.start.compareTo(this.end) > 0)
         throw new IllegalArgumentException(
             this.start + " after " + this.end);
}
```

> With the new constructor in place, the previous attack will have no effect on the Period instance. Note that **defensive copies are made** **before** **checking the validity of the parameters (Item 49), and the validity check is performed on the copies rather than on the originals.** While this may seem unnatural, it is necessary. It protects the class against changes to the parameters from another thread during the *window of vulnerability* between the time the parameters are checked and the time they are copied. In the computer security community, this is known as a *time-of-check/time-of-use* or *TOCTOU* attack [Viega01].

使用了这个新的构造器后，之前的攻击对于Period实例就没有用了。注意，**保护性拷贝是在检查参数的有效性之前进行的，并且有效性检查是在拷贝对象而不是原对象上进行。**虽然这个看起来有点不自然，但是它可以让这个类避免出现这样的情况：在对象检查和对象复制中间的危险期，另一个线程修改了其参数。在计算机安全领域，这个被称为*time-of-check/time-of-use* 或者 *TOCTOU* 攻击 [Viega01]。

> Note also that we did not use Date’s clone method to make the defensive copies. Because Date is nonfinal, the clone method is not guaranteed to return an object whose class is java.util.Date: it could return an instance of an untrusted subclass that is specifically designed for malicious mischief. Such a subclass could, for example, record a reference to each instance in a private static list at the time of its creation and allow the attacker to access this list. This would give the attacker free rein over all instances. To prevent this sort of attack, **do not use the** **clone** **method to make a defensive copy of a parameter whose type is subclassable by untrusted parties.**

还需要注意的是，在做保护性拷贝的时候，并没有使用Date的clone方法。因为Data类是非final的，无法保证它的clone方法返回的对象的类就是java.util.Date，可能返回的处于恶意设计的不可信的子类。比如，这样的一个子类可能在创建的时候把每个实例的引用都保存到一个私有静态列表中，然后允许攻击者访问这个列表。这样攻击者可以自由访问这些实例。为了防止这种攻击，**在做参数的保护性拷贝的时候，如果该参数可能被不可信的用户子类化，就不要使用clone方法。**

> While the replacement constructor successfully defends against the previous attack, it is still possible to mutate a Period instance, because its accessors offer access to its mutable internals:

虽然这个替换后的构造器可以抵御前面的那种攻击，但是它还是可能改变Period实例，因为其访问提供了它的可变内部域的访问，这种攻击如下：

```java 
// Second attack on the internals of a Period instance
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end); 
p.end().setYear(78); // Modifies internals of p!
```

> To defend against the second attack, merely modify the accessors to **return defensive copies of mutable internal fields:**

要抵抗第二种攻击，我们只需要修改这个访问器，**让它返回可变内部域的保护性拷贝**就可以了，如下：

```java
// Repaired accessors - make defensive copies of internal fields
   public Date start() {
       return new Date(start.getTime());
   }
   public Date end() {
       return new Date(end.getTime());
   }
```

> With the new constructor and the new accessors in place, Period is truly immutable. No matter how malicious or incompetent a programmer, there is simply no way to violate the invariant that the start of a period does not follow its end (without resorting to extralinguistic means such as native methods and reflection). This is true because there is no way for any class other than Period itself to gain access to either of the mutable fields in a Period instance. These fields are truly encapsulated within the object.

使用了这个新的构造器和新的访问器替换后，这个Period就是真的不可变了。不管这个程序员多么恶意，或者多么笨笨，都绝不会违反”开始时间早于结束时间“这一约束条件了（除了那些语言之外的方法，比如本地方法，和反射）。确实如此，因为对于除了Period自己以外其他的类，没有办法访问Period实例里的任何可变域。这些域是真正地封装在了对象内部。

> In the accessors, unlike the constructor, it would be permissible to use the clone method to make the defensive copies. This is so because we know that the class of Period’s internal Date objects is java.util.Date, and not some untrusted subclass. That said, you are generally better off using a constructor or static factory to copy an instance, for reasons outlined in Item 13.

和构造器不一样的是，在访问器中，是可以使用clone方法来进行保护性拷贝的。因为我们知道这个Period内部的Date对象就是java.util.Date，不可能是其他的不可信的子类。也就说，我们最好是使用构造器或者静态工厂方法来复制实例，理由详见Item13。

> Defensive copying of parameters is not just for immutable classes. Any time you write a method or constructor that stores a reference to a client-provided object in an internal data structure, think about whether the client-provided object is potentially mutable. If it is, think about whether your class could tolerate a change in the object after it was entered into the data structure. If the answer is no, you must defensively copy the object and enter the copy into the data structure in place of the original. For example, if you are considering using a client-provided object reference as an element in an internal Set instance or as a key in an internal Map instance, you should be aware that the invariants of the set or map would be corrupted if the object were modified after it is inserted.

对参数进行保护性拷贝不仅仅适合与不可变类。在每一次你写的方法或者构造器，需要将一个客户端提供的对象的引用保存到类的内部数据结构的时候，就需要思考这个类是不是可变的。如果是的话，就需要考虑你的类能不能容忍这个对象在假如数据结构后被改变。如果不能的话，你就必须对这个对象进行保护性拷贝，然后用这个拷贝结果替换原始对象来作为内部数据中的元素。比如，如果你考虑使用一个客户端提供的对象引用来作为一个内部的Set实例的元素或者作为内部Map实例的key的时候，你必须保证这个map和set中约束条件，不会因为这个对象在插入后被修改，而被破坏。

> The same is true for defensive copying of internal components prior to returning them to clients. Whether or not your class is immutable, you should think twice before returning a reference to an internal component that is mutable. Chances are, you should return a defensive copy. Remember that nonzero-length arrays are always mutable. Therefore, you should always make a defensive copy of an internal array before returning it to a client. Alternatively, you could return an immutable view of the array. Both of these techniques are shown in Item 15.

优先返回内部组件的保护性拷贝给客户端，也是同样的道理。不管你的类是不是可变的，在返回可变内部组件的引用给客户端的时候，都应该仔细思考一下。如果有需要，就应该返回保护性拷贝。记住长度非0的数组总是可变的。因此你应该个客户端返回一个内部数组的保护性拷贝。或者模拟也可以返回一个数组的不可变试图，这些技术在Item15中都介绍过了。

> Arguably, the real lesson in all of this is that you should, where possible, use immutable objects as components of your objects so that you that don’t have to worry about defensive copying (Item 17). In the case of our Period example, use Instant (or LocalDateTime or ZonedDateTime), unless you’re using a release prior to Java 8. If you are using an earlier release, one option is to store the primitive long returned by Date.getTime() in place of a Date reference.

可以这么说，本节真正的含义是，只要有可能都使用不可变对象作为对象的组件，那样就不需要考虑保护性拷贝的问题（Item17）。在我们的Period例子里，除非你使用的是Java8之前的版本，否则你都应该使用Instant (或者 LocalDateTime 或者 ZonedDateTime)。如果你使用的是更早的版本，还有一个选择是保存Date.getTime()返回的基本类型long来替换Date引用。

> There may be a performance penalty associated with defensive copying and it isn’t always justified. If a class trusts its caller not to modify an internal component, perhaps because the class and its client are both part of the same package, then it may be appropriate to dispense with defensive copying. Under these circumstances, the class documentation should make it clear that the caller must not modify the affected parameters or return values.

保护性拷贝可能会带来性能损失，而且它有时候也不合适。如果一个类信任它的调用者都不会修改内部组件，比如这个类和其客户端都在同一个包里，这种时候省略掉保护性拷贝就是合适的。在这种情况下，类文档说明应该清楚地声明，调用者不应该修改受影响的参数和返回值。

> Even across package boundaries, it is not always appropriate to make a defensive copy of a mutable parameter before integrating it into an object. There are some methods and constructors whose invocation indicates an explicit *handoff* of the object referenced by a parameter. When invoking such a method, the client promises that it will no longer modify the object directly. A method or constructor that expects to take ownership of a client-provided mutable object must make this clear in its documentation.

即使在跨包的作用范围，在把一个可变参数变成对象的一部分之前，进行保护性拷贝也不总是合适的。有一些方法和构造器的调用，要求参数所引用的对象必须有一个显示的交接过程。但调用这样的方法的时候，客户端就保证不会在直接修改这个对象了。希望获得可不断提供的可变对象的所有权的方法或者构造器，必须在文档中说明。

> Classes containing methods or constructors whose invocation indicates a transfer of control cannot defend themselves against malicious clients. Such classes are acceptable only when there is mutual trust between a class and its client or when damage to the class’s invariants would harm no one but the client. An example of the latter situation is the wrapper class pattern (Item 18). Depending on the nature of the wrapper class, the client could destroy the class’s invariants by directly accessing an object after it has been wrapped, but this typically would harm only the client.

如果类的方法和构造器的调用需要转移对象的控制权，那么这个类就无法抵御恶意客户端的攻击。这种类只有在以下两种情况下是可以接受的：类和客户端之前互相信任，或者破坏了类的约束条件只会对客户端造成危害。包装类（Item18）是后面这种情况的例子。根据包装类的本质特征，在对象被包装后，客户端如果直接访问对象就会破坏类的约束性条件，但是通常来说，受到影响的只有客户端自己。

> In summary, if a class has mutable components that it gets from or returns to its clients, the class must defensively copy these components. If the cost of the copy would be prohibitive *and* the class trusts its clients not to modify the components inappropriately, then the defensive copy may be replaced by documentation outlining the client’s responsibility not to modify the affected components.

总结一下，如果一个类有来自客户端的可变组件，或者需要将可变组件返回给客户端，这个类就应该对这些组件进行保护性拷贝。如果这个拷贝代价比较大，以及这个类信任它的客户端不会不恰当地修改这些组件，保护性拷贝就可以省略掉，并使用文档说明不要修改受影响的组件是客户端的责任。