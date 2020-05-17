### Item62 当其他类型合适的时候，避免使用字符串

> Strings are designed to represent text, and they do a fine job of it. Because strings are so common and so well supported by the language, there is a natural tendency to use strings for purposes other than those for which they were designed. This item discusses a few things that you shouldn’t do with strings.

String是专门设计来表示文本的，它在这方面也确实做得很好。由于String的使用很普遍，Java语言也支持得很好，所有当一些不适合使用字符串的时候，人们也会有一种自然的倾向选择使用字符创，本节中将介绍一些不应该使用字符串的场景。

> **Strings are poor substitutes for other value types.** When a piece of data comes into a program from a file, from the network, or from keyboard input, it is often in string form. There is a natural tendency to leave it that way, but this tendency is justified only if the data really is textual in nature. If it’s numeric, it should be translated into the appropriate numeric type, such as int, float, or BigInteger. If it’s the answer to a yes-or-no question, it should be translated into an appropriate enum type or a boolean. More generally, if there’s an appropriate value type, whether primitive or object reference, you should use it; if there isn’t, you should write one. While this advice may seem obvious, it is often violated.

**String不适合代替其他的值类型。**当程序中的数据从文件，网络，或者键盘输入的时候，通常都是String形式的。就很容易自然地让它保留这种形式。但是只有当这个数据确实是文本数据的时候，这种倾向才是合适的。如果是数值型的，它就应该被转换为合适的数值类型，比如int，float或者BigInteger。如果它是yes-or-no问题的答案，它就应该被转换为合适的枚举类型或者boolean。通常情况下，如果有一个合适的类型，不管是基本类型和式对象引用，你都应该使用它，如果没有的话，你就应该自己写一个。虽然这个建议看起来很明显，但是它经常被违反。

> **Strings are poor substitutes for enum types.** As discussed in Item 34, enums make far better enumerated type constants than strings.

**String不适合代替枚举类型。**正如Item34里介绍的那样，enum比String更适合表示枚举类型常量。

> **Strings are poor substitutes for aggregate types.** If an entity has multiple components, it is usually a bad idea to represent it as a single string. For example, here’s a line of code that comes from a real system—identifier names have been changed to protect the guilty:

**String不适合代替聚合类型。**如果一个实体拥有几个组件，通常来说，使用单个String来表示是一个非常不好的方法。比如，下面是一个来自真实系统的代码——为了避免纠纷，其标识符的名字已经被修改了：

```java
 // Inappropriate use of string as aggregate type
   String compoundKey = className + "#" + i.next();
```

> This approach has many disadvantages. If the character used to separate fields occurs in one of the fields, chaos may result. To access individual fields, you have to parse the string, which is slow, tedious, and error-prone. You can’t provide equals, toString, or compareTo methods but are forced to accept the behavior that String provides. A better approach is simply to write a class to represent the aggregate, often a private static member class (Item 24).

这种方法有很多的缺点。如果这个用来分割各个域的字符出现在了某个域里就会造成混乱。为了访问单独的域值，你必须要分解这个字符串，这个过程慢、单调乏味、容易出错。你也不能提供专门的eqauls、toString或者compareTo方法，只能被迫解决String提供的行为。一个更好的方法是简单的写一个类来表示这个聚合，通常情况下，这个类是一个私有静态成员类（Item24）。

> **Strings are poor substitutes for capabilities.** Occasionally, strings are used to grant access to some functionality. For example, consider the design of a thread-local variable facility. Such a facility provides variables for which each thread has its own value. The Java libraries have had a thread-local variable facility since release 1.2, but prior to that, programmers had to roll their own. When confronted with the task of designing such a facility many years ago, several people independently came up with the same design, in which client-provided string keys are used to identify each thread-local variable:

**String不适合代替能力表。**有的时候，String会用在对某种功能进行授权访问上。比如，线程本地变量机制的设计。这种机制针对每个变量，每个线程都有自己的值。Java类库中自1.2版本开始，就已经有了线程本地变量机制，但是在这之前，程序员都只能自己提供这样的机制。当很多年前，面对设计这种机制的任务的时候，有一些人都提出了同样的方法，使用客户端提供了String键来对每个局部变量进行访问。代码如下：

