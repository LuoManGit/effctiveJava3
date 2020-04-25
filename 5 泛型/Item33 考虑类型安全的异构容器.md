### Item33 考虑类型安全的异构容器

> Common uses of generics include collections, such as Set<E> and Map<K,V>, and single-element containers, such as ThreadLocal<T> and AtomicReference<T>. In all of these uses, it is the container that is parameterized. This limits you to a fixed number of type parameters per container. Normally that is exactly what you want. A Set has a single type parameter, representing its element type; a Map has two, representing its key and value types; and so forth.

泛型最常见的使用有集合（比如Set<E>和Map<K,V>）和单个元素容器（比如ThreadLocal<T> 和AtomicReference<T>）。在这些用法中，都是被参数化了的容器。限制了每个容器只能有固定数目的类型参数。一般来说，这正是你想要的。Set只有一个类型参数，表示其元素类型，Map有两个类型参数，表示其键和值的类型，等等。

> Sometimes, however, you need more flexibility. For example, a database row can have arbitrarily many columns, and it would be nice to be able to access all of them in a typesafe manner. Luckily, there is an easy way to achieve this effect. The idea is to parameterize the *key* instead of the *container*. Then present the parameterized key to the container to insert or retrieve a value. The generic type system is used to guarantee that the type of the value agrees with its key.

但是有时候，你可能需要更多的灵活性。比如，数据库行有任意数量的列，如果这个列都可以类型安全的访问就太好了。幸运的是，有一种很简单的方法可以达到这个效果。这个方法就是用参数化键来替代参数化容器。然后将参数化的键交给容器，来插入或者获取值。泛型类型系统可以保证这个值的类型和它的键相符。

> As a simple example of this approach, consider a Favorites class that allows its clients to store and retrieve a favorite instance of arbitrarily many types. The Class object for the type will play the part of the parameterized key. The reason this works is that class Class is generic. The type of a class literal is not simply Class, but Class<T>. For example, String.class is of type Class<String>, and Integer.class is of type Class<Integer>. When a class literal is passed among methods to communicate both compile-time and runtime type information, it is called a *type token* [Bracha04].

下面简单的示范一下这个方法的例子，Favorites类的客户端可以保存并获取，任意多类型的一个最喜欢的实例。类型的Class对象就是参数化的key。这种方法可以正常工作的原因，在于class类是泛型的。类字面量类型不是简单的Class，而是Class<T>。比如，String.class的类型就是Class<String>，Integer.class的类型是Class<Integer>。当类字面量用在方法中，用来传递编译时和运行时类型信息的时候，被称为“类型令牌”[Beacha04]。

> The API for the Favorites class is simple. It looks just like a simple map, except that the key is parameterized instead of the map. The client presents a Class object when setting and getting favorites. Here is the API:

Favarites类的API很简单，除了他的key是参数化的以外，它看起来就像是一个简单的map。客户端在设置和获取最喜欢的实例时，传递一个Class对象。下面是他的API:

```java
 // Typesafe heterogeneous container pattern - API
   public class Favorites {
       public <T> void putFavorite(Class<T> type, T instance);
       public <T> T getFavorite(Class<T> type);
	 }
```

> Here is a sample program that exercises the Favorites class, storing, retrieving, and printing a favorite String, Integer, and Class instance:

下面是一个Favorite类的简单的使用，它保存、获取、并打印了一个最喜欢的String、Integer、和Class实例：

```java
// Typesafe heterogeneous container pattern - client
   public static void main(String[] args) {
       Favorites f = new Favorites();
       f.putFavorite(String.class, "Java");
       f.putFavorite(Integer.class, 0xcafebabe);
       f.putFavorite(Class.class, Favorites.class);
     	 String favoriteString = f.getFavorite(String.class);
       int favoriteInteger = f.getFavorite(Integer.class);
       Class<?> favoriteClass = f.getFavorite(Class.class);
       System.out.printf("%s %x %s%n", favoriteString,
           favoriteInteger, favoriteClass.getName());
}
```

> As you would expect, this program prints Java cafebabe Favorites. Note, incidentally, that Java’s printf method differs from C’s in that you should use %n where you’d use \n in C. The %n generates the applicable platform-specific line separator, which is \n on many but not all platforms.

正如你所期望的那样，这个程序会打印“Java cafebabe Favorites”。需要注意的是，java的printf方法和C语言里面的有点不同，在C里需要使用\n的时候，而java里使用%n，这个%n用于特定平台生成行分隔符，而\n用于大部分的平台生成行分隔符，但是也并非全部。

