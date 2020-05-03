### Item43 方法引用优于lambdas

> The primary advantage of lambdas over anonymous classes is that they are more succinct. Java provides a way to generate function objects even more succinct than lambdas: *method references*. Here is a code snippet from a program that maintains a map from arbitrary keys to Integer values. If the value is interpreted as a count of the number of instances of the key, then the program is a multiset implementation. The function of the code snippet is to associate the number 1 with the key if it is not in the map and to increment the associated value if the key is already present:

lambda相对于匿名类最主要的优势就是要简洁一些。java还提供了一种生成函数对象的方法，比lambdas还要简洁一些，它就是方法引用。下面是来自一个包含一个从任意键到Integer值的映射的程序片段。如果这个值解释为键实例的个数的话，这个程序就是一个多集合实现。这个代码片段的功能是：如果这个键在map里不存在的话，就和1关联起来，如果已经存在的话，就递增其关联的值。代码如下：

```java
map.merge(key, 1, (count, incr) -> count + incr);
```

> Note that this code uses the merge method, which was added to the Map interface in Java 8. If no mapping is present for the given key, the method simply inserts the given value; if a mapping is already present, merge applies the given function to the current value and the given value and overwrites the current value with the result. This code represents a typical use case for the merge method.

注意这段代码中使用的merge方法，是Java8中添加到Map接口中的。如果map中没有给定的这个键，这个方法就会直接插入这个值；如果map里有这个键，merge方法就会在当前值和给定值上应用给定的函数，然后用得到的结果重写当前值。这段代码展示了merge方法的一个常见的使用例子。

> The code reads nicely, but there’s still some boilerplate. The parameters count and incr don’t add much value, and they take up a fair amount of space. Really, all the lambda tells you is that the function returns the sum of its two arguments. As of Java 8, Integer (and all the other boxed numerical primitive types) provides a static method sum that does exactly the same thing. We can simply pass a reference to this method and get the same result with less visual clutter:

这段代码读起来很不错，但是还是有样板代码。参数count和incr不会增加什么价值，还占用了一些空间。实际上，这个lambda表达式就是说这个返回会返回这两个参数的和。在Java8中，Integer（和其他的所有的数字类基本类型封装类）提供了一个静态方法sum，也能做到同样的事情。我们可以直接给这个方法传递一个引用，然后就可以达到同样的效果，同时还可以减少视觉上的混乱。代码如下：

```
map.merge(key, 1, Integer::sum);
```

> The more parameters a method has, the more boilerplate you can eliminate with a method reference. In some lambdas, however, the parameter names you choose provide useful documentation, making the lambda more readable and maintainable than a method reference, even if the lambda is longer.

一个方法包含的参数越多，使用方法引用能消除的样本代码也就却多。但是，在一些lambdas里，你命名的参数可以提供一些有用的信息，即使这个lambdas更长一些，但lambda的可读性和可维护性都比方法引用高一些。

> There’s nothing you can do with a method reference that you can’t also do with a lambda (with one obscure exception—see JLS, 9.9-2 if you’re curious). That said, method references usually result in shorter, clearer code. They also give you an out if a lambda gets too long or complex: You can extract the code from the lambda into a new method and replace the lambda with a reference to that method. You can give the method a good name and document it to your heart’s content.

只要你能用方法引用实现的，你都能使用lambda来做（有一个很复杂了例外，如果感兴趣的话，详见 JLS, 9.9-2 ）。也就是说，方法引用往往会使代码更加简短。如果lambda表达式太长，太复杂，也有另外一种选择：你可以从lambda里提取出新的代码然后放在一个新的方法里，并使用这个方法的而引用来代替lambda。你还可以给这个方法起个好名字，并按照你的意愿写说明文档。

> If you’re programming with an IDE, it will offer to replace a lambda with a method reference wherever it can. You should usually, but not always, take the IDE up on the offer. Occasionally, a lambda will be more succinct than a method reference. This happens most often when the method is in the same class as the lambda. For example, consider this snippet, which is presumed to occur in a class named GoshThisClassNameIsHumongous:

如果你在使用IDE进行变成，它可能会在所有能使用方法引用替换lambda的地方，提供这个的转换。在通常情况下（但不总是），你可以采用IDE的建议。有时候，lambda可能比方法引用更简洁一些。这种情况大都发生在这个方法和lambda在同一个类里的时候。比如，下面这段代码，假如它发生在一个名为GoshThisClassNameIsHumongous的类里：

```java
service.execute(GoshThisClassNameIsHumongous::action);
```

> The lambda equivalent looks like this:

它对应的lambda长这样：

```java
service.execute(() -> action());
```

> The snippet using the method reference is neither shorter nor clearer than the snippet using the lambda, so prefer the latter. Along similar lines, the Function interface provides a generic static factory method to return the identity function, Function.identity(). It’s typically shorter and cleaner *not* to use this method but to code the equivalent lambda inline: x -> x.

这个使用方法运用的代码片段还没有lambda的简洁清晰，因此应该使用lambda的。同样的还有，Function接口提供的泛型静态工厂方法Function.identity()，返回一个相同对象的函数。通常来说来没有直接使用lambda： x -> x更简洁。

> Many method references refer to static methods, but there are four kinds that do not. Two of them are *bound* and *unbound* instance method references. In bound references, the receiving object is specified in the method reference. Bound references are similar in nature to static references: the function object takes the same arguments as the referenced method. In unbound references, the receiving object is specified when the function object is applied, via an additional parameter before the method’s declared parameters. Unbound references are often used as mapping and filter functions in stream pipelines (Item 45). Finally, there are two kinds of *constructor* references, for classes and arrays. Constructor references serve as factory objects. All five kinds of method references are summarized in the table below:

虽然很多的方法引用都是静态方法，但是还有4中不是静态方法。其中的两种是有限制的和无限制的实例方法引用，在有限制的引用中，接受的对象是在方法引用中指定的。有限制的方法引用本质上类似于静态方法引用：函数随想的参数和被引用的方法相同。在无限制的引用中，接受的对象是在在函数对象应用时，通过在声明函数前面额外添加一个参数来指定的，无限制的引用通常被用在Stream管道中作为映射和过滤函数。最后，还有两种针对类和对象的构造器引用，构造器引用充当工厂对象。这五种方法引用都总结在下面这张表里了：

| 方法引用类型 | 例子                   | 对应的Lambda                                        |
| ------------ | ---------------------- | --------------------------------------------------- |
| 静态         | Integer::parseInt      | str -> Integer.parseInt(str)                        |
| 有限制       | Instant.now()::isAfter | Instant then = Instant.now();  t -> then.isAfter(t) |
| 无限制       | String::toLowerCase    | str -> str.toLowerCase()                            |
| 类构造器     | TreeMap<K,V>::new      | () -> new TreeMap<K,V>                              |
| 数组构造器   | int[]::new             | len -> new int[len]                                 |

> In summary, method references often provide a more succinct alternative to lambdas. **Where method references are shorter and clearer, use them; where they aren’t, stick with lambdas.**

总结一下，方法引用相对于lambdas，通常来说都更简洁明了。**如果方法引用更简洁明了，就用方法引用；如果方法引用并不简洁，就用lambda。**