```java
// Broken - inappropriate use of string as capability!
   public class ThreadLocal {
       private ThreadLocal() { } // Noninstantiable
       // Sets the current thread's value for the named variable.
       public static void set(String key, Object value);
			 // Returns the current thread's value for the named variable.
       public static Object get(String key);
   }
```

> The problem with this approach is that the string keys represent a shared global namespace for thread-local variables. In order for the approach to work, the client-provided string keys have to be unique: if two clients independently decide to use the same name for their thread-local variable, they unintentionally share a single variable, which will generally cause both clients to fail. Also, the security is poor. A malicious client could intentionally use the same string key as another client to gain illicit access to the other client’s data.

这种方法的问题是，这些线程本地变量的String键共用了一个全局命名空间。为了让这种方法可以工作，各个客户端提供的String键就必须不一样。如果有两个客户端各自决定为其线程本地变量使用相同的名字，那么他们就会无意中共用了一个变量，这就会导致这两个客户端都出错。也因此，特别不安全。一个怀有恶意的客户端可能专门使用和其他客户端相同的string键，然后就可以非法地回去其他客户端的数据了。

> This API can be fixed by replacing the string with an unforgeable key (sometimes called a *capability*):

这个API可以用一个不可伪造的键（有时也称为能力）来代替这个String，以解决这个问题。代码如下：

```java
public class ThreadLocal {
       private ThreadLocal() { }  // Noninstantiable
       public static class Key {  // (Capability)
           Key() { }
       }
       // Generates a unique, unforgeable key
       public static Key getKey() {
           return new Key();
       }
       public static void set(Key key, Object value);
       public static Object get(Key key);
}
```

> While this solves both of the problems with the string-based API, you can do much better. You don’t really need the static methods anymore. They can instead become instance methods on the key, at which point the key is no longer a key for a thread-local variable: it *is* a thread-local variable. At this point, the top-level class isn’t doing anything for you anymore, so you might as well get rid of it and rename the nested class to ThreadLocal:

虽然这个API解决了基于String的API的两个问题，但是你还可以做得更好。你不在真的需要静态方法了。他们可以变成key上的实例方法，同时这个key也不再是一个线程本地变量的key，而是一个线程本地变量。在这种情况下，这个顶级类也没啥用了，因此你可以删除它，然后把嵌套类的名字改为ThreadLocal。代码如下：

```java
public final class ThreadLocal {
             public ThreadLocal();
             public void set(Object value);
             public Object get();
}
```

> This API isn’t typesafe, because you have to cast the value from Object to its actual type when you retrieve it from a thread-local variable. It is impossible to make the original String-based API typesafe and difficult to make the Key-based API typesafe, but it is a simple matter to make this API typesafe by making ThreadLocal a parameterized class (Item 29):

这个API不是类型安全的，因为当你从一个线程本地变量中读取出值的时候，还必须要把类型从Object转化为真正的类型。要把前面的基于String的API做成类型安全的是不可能的，要把基于Key的API做成类型安全的也很麻烦，但是要把上面这个API做成类型安全的是很简单的，只需要把ThreadLocal变成一个参数化类就可以了。代码如下：

```java
public final class ThreadLocal<T> {
             public ThreadLocal();
             public void set(T value);
             public T get();
}
```

> This is, roughly speaking, the API that java.lang.ThreadLocal provides. In addition to solving the problems with the string-based API, it is faster and more elegant than either of the key-based APIs.

粗略来说，这就是java.lang.ThreadLocal提供的API。除了解决了基于String的API的问题，相对于其他两种基于Key的API，这个更快也更优雅。

> To summarize, avoid the natural tendency to represent objects as strings when better data types exist or can be written. Used inappropriately, strings are more cumbersome, less flexible, slower, and more error-prone than other types. Types for which strings are commonly misused include primitive types, enums, and aggregate types.

总结一下，如果存在更加合适数据类型（或者可以编写一个）的时候，要避免习惯地使用字符串来表示对象。使用不当的话，String比其他类型会更加笨重、不灵活、慢、容易出错。很容易被错用成String的类型主要有基本类型、枚举类型和聚合类型。