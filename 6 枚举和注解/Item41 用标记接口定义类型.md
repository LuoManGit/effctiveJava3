### Item41 用标记接口定义类型

> A *marker interface* is an interface that contains no method declarations but merely designates (or “marks”) a class that implements the interface as having some property. For example, consider the Serializable interface (Chapter 12). By implementing this interface, a class indicates that its instances can be written to an ObjectOutputStream (or “serialized”).

标记接口是一个不包含任何方法声明的接口，只是用来指明（或者标记）实现了该类的接口具有某种属性。比如，Serializable接口（Chapter 12）。通过实现这个接口，类表明它的实例可以被写到ObjectOutputStream里（或者说，被序列化）。

> You may hear it said that marker annotations (Item 39) make marker interfaces obsolete. This assertion is incorrect. Marker interfaces have two advantages over marker annotations. First and foremost, **marker interfaces define a type that is implemented by instances of the marked class; marker annotations do not.** The existence of a marker interface type allows you to catch errors at compile time that you couldn’t catch until runtime if you used a marker annotation.

你可能听说过，标记注解的出现使得标记接口过时了。这种说法是不正确的，相对于标记注解，标记接口有两个优势。第一个也是最重要的一个，**标记接口定义的类型，是通过被标记类的实例来实现的，而标记注解就不是这样的。**标记接口类型的存在使得你在编译器就能发现一些错误，而如果你想用标记注解的话，这些错误，就要运行时才能发现了。

> Java’s serialization facility (Chapter 6) uses the Serializable marker interface to indicate that a type is serializable. The ObjectOutputStream.writeObject method, which serializes the object that is passed to it, requires that its argument be serializable. Had the argument of this method been of type Serializable, an attempt to serialize an inappropriate object would have been detected at compile time (by type checking). Compile-time error detection is the intent of marker interfaces, but unfortunately, the ObjectOutputStream.write API does not take advantage of the Serializable interface: its argument is declared to be of type Object, so attempts to serialize an unserializable object won’t fail until runtime.

Java的序列化机制使用Serializable标记接口，来表明这个类型是可序列化的。方法ObjectOutputStream.writeObject，用来将传进来的对象序列化，需要参数是可序列化的。把这个方法的参数类型设定为Serializable的话，当你试图序列化那些不合适的对象时，在编译时就会被（类型检查）检查出来。标记接口的目的就是进行编译时的错误检查，但是遗憾的是，ObjectOutputStream.write的API并没有利用Serializable接口的有四，它的参数被声明为Object类型，因此当试图序列化一个不可序列化的对象的时候，直到运行时才会失败。

> **Another advantage of marker interfaces over marker annotations is that they can be targeted more precisely.** If an annotation type is declared with target ElementType.TYPE, it can be applied to *any* class or interface. Suppose you have a marker that is applicable only to implementations of a particular interface. If you define it as a marker interface, you can have it extend the sole interface to which it is applicable, guaranteeing that all marked types are also subtypes of the sole interface to which it is applicable.

**标记接口相对于标记注解的另一个优点是，它们可以更精确地进行锁定。**如果一个注解类型声明其目标为ElementType.TYPE，它就可以被用在任何的类和接口上。假如你有一个标记只适合在某个特定接口的实现上使用。如果你将它定义为了一个标记接口，你就可以将唯一的接口扩展成合适的接口，保证所有被标记的接口都是这个唯一接口的子类型。

> Arguably, the Set interface is just such a *restricted marker interface*. It is applicable only to Collection subtypes, but it adds no methods beyond those defined by Collection. It is not generally considered to be a marker interface because it refines the contracts of several Collection methods, including add, equals, and hashCode. But it is easy to imagine a marker interface that is applicable only to subtypes of some particular interface and does *not* refine the contracts of any of the interface’s methods. Such a marker interface might describe some invariant of the entire object or indicate that instances are eligible for processing by a method of some other class (in the way that the Serializable interface indi- cates that instances are eligible for processing by ObjectOutputStream).

Set 接口可以说就是这样一个有限制的标记接口。它只适用于集合的子类型，又没有添加除了Collection定义的方法之外的所有方法。通常来说，我们不把Set接口当做是标记接口，因为它重新定义了几个Collection方法的约定，比如add，equals和hashCode。但是，不难想象有一个标记接口只适用于一些特定接口的子类型，也没有重新定义接口中任何方法的约定，这样的标记接口可以描述整个对象的约束条件，或者表明这些实例可以被其他类的方法处理（就像Serializable接口表明其实例可以通过ObjectOutputStream进行处理一样）。

> **The chief advantage of marker annotations over marker interfaces is that they are part of the larger annotation facility.** Therefore, marker annotations allow for consistency in annotation-based frameworks.

标记注解相对于标记接口的主要的有点是，他们是庞大的注解机制中的一部分。因此，注解标记在支持注解的框架中保持了一致性。

> So when should you use a marker annotation and when should you use a marker interface? Clearly you must use an annotation if the marker applies to any program element other than a class or interface, because only classes and interfaces can be made to implement or extend an interface. If the marker applies only to classes and interfaces, ask yourself the question “Might I want to write one or more methods that accept only objects that have this marking?” If so, you should use a marker interface in preference to an annotation. This will make it possible for you to use the interface as a parameter type for the methods in question, which will result in the benefit of compile-time type checking. If you can convince yourself that you’ll never want to write a method that accepts only objects with the marking, then you’re probably better off using a marker annotation. If, additionally, the marking is part of a framework that makes heavy use of annotations, then a marker annotation is the clear choice.

那么，什么时候应该使用标记注解，又什么时候应该使用标注借口呢？很显然，当这个标记需要应用在既不是类也不是接口的其他程序元素上的时候，你必须使用注解，因为只有类和接口才可以实现或者继承接口。如果这个标记只需要要应用在类和接口上，你就可以问自己一个问题：”我想写一些只接受 有这个标记的对象 的方法吗？“如果是的话，那么标记接口就应该优先于标记注解。因为这个使得你可以在 前面的问题中的方法里 使用接口作为参数的类型，这样就可以享受到类型检查的好处。如果你确信你不会编写只接受标记对象的方法，就应该优先使用标记注解。额外的，如果你使用的框架中使用了大量的注解用于标记，那么使用标记注解就是一个明智的选择。

> In summary, marker interfaces and marker annotations both have their uses. If you want to define a type that does not have any new methods associated with it, a marker interface is the way to go. If you want to mark program elements other than classes and interfaces or to fit the marker into a framework that already makes heavy use of annotation types, then a marker annotation is the correct choice. **If you find yourself writing a marker annotation type whose target is** **ElementType.TYPE, take the time to figure out whether it really should be an annotation type or whether a marker interface would be more appropriate.**

总结一下，标记接口和标记注解都有他们的用处。如果你想定义一个没有任何新的方法和它关联的类型（*？？？*）就应该选择标记接口。如果你想标记一个既不是类又不是皆苦的程序元素，或者标记需要使用于一个大量使用注解类型的框架，就选择标记注解。**如果发现自己写了一个标记注解类型，它的目标是ElementType.TYPE，就需要花点时候你搞清楚，确实应该是注解类型，还是标记接口要更合适些。**

> In a sense, this item is the inverse of Item 22, which says, “If you don’t want to define a type, don’t use an interface.” To a first approximation, this item says, “If you do want to define a type, do use an interface.”

从某种意义上来说，本节的说法就是Item22的说法是反着的。Item22说：”如果你不想定义一个类型，就不要使用接口。“本节大概是说：”如果你想定义个类型，就用接口。“（*怎么就反了啊啊啊？*）

















