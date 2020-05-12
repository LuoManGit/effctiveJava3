### Item55 谨慎地返回Optionals

> Prior to Java 8, there were two approaches you could take when writing a method that was unable to return a value under certain circumstances. Either you could throw an exception, or you could return null (assuming the return type was an object reference type). Neither of these approaches is perfect. Exceptions should be reserved for exceptional conditions (Item 69), and throwing an exception is expensive because the entire stack trace is captured when an exception is created. Returning null doesn’t have these shortcomings, but it has its own. If a method returns null, clients must contain special-case code to deal with the possibility of a null return, unless the programmer can *prove* that a null return is impossible. If a client neglects to check for a null return and stores a null return value away in some data structure, a NullPointerException may result at some arbitrary time in the future, at some place in the code that has nothing to do with the problem.

在Java8之前，当写一个在某种情景下无法返回值的方法时，有两种方法。你可以抛出异常，或者返回null（假设你返回的类型是对象引用类型）。这两种方法都不是完美的。异常需要额外的代码来处理，并且抛出异常的代价是很高的，因为在生成一个异常的时候，需要捕获整个栈轨迹。返回null虽然没有这些缺点，但是它有它自己的缺点。如果方法返回null，那么其客户端就必须包含特殊的代码来处理可能的null返回，除非这个程序员可以证明，不可能返回空。如果客户端并没有检查null返回，就把这个null返回值保存在了一些数据结构里，那么就可能会未来的某个时间点，在毫不相关的代码中，生成一个NullPointerException。

> In Java 8, there is a third approach to writing methods that may not be able to return a value. The Optional<T> class represents an immutable container that can hold either a single non-null T reference or nothing at all. An optional that contains nothing is said to be *empty*. A value is said to be *present* in an optional that is not empty. An optional is essentially an immutable collection that can hold at most one element. Optional<T> does not implement Collection<T>, but it could in principle.

在Java8中，还有第三种方法可以编写无法返回值的方法。Optional<T> 类表示一个可以持有单个非空T引用或者什么都没有的不可变容器。一个什么都没有的Optional被称为是空的。一个非空的Optional的值被称为存在。Optional本质上是一个最多可以持有一个元素的容器。Optional<T>并没有实现Collection<T>，虽然理论上可以实现。

> A method that conceptually returns a T but may be unable to do so under certain circumstances can instead be declared to return an Optional<T>. This allows the method to return an empty result to indicate that it couldn’t return a valid result. An Optional-returning method is more flexible and easier to use than one that throws an exception, and it is less error-prone than one that returns null.

对于理论上返回T值，但是在某些情况下无法返回值的方法，可以声明为返回一个Optional<T>。这样可以允许方法在无法返回合法值的时候返回一个空的结果，一个返回Optional的方法比抛出异常的方法更加灵活简单，比返回null的更不容易出错。

> In Item 30, we showed this method to calculate the maximum value in a collection, according to its elements’ natural order:

在Item30中，我们展示了这个方法，来根据集合元素的自然顺序计算最大值的方法：

```java
// Returns maximum value in collection - throws exception if empty
public static <E extends Comparable<E>> E max(Collection<E> c) { 
  		 if (c.isEmpty())
           throw new IllegalArgumentException("Empty collection");
       E result = null;
       for (E e : c)
           if (result == null || e.compareTo(result) > 0)
               result = Objects.requireNonNull(e);
       return result;
}
```

> This method throws an IllegalArgumentException if the given collection is empty. We mentioned in Item 30 that a better alternative would be to return Optional<E>. Here’s how the method looks when it is modified to do so:

如果给定的集合是空的，这个方法会抛出一个IllegalArgumentException。我们在Item30中提到过，还有一个更好的选择是染回Optional<E>，现在让我们来看看这样做的话，这个方法是什么样的：

```java
// Returns maximum value in collection as an Optional<E>
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
		if (c.isEmpty())
				return Optional.empty();
    E result = null;
    for (E e : c)
         if (result == null || e.compareTo(result) > 0)
              result = Objects.requireNonNull(e);
  	return Optional.of(result); 
}
```

> As you can see, it is straightforward to return an optional. All you have to do is to create the optional with the appropriate static factory. In this program, we use two: Optional.empty() returns an empty optional, and Optional.of(value) returns an optional containing the given non-null value. It is a programming error to pass null to Optional.of(value). If you do this, the method responds by throwing a NullPointerException. The Optional.ofNullable(value) method accepts a possibly null value and returns an empty optional if null is passed in. **Never return a null value from an** **Optional-returning method:** it defeats the entire purpose of the facility.

