### Item47 作为返回类型，Collection优先于Stream

> Many methods return sequences of elements. Prior to Java 8, the obvious return types for such methods were the collection interfaces Collection, Set, and List; Iterable; and the array types. Usually, it was easy to decide which of these types to return. The norm was a collection interface. If the method existed solely to enable for-each loops or the returned sequence couldn’t be made to implement some Collection method (typically, contains(Object)), the Iterable interface was used. If the returned elements were primitive values or there were stringent performance requirements, arrays were used. In Java 8, streams were added to the platform, substantially complicating the task of choosing the appropriate return type for a sequence-returning method.

有一些方法返回元素的序列。在Java8之前，这种方法的返回类型很明显，有 集合接口Collection，Set，List；Iterable；和数组类型。很容易就能决定返回的类型。通常都是集合接口；如果这个方法真能允许for-each循环或者返回的序列不能实现集合中的一些方法（通常来说，比如contains(Object))，那么就应该用Iterable；如果返回的元素是基本类型，或者有严格的性能要求，就用数组。在Java8里，平台里新增了stream，很大程度上使得这种返回序列的方法的返回类型的选择，变得更复杂了。

> You may hear it said that streams are now the obvious choice to return a sequence of elements, but as discussed in Item 45, streams do not make iteration obsolete: writing good code requires combining streams and iteration judiciously. If an API returns only a stream and some users want to iterate over the returned sequence with a for-each loop, those users will be justifiably upset. It is especially frustrating because the Stream interface contains the sole abstract method in the Iterable interface, and Stream’s specification for this method is compatible with Iterable’s. The only thing preventing programmers from using a for-each loop to iterate over a stream is Stream’s failure to extend Iterable.

你可能听说过，现在stream是返回元素序列最明显的选择了，但是正如Item45里面所讨论的那样，Stream并没有淘汰掉迭代，要写好的代码，还是需要巧妙地将stream和迭代结合起来。如果一个API返回的仅仅是一个stream，而用户想要在一个for-each循环里迭代这个返回序列，这时候，用户就会很苦恼。Stream接口中包含了Iterable接口中的唯一的抽象方法，Stream的该方法的声明也使用于Iterable。唯一使得程序员不能使用for-each循环来迭代stream的原因，是Stream并没有继承Iterable。

> Sadly, there is no good workaround for this problem. At first glance, it might appear that passing a method reference to Stream’s iterator method would work. The resulting code is perhaps a bit noisy and opaque, but not unreasonable:

遗憾的是，对于这个问题，也没有好的变通的方法。乍一看，好像传递一个Stream的iterable方法的方法引用好像可以工作，这样得到的代码虽然有些繁杂，但是也还是可以理解：

```java
// Won't compile, due to limitations on Java's type inference
for (ProcessHandle ph : ProcessHandle.allProcesses()::iterator) { 
		// Process the process
}
```

> Unfortunately, if you attempt to compile this code, you’ll get an error message:

不幸地是，当你编译这段代码的时候，会获得一个错误信息如下：

```java
Test.java:6: error: method reference not expected here
for (ProcessHandle ph : ProcessHandle.allProcesses()::iterator) {
                        ^
```

> In order to make the code compile, you have to cast the method reference to an appropriately parameterized Iterable:

为了让这段代码可以编译，你必须要把这个方法引用转换成一个合适的参数化的Iterable，如下:

```java
// Hideous workaround to iterate over a stream
   for (ProcessHandle ph : (Iterable<ProcessHandle>) ProcessHandle.allProcesses()::iterator)
```

> This client code works, but it is too noisy and opaque to use in practice. A better workaround is to use an adapter method. The JDK does not provide such a method, but it’s easy to write one, using the same technique used in-line in the snippets above. Note that no cast is necessary in the adapter method because Java’s type inference works properly in this context:

这个客户端代码可以工作，但是在实际使用中太麻烦，也太难懂了。一个更好的变通方法是使用一个适配方法，JDK并没有提供这样的方法，但是使用前面代码里用到的技术，这个方法很容易写。注意，在适配器方法中就不需要类型转换了，因为在这里的上下文中，java的类型推导可以完成这项工作：

```java
// Adapter from  Stream<E> to Iterable<E>
   public static <E> Iterable<E> iterableOf(Stream<E> stream) {
       return stream::iterator;
}
```

> With this adapter, you can iterate over any stream with a for-each statement:

有了这个适配器以后，你就可以在for-each循环里迭代任何stream了。

```java
for(ProcessHandle p:iterableOf(ProcessHandle.allProcesses())){ 
		// Process the process
}
```

> Note that the stream versions of the Anagrams program in Item 34 use the Files.lines method to read the dictionary, while the iterative version uses a scanner. The Files.lines method is superior to a scanner, which silently swallows any exceptions encountered while reading the file. Ideally, we would have used Files.lines in the iterative version too. This is the sort of compromise that programmers will make if an API provides only stream access to a sequence and they want to iterate over the sequence with a for-each statement.

