### Item46 优先选择Stream中无副作用的函数

> If you’re new to streams, it can be difficult to get the hang of them. Merely expressing your computation as a stream pipeline can be hard. When you succeed, your program will run, but you may realize little if any benefit. Streams isn’t just an API, it’s a paradigm based on functional programming. In order to obtain the expressiveness, speed, and in some cases parallelizability that streams have to offer, you have to adopt the paradigm as well as the API.

如果你刚刚接触到stream，要掌握它的精髓很困难。只是将自己的计算用stream pipeline来表示都已经很难了。即使你成功了，你的程序也可以运行，但是你可能感觉这么做也没有多大的好处。Stream不仅仅是一个API，它还是基于函数编程的范例。为了获得stream提供的表达性，速度，以及某些情况下的并行，就必须要像采用api一样采用范例。

> The most important part of the streams paradigm is to structure your computation as a sequence of transformations where the result of each stage is as close as possible to a *pure function* of the result of the previous stage. A pure function is one whose result depends only on its input: it does not depend on any mutable state, nor does it update any state. In order to achieve this, any function objects that you pass into stream operations, both intermediate and terminal, should be free of side-effects.

Stream 范例最重要的部分就是将你的计算分解成一系列的变换，这些变换每一步的结果都尽可能接近上一步接口的纯函数。纯函数是指结果只依赖输入的函数，它不依赖任何可变的状态，也不会修改任何状态。为了达到这个目的，你传递给stream的中间操作，和终止操作，都应该是没有副作用的。

> Occasionally, you may see streams code that looks like this snippet, which builds a frequency table of the words in a text file:

有时候，你可能会看到想下面这个片段的stream代码，它在一个text文件中创建了一个词频表：

```java
// Uses the streams API but not the paradigm--Don't do this!
   Map<String, Long> freq = new HashMap<>();
   try (Stream<String> words = new Scanner(file).tokens()) {
       words.forEach(word -> {
           freq.merge(word.toLowerCase(), 1L, Long::sum);
       }); 
   }
```

> What’s wrong with this code? After all, it uses streams, lambdas, and method references, and gets the right answer. Simply put, it’s not streams code at all; it’s iterative code masquerading as streams code. It derives no benefits from the streams API, and it’s (a bit) longer, harder to read, and less maintainable than the corresponding iterative code. The problem stems from the fact that this code is doing all its work in a terminal forEach operation, using a lambda that mutates external state (the frequency table). A forEach operation that does anything more than present the result of the computation performed by a stream is a “bad smell in code,” as is a lambda that mutates state. So how should this code look?

这段代码错在哪里了？毕竟，它用了stream，lambda和方法引用，还得到了正确的答案。简单来说，这根本就不是stream代码；是它的迭代代码伪装成了stream代码。它不能从stream API中获得任何的好处，而且比对应的迭代代码还要长一些，更难阅读，也不好维护。这个问题的根源在于这个代码在终止操作forEach里做了所有的工作，使用一个lambda修改了外部的状态（即词频表）。在forEach操作中不只是展示stream计算的结果，这在代码中并非好事。改变状态的lambda也是如此。那么正确的代码应该是什么样的呢？如下：

```java
// Proper use of streams to initialize a frequency table
   Map<String, Long> freq;
   try (Stream<String> words = new Scanner(file).tokens()) {
       freq = words.collect(groupingBy(String::toLowerCase, counting()));
}
```

> This snippet does the same thing as the previous one but makes proper use of the streams API. It’s shorter and clearer. So why would anyone write it the other way? Because it uses tools they’re already familiar with. Java programmers know how to use for-each loops, and the forEach terminal operation is similar. But the forEach operation is among the least powerful of the terminal operations and the least stream-friendly. It’s explicitly iterative, and hence not amenable to parallelization. **The** **forEach** **operation should be used only to report the result of a stream computation, not to perform the computation.** Occasionally, it makes sense to use forEach for some other purpose, such as adding the results of a stream computation to a preexisting collection.

这段代码做了和前面的代码相同的事情，但是这段代码才是正确地使用了StreamAPI，它也更加简洁明了。那么为什么还会有人以其他的方法编写呢？这是因为人们总是习惯于使用自己熟悉的工具。Java程序员都知道怎么使用for-each循环，然后这个forEach终止操作又差不多。然而这个forEach操作是终止操作里最没有威力的，对stream也不那么友好。它是显示迭代的，因此也不适用于并行。**这个forEach操作只能用来报告stream计算的结果，不应该用来执行计算。**有的时候，使用forEach来达到某些目的也是有意义的，比如往一个已经存在的集合中添加stream计算的结果。

