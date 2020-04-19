### Item3 使用私有构造器或枚举类型来强化单例

> A singleton is simply a class that is instantiated exactly once [Gamma95]. Singletons typically represent either a stateless object such as a function (Item 24) or a system component that is intrinsically unique. **Making a class a singleton can make it difficult to test its clients** because it’s impossible to substitute a mock implementation for a singleton unless it implements an interface that serves as its type.

单例是指仅仅被实例化一次的类，通常用来表示一个无状态的对象，比如函数（Item24），或者本来就是唯一的系统组件。**使类成为单例会使得它的客户端难以调试**，因为除非这个单例实现一个可以充当其类型的接口，否则无法用模拟实现去替代这个单例。

> There are two common ways to implement singletons. Both are based on keeping the constructor private and exporting a public static member to provide access to the sole instance. In one approach, the member is a final field:

通常有两种方式实现单例。这两种方式都是将构造器私有化，然后提供一个公有的静态成员来访问这个唯一的实例。在第一种方式中，这个静态成员是final的，如下所示：

```java
// Singleton with public final field
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public void leaveTheBuilding() { ... } 
}
```

> The private constructor is called only once, to initialize the public static final field *Elvis.INSTANCE*. The lack of a public or protected constructor guarantees a “monoelvistic” universe: exactly one Elvis instance will exist once the Elvis class is initialized—no more, no less. Nothing that a client does can change this, with one caveat: a privileged client can invoke the private constructor reflectively (Item 65) with the aid of the *AccessibleObject.setAccessible* method. If you need to defend against this attack, modify the constructor to make it throw an exception if it’s asked to create a second instance.

代码中，私有的构造器只会在初始化公有静态final域 Elvis.INSTANCE 的时候被调用一次，public或者protected构造器的缺失保证了Elvis实例的全局唯一性，即Elvis实例一旦创建，就只存在一个实例，不多也不少。没有任何的客户端可以改变这点。但是有一个例外，一个拥有特权的客户端可以调用AccessibleObject.setAccessible方法通过反射调用private构造器。如果需要抵御这种攻击，可以修改构造器，使得他在第二次被调用的时候抛出异常。

> In the second approach to implementing singletons, the public member is a static factory method:

在第二种方式中，公有成员是一个静态工厂方法，代码如下：

```java
// Singleton with static factory
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public static Elvis getInstance() { 
        return INSTANCE; 
    }
    public void leaveTheBuilding() { ... } 
}
```

> All calls to *Elvis.getInstance* return the same object reference, and no other *Elvis* instance will ever be created (with the same caveat mentioned earlier).

代码中， 每次调用Elvis.getInstance都将返回一个相同的对象引用，并且其他的Elvis实例永远都不会被创建（同样地，除了前面提到的拥有特权的客户端以外）。

> The main advantage of the public field approach is that the API makes it clear that the class is a singleton: the public static field is final, so it will always contain the same object reference. The second advantage is that it’s simpler.
>
> One advantage of the static factory approach is that it gives you the flexibility to change your mind about whether the class is a singleton without changing its API. The factory method returns the sole instance, but it could be modified to return, say, a separate instance for each thread that invokes it. A second advantage is that you can write a generic singleton factory if your application requires it (Item 30). A final advantage of using a static factory is that a method reference can be used as a supplier, for example *Elvis::instance* is a *Supplier<Elvis>*. Unless one of these advantages is relevant, the public field approach is preferable.

第一种方式的主要的优势是其api能让人清晰地知道该类是一个单例，因为它的公有静态域有final的，它一直保持着相同的对象引用。第二种方式的优势是更简单。

采用第二种方式的一个优点是它非常灵活，比如当你想将该类改为不是单例时，可以不用修改其API；一般情况下，getInstance方法返回唯一实例，我们也可以进行修改，比如改成对每个调用它的线程返回不同实例。第二个优点是，如果有需要的话，你可以写一个泛型单例工厂（Item30）。最后一个优点是可以通过方法引用（method reference）来作为一个提供者，比如Elvis::getInstance 就是一个Supplier<Elvis>。在实际应用中，除非上述第二种方式的优点中的某一个很重要，否则还是建议使用一种方式。

> To make a singleton class that uses either of these approaches serializable(Chapter 12), it is not sufficient merely to add implements Serializable to its declaration. To maintain the singleton guarantee, declare all instance fields transient and provide a readResolve method (Item 89). Otherwise, each time a serialized instance is deserialized, a new instance will be created, leading, in the case of our example, to spurious Elvis sightings. To prevent this from happening, add this readResolve method to the Elvis class:

对于使用上述两种方式实现的单例而言，若要将其序列化，仅仅在声明中添加实现Serializable接口是不够的。为了保证单例，必须将所有的实例域声明为trasient，并且提供一个readResolve方法。否则，每次反序列化一个序列化的实例时，都将创建一个新的实例。比如在前面的例子里，将出现一个假冒的Elvis实例。为了防止这种情况出现，在Elvis类里添加readResolve方法如下：

```java
// readResolve method to preserve singleton property
private Object readResolve() {
// Return the one true Elvis and let the garbage collector // take care of the Elvis impersonator.
    return INSTANCE; 
}
```

> A third way to implement a singleton is to declare a single-element enum:

第三种实现单例的方式是声明一个包含单个元素的枚举，如下：

```java
// Enum singleton - the preferred approach
public enum Elvis { 
    INSTANCE;
    public void leaveTheBuilding() { ... } 
}
```

> This approach is similar to the public field approach, but it is more concise, provides the serialization machinery for free, and provides an ironclad guarantee against multiple instantiation, even in the face of sophisticated serialization or reflection attacks. This approach may feel a bit unnatural, but **a single-element enum type is often the best way to implement a singleton**. Note that you can’t use this approach if your singleton must extend a superclass other than Enum(though you can declare an enum to implement interfaces).

这种方式和第一种方式有点像，但是枚举类的实现更加简洁，并且提供了免费的序列化机制，同时及时是面对序列化或者反射攻击时，也能完全保证单例。这种实现虽然应用不广泛，但是**只有一个元素的枚举类型确实是最好的实现单例的方式。**需要注意的是，使用这种方式的单例类无法继承除了Enum外的其他父类（但可以声明枚举实现接口）。

### 