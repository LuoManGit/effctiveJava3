### Item13 谨慎地覆盖clone

> The Cloneable interface was intended as a *mixin interface* (Item 20) for classes to advertise that they permit cloning. Unfortunately, it fails to serve this purpose. Its primary flaw is that it lacks a clone method, and Object’s clone method is protected. You cannot, without resorting to *reflection* (Item 65), invoke clone on an object merely because it implements Cloneable. Even a reflective invocation may fail, because there is no guarantee that the object has an accessible clone method. Despite this flaw and many others, the facility is in reasonably wide use, so it pays to understand it. This item tells you how to implement a well-behaved clone method, discusses when it is appropriate to do so, and presents alternatives.
>
> So what *does* Cloneable do, given that it contains no methods? It determines the behavior of Object’s protected clone implementation: if a class implements Cloneable, Object’s clone method returns a field-by-field copy of the object; otherwise it throws CloneNotSupportedException. This is a highly atypical use of interfaces and not one to be emulated. Normally, implementing an interface says something about what a class can do for its clients. In this case, it modifies the behavior of a protected method on a superclass.

Cloneable结果被设计为类的mixin接口（详见Item20），用来声明这个类允许clone。不幸的是，它并不能达到这个目标，因为它并没有clone方法，而且Object的clone方法是受保护的（protected）。你不能仅仅因为这个类实现了Cloneable接口，就可以调用它的clone方法，除非使用反射（Item65)。甚至有时候反射也会失败，因为没有办法保证这个对象有一个可访问的clone方法。虽然有很多的问题，但是这种方法确实被广泛使用了，因此还是值得我们去学习的。在本条里，将告诉你如何去实现一个表现不错的clone方法，也讨论了还何时应该这么做，还提供了一些可以替代的方案。

那么啥方法都没有的Cloneable方法有啥用呢？它确定了Object中被保护的clone方法的具体的实现方式，若果一个类实现了Cloneable接口，对象的clone方法返回对该对象的一个拷贝，该拷贝是通过对原对象的每个域的挨着挨着的复制得到的；否则clone方法就将抛出CloneNotSupportedException。这是接口的一个反常的使用方式，不应该去模仿。通常情况下，我们实现一个接口就是声明这个对象可以为客户端提供哪些功能。而在Cloneable这里，它修改了父类的被保护方法clone的行为。

> Though the specification doesn’t say it, **in practice, a class implementing** **Cloneable** **is expected to provide a properly functioning public** **clone** **method.** In order to achieve this, the class and all of its superclasses must obey a complex, unenforceable, thinly documented protocol. The resulting mechanism is fragile, dangerous, and *extralinguistic*: it creates objects without calling a constructor.
>
> The general contract for the clone method is weak. Here it is, copied from the Object specification :

虽然规范里并没有明说，**但是在实际应用中，一个实现了Cloneable的类需要提供一个功能合适的public的clone方法。**为了达到这个目的，这个类和它的父类都必须要遵守一个复杂的、不强制实施的（unenforceable）、没有文档说明的协议。这导致这种机制是易错的、危险的、语言之外（extralinguistic）的。

clone方法的通用约定是非常弱的，下面是从Object规范里的复制来的约定内容

> Creates and returns a copy of this object. The precise meaning of “copy” may depend on the class of the object. The general intent is that, for any object x, the expression
>
> x.clone() != x 
>
> will be true, and the expression
>
> x.clone().getClass() == x.getClass()
>
> will be true, but these are not absolute requirements. While it is typically the case that
>
> x.clone().equals(x)
>
> will be true, this is not an absolute requirement.

创建并返回一个这个对象的拷贝。拷贝的具体的定义取决于这个对象的类，通常来说，对于一个对象x，表达式x.clone() != x 

应该返回true，同时表达式

x.clone().getClass() == x.getClass()

也应该返回true，但是这些都不是绝对的要求，通常这个表达式

x.clone().equals(x)

也应该返回true，但是这也不是绝对的要求。

> By convention, the object returned by this method should be obtained by calling super.clone. If a class and all of its superclasses (except Object) obey this convention, it will be the case that
>
> x.clone().getClass() == x.getClass().
>
> By convention, the returned object should be independent of the object being cloned. To achieve this independence, it may be necessary to modify one or more fields of the object returned by super.clone before returning it.