> The improved code uses a *collector*, which is a new concept that you have to learn in order to use streams. The Collectors API is intimidating: it has thirty- nine methods, some of which have as many as five type parameters. The good news is that you can derive most of the benefit from this API without delving into its full complexity. For starters, you can ignore the Collector interface and think of a collector as an opaque object that encapsulates a *reduction* strategy. In this context, reduction means combining the elements of a stream into a single object. The object produced by a collector is typically a collection (which accounts for the name collector).

这个升级版的代码中使用了collector，这是你要使用stream必须学习的一个新的概念。这个CollectorsAPI有点可怕，它有39个方法，其中有一些方法有多达5个参数类型。好消息是，即使你没有完全搞懂这些API，你也可以从中获得绝大多数的好处。作为初学者，你可以忽略Collector接口，把collector想象成一个包含“聚合”策略的不透明对象。在这个上下文中，聚合的意思是把stream中的元素组合成单个对象。collector对象通常会生成一个集合（被称为名称收集器）。

> the collectors for gathering the elements of a stream into a true Collection are straightforward. There are three such collectors: toList(), toSet(), and toCollection(collectionFactory). They return, respectively, a set, a list, and a programmer-specified collection type. Armed with this knowledge, we can write a stream pipeline to extract a top-ten list from our frequency table.

把stream里的所有元素放在一个真的集合中很简单。有三个这样的集合：toList(), toSet(), 和toCollection(collectionFactory)。它们分别返回list，set和指定的集合类型。有了这些知识以后，我们就可以写一个stream pipeline来从前面的词频表里提取一个top10的列表了。

```java
// Pipeline to get a top-ten list of words from a frequency table
List<String> topTen = freq.keySet().stream() 
  .sorted(comparing(freq::get).reversed()) 
  .limit(10)
  .collect(toList());
```

> Note that we haven’t qualified the toList method with its class, Collectors. **It is customary and wise to statically import all members of** **Collectors because** **it makes stream pipelines more readable.**

需要注意的是，toList方法并没有指定它的类Collectors。**静态导入Collectors的所有成员是惯例，也是非常明智的，这样做可以提升stream pipeline的可读性。

> The only tricky part of this code is the comparator that we pass to sorted, comparing(freq::get).reversed(). The comparing method is a comparator construction method (Item 14) that takes a key extraction function. The function takes a word, and the “extraction” is actually a table lookup: the bound method reference freq::get looks up the word in the frequency table and returns the number of times the word appears in the file. Finally, we call reversed on the comparator, so we’re sorting the words from most frequent to least frequent. Then it’s a simple matter to limit the stream to ten words and collect them into a list.

这段代码中唯一有技巧的地方就是我们传递进去排序的comparator--comparing(freq::get).reversed()。这个comparing方法是一个比较器构造方法（Item14），带一个键提取函数。这个提取实际上是一个表查找，有限制的方法引用freq::get 在词频表里查找单词，并返回其在文件中出现的次数。最后我们在这个comparator上调用reversed，然后我们就把这些词按照出现频率进行了从高到低的排序。然后把这个stream限制在10个元素，然后放到list里就很简单了。

> The previous code snippets use Scanner’s stream method to get a stream over the scanner. This method was added in Java 9. If you’re using an earlier release, you can translate the scanner, which implements Iterator, into a stream using an adapter similar to the one in Item 47 (streamOf(Iterable<E>)).

前面的代码片段使用了Scanner的stream方法来获取scanner上的stream。这个方法是Java9里面添加的，如果你使用的版本早一些，你可以使用类似于Item47里面的适配器(streamOf(Iterable<E>))，把这个实现了Iterator的scanner转化为stream。

> So what about the other thirty-six methods in Collectors? Most of them exist to let you collect streams into maps, which is far more complicated than collecting them into true collections. Each stream element is associated with a key *and a value*, and multiple stream elements can be associated with the same key.

那么Collectors里剩下的36个方法又是什么呢？他们中大部分都是让你把stream聚合到map里的，这比把stream聚合到真的集合中，要复杂得多。每一个stream元素都和一个键和值相关，而且可能几个stream元素和同一个键相关。