注意，在Item45（*原书中是错误的*）中的Anagram程序的stream版本中使用了Files.lines方法来读取字典文件，而迭代的版本使用了Scanner。Files.lines方法比Scanner方法要好一些，因为Scanner会吞掉读取文件时遇到的所有的异常。理想的情况是，我们也能在迭代版本中使用File.lines。当API只能提供stream访问序列，而他们想通过for-each循环迭代序列的时候，这是程序员做出的妥协。

> Conversely, a programmer who wants to process a sequence using a stream pipeline will be justifiably upset by an API that provides only an Iterable. Again the JDK does not provide an adapter, but it’s easy enough to write one:

同样的，当API只提供了序列的Iterable，而程序员想使用stream pipeline来访问序列的时候，也会有些沮丧。同样JDK也没有提供适配器，但是也很容易写，如下：

```java
// Adapter from Iterable<E> to Stream<E>
   public static <E> Stream<E> streamOf(Iterable<E> iterable) {
       return StreamSupport.stream(iterable.spliterator(), false);
}
```

> If you’re writing a method that returns a sequence of objects and you know that it will only be used in a stream pipeline, then of course you should feel free to return a stream. Similarly, a method returning a sequence that will only be used for iteration should return an Iterable. But if you’re writing a public API that returns a sequence, you should provide for users who want to write stream pipelines as well as those who want to write for-each statements, unless you have a good reason to believe that most of your users will want to use the same mechanism.

如果你写了一个返回对象序列的方法，而且你知道它只会被用在stream里，那么你当然就应该返回stream。同样地，一个方法返回的序列如果只会被用来迭代，就应该放回一个Iterable。但是如果你写的是公开的返回序列的API，你应该既为那些想要用于stream的用户提供API，也应该为那些想要做for-each循环的用户提供API，除非你有很好的理由认定绝大部分你的用户都将使用同一种机制。

> The Collection interface is a subtype of Iterable and has a stream method, so it provides for both iteration and stream access. Therefore, **Collection** **or an appropriate subtype is generally the best return type for a public, sequence- returning method.** Arrays also provide for easy iteration and stream access with the Arrays.asList and Stream.of methods. If the sequence you’re returning is small enough to fit easily in memory, you’re probably best off returning one of the standard collection implementations, such as ArrayList or HashSet. But **do not store a large sequence in memory just to return it as a collection.**

Collection接口即是Iterable的子类型又有一个stream方法，因此他可以提供迭代和stream访问方法。因此，**对于公开的，返回序列的方法，其返回类型最好是Collection或者其他合适子类型。**通过Arrays.asList 和 Stream.of methods方法，数组也能提供简单迭代和stream访问。如果你返回的序列在内存中比较小，最好返回标准的集合实现，比如ArrayList或者HashSet。但是**千万不要在内存中保存巨大的序列，然后只是将它作为集合返回**。

> If the sequence you’re returning is large but can be represented concisely, consider implementing a special-purpose collection. For example, suppose you want to return the *power set* of a given set, which consists of all of its subsets. The power set of {*a*, *b*, *c*} is {{}, {*a*}, {*b*}, {*c*}, {*a*, *b*}, {*a, c*}, {*b*, *c*}, {*a*, *b*, *c*}}. If a set has *n* elements, its power set has 2*n*. Therefore, you shouldn’t even consider storing the power set in a standard collection implementation. It is, however, easy to implement a custom collection for the job with the help of AbstractList.

如果你要返回的序列很大，但是该序列可以准确地秒速，就可以考虑实现一个专用的集合。比如，你想返回一个给定的set的幂集，幂集包含它的所有的子集。比如{*a*, *b*, *c*} 的幂集就是 {{}, {*a*}, {*b*}, {*c*}, {*a*, *b*}, {*a, c*}, {*b*, *c*}, {*a*, *b*, *c*}}。如果一个set有n个元素，那么它的幂集就有2^n个元素。因此，你根本你就不应该考虑把这个幂集保存到标准的集合实现中。但是，使用AbstractList，我们可以很容易地实现一个定制的集合。

> The trick is to use the index of each element in the power set as a bit vector, where the *n*th bit in the index indicates the presence or absence of the *n*th element from the source set. In essence, there is a natural mapping between the binary numbers from 0 to 2*n* − 1 and the power set of an *n*-element set. Here’s the code:

这里的技巧是，使用每个元素在幂集中的索引作为一个位向量，索引中的第n位的值表示原集合中第n个元素是否存在。本质上来说，从0到2^n-1的二进制数到n个元素集合的幂集之间有一个自然映射。下面是代码：