正如你所看到的那样，它直接返回了一个optional，你需要做的就是选择合适的静态工厂方法来创建一个optional。在这个程序中，我们使用了两个：Optional.empty() 用来返回一个空的optional；Optional.of(value)用来返回一个包含给定的非空值的optional。当给传递null给Optional.of(value)的时候，会出现程序错误。如果你这样做了，这个方法机会抛出一个NullPointerException。Optional.ofNullable(value)方法可以接受一个空值，当你传入空值的时候，它会返回一个空的optional。**永远不要从一个返回Optional的方法中返回一个null**：这样做就违背了这个技术的初衷。

> Many terminal operations on streams return optionals. If we rewrite the max method to use a stream, Stream’s max operation does the work of generating an optional for us (though we do have to pass in an explicit comparator):

在stream的很多终止操作中都返回了Optional。如果我们使用stream来重写max方法，这个Stream的max操作就会做生成optional的工作（还是需要传入一个显式的比较器）：

```java
// Returns max val in collection as Optional<E> - uses stream
   public static <E extends Comparable<E>>
           Optional<E> max(Collection<E> c) {
       return c.stream().max(Comparator.naturalOrder());
   }
```

> So how do you choose to return an optional instead of returning a null or throwing an exception? **Optionals are similar in spirit to checked exceptions** (Item 71), in that they *force* the user of an API to confront the fact that there may be no value returned. Throwing an unchecked exception or returning a null allows the user to ignore this eventuality, with potentially dire consequences. However, throwing a checked exception requires additional boilerplate code in the client.

那么怎么去选择返回optional，还是null，还是抛出异常呢？**Optional本质上和受检异常很相似**（Item71）。他们强迫API的用户必须面对没有值返回的事实。抛出非受检异常或者返回null，都允许用户忽略掉这种情况，使得代码一致存在潜在的问题。然而，抛出受检异常又需要客户端增加一些样板代码。

> If a method returns an optional, the client gets to choose what action to take if the method can’t return a value. You can specify a default value:

如果一个方法返回的是一个optional，客户端就必须选择如果这个方法不能返回值的时候，应该采取什么操作。你可以指定一个默认的值，如下：

```java
// Using an optional to provide a chosen default value 
String lastWordInLexicon = max(words).orElse("No words...");
```

> or you can throw any exception that is appropriate. Note that we pass in an exception factory rather than an actual exception. This avoids the expense of creating the exception unless it will actually be thrown:

或者你可以抛出一个合适的异常，注意，我们只是传一个异常工厂，而不是传一个真正的异常，这样就可以避免创建异常的开销，除非真正需要抛出异常。代码如下：

```java
// Using an optional to throw a chosen exception
Toy myToy = max(toys).orElseThrow(TemperTantrumException::new);
```

> If you can *prove* that an optional is nonempty, you can get the value from the optional without specifying an action to take if the optional is empty, but if you’re wrong, your code will throw a NoSuchElementException:

如果你可以证明这个optional是非空的，你可以直接从optional里获取这个值，不需要指定当optional为空的时候的行为。但是如果你的证明是错误的，那么你的代码就会抛出NoSuchElementException。代码如下：

```java
// Using optional when you know there’s a return value 
Element lastNobleGas = max(Elements.NOBLE_GASES).get();
```

> Occasionally you may be faced with a situation where it’s expensive to get the default value, and you want to avoid that cost unless it’s necessary. For these situations, Optional provides a method that takes a Supplier<T> and invokes it only when necessary. This method is called orElseGet, but perhaps it should have been called orElseCompute because it is closely related to the three Map methods whose names begin with compute. There are several Optional methods for dealing with more specialized use cases: filter, map, flatMap, and ifPresent. In Java 9, two more of these methods were added: or and ifPresentOrElse. If the basic methods described above aren’t a good match for your use case, look at the documentation for these more advanced methods and see if they do the job.

有些时候你可能会遇到这样的情况，获取默认值的开销比较大，希望在没有必要的时候避免这个开销。针对这种情况，Optional提供了一个方法，参数为 Supplier<T>，只在需要的时候，才会调用它。这个方法的名字是orElseGet，但是可能叫orElseCompute要好一些，因为和它紧密相关的三个Map的方法的名称都是以compute开头的。还有几个Optional方法用来处理更加特殊的场景：filter，map，flatMap和ifPresent。在Java9中，又新增了两个方法：or和ifPresentOrElse。如果前面讲的这些基本方法都无法匹配你的使用场景，你可以查看文档寻找更加高级的方法，看看他们是不是满足需求。