按照约定，clone方法返回的对象应该通过调用super.clone来获得。如果一个类和她所有的除了Object外的父类都遵守了这个约定，那么 x.clone().getClass() == x.getClass() 表达式就会返回true。

按照约定，返回的对象必须独立于被拷贝的对象，为了达到这个目标，在调用super.clone()后，返回对象前，修改返回对象的一些域是很有必要的。

> This mechanism is vaguely similar to constructor chaining, except that it isn’t enforced: if a class’s clone method returns an instance that is *not* obtained by call- ing super.clone but by calling a constructor, the compiler won’t complain, but if a subclass of that class calls super.clone, the resulting object will have the wrong class, preventing the subclass from clone method from working properly. If a class that overrides clone is final, this convention may be safely ignored, as there are no subclasses to worry about. But if a final class has a clone method that does not invoke super.clone, there is no reason for the class to implement Cloneable, as it doesn’t rely on the behavior of Object’s clone implementation.

这种机制和构造器调用链有点相似，但是这个不是强制的。如果一个类的clone方法返回的对象不是通过调用super.clone()获得的，而是调用构造器获得的，此时编译器也不会报错。但是如果这个类的子类，调用了super.clone方法，返回的对象的类就是错误的，使得这个子类的clone方法无法正常工作。如果覆盖clone方法的类是final的，那么这个问题就可以安全地忽略掉，因为没有子类会出问题。如果一个final类的clone方法里没有调用super.clone方法，那么也就没有必要实现Cloneable接口了，因为它完全不依赖Object的clone方法的实现行为。

> Suppose you want to implement Cloneable in a class whose superclass provides a well-behaved clone method. First call super.clone. The object you get back will be a fully functional replica of the original. Any fields declared in your class will have values identical to those of the original. If every field contains a primitive value or a reference to an immutable object, the returned object may be exactly what you need, in which case no further processing is necessary. This is the case, for example, for the PhoneNumber class in Item 11, but note that **immutable classes should never provide a** **clone** **method** because it would merely encourage wasteful copying. With that caveat, here’s how a clone method for PhoneNumber would look:

假如你在一个类里实现Cloneable接口，这个类的父类已经提供了表现不错的clone方法。第一步就是调用super.clone()方法，这个方法将返回一个和原始对象功能完全相同的对象，该对象的每个声明的域值都将和原始对象一致。如果每个域包含的是基本类型，或者是不可变类的引用，那么这个返回的对象就正是你所需要的，不再需要进行其他的处理，比如Item11里的PhoneNumber就是这种类型。**需要注意的是，不可变类永远都不需要提供clone方法**，这只会导致出现不必要的复制。根据这种方式，PhoneNUmber里的clone方法应该如下所示：

```java
// Clone method for class with no references to mutable state
   @Override public PhoneNumber clone() {
       try {
           return (PhoneNumber) super.clone();
       } catch (CloneNotSupportedException e) {
           throw new AssertionError();  // Can't happen
       }
}
```

> In order for this method to work, the class declaration for PhoneNumber would have to be modified to indicate that it implements Cloneable. Though Object’s clone method returns Object, this clone method returns PhoneNumber. It is legal and desirable to do this because Java supports *covariant return types*. In other words, an overriding method’s return type can be a subclass of the overridden method’s return type. This eliminates the need for casting in the client. We must cast the result of super.clone from Object to PhoneNumber before returning it, but the cast is guaranteed to succeed.

为了让这个方法生效，我们需要把PhoneNumber的类声明上明确表示实现Cloneable。虽然Object的clone方法返回Object，但这个clone方法返回的是PhoneNumber。这个是合法的，并且推荐这么做，因为java是支持协变返回类型（convariant return types）的。换句话说，一个覆盖方法可以返回原方法的返回类型的子类。这样在客户端里就不需要进行转换了。因此我们在返回这个对象前，必须要将super.clone方法的返回值转换成PhoneNumber类，这个类型转换是一定会成功的。

> The call to super.clone is contained in a try-catch block. This is because Object declares its clone method to throw CloneNotSupportedException, which is a *checked exception*. Because PhoneNumber implements Cloneable, we know the call to super.clone will succeed. The need for this boilerplate indicates that CloneNotSupportedException should have been unchecked (Item 71).