> The simplest map collector is toMap(keyMapper, valueMapper), which takes two functions, one of which maps a stream element to a key, the other, to a value. We used this collector in our fromString implementation in Item 34 to make a map from the string form of an enum to the enum itself:

最简单的map收集器就是toMap(keyMapper, valueMapper)，它只有两个函数，一个把stream元素映射到key上，另一个映射到value上。在Item34的fromString的实现中使用了这个收集器，来创建一个枚举String形式到枚举本身的映射：

```java
// Using a toMap collector to make a map from string to enum
   private static final Map<String, Operation> stringToEnum =
       Stream.of(values()).collect(toMap(Object::toString, e -> e));
```

> This simple form of toMap is perfect if each element in the stream maps to a unique key. If multiple stream elements map to the same key, the pipeline will terminate with an IllegalStateException.

如果stream里的每一个元素都可以映射到不同的key上，那么toMap这种简单的形式就很完美。如果多个stream元素映射到同一个key上，pipeline就会以IllegalStateException终止。

> The more complicated forms of toMap, as well as the groupingBy method, give you various ways to provide strategies for dealing with such collisions. One way is to provide the toMap method with a *merge function* in addition to its key and value mappers. The merge function is a BinaryOperator<V>, where V is the value type of the map. Any additional values associated with a key are combined with the existing value using the merge function, so, for example, if the merge function is multiplication, you end up with a value that is the product of all the values associated with the key by the value mapper.

toMap的更加复杂的形式，和groupingBy方法一样，给了你很多方式来提供处理这种冲突的策略。一种方法就是在toMap方法中，除了key和value的mapper以外，提供一个合并的函数。这个合并的函数是一个BinaryOperator<V>。这个V表示的是map值的类型。和一个key相关的新增的值使用这个合并的函数和已经存在的值进行合并。因此，比如，这个合并函数时一个乘法，那么你的得到的值就是通过mapper和key关联的所有的value的积。

> The three-argument form of toMap is also useful to make a map from a key to a chosen element associated with that key. For example, suppose we have a stream of record albums by various artists, and we want a map from recording artist to best-selling album. This collector will do the job.

toMap的三个参数的形式还可以用来生成key到 与之关联的元素中被选中的值的映射。比如，我们现在有一个不同歌手的唱片记录的stream，我们想要一个从歌手到其最佳销量唱片的映射，这个collector就可以做到。代码如下：

```java
// Collector to generate a map from key to chosen element for key
Map<Artist, Album> topHits = albums.collect( 
  toMap(Album::artist, a->a, maxBy(comparing(Album::sales))));
```

> Note that the comparator uses the static factory method maxBy, which is statically imported from BinaryOperator. This method converts a Comparator<T> into a BinaryOperator<T> that computes the maximum implied by the specified comparator. In this case, the comparator is returned by the comparator construction method comparing, which takes the key extractor function Album::sales. This may seem a bit convoluted, but the code reads nicely. Loosely speaking, it says, “convert the stream of albums to a map, mapping each artist to the album that has the best album by sales.” This is surprisingly close to the problem statement.

需要注意的是，这个比较器使用 了静态工厂方法maxBy，是来自BinaryOperator的静态导入。这个方法吧一个Comparator<T>转换成一个BInaryOperation<T>，用来计算根据这个指定的比较器得到的最大值，在这个例子中，比较器是通过比较器构造方法comparing返回的，它的参数是键提取函数Album::sales。虽然这个看起来有些盘根错节，但是这个代码读起来非常清晰。大概来看，它就是说“把这个唱片的stream转换成一个map，这个map是每个歌手到其销量最佳的唱片的映射”。非常接近问题本身的陈述。

> Another use of the three-argument form of toMap is to produce a collector that imposes a last-write-wins policy when there are collisions. For many streams, the results will be nondeterministic, but if all the values that may be associated with a key by the mapping functions are identical, or if they are all acceptable, this collector’s s behavior may be just what you want:

三个参数的toMap形式还有一个用法，创建一个当出现冲突时，强制保存最后写入的数据的收集器。对于大部分stream而言，结果是不确定的，但是如果和一个key关联的所有的值都是相同的，或者都是可接受的，那么下面这个收集器的行为就正是你想要的：