> In case none of these methods meets your needs, Optional provides the isPresent() method, which may be viewed as a safety valve. It returns true if the optional contains a value, false if it’s empty. You can use this method to perform any processing you like on an optional result, but make sure to use it wisely. Many uses of isPresent can profitably be replaced by one of the methods mentioned above. The resulting code will typically be shorter, clearer, and more idiomatic.

如果没有一个方法可以满足你的需求，Optional提供了一个isPresent()方法，它可以被当做是一个安全阈。当这个optional包含一个值的时候就返回true，否则返回false。你可以根据这个方法的结果来执行任何你想要的处理，但是必须要确保使用正确。大部分的isPresent使用都可以使用前面的方法来有效地替代，得到的代码通常来说要更简单，清晰和符合语言习惯一些。

> For example, consider this code snippet, which prints the process ID of the parent of a process, or N/A if the process has no parent. The snippet uses the ProcessHandle class, introduced in Java 9:

比如，下面这段代码，打印一个进程的父进程ID，如果没有父进程打印N/A 。这个片段使用了Java9里新增的ProcessHandle类，如下：

```java
Optional<ProcessHandle> parentProcess = ph.parent(); 
System.out.println("Parent PID: " +
         (parentProcess.isPresent() ? String.valueOf(parentProcess.get().pid()) : "N/A"));
```

> The code snippet above can be replaced by this one, which uses Optional’s map function:

上面的这段代码可以使用下面这段使用了Optional的map函数的代码替代：

```java
System.out.println("Parent PID: " +
ph.parent().map(h -> String.valueOf(h.pid())).orElse("N/A"));
```

> When programming with streams, it is not uncommon to find yourself with a Stream<Optional<T>> and to require a Stream<T> containing all the elements in the nonempty optionals in order to proceed. If you’re using Java 8, here’s how to bridge the gap:

当使用stream编程的时候，你会常常发现你有一个Stream<Optional<T>> ，而你需要处理的是一个包含optional里面的非空的元素的Stream<T>。如果你使用Java8的话，可以使用下面这样的代码来搭桥：

```java
streamOfOptionals
       .filter(Optional::isPresent)
       .map(Optional::get)
```

> In Java 9, Optional was outfitted with a stream() method. This method is an adapter that turns an Optional into a Stream containing an element if one is present in the optional, or none if it is empty. In conjunction with Stream’s flatMap method (Item 45), this method provides a concise replacement for the code snippet above:

在Java9中，Optional还配有一个stream()方法。这个方法是一个适配器，可以把一个Optional转换为一个Stream，如果optional中存在值的话，这个stream包含一个元素；如果optional为空的话，就不包含元素。结合stream的flatMap方法（Item45），可以简洁地替代上面的代码:

```java
streamOfOptionals.
       .flatMap(Optional::stream)
```

> Not all return types benefit from the optional treatment. **Container types, including collections, maps, streams, arrays, and optionals should not be wrapped in optionals.** Rather than returning an empty Optional<List<T>>, you should simply return an empty List<T> (Item 54). Returning the empty container will eliminate the need for client code to process an optional. The ProcessHandle class does have the arguments method, which returns Optional<String[]>, but this method should be regarded as an anomaly that is not to be emulated.

并不是所有的返回类型都能受益于optional的返回类型。**容器类型，包括Collection，map，stream，数组，以及optional都不应该再用optional进行包装了。你应该放回一个空的List<T>，而不是空的Optional<List<T>>。返回一个空的容器可以让客户端不再需要去处理optional。ProcessHandle类确实有一个arguments方法，返回一个Optional<String[]>。这个方法应该被看作是反常的，不应该去模仿。

> So when should you declare a method to return Optional<T> rather than T? As a rule, **you should declare a method to return** **Optional** **if it might not be able to return a result** **and** **clients will have to perform special processing if no result is returned.** That said, returning an Optional<T> is not without cost. An Optional is an object that has to be allocated and initialized, and reading the value out of the optional requires an extra indirection. This makes optionals inappropriate for use in some performance-critical situations. Whether a particular method falls into this category can only be determined by careful measurement (Item 67).