> A Favorites instance is *typesafe*: it will never return an Integer when you ask it for a String. It is also *heterogeneous*: unlike an ordinary map, all the keys are of different types. Therefore, we call Favorites a *typesafe heterogeneous container*.

这个Favorites实例是类型安全的：当你请求一个String的时候，绝不会给你返回一个Integer。同时，它也是异构的，和普通的map不一样，它所有的key的类型都不同，因此我们把Favorites称为类型安全的异构容器。

> The implementation of Favorites is surprisingly tiny. Here it is, in its entirety:

Favorites的实现小得出奇，下面是它全部的代码：

```java
// Typesafe heterogeneous container pattern - implementation
public class Favorites {
	private Map<Class<?>, Object> favorites = new HashMap<>();
  public <T> void putFavorite(Class<T> type, T instance) {
    favorites.put(Objects.requireNonNull(type), instance);
	}
	public <T> T getFavorite(Class<T> type) { 
		return type.cast(favorites.get(type));
	}
}
```

> There are a few subtle things going on here. Each Favorites instance is backed by a private Map<Class<?>, Object> called favorites. You might think that you couldn’t put anything into this Map because of the unbounded wildcard type, but the truth is quite the opposite. The thing to notice is that the wildcard type is nested: it’s not the type of the map that’s a wildcard type but the type of its key. This means that every key can have a *different* parameterized type: one can be Class<String>, the next Class<Integer>, and so on. That’s where the heterogeneity comes from.

这里有一些有点微妙的事情。每个Favorites实例都包含一个名为favorites的私有Map<Class<?>, Object>对象。由于这里使用了无限制的通配符类型，你可能会觉得不能往这个Map里放任何的东西，但事实却是相反的。需要注意的是这个通配符是嵌套的：这个map不是通配符类型的，而他的key才是通配符类型的。这意味着每一个key可以拥有一个不同参数化类型：一个可以是Class<String>，另一个可是是Class<Integer>，等等。异构就是从这里来的。

> The next thing to notice is that the value type of the favorites Map is simply Object. In other words, the Map does not guarantee the type relationship between keys and values, which is that every value is of the type represented by its key. In fact, Java’s type system is not powerful enough to express this. But we know that it’s true, and we take advantage of it when the time comes to retrieve a favorite.

另外一个需要注意的是，这个favorites Map的值类型就是就简单的Object。换句话说，这个map并没有保证其键和值之间的类型关系，即不能保证每一个值就是其键代表的类型。事实上，Java的类型系统也没有强大到可以表示这个关系。但是，我们知道这就是事实，而且我们在获取favorite的时候额利用了这一点。

> The putFavorite implementation is trivial: it simply puts into favorites a mapping from the given Class object to the given favorite instance. As noted, this discards the “type linkage” between the key and the value; it loses the knowledge that the value is an instance of the key. But that’s OK, because the getFavorites method can and does reestablish this linkage.

putFavorite方法的实现很简单：它只是将给定的Class对象和给定的最喜欢的实例之间的映射方法哦了favorites 这个map里。需要注意的是，这个方法，把键和值之间的“类型关系”丢弃了；它丢失了值是键的一个实例这个信息。但是这样做是OK的，因为getFavorite方法可以重建这个关系，它也确实这么做了。

> The implementation of getFavorite is trickier than that of putFavorite. First, it gets from the favorites map the value corresponding to the given Class object. This is the correct object reference to return, but it has the wrong compile-time type: it is Object (the value type of the favorites map) and we need to return a T. So, the getFavorite implementation *dynamically casts* the object reference to the type represented by the Class object, using Class’s cast method.

getFavorite的实现要比putFavorite方法要复杂一些。首先它从favorites map里获取了给定的Class 对象对应的值，它返回的对象引用时正确的，但是它的编译时类型确实错误的：它是一个Object类型（即favorites map的值类型），而我么你需要返回一个T类型。因此getFavorite的实现使用Class的cast方法，将对象引用 动态地转换成了Class对象代表的类型。

> The cast method is the dynamic analogue of Java’s cast operator. It simply checks that its argument is an instance of the type represented by the Class object. If so, it returns the argument; otherwise it throws a ClassCastException. We know that the cast invocation in getFavorite won’t throw ClassCastException, assuming the client code compiled cleanly. That is to say, we know that the values in the favorites map always match the types of their keys.