对super.clone方法的调用时包含在一个try-catch块中的，这是因为Object声明了其clone方法会抛出CloneNotSupportedException，这是一个受检异常（checked exception）。因为PhoneNumber实现了Cloneable接口，因此我们知道super.clone方法的调用一定会成功。上面的样本代码也指明了CloneNotSupportedException异常将不会被检测到。

> If an object contains fields that refer to mutable objects, the simple clone implementation shown earlier can be disastrous. For example, consider the Stack class in Item 7:

如果这个对象包含可变对象的引用，那么前面那个简单的clone实现就会带来灾难性的问题。比如，考虑在Item7里面的Stack类：

```java
public class Stack {
       private Object[] elements;
       private int size = 0;
       private static final int DEFAULT_INITIAL_CAPACITY = 16;
       public Stack() {
           this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
			 }
       public void push(Object e) {
           ensureCapacity();
           elements[size++] = e;
			 }
       public Object pop() {
           if (size == 0)
               throw new EmptyStackException();
           Object result = elements[--size];
           elements[size] = null; // Eliminate obsolete reference
           return result;
			 }
       // Ensure space for at least one more element.
       private void ensureCapacity() {
           if (elements.length == size)
               elements = Arrays.copyOf(elements, 2 * size + 1);
			 } 
}
```

> Suppose you want to make this class cloneable. If the clone method merely returns super.clone(), the resulting Stack instance will have the correct value in its size field, but its elements field will refer to the same array as the original Stack instance. Modifying the original will destroy the invariants in the clone and vice versa. You will quickly find that your program produces nonsensical results or throws a NullPointerException.
>
> This situation could never occur as a result of calling the sole constructor in the Stack class. **In effect, the** **clone** **method functions as a constructor; you must ensure that it does no harm to the original object and that it properly establishes invariants on the clone**. In order for the clone method on Stack to work properly, it must copy the internals of the stack. The easiest way to do this is to call clone recursively on the elements array:

假如你想把这个类做成可clone的，如果它的clone方法也只是返回super.clone()，返回的Stack实例的size域将有正确的值，但是elements域却和原始的Stack实例指向了同一个数组。改变原始对象就会破坏复制对象的约束条件，反之亦然。很快你就会发现你的程序产生了一些毫无意义的结果，甚至会抛出NullPointerException。

如果直接调用Stack的唯一的构造器的话，这种情形就不会出现了。**实际上，clone方法就是另一个构造器，必须要保证不会破坏原始的对象，同时正确地建立克隆对象的约束条件。** 为了防Stack上的clone方法正常工作，必须要拷贝栈里的对象。最简单的方法就是在elements数组上递归的调用其clone方法。如下：

```java
// Clone method for class with references to mutable state
   @Override public Stack clone() {
       try {
           Stack result = (Stack) super.clone();
           result.elements = elements.clone();
           return result;
       } catch (CloneNotSupportedException e) {
           throw new AssertionError();
			 } 
}
```

> Note that we do not have to cast the result of elements.clone to Object[]. Calling clone on an array returns an array whose runtime and compile-time types are identical to those of the array being cloned. This is the preferred idiom to duplicate an array. In fact, arrays are the sole compelling use of the clone facility.
>
> Note also that the earlier solution would not work if the elements field were final because clone would be prohibited from assigning a new value to the field. This is a fundamental problem: like serialization, **the** **Cloneable** **architecture is incompatible with normal use of final fields referring to mutable objects**, except in cases where the mutable objects may be safely shared between an object and its clone. In order to make a class cloneable, it may be necessary to remove final modifiers from some fields.

需要注意的是，我们不需要将elements.clone的结果转换为Object[]。因为调用对象上的clone返回的数组的编译时类型和原始数组的类型一致。这是复制数组的最佳方式。实际上，数组是clone方法的唯一吸引人的用法。

还需要注意的是，如果elements域是final的，前面的方法就不能正常工作，因为在其clone方法里，不能将新的值赋给一个final域。这是一个根本问题：就像序列化一样。**Cloneable的架构和指向可变对象的final域的正常用法是不兼容的，**除非，这个可变对象可以在源对象和其复制对象中安全的共用。为了使得一个类可clone，就有必要将一些域的final修饰符去掉。

> It is not always sufficient merely to call clone recursively. For example, suppose you are writing a clone method for a hash table whose internals consist of an array of buckets, each of which references the first entry in a linked list of key-value pairs. For performance, the class implements its own lightweight singly linked list instead of using java.util.LinkedList internally:

有的时候仅仅递归的调用clone方法还不够。比如，假如你要写一个hash表的clone方法，这个hash表包含一个散列桶数组，每个散列通指向键值对列表的第一项。处于性能的考虑，这个类自己实现了轻量级的链表，没有使用java.util.LinkedList。如下：

```java
public class HashTable implements Cloneable {
       private Entry[] buckets = ...;
       private static class Entry {
           final Object key;
           Object value;
           Entry  next;
           Entry(Object key, Object value, Entry next) {
               this.key   = key;
               this.value = value;
               this.next  = next;
					 } 
       }
       ... // Remainder omitted
   }
```

> Suppose you merely clone the bucket array recursively, as we did for Stack:

假设你只是循环地克隆了这个散列桶，就像Stack里的一样，如下：

```java
 // Broken clone method - results in shared mutable state!
   @Override public HashTable clone() {
       try {
           HashTable result = (HashTable) super.clone();
           result.buckets = buckets.clone();
           return result;
       } catch (CloneNotSupportedException e) {
           throw new AssertionError();
			 }
}
```

> Though the clone has its own bucket array, this array references the same linked lists as the original, which can easily cause nondeterministic behavior in both the clone and the original. To fix this problem, you’ll have to copy the linked list that comprises each bucket. Here is one common approach:

虽然克隆得到的hash表有自己的桶数组，但是这个数组引用的链表和原始的hash表一致，这将导致很容易地在clone的和原始的对象中出现不确定的行为。为了解决这个问题，必须要单独赋值组成每个链表的桶，下面是一个常见的实现：

```java
// Recursive clone method for class with complex mutable state
   public class HashTable implements Cloneable {
       private Entry[] buckets = ...;
       private static class Entry {
           final Object key;
           Object value;
           Entry  next;
           Entry(Object key, Object value, Entry next) {
               this.key   = key;
               this.value = value;
               this.next  = next;
					 }
					 // Recursively copy the linked list headed by this Entry
           Entry deepCopy() {
               return new Entry(key, value,
                   next == null ? null : next.deepCopy());
					 } }
       @Override public HashTable clone() {
           try {
               HashTable result = (HashTable) super.clone();
               result.buckets = new Entry[buckets.length];
               for (int i = 0; i < buckets.length; i++)
                   if (buckets[i] != null)
                       result.buckets[i] = buckets[i].deepCopy();
               return result;
           } catch (CloneNotSupportedException e) {
               throw new AssertionError();
           }
      }
       ... // Remainder omitted
   }
```

> The private class HashTable.Entry has been augmented to support a “deep copy” method. The clone method on HashTable allocates a new buckets array of the proper size and iterates over the original buckets array, deep-copying each nonempty bucket. The deepCopy method on Entry invokes itself recursively to copy the entire linked list headed by the entry. While this technique is cute and works fine if the buckets aren’t too long, it is not a good way to clone a linked list because it consumes one stack frame for each element in the list. If the list is long, this could easily cause a stack overflow. To prevent this from happening, you can replace the recursion in deepCopy with iteration:

私有类HashTable.Entry被增强了，添加了一个deepCopy方法。HashTable里的clone方法创建了一个大小合适的新的桶数组，然后循环遍历原始数组，在每个非空的节点上调用deepCopy方法，复制到新的数组里。Entry中的deepCopy方法通过递归调用自己来从头复制整个链表。虽然这种方式很简洁，当链表不长的时候也能正常工作，但是使用这种方式来clone数组并不是一个很好的方法，因为他会为链表中的每个元素生成栈帧。如果链表太长的话，可能会导致出现栈溢出。为了避免这个问题，你可以在deepCopy里使用迭代（iteration）来代替递归。

```java
// Iteratively copy the linked list headed by this Entry
   Entry deepCopy() {
      Entry result = new Entry(key, value, next);
      for (Entry p = result; p.next != null; p = p.next)
				p.next = new Entry(p.next.key, p.next.value, p.next.next);
      return result;
}
```

> A final approach to cloning complex mutable objects is to call super.clone, set all of the fields in the resulting object to their initial state, and then call higher- level methods to regenerate the state of the original object. In the case of our HashTable example, the buckets field would be initialized to a new bucket array, and the put(key, value) method (not shown) would be invoked for each key-value mapping in the hash table being cloned. This approach typically yields a simple, reasonably elegant clone method that does not run as quickly as one that directly manipulates the innards of the clone. While this approach is clean, it is antithetical to the whole Cloneable architecture because it blindly overwrites the field-by-field object copy that forms the basis of the architecture.