那么什么时候应该将方法声明为返回Optional<T>而不是T呢？规则是**如果一个方法可能不能返回一个结果，并且客户端必须对没有结果的情况进行额外的处理时，就应该把方法声明为返回Optional。**也就是会说，返回一个Optional<T>不是没有开销的。Optional是一个必须要创建和初始化的对象，以及从optional里面读取值也需要额外的处理。这些都使得在一个性能要求严格的情况下，不太适合使用Optional。一个特定的方法是否属于这一类方法，必须经过严格的测试后才能确定（Item67）。

> Returning an optional that contains a boxed primitive type is prohibitively expensive compared to returning a primitive type because the optional has two levels of boxing instead of zero. Therefore, the library designers saw fit to provide analogues of Optional<T> for the primitive types int, long, and double. These optional types are OptionalInt, OptionalLong, and OptionalDouble. They contain most, but not all, of the methods on Optional<T>. Therefore, **you should never return an optional of a boxed primitive type,** with the possible exception of the “minor primitive types,” Boolean, Byte, Character, Short, and Float.

返回一个基本类型封装类的option的成本比返回基本类型要高得多。因为这样的optional是两层包装，而基本类型是0层包装。因此类库的设计者为基本类型int，long和double提供了模拟的 Optional<T>。这些optional类型是OptionalInt, OptionalLong, 和 OptionalDouble。他们包括了大部分而不是全部的Optional<T>里面的方法。因此，**你永远不应该返回基本类型封装类的optional，**小型基本类型”Boolean, Byte, Character, Short, 和 Float"除外。

> Thus far, we have discussed returning optionals and processing them after they are returned. We have not discussed other possible uses, and that is because most other uses of optionals are suspect. For example, you should never use optionals as map values. If you do, you have two ways of expressing a key’s logical absence from the map: either the key can be absent from the map, or it can be present and map to an empty optional. This represents needless complexity with great potential for confusion and errors. More generally, **it is almost never appropriate to use an optional as a key, value, or element in a collection or array.**

到目前为止，我们已经介绍了怎么返回optional以及返回后如何处理optional。我们没有讨论过其他的用法，因为大部分的其他用法都不靠谱。比如，你永远都不应该使用optional作为map 的值，如果你这么做的话，对于map中key的逻辑缺失就会有两种情况：一种是就是没有这个key，第二种就是存在这个key，对应的映射是一个空的optional。这样做，既增加没有用的复杂度，还让人迷惑，容易出错。更普遍地说，**永远不要使用optional作为key，value，或者集合或数组中的元素。

> This leaves a big question unanswered. Is it ever appropriate to store an optional in an instance field? Often it’s a “bad smell”: it suggests that perhaps you should have a subclass containing the optional fields. But sometimes it may be justified. Consider the case of our NutritionFacts class in Item 2. A NutritionFacts instance contains many fields that are not required. You can’t have a subclass for every possible combination of these fields. Also, the fields have primitive types, which make it awkward to express absence directly. The best API for NutritionFacts would return an optional from the getter for each optional field, so it makes good sense to simply store those optionals as fields in the object.

还有一个很大的问题，没有回答。把optional保存在一个实例域中是否合适？答案让人感觉到不太舒服：建议你在子类中包含optional数据域。有的时候，确实是合适的。比如Item2里面NutritionFacts类，一个NutritionFacts实例包含很多不需要的域，你不嗯呢该为每一个这些可能的域的组合写一个子类，并且，这些域还有基本类型，很难让其直接表示这种缺失。NutritionFacts最好的API就是在每个域的getter方法中返回一个optional，因此，在对象的域中使用optional直接保存，是很有意义的。

> In summary, if you find yourself writing a method that can’t always return a value and you believe it is important that users of the method consider this possibility every time they call it, then you should probably return an optional. You should, however, be aware that there are real performance consequences associated with returning optionals; for performance-critical methods, it may be better to return a null or throw an exception. Finally, you should rarely use an optional in any other capacity than as a return value.

总结一下，如果你发现你自己写的方法，不能总是返回一个值，并且，你认为这个方法的用户在每一次调用方法的时候，都必须要考虑到这种可能性，你可能就应该返回一个optional。然而，你也应该意识到，在返回optional的时候，会有性能问题，对于性能要求严格的方法，可能还是返回null或者抛出异常要更好一些。最后你尽量不要将optional用在除了返回值的其他地方。