```java
// Collector to impose last-write-wins policy
   toMap(keyMapper, valueMapper, (v1, v2) -> v2)
```

> The third and final version of toMap takes a fourth argument, which is a map factory, for use when you want to specify a particular map implementation such as an EnumMap or a TreeMap.
>
> There are also variant forms of the first three versions of toMap, named toConcurrentMap, that run efficiently in parallel and produce ConcurrentHashMap instances.

第三种也是最后一种toMap版本有四个参数，多出来的那个是map工厂，当你想指定一个特殊的map实现（比如EnumMap和TreeMap的时候就可以使用。

toMap的前面的三个版本都有对应的变体形式，名为toConcurrentMap，可以在并行状态下高效运行并生成ConcurrentHashMap实例。

> In addition to the toMap method, the Collectors API provides the groupingBy method, which returns collectors to produce maps that group elements into categories based on a *classifier function*. The classifier function takes an element and returns the category into which it falls. This category serves as the element’s map key. The simplest version of the groupingBy method takes only a classifier and returns a map whose values are lists of all the elements in each category. This is the collector that we used in the Anagram program in Item 45 to generate a map from alphabetized word to a list of the words sharing the alphabetization:

除了toMap方法以外，Collectors API还提供了groupingBy方法，可以返回生成map的collector，根据分类函数将元素分成几组。这个分类函数的参数是元素，返回这个元素所属的种类。这个种类就是这个元素在map中的key。groupingBy最简单的版本只有一个分类器，返回一个map，值为每个种类的所有元素的list。也就是我们在Item45的Anagram程序中使用的，用来生成从按字母排序的单词到字母排序相同的单词列表的映射的收集器。如下：

```java
words.collect(groupingBy(word -> alphabetize(word)))
```

> If you want groupingBy to return a collector that produces a map with values other than lists, you can specify a *downstream collector* in addition to a classifier. A downstream collector produces a value from a stream containing all the elements in a category. The simplest use of this parameter is to pass toSet(), which results in a map whose values are sets of elements rather than lists.

如果你想让groupBy返回的收集器创建的map的值不是list，你可以在分类器外，在指定一个下游收集器。这个收集器创建生成的值来自于一个包含某个类别中的所有元素的stream。这个参数最简单的用法就是传一个toSet()。得到的map的值就是元素的set而不是list了。

> Alternatively, you can pass toCollection(collectionFactory), which lets you create the collections into which each category of elements is placed. This gives you the flexibility to choose any collection type you want. Another simple use of the two-argument form of groupingBy is to pass counting() as the downstream collector. This results in a map that associates each category with the *number* of elements in the category, rather than a collection containing the elements. That’s what you saw in the frequency table example at the beginning of this item:

另外一种方法是传递toCollection(collectionFactory)，便可以自己创建存放每个类别的元素的集合了，给了你选择任何集合类型的灵活性。groupingBy的两个参数版本的另外一个简单的用法是传递一个counting作为下游收集器，这样在map中生成的和每个类别关联的结果就是这个类别的元素数量了，而不是包含这些元素的集合了。在本Item的前面的词频表例子中就使用了这个方法：

```java
Map<String, Long> freq = words .collect(groupingBy(String::toLowerCase, counting()));
```

> The third version of groupingBy lets you specify a map factory in addition to a downstream collector. Note that this method violates the standard telescoping argument list pattern: the mapFactory parameter precedes, rather than follows, the downStream parameter. This version of groupingBy gives you control over the containing map as well as the contained collections, so, for example, you can specify a collector that returns a TreeMap whose values are TreeSets.

groupBy的第三个版本除了下游收集器外，还允许你指定一个map工厂。需要注意的是这个方法违反了标准的可伸缩参数列表模式：mapFactory参数在downStream参数前面了。这个版本的Map，让你既可以控制map的类型也可以控制包含的集合的类型，比如，你可以指定一个收集器，返回一个值为TreeSet的TreeMap。

> The groupingByConcurrent method provides variants of all three overloadings of groupingBy. These variants run efficiently in parallel and produce ConcurrentHashMap instances. There is also a rarely used relative of groupingBy called partitioningBy. In lieu of a classifier method, it takes a predicate and returns a map whose key is a Boolean. There are two overloadings of this method, one of which takes a downstream collector in addition to a predicate.