克隆复杂的可变对象的最后一种方法是，先调用super.clone方法，然后将返回对象的所有域设置成初始化状态，最后调用一些高级方法来重新生成对象的状态。在HashTable这个例子里，buckets域应该被初始化为一个新的bucket数组，然后对原始hashTable中的每一对键值对，在新的对象上调用put(key, value) 方法 (代码中未展示，可以自己想像)。这种做法写的clone方法，往往比较简单、合理、优美。但是运行速度可能比直接进行内部数据复制的clone方法慢一些。虽然这种方法很简单，但是它和整个Cloneable体系结构是对立的，因为它完全重写了Cloneable体系结构的一个一个域进行复制的机制。

> Like a constructor, a clone method must never invoke an overridable method on the clone under construction (Item 19). If clone invokes a method that is overridden in a subclass, this method will execute before the subclass has had a chance to fix its state in the clone, quite possibly leading to corruption in the clone and the original. Therefore, the put(key, value) method discussed in the previous paragraph should be either final or private. (If it is private, it is presumably the “helper method” for a nonfinal public method.)

和构造器类似，clone方法在构造过程中绝对不能调用可以被覆盖的方法（Item19）。如果clone方法调用了一个在子类中覆盖了的方法，那么这个方法就会再子类修正克隆对象的状态之前被调用，很有可能导致clone对象和原始对象不一致。因此，前面讨论的put(key, value)方法就应该是final的或者private的（如果一个方法是private的，那这个方法一般都是非final、public方法的“辅助方法”）。

> Object’s clone method is declared to throw CloneNotSupportedException, but overriding methods need not. **Public** **clone** **methods should omit the** **throws** **clause**, as methods that don’t throw checked exceptions are easier to use (Item 71).
>
> You have two choices when designing a class for inheritance (Item 19), but whichever one you choose, the class should *not* implement Cloneable. You may choose to mimic the behavior of Object by implementing a properly functioning protected clone method that is declared to throw CloneNotSupportedException. This gives subclasses the freedom to implement Cloneable or not, just as if they extended Object directly. Alternatively, you may choose *not* to implement a working clone method, and to prevent subclasses from implementing one, by providing the following degenerate clone implementation:

Object的clone方法会抛出CloneNotSupportedException，但是覆盖后的方法却不用 抛异常。公用的clone方法应该省略掉throws声明，因为不抛出受检异常的方法用起来更简单些。

<span id="Item13inheritance">当你在设计一个用来继承的类的时候，你有两种方法可以选择（item19）</span>，但不论你选择哪一种，都不应该实现Cloneable。你可以选择像Object的clone方法一样，实现一个功能合适、声明抛出CloneNotSupportedException的、protected的clone方法。它使得子类可以选择是否实现Cloneable接口，就像是直接继承的Object类一样。或者，你可以选择提供一个退化的clone实现如下，实现一个无效的clone方法，还可以防止子类实现。

```java
// clone method for extendable class not supporting Cloneable
@Override
protected final Object clone() throws CloneNotSupportedException {
       throw new CloneNotSupportedException();
   }

```

> There is one more detail that bears noting. If you write a thread-safe class that implements Cloneable, remember that its clone method must be properly synchronized, just like any other method (Item 78). Object’s clone method is not synchronized, so even if its implementation is otherwise satisfactory, you may have to write a synchronized clone method that returns super.clone().

还有一点需要注意。如果你要写一个线程安全的类并实现Cloneable，记住它的clone方法必须是同步的，和其他的方法一样（Item78）。Object的clone方法不是同步的，因此即使它的实现很满意，也需要写一个同步的clone方法来调用super.clone()。

> To recap, all classes that implement Cloneable should override clone with a public method whose return type is the class itself. This method should first call super.clone, then fix any fields that need fixing. Typically, this means copying any mutable objects that comprise the internal “deep structure” of the object and replacing the clone’s references to these objects with references to their copies. While these internal copies can usually be made by calling clone recursively, this is not always the best approach. If the class contains only primitive fields or references to immutable objects, then it is likely the case that no fields need to be fixed. There are exceptions to this rule. For example, a field representing a serial number or other unique ID will need to be fixed even if it is primitive or immutable.