这个cast方法是Java的cast操作的一个动态模拟。它简单的检查了这个参数是不是这个Class对象所表示的类，如果是的话，就返回这个参数；如果不是，就抛出ClassCastException。我们知道，只要客户端代码可以干干净净地编译，这个getFavorite里的cast调用就不会抛出ClassCastException。也就是说，我们知道这个favorites map里的值总是和他们的key相匹配的。

> So what does the cast method do for us, given that it simply returns its argument? The signature of the cast method takes full advantage of the fact that class Class is generic. Its return type is the type parameter of the Class object:

既然cast方法只是简单的返回了参数，那这个方法对我们来说有什么用呢？这个cast方法的签名很好的利用了这个Class类是泛型这一优势。它的返回类型就是Class对象的类型参数，如下：

```java
public class Class<T> {
       T cast(Object obj);
}
```

> This is precisely what’s needed by the getFavorite method. It is what allows us to make Favorites typesafe without resorting to an unchecked cast to T.

这个恰恰就是getFavorite方法所需要的。使得我们即可以保证Favorites类型安全，有不需要借助于非受检转换将其转换为T类型。

> There are two limitations to the Favorites class that are worth noting. First, a malicious client could easily corrupt the type safety of a Favorites instance, by using a Class object in its raw form. But the resulting client code would generate an unchecked warning when it was compiled. This is no different from a normal collection implementations such as HashSet and HashMap. You can easily put a String into a HashSet<Integer> by using the raw type HashSet (Item 26). That said, you can have runtime type safety if you’re willing to pay for it. The way to ensure that Favorites never violates its type invariant is to have the putFavorite method check that instance is actually an instance of the type represented by type, and we already know how to do this. Just use a dynamic cast:

这Favorites类有两点限制需要注意。第一个就是，恶意客户端可以通过使用原生Class对象很轻易地破坏Favorites实例的类型安全。但是这样的客户端代码在编译时会生成非受检的警告。这点和普通的集合（比如HashSet和HashMap）一样。通过HashSet的原生类型，你可以很容易的将String放到HashSet<Integer>里去。也就是说，如果愿意付出一点代价，就可以拥有运行时类型安全。保证Favorites永远不会破坏其类型约束的方法是：在putFavorite方法里检查这个实例的实例是不是就是这个Class对象所代表的类型，我们已经知道如何去检查了，直接用动态cast就好了。代码如下：

```java
// Achieving runtime type safety with a dynamic cast
public <T> void putFavorite(Class<T> type, T instance) {
     favorites.put(type, type.cast(instance));
}
```

> There are collection wrappers in java.util.Collections that play the same trick. They are called checkedSet, checkedList, checkedMap, and so forth. Their static factories take a Class object (or two) in addition to a collection (or map). The static factories are generic methods, ensuring that the compile-time types of the Class object and the collection match. The wrappers add reification to the col- lections they wrap. For example, the wrapper throws a ClassCastException at runtime if someone tries to put a Coin into your Collection<Stamp>. These wrappers are useful for tracking down client code that adds an incorrectly typed element to a collection, in an application that mixes generic and raw types.

在java.util.Collections中的一些集合包装器也使用了这样的技巧。他们被称为checkedSet、checkedList和checkedMap等等。它们的静态工厂方法，除了需要一个集合（或者map）以外，还需要一个Class对象（或者两）。这些静态工厂方法都是泛型方法，用来保证编译时Class对象的类型和集合类型匹配。包装类给他们包装的集合添加了具体化。比如，当有人想往Collection<Stamp>里放一个Coin的时候，包装类就会抛出ClassCastException。这些包装类可以在泛型和原生类型或者的应用中，用来追溯是那段客户端代码把错误的类型的元素添加到了集合中。

> The second limitation of the Favorites class is that it cannot be used on a non-reifiable type (Item 28). In other words, you can store your favorite String or String[], but not your favorite List<String>. If you try to store your favorite List<String>, your program won’t compile. The reason is that you can’t get a Class object for List<String>. The class literal List<String>.class is a syntax error, and it’s a good thing, too. List<String> and List<Integer> share a single Class object, which is List.class. It would wreak havoc with the internals of a Favorites object if the “type literals” List<String>.class and List<Integer>.class were legal and returned the same object reference. There is no entirely satisfactory workaround for this limitation.

Favorites类的第二个限制就是他们不能被用在不可具体化的类型中。也就是说，你可以保存你最喜欢的String或者String[]，但是不能保存你最喜欢List<String>。当你想存储你最喜欢的List<String>的时候，你的程序就无法编译。原因在于无法获取一个List<String>的Class对象。类字面量List<String>.class是一个语法错误，这是一个好事。List<String>和List<Integer>共用一个Class对象，就是List.class。如果"类型字面量"List<String>和List<Integer>是合法的，就会破坏Favorites对象的内部结构，对这两种字面量，返回同一个对象引用。对于这个限制，目前还没有让人满意的解决方案。