groupingByConcurrent这个方法提供了groupingBy的三种重载的变体。这些变体可以高效地并行运行，并生成ConcurrentHashMap实例。还有一个和groupBy相关的很少使用的方法partitioningBy，它使用一个断言替代了分类函数，返回的map的key是一个Boolean。这个方法有两个重载，另外一个除了断言以外，还有一个下游收集器。

> The collectors returned by the counting method are intended *only* for use as downstream collectors. The same functionality is available directly on Stream, via the count method, so **there is never a reason to say** **collect(counting())**. There are fifteen more Collectors methods with this property. They include the nine methods whose names begin with summing, averaging, and summarizing (whose functionality is available on the corresponding primitive stream types). They also include all overloadings of the reducing method, and the filtering, mapping, flatMapping, and collectingAndThen methods. Most programmers can safely ignore the majority of these methods. From a design perspective, these collectors represent an attempt to partially duplicate the functionality of streams in collectors so that downstream collectors can act as “ministreams.”

counting方法返回的收集器只用在下游收集器中，在Stream中有类似的功能的方法count，**因此压根没有什么理由使用collect(counting())**。在Collectors的方法中有15个方法有这样的属性。其中9个方法的名字以summing, averaging, 和summarizing开头，相应的基本类型结尾（相应的基本类型就有这样的功能）。他们还包括reducing, filtering, mapping, flatMapping, 和collectingAndThen 方法的所有的重载。大部分程序员可以放心地忽略大部分这些方法。从设计者的角度来看，这些收集器试图在collectors里部分复制Stream的功能，以让下游收集器可以像一个“小stream”一样工作。

> There are three Collectors methods we have yet to mention. Though they are in Collectors, they don’t involve collections. The first two are minBy and maxBy, which take a comparator and return the minimum or maximum element in the stream as determined by the comparator. They are minor generalizations of the min and max methods in the Stream interface and are the collector analogues of the binary operators returned by the like-named methods in BinaryOperator. Recall that we used BinaryOperator.maxBy in our best-selling album example.

这里还有三个Collectors方法我们还没有提到，虽然他们在Collectors里，但是他们并不需要集合。前两个是maxBy和minBy，参数是一个Comparator，然后返回由这个比较器决定的stream里最大值和最小值。他们是Stream接口里max和min的粗略概括，也是collector对BinaryOperator里返回二元操作的同名的方法的模拟。回顾一下，我们在最佳销量唱片的例子里使用了BinaryOperator.maxBy方法。

> The final Collectors method is joining, which operates only on streams of CharSequence instances such as strings. In its parameterless form, it returns a collector that simply concatenates the elements. Its one argument form takes a single CharSequence parameter named delimiter and returns a collector that joins the stream elements, inserting the delimiter between adjacent elements. If you pass in a comma as the delimiter, the collector returns a comma-separated values string (but beware that the string will be ambiguous if any of the elements in the stream contain commas). The three argument form takes a prefix and suffix in addition to the delimiter. The resulting collector generates strings like the ones that you get when you print a collection, for example [came, saw, conquered].

最后一个Collector方法是joining，它只能在CharSequence实例（比如String）的stream上面使用。在它的无参数的版本中，它返回一个简单将元素串联起来的收集器。它的有一个参数的版本，有一个CharSequence参数名为delimiter（分隔符），返回的收集器，将所有的元素连接起来，并且在相邻的元素中间插入分隔符。如果你传了一个逗号作为分隔符，那么这个收集器就会返回一个逗号分隔的值字符串（但是，要小心的是，当元素中本身就包含逗号的时候，生成的字符串会有歧义）。它的三个参数的版本，除了delimiter以外，还有一个prefix和suffix参数。得到的收集器生成的字符串就和你打印集合的时候差不多，比如[came, saw, conquered]。

> In summary, the essence of programming stream pipelines is side-effect-free function objects. This applies to all of the many function objects passed to streams and related objects. The terminal operation forEach should only be used to report the result of a computation performed by a stream, not to perform the computation. In order to use streams properly, you have to know about collectors. The most important collector factories are toList, toSet, toMap, groupingBy, and joining.

总结一下，编写stream pipeline的本质是无副作用的函数对象。这适用于所有传入stream和相关对象的很多的函数对象。终止操作forEach只能用来报告stream运算的结果，不应该用来进行计算。为了要更好地使用stream，你必须要了解collector。其中最重要的collector工厂是toList，toSet，toMap，groupingBy，和joining。