```java
// Returns the power set of an input set as custom collection
   public class PowerSet {
      public static final <E> Collection<Set<E>> of(Set<E> s) {
         List<E> src = new ArrayList<>(s);
         if (src.size() > 30)
						throw new IllegalArgumentException("Set too big " + s); 
         return new AbstractList<Set<E>>() {
            @Override public int size() {
               return 1 << src.size(); // 2 to the power srcSize
            }
            @Override public boolean contains(Object o) {
               return o instanceof Set && src.containsAll((Set)o);
            }
            @Override public Set<E> get(int index) {
               Set<E> result = new HashSet<>();
               for (int i = 0; index != 0; i++, index >>= 1)
                  if ((index & 1) == 1)
                     result.add(src.get(i));
               return result;
            }
         }; 
      }
}
```

> Note that PowerSet.of throws an exception if the input set has more than 30 elements. This highlights a disadvantage of using Collection as a return type rather than Stream or Iterable: Collection has an int-returning size method, which limits the length of the returned sequence to Integer.MAX_VALUE, or 2^31 − 1. The Collection specification does allow the size method to return 2^31 − 1 if the collection is larger, even infinite, but this is not a wholly satisfying solution.

需要注意的是，当输入的set超过30个元素的时候，PowerSet.of方法会抛出一个异常。这就是使用集合作为返回类型，相对于使用Stream或者Iterable的一个主要的缺点：Collection有一个返回值为int的size方法，限制了返回的序列的长度必须小于等于Integer.MAX_VALUE 即2^31-1。如果集合更大，甚至无限长，集合规范确实允许size方法返回2^31-1，但是这并不是让人满意的解决方案。

> In order to write a Collection implementation atop AbstractCollection, you need implement only two methods beyond the one required for Iterable: contains and size. Often it’s easy to write efficient implementations of these methods. If it isn’t feasible, perhaps because the contents of the sequence aren’t predetermined before iteration takes place, return a stream or iterable, whichever feels more natural. If you choose, you can return both using two separate methods.

为了基于AbstractCollection写一个Collection实现，除了Iterable需要的方法外，你只需要实现两个方法：contains和size。要写这些方法的有效实现很简单。如果这个不可行，很可能是因为，在迭代出现之前，这个序列的内容不能确定，因此返回感觉比较自然的一个stream或者Iterable。如果你选择的话，你可以分别用两个方法来返回。

> There are times when you’ll choose the return type based solely on ease of implementation. For example, suppose you want to write a method that returns all of the (contiguous) sublists of an input list. It takes only three lines of code to generate these sublists and put them in a standard collection, but the memory required to hold this collection is quadratic in the size of the source list. While this is not as bad as the power set, which is exponential, it is clearly unacceptable. Implementing a custom collection, as we did for the power set, would be tedious, more so because the JDK lacks a skeletal Iterator implementation to help us.

很多时候，你都会根据简单的实现来选择返回类型。比如，假如你想写一个方法，返回输入list的所有的（相邻的）子列表。只需要三行代码，来生成子序列，把他们都放在一个标准集合里，但是来保存这些集合所需要的内存在原list大小的平方级，虽然没有指数级别的幂集那么可怕，但是很显然，也是不能接受的。像针对幂集那样实现一个特定的集合，又有些繁琐，尤其是JDK中没有提供基本的Iterator实现来帮助我们，就更麻烦了。

> It is, however, straightforward to implement a stream of all the sublists of an input list, though it does require a minor insight. Let’s call a sublist that contains the first element of a list a *prefix* of the list. For example, the prefixes of (*a*, *b*, *c*) are (*a*), (*a*, *b*), and (*a*, *b*, *c*). Similarly, let’s call a sublist that contains the last ele- ment a *suffix*, so the suffixes of (*a*, *b*, *c*) are (*a*, *b*, *c*), (*b*, c), and (*c*). The insight is that the sublists of a list are simply the suffixes of the prefixes (or identically, the prefixes of the suffixes) and the empty list. This observation leads directly to a clear, reasonably concise implementation:

但是，实现输入list的子序列的stream还是很简单的，虽然它需要一些洞察力。让我们把包含列表中第一个元素的子序列称为list的前缀，比如(*a*, *b*, *c*) 的前缀就是 (*a*), (*a*, *b*),  (*a*, *b*, *c*)。同样地，让我们把包含列表中最后一个元素的子列表称为后缀，那么(*a*, *b*, *c*) 的后缀就是 (*a*, *b*, *c*), (*b*, c), 和(*c*)。考察洞察力的地方就是一个元素的子列表就是前缀的后缀（或者说后缀的前缀）和一个空的集合。有了这样的观察，就可以得到一个简单，准确的实现了。代码如下：