总结一下，所有实现了Cloneable的类都必须要提供一个公用的方法来覆盖clone方法，且返回类型是这个类自己。这个方法第一步调用super.clone()，然后修正所有需要修正的域。通常来说，这就意味着要复制所有包含深层内部结构的可变对象，并且将clone对象中这些对象的引用替换成这些复制对象的引用。虽然这些内部复制通常可以通过递归调用clone方法来实现，但这并不是最好的方式。如果这个类只包含基本类型或者是不可变对象的引用，就基本没有域需要修正。这个规则也有例外，比如，一个表示序列号或者唯一ID的域，即使是基本数据类型或者不可变的，也需要被修正。

> Is all this complexity really necessary? Rarely. If you extend a class that already implements Cloneable, you have little choice but to implement a wellbehaved clone method. Otherwise, you are usually better off providing an alternative means of object copying. **A better approach to object copying is to provide a** **copy constructor or copy factory**.A copy constructor is simply a constructor that takes a single argument whose type is the class containing the constructor, for example,

这么复杂有必要吗？很少有必要。只有当你确实要继承一个已经实现了Cloneable的类时，除了实现一个好的clone方法以外，别无选择。其他情况下，你通常可以提供一个其他的方法来代替使用对象复制。**相对于对象复制，更好的方法是提供一个复制构造器或者复制工厂。**一个复制构造器就是一个构造器，只有一个类型和构造器所在类相同的参数，如下：

```java
// Copy constructor
   public Yum(Yum yum) { ... };
```

> A copy factory is the static factory (Item 1) analogue of a copy constructor:

一个复制工厂是类似复制构造器的一个静态工厂（Item1）。如下：

```java
// Copy factory
   public static Yum newInstance(Yum yum) { ... };
```

> The copy constructor approach and its static factory variant have many advantages over Cloneable/clone: they don’t rely on a risk-prone extralinguistic object creation mechanism; they don’t demand unenforceable adherence to thinly documented conventions; they don’t conflict with the proper use of final fields; they don’t throw unnecessary checked exceptions; and they don’t require casts.
>
> Furthermore, a copy constructor or factory can take an argument whose type is an interface implemented by the class. For example, by convention all general- purpose collection implementations provide a constructor whose argument is of type Collection or Map. Interface-based copy constructors and factories, more properly known as *conversion constructors* and *conversion factories*, allow the client to choose the implementation type of the copy rather than forcing the client to accept the implementation type of the original. For example, suppose you have a HashSet, s, and you want to copy it as a TreeSet. The clone method can’t offer this functionality, but it’s easy with a conversion constructor: new TreeSet<>(s).

相对于Cloneable/clone而言，复制构造器以及其静态工厂变体（即复制工厂）有以下几个优势：首先它们不依赖有风险的、语言之外的对象创建机制；其次它们不要求遵守还没有制定好的文档规范；然后它们也不会和final域的正常使用产生冲突；它们也不会抛出不必要的受检异常；最后它们还不需要转换类型。

此外，复制构造器或者工厂还可以使用这个实现的某个接口作为其参数类型。比如，按照约定，所有的集合类型的实现都提供了一个构造器，其参数类型是Collection或者Map。基于接口的复制构造器和工厂（更准确的叫法是转换构造器和转换工厂）允许客户端选择复制对象的实现类型，而不是强迫客户端使用原始类型。比如，假如你有一个HashSet，然后你想把它复制成一个TreeSet，这种情况下，clone方法就做不到了，但是使用转换构造器就很容易：new TreeSet<>(s)。

> Given all the problems associated with Cloneable, new interfaces should not extend it, and new extendable classes should not implement it. While it’s less harmful for final classes to implement Cloneable, this should be viewed as a performance optimization, reserved for the rare cases where it is justified (Item 67). As a rule, copy functionality is best provided by constructors or factories. A notable exception to this rule is arrays, which are best copied with the clone method.

既然所有的问题都和Cloneable接口有关，那么新的接口就不应该继承这个接口，新的可扩展的类也不应该实现它。即使对于final类而言，实现Cloneable危害比较小，但是这个应该被看做是性能优化，只有在极少数确实需要的情况下才使用。总之，复制功能最好是通过构造器或者工厂方法来实现，这个规则的例外是数组，其最好的复制方法就是clone方法。

### 