> The type tokens used by Favorites are unbounded: getFavorite and putFavorite accept any Class object. Sometimes you may need to limit the types that can be passed to a method. This can be achieved with a *bounded type token*, which is simply a type token that places a bound on what type can be represented, using a bounded type parameter (Item 30) or a bounded wildcard (Item 31).

Favorites里使用的类型令牌是无限制的：getFavorite和putFavorite都可以接受任何的Class对象。有的时候你需要限制传递给方法的类型，就可以使用有限制的类型令牌，也就是在类型令牌中，代表类型的地方使用有限制的类型参数（Item30），或者是有限制的通配符（Item31）。

> The annotations API (Item 39) makes extensive use of bounded type tokens. For example, here is the method to read an annotation at runtime. This method comes from the AnnotatedElement interface, which is implemented by the reflective types that represent classes, methods, fields, and other program elements:

注解API（Item39）中广泛使用有有限制的类型令牌。比如，下面这个方法可以在运行时读取注解。这个方法来自AnnotatedElement接口，是通过表示类、方法、域或者其他的程序元素的反射类型来实现的。方法声明如下：

```java
public <T extends Annotation> T getAnnotation(Class<T> annotationType);
```

> The argument, annotationType, is a bounded type token representing an annotation type. The method returns the element’s annotation of that type, if it has one, or null, if it doesn’t. In essence, an annotated element is a typesafe heterogeneous container whose keys are annotation types.

这个参数annotationType是一个表示Annotation类型的受限制的类型令牌。如果这个元素有这种类型的注解，这个方法就返回这个类型对应的注解，如果没有，就返回null。本质上，一个被注解的元素就是一个类型安全的异构容器，其key就是注解类型。

> Suppose you have an object of type Class<?> and you want to pass it to a method that requires a bounded type token, such as getAnnotation. You could cast the object to Class<? extends Annotation>, but this cast is unchecked, so it would generate a compile-time warning (Item 27). Luckily, class Class provides an instance method that performs this sort of cast safely (and dynamically). The method is called asSubclass, and it casts the Class object on which it is called to represent a subclass of the class represented by its argument. If the cast succeeds, the method returns its argument; if it fails, it throws a ClassCastException.

假如你有一个对象的类型是Class<?>，你想把这个传递给一个参数为受限制类型令牌的方法，比如getAnnotation方法，你可以将这个对象转化为Class<? extends Annotation>类型，但是这个转换是非受检的，在编译时可能会生成警告（Item27）。幸运的是class这个类提供了一个实例方法，可以安全（动态）地执行这种转换。这个方法名为asSubClass，它将调用这个方法的class对象表示的类，转换为其参数代表的类的一个子类。如果这个转换成功了，这个方法就会回它的参数；如果失败了，就抛出ClassCastException。

> Here’s how you use the asSubclass method to read an annotation whose type is unknown at compile time. This method compiles without error or warning:

下面展示了如何使用asSubclass方法，来读取一个编译时类型未知的注解。这个方法在编译时，不会有error和warning：

```java
// Use of asSubclass to safely cast to a bounded type token
static Annotation getAnnotation(AnnotatedElement element, String annotationTypeName) {
		Class<?> annotationType = null; // Unbounded type token 
  	try {
    		annotationType = Class.forName(annotationTypeName);
    } catch (Exception ex) {
        throw new IllegalArgumentException(ex);
    }
		return element.getAnnotation(annotationType.asSubclass(Annotation.class));
}
```

> In summary, the normal use of generics, exemplified by the collections APIs, restricts you to a fixed number of type parameters per container. You can get around this restriction by placing the type parameter on the key rather than the container. You can use Class objects as keys for such typesafe heterogeneous containers. A Class object used in this fashion is called a type token. You can also use a custom key type. For example, you could have a DatabaseRow type representing a database row (the container), and a generic type Column<T> as its key.

总结一下，集合API说明了泛型的一般用法，限制每个容器只能有固定数目的类型参数。你可以通过把可类型参数放在键上而不是容器上，来避开这一限制。你在类型安全的异构容器中，使用Class对象来表示键。以这种方式使用的Class对象被称为类型令牌。你也可以使用定制的键类型，比如，可以有一个DatabaseRow类型来表示数据库行（容器），然后使用泛型类型Column<T>来作为它的键。

