```java
// Returns a stream of all the sublists of its input list
   public class SubLists {
      public static <E> Stream<List<E>> of(List<E> list) {
         return Stream.concat(Stream.of(Collections.emptyList()),
            prefixes(list).flatMap(SubLists::suffixes));
      }
      private static <E> Stream<List<E>> prefixes(List<E> list) {
         return IntStream.rangeClosed(1, list.size())
           .mapToObj(end -> list.subList(0, end));
      }
			private static <E> Stream<List<E>> suffixes(List<E> list) {
         return IntStream.range(0, list.size())
            .mapToObj(start -> list.subList(start, list.size()));
      } 
   }
```

> Note that the Stream.concat method is used to add the empty list into the returned stream. Also note that the flatMap method (Item 45) is used to generate a single stream consisting of all the suffixes of all the prefixes. Finally, note that we generate the prefixes and suffixes by mapping a stream of consecutive int values returned by IntStream.range and IntStream.rangeClosed. This idiom is, roughly speaking, the stream equivalent of the standard for-loop on integer indices. Thus, our sublist implementation is similar in spirit to the obvious nested for-loop:

注意，Stream.concat方法用来往返回的stream里添加空的list。还要注意，这个flatMap方法（详见Item45）是用来生成包含所有前缀的后缀的stream的。最后注意，生成前缀和后缀的方法是对，由IntStream.range 和IntStream.rangeClosed生成的连续int值的stream，映射得到的。通俗地说，这就是int索引上标准for循环的stream版本，因此我们子序列的实现本质和显示嵌套循环类似：

```java
for (int start = 0; start < src.size(); start++)
       for (int end = start + 1; end <= src.size(); end++)
           System.out.println(src.subList(start, end));
```

> It is possible to translate this for-loop directly into a stream. The result is more concise than our previous implementation, but perhaps a bit less readable. It is similar in spirit to the streams code for the Cartesian product in Item 45:

也可以把这个for循环直接转换为stream。这个实现比我们前面的实现要简洁一些，但是有点难读。本质上和Item45里面的笛卡尔积的stream代码很相似：

```java
// Returns a stream of all the sublists of its input list
   public static <E> Stream<List<E>> of(List<E> list) {
      return IntStream.range(0, list.size())
        .mapToObj(start ->
                  	IntStream.rangeClosed(start + 1, list.size())
                  	.mapToObj(end -> list.subList(start, end)))
        .flatMap(x -> x);
```

> Like the for-loop that precedes it, this code does *not* emit the empty list. In order to fix this deficiency, you could either use concat, as we did in the previous version, or replace 1 by (int) Math.signum(start) in the rangeClosed call.
>
> Either of these stream implementations of sublists is fine, but both will require some users to employ a Stream-to-Iterable adapter or to use a stream in places where iteration would be more natural. Not only does the Stream-to-Iterable adapter clutter up client code, but it slows down the loop by a factor of 2.3 on my machine. A purpose-built Collection implementation (not shown here) is considerably more verbose but runs about 1.4 times as fast as our stream-based implementation on my machine.

和前面的for循环的版本一样，这个代码不包含空的列表。为了解决这个问题，你可以向前面一样使用concat。或者在rangeClosed方法里使用Math.signum(start) 来替换1。

这几种版本的子列表的stream实现都还可以，但是在更适合用迭代的时候，还是需要用户实现一个Stream-to-Iterable适配器，或者直接使用stream。不仅仅是Stream-to-Iterable会打乱客户端代码，同时还会降低循环的速度，在作者的机器上，就降低了2.3倍。而一个特定的Collection实现（在这里没有展示），虽然代码冗长了一些，但是速度却比基于stream的实现快了1.4倍。

> In summary, when writing a method that returns a sequence of elements, remember that some of your users may want to process them as a stream while others may want to iterate over them. Try to accommodate both groups. If it’s feasible to return a collection, do so. If you already have the elements in a collection or the number of elements in the sequence is small enough to justify creating a new one, return a standard collection such as ArrayList. Otherwise, consider implementing a custom collection as we did for the power set. If it isn’t feasible to return a collection, return a stream or iterable, whichever seems more natural. If, in a future Java release, the Stream interface declaration is modified to extend Iterable, then you should feel free to return streams because they will allow for both stream processing and iteration.

总的来说，当你在写一个返回元素序列的方法的时候，记住，有一些人想要把他们当做stream来处理，而又有一些人想要进行迭代。要努力做到两种都顾及到。如果可以返回一个集合，就返回集合。如果你可以在集合中保存这些元素，并且序列中元素的数目比较小，那么创建一个新的集合，并返回标准的集合比如ArrayList。否则，就要考虑实现一个特定的集合，就像我们前面为幂集所做的那样。如果无法返回集合的话，就返回一个stream或者是iterable，哪个更自然就返回哪一个。如果在后面的java版本中，stream接口声明修改为继承了Iterable，你就可以直接返回stream了，因为他们即可以进行stream操作，又可以迭代。