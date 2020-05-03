### Item45 谨慎地使用Stream

> The streams API was added in Java 8 to ease the task of performing bulk operations, sequentially or in parallel. This API provides two key abstractions: the *stream*, which represents a finite or infinite sequence of data elements, and the *stream pipeline*, which represents a multistage computation on these elements. The elements in a stream can come from anywhere. Common sources include collections, arrays, files, regular expression pattern matchers, pseudorandom number generators, and other streams. The data elements in a stream can be object references or primitive values. Three primitive types are supported: int, long, and double.

在Java8中，添加了流API来简化串行和并行的大批量计算任务。这个API提供了两个关键的抽象：流，表示有限或者无限数据元素序列；流管道，表示在这些元素上的多步计算。stream中的元素可以来自任何位置，常见的来源包括集合，数组，文件，正则表达式模式匹配器，伪随机数生成器，或者其他的流。在流里的数据原始可以是对象引用，也可以是基本类型值，支持的三种基本类型是：int ,long, 和 double。

>A stream pipeline consists of a source stream followed by zero or more *intermediate operations* and one *terminal operation*. Each intermediate operation transforms the stream in some way, such as mapping each element to a function of that element or filtering out all elements that do not satisfy some condition. Intermediate operations all transform one stream into another, whose element type may be the same as the input stream or different from it. The terminal operation performs a final computation on the stream resulting from the last intermediate operation, such as storing its elements into a collection, returning a certain element, or printing all of its elements.

一个Stream Pipeline中包括一个源Stream，紧跟着0个或多个中间操作，和一个终止操作。每一个中间操作都使用一些方式对Stream进行转换，比如把每个元素映射到这个元素的函数上，或者过滤掉不满足条件的所有元素。中间操作总是把一个Stream转换成另一个Stream，它的元素类型可能和输入Stream相同，也可能不同。终止操作会对从最后一个中间操作得到的Stream进行最后的计算，比如把元素保存在集合中，返回一个特定的元素，或者打印所有的元素。

> Stream pipelines are evaluated *lazily*: evaluation doesn’t start until the terminal operation is invoked, and data elements that aren’t required in order to complete the terminal operation are never computed. This lazy evaluation is what makes it possible to work with infinite streams. Note that a stream pipeline without a terminal operation is a silent no-op, so don’t forget to include one.

Stream pipelines的计算都是lazy的：要知道终止操作被调用的时候，计算才会开始。完成终止操作不需要的数据元素永远都不会被计算到。这种lazy计算机制使得无限Stream成为可能。需要注意的是，一个没有终止计算的Stream pipeline就是一个静默的无操作指令，因此千万不要忘记写终止操作。

> The streams API is *fluent*: it is designed to allow all of the calls that comprise a pipeline to be chained into a single expression. In fact, multiple pipelines can be chained together into a single expression.

Stream的API是流式的：这样的设计使得pipeline中的所有的调用可以链接成一个表达式。事实上，多个pipeline也能链接成一个表达式。

> By default, stream pipelines run sequentially. Making a pipeline execute in parallel is as simple as invoking the parallel method on any stream in the pipeline, but it is seldom appropriate to do so (Item 48).

默认情况下，Stream pipeline都是串行地计算的。要让一个pipeline并行地计算，只需要在pipeline中的每一个Stream上都调用并行方法就可以了，但是只有很少的情况才适合这么做（Item48）。

> The streams API is sufficiently versatile that practically any computation can be performed using streams, but just because you can doesn’t mean you should. When used appropriately, streams can make programs shorter and clearer; when used inappropriately, they can make programs difficult to read and maintain. There are no hard and fast rules for when to use streams, but there are heuristics.

Stream 的API多种多样，足够用Stream进行所有的计算，但是可以做到，并不意味着你应该这做。当使用得当的时候，Stream可以让程序更加简洁明了；但是使用不当的时候，Stream可以让程序很难阅读和维护。使用Stream没有捷径，但是还是可以通过一些例子有所启发。

> Consider the following program, which reads the words from a dictionary file and prints all the anagram groups whose size meets a user-specified minimum. Recall that two words are anagrams if they consist of the same letters in a different order. The program reads each word from a user-specified dictionary file and places the words into a map. The map key is the word with its letters alphabetized, so the key for "staple" is "aelpst", and the key for "petals" is also "aelpst": the two words are anagrams, and all anagrams share the same alphabetized form (or *alphagram*, as it is sometimes known). The map value is a list containing all of the words that share an alphabetized form. After the dictionary has been processed, each list is a complete anagram group. The program then iterates through the map’s values() view and prints each list whose size meets the threshold:

来看下面这段程序，从字典文件中读取单词，然后打印所有的大小符合用户指定的最小值的通字母异序词组合。声明一下，同字母异序词就是包含相同字母但是顺序不同的两个词。这个程序会读取用户指定的字典文件中读取每一个单词，然后把它们放在map里。这个map的key就是这些字符的按字母顺序排列，比如staple的key就是aelpst，而petals的key也是aelpst，因此，staple和petals就是同字母异序词，并且所有的同字母异序词都有相同的按字母排序形式。这个map的值是一个列表，包含具有同一个按字母排序形式的所有的单词。但这个字典处理完以后，每个列表都是一个同字母异序词组。程序通过迭代map的value（）视图，就会以打印出大小满足阈值的每个list：

```java
// Prints all large anagram groups in a dictionary iteratively
public class Anagrams {
  public static void main(String[] args) throws IOException {
      File dictionary = new File(args[0]);
      int minGroupSize = Integer.parseInt(args[1]);
      Map<String, Set<String>> groups = new HashMap<>();
      try (Scanner s = new Scanner(dictionary)) {
				while (s.hasNext()) {
					String word = s.next(); 
        	groups.computeIfAbsent(alphabetize(word),
                       (unused) -> new TreeSet<>()).add(word);
        } 
      }
      for (Set<String> group : groups.values())
          if (group.size() >= minGroupSize)
						System.out.println(group.size() + ": " + group);
  }
  private static String alphabetize(String s) {
      char[] a = s.toCharArray();
      Arrays.sort(a);
      return new String(a);
  } 
}
```

> One step in this program is worthy of note. The insertion of each word into the map, which is shown in bold, uses the computeIfAbsent method, which was added in Java 8. This method looks up a key in the map: If the key is present, the method simply returns the value associated with it. If not, the method computes a value by applying the given function object to the key, associates this value with the key, and returns the computed value. The computeIfAbsent method simplifies the implementation of maps that associate multiple values with each key.

在这个程序中的一步值得一提。将每一个单词插入到map中的操作，使用了Java8中加入的computeIfAbsent方法。这个方法会在map中查找这个key：如果这个key存在，就返回和它关联的值；如果不存在，这个方法就会应用给定的函数对象计算出一个值，关联到这个key上，并返回这个值。computeIfAbsent方法进化了每个key关联多个值的map实现。

> Now consider the following program, which solves the same problem, but makes heavy use of streams. Note that the entire program, with the exception of the code that opens the dictionary file, is contained in a single expression. The only reason the dictionary is opened in a separate expression is to allow the use of the try-with-resources statement, which ensures that the dictionary file is closed:

现在来看看下面这段代码，它解决了同样的问题，但是过度地使用了Stream。主要注意的是，整个程序，除了打开文件的代码以外，都包括在了一个表达式里。把打开文件的代码放在单独的表达式里的唯一原因是为了允许使用try-with-resources代码块，这样才能保证字典文件的关闭。代码如下：

```java
// Overuse of streams - don't do this!
   public class Anagrams {
     public static void main(String[] args) throws IOException {
       Path dictionary = Paths.get(args[0]);
       int minGroupSize = Integer.parseInt(args[1]);
         try (Stream<String> words = Files.lines(dictionary)) {
           words.collect(
             groupingBy(word -> word.chars().sorted()
                         .collect(StringBuilder::new,
                           (sb, c) -> sb.append((char) c),
                           StringBuilder::append).toString()))
             .values().stream()
             .filter(group -> group.size() >= minGroupSize)
						 .map(group -> group.size() + ": " + group)
						 .forEach(System.out::println);
         } 
     }
}
```

> If you find this code hard to read, don’t worry; you’re not alone. It is shorter, but it is also less readable, especially to programmers who are not experts in the use of streams. **Overusing streams makes programs hard to read and maintain.**
>
> Luckily, there is a happy medium. The following program solves the same problem, using streams without overusing them. The result is a program that’s both shorter and clearer than the original:

你会发现这段代码很难阅读，不过不要担心，你不是孤单一个人。这个代码确实短一些，但是可读性太差了，尤其是对于那些不擅长使用Stream的程序员而言。**过度使用Stream会导致程序很难读也很难维护。**

幸运得是，这里有一个快乐的中间版本。下面这段程序也解决了相同的问题，使用了Stream但是没有过度使用，这样得到的程序比原始代码更短，也更清晰。代码如下：

```java
// Tasteful use of streams enhances clarity and conciseness
   public class Anagrams {
      public static void main(String[] args) throws IOException {
         Path dictionary = Paths.get(args[0]);
         int minGroupSize = Integer.parseInt(args[1]);
         try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(groupingBy(word -> alphabetize(word)))
							.values().stream()
							.filter(group -> group.size() >= minGroupSize) 
              .forEach(g -> System.out.println(g.size() + ": " + g));
         }
      }
      // alphabetize method is the same as in original version
   }
```

> Even if you have little previous exposure to streams, this program is not hard to understand. It opens the dictionary file in a try-with-resources block, obtaining a stream consisting of all the lines in the file. The stream variable is named words to suggest that each element in the stream is a word. The pipeline on this stream has no intermediate operations; its terminal operation collects all the words into a map that groups the words by their alphabetized form (Item 46). This is exactly the same map that was constructed in both previous versions of the program. Then a new Stream<List<String>> is opened on the values() view of the map. The elements in this stream are, of course, the anagram groups. The stream is filtered so that all of the groups whose size is less than minGroupSize are ignored, and finally, the remaining groups are printed by the terminal operation forEach.

即使你之前没怎么接触过stream，这段程序也不难懂。它在一个try-with-resources块里打开了一个字典文件，获取到了包含文件中所有行的stream。这个stream取名为words表示该stream里每一个元素都是一个单词。这个Stream上的pipeline没有中间操作，它的终止操作将所有的单词按照它们的按字母顺序形式分组（Item46）映射到一个map里。这里得到的map和前面两个版本得到的map是一样的。然后程序又在map的values()视图上打开了一个新的Stream。这个stream里面的元素就是同字母异序词组。这个stream经过了过滤，忽略了所有大小小于minGroupSize的词组。最后使用终止操作forEach打印了所有的剩下的词组。

> Note that the lambda parameter names were chosen carefully. The parameter g should really be named group, but the resulting line of code would be too wide for the book. **In the absence of explicit types, careful naming of lambda parameters is essential to the readability of stream pipelines.**

需要注意的是，这个lambda的参数的名字应该好好挑选。比如这个参数g本来应该用group，但是这样得到的代码的长度对于书本来说就太长了。**由于缺少显示类型，因此仔细地给lambda参数命名对于Stream pipeline的可读性而言十分重要。**

> Note also that word alphabetization is done in a separate alphabetize method. This enhances readability by providing a name for the operation and keeping implementation details out of the main program. **Using helper methods is even more important for readability in stream pipelines than in iterative code** because pipelines lack explicit type information and named temporary variables.

还需要注意的是，单词的按字母排序的计算是在一个单独的alphabetize方法里面进行的。这样通过给这个操作命名和把具体的实现放在main程序外，提高了代码的可读性。**在stream pipeline中使用辅助方法对可读性的帮助比在迭代代码中更大**，因为pipeline缺少显示类型信息和有名字的临时变量。

> The alphabetize method could have been reimplemented to use streams, but a stream-based alphabetize method would have been less clear, more difficult to write correctly, and probably slower. These deficiencies result from Java’s lack of support for primitive char streams (which is not to imply that Java should have supported char streams; it would have been infeasible to do so). To demonstrate the hazards of processing char values with streams, consider the following code:

这个alphabetize方法也可以通过stream来重新实现，但是基于stream的alphabetize方法不那么清晰，也很难写对，也可能更慢一些。造成这些缺点的原因在于Java不支持char基本类型的stream（这也并不一维这java应该支持char Stream，也不可能支持）。为了说明使用stream处理char值的危害，看看下面这段代码：

```java
"Hello world!".chars().forEach(System.out::print);
```

> You might expect it to print Hello world!, but if you run it, you’ll find that it prints 721011081081113211911111410810033. This happens because the elements of the stream returned by "Hello world!".chars() are not char values but int values, so the int overloading of print is invoked. It is admittedly confusing that a method named chars returns a stream of int values. You *could* fix the program by using a cast to force the invocation of the correct overloading:

你可能会希望打印出“Hello world！”,但是当你运行的时候，你会发现打印的是721011081081113211911111410810033。这个之所以会出现，是因为"Hello world!".chars()返回的stream的元素是int类型的而不是char类型的，因此调用的也是print的int重载。一个叫chars的方法返回的stream却是int值，这点却是很让疑惑。你可以使用一个转换来强制调用正确的重载方法，以解决这个问题。代码如下：

```
"Hello world!".chars().forEach(x -> System.out.print((char) x));
```

> but ideally you should **refrain from using streams to process** **char** **values.**

但是，最好还是不要使用stream来处理char值。

> When you start using streams, you may feel the urge to convert all your loops into streams, but resist the urge. While it may be possible, it will likely harm the readability and maintainability of your code base. As a rule, even moderately complex tasks are best accomplished using some combination of streams and iteration, as illustrated by the Anagrams programs above. So **refactor existing code to use streams and use them in new code only where it makes sense to do so.**

当你开始使用stream的时候，你可能会有强烈的欲望来把所有的步骤都转换为stream，但是请抑制住这种欲望。虽然这样做是可能的，但是这会严重损害代码的可读性和可维护性。一般来说，即使是相当复杂的任务，也是使用stream和迭代的结合来实现的，就像前面的Anagrams程序一样。因此**只在确实有意义的地方，才使用stream来重构代码或者在新的代码中使用stream。**

> As shown in the programs in this item, stream pipelines express repeated computation using function objects (typically lambdas or method references), while iterative code expresses repeated computation using code blocks. There are some things you can do from code blocks that you can’t do from function objects: 
>
> - From a code block,you can read or modify any local variable in scope;from a lambda, you can only read final or effectively final variables [JLS 4.12.4], and you can’t modify any local variables.
> - From a code block, you can return from the enclosing method, break or continue an enclosing loop, or throw any checked exception that this method is declared to throw; from a lambda you can do none of these things.

就像前面的代码展示的那样Stream pipeline是通过函数对象（通常使用lambda和方法引用）来表示重复的计算的，而在迭代的代码中使用代码块来表示重复的计算。下面这几件事只能通过代码块完成，不能通过函数对象来完成：

- 在代码块中，你可以读取、修改任何作用域内的本地变量；在在lambda里，你只能读取final或者有效final变量（*自声明后，没有修改过的变量*），并且不能修改任何本地变量。
- 在代码块中，你可以从外围方法中返回、在外围循环中break或者continue，或者抛出方法中声明要抛出的受检异常。而在lambda中，这些都做不到。

> If a computation is best expressed using these techniques, then it’s probably not a good match for streams. Conversely, streams make it very easy to do some things:
>
> - Uniformly transform sequences of elements 
> - Filter sequences of elements
> - Combine sequences of elements using a single operation (for example to add them, concatenate them, or compute their minimum)
> - Accumulate sequences of elements into a collection,perhaps grouping them by some common attribute
> - Search a sequence of elements for an element satisfying some criterion
>
> If a computation is best expressed using these techniques, then it is a good candidate for streams.

如果一个计算必须使用这些技术来表示，那么它就可能不适合stream。反之，stream会让下面这些情况变得很简单：

- 统一转换一系列的元素。
- 过滤元素序列。
- 使用单个操作来合并元素序列（比如，求和，连接，或者求最小值）。
- 将元素序列聚合到集合中，比如，按照一些公共属性进行分组。
- 在元素序列中寻找满足一些条件的元素。

如果一个计算使用这些技术就能很好的表示，那么它就非常适合用stream。

> One thing that is hard to do with streams is to access corresponding elements from multiple stages of a pipeline simultaneously: once you map a value to some other value, the original value is lost. One workaround is to map each value to a *pair object* containing the original value and the new value, but this is not a satisfying solution, especially if the pair objects are required for multiple stages of a pipeline. The resulting code is messy and verbose, which defeats a primary purpose of streams. When it is applicable, a better workaround is to invert the mapping when you need access to the earlier-stage value.

使用stream很难做到，从pipeline的多个状态中同时获取到对应的元素：一旦你把一个值映射成了另一个值，那个原始的值就丢了。有一种变通的发放就是把每个值都映射成包含原始值和新值的对象对，但是这并不是一个很好的解决方法，尤其是pipeline中有很多状态都需要对象对的时候，得到的代码就会很混乱繁杂，也就违背了stream的目的。当有需要访问较早阶段的值的时候，一个更好的变通方法是把这个映射倒过来。

> For example, let’s write a program to print the first twenty *Mersenne primes*. To refresh your memory, a *Mersenne number* is a number of the form 2^*p* − 1. If *p* is prime, the corresponding Mersenne number *may* be prime; if so, it’s a Mersenne prime. As the initial stream in our pipeline, we want all the prime numbers. Here’s a method to return that (infinite) stream. We assume a static import has been used for easy access to the static members of BigInteger:

举个例子，我们来写一个打印前20个梅森素数的程序。解释一下，梅森数就是一个形式为2^P-1的数，若p是素数，且对应梅森数也是素数，那么我们就称之为梅森素数。在我们的pipeline中的初始化stream中，我们想要所有的素数。下面是返回无限stream的方法，为了让代码简单一些，我们假设BigInteger中的而所有的静态成员都使用了静态导入。

```java
static Stream<BigInteger> primes() {
       return Stream.iterate(TWO, BigInteger::nextProbablePrime);
}
```

> The name of the method (primes) is a plural noun describing the elements of the stream. This naming convention is highly recommended for all methods that return streams because it enhances the readability of stream pipelines. The method uses the static factory Stream.iterate, which takes two parameters: the first element in the stream, and a function to generate the next element in the stream from the previous one. Here is the program to print the first twenty Mersenne primes:

primes方法的名字是一个复数名词，很好地描述了stream中的元素。推荐所有返回stream的方法都使用这个命名习惯，因为这样会增强stream pipeline的可读性。这个方法使用了静态工厂方法Stream.iterate，这个方法有两个参数，stream中的第一个元素，和用来根据前一个元素生成下一个元素的函数。下面是用来打印前20个梅森素数的程序：

```java
public static void main(String[] args) {
       primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
           .filter(mersenne -> mersenne.isProbablePrime(50))
           .limit(20)
           .forEach(System.out::println);
}
```

> This program is a straightforward encoding of the prose description above: it starts with the primes, computes the corresponding Mersenne numbers, filters out all but the primes (the magic number 50 controls the probabilistic primality test), limits the resulting stream to twenty elements, and prints them out.

这个程序是对上面的描述的简单的编码实现：它从primes开始，然后计算对应的梅森值，然后过滤出所有的素数（这个神奇的50用来控制概率素性测试），把得到的stream限制在20个元素，然后把它们打印出来。

> Now suppose that we want to precede each Mersenne prime with its exponent (*p*). This value is present only in the initial stream, so it is inaccessible in the terminal operation, which prints the results. Luckily, it’s easy to compute the ex- ponent of a Mersenne number by inverting the mapping that took place in the first intermediate operation. The exponent is simply the number of bits in the binary representation, so this terminal operation generates the desired result:

现在，假设我们还想在每个梅森素数之前打印它对应的指数p。这个值只存在与初始化的stream中，因此在打印结果的终止操作中访问不到。幸运地是，我们可以很容易地计算出梅森素数的指数，只需要将第一步中间操作中的映射计算反过来就好了。这个指数刚好就是他的二进制表示的位数，因此下面这个终止操作就可生成我们想要的结果。

```java
.forEach(mp -> System.out.println(mp.bitLength() + ": " + mp));
```

> There are plenty of tasks where it is not obvious whether to use streams or iteration. For example, consider the task of initializing a new deck of cards. Assume that Card is an immutable value class that encapsulates a Rank and a Suit, both of which are enum types. This task is representative of any task that requires computing all the pairs of elements that can be chosen from two sets. Mathematicians call this the *Cartesian product* of the two sets. Here’s an iterative implementation with a nested for-each loop that should look very familiar to you:

还是有很多任务，在使用stream还是迭代方面不是很明显。比如，一个初始化一副牌的任务，假定Card是一个包含枚举Rank和枚举Suit的不可变类。这个任务是一个典型的需要计算从两个集合中选择所有元素对的任务。数学上，称之为两个集合的笛卡尔积。下面是一个你很熟悉的，使用嵌套for-each的迭代实现。代码如下：

```java
// Iterative Cartesian product computation
   private static List<Card> newDeck() {
       List<Card> result = new ArrayList<>();
       for (Suit suit : Suit.values())
           for (Rank rank : Rank.values())
               result.add(new Card(suit, rank));
       return result;
   }
```

> And here is a stream-based implementation that makes use of the intermediate operation flatMap. This operation maps each element in a stream to a stream and then concatenates all of these new streams into a single stream (or *flattens* them). Note that this implementation contains a nested lambda, shown in boldface:

下面是一个基于stream的实现，使用了中间操作flatMap。这个操作将一个stream中的每一个元素都映射到另一个stream中，然后把这些新的stream合并到一个stream中（或者说，扁平化）。需要注意的是，这个实现包含一个嵌套的lambda，如粗体所示：

```java
// Stream-based Cartesian product computation
   private static List<Card> newDeck() {
       return Stream.of(Suit.values())
         .flatMap(suit ->
                  Stream.of(Rank.values())
                  		.map(rank -> new Card(suit, rank))) 
         .collect(toList());
   }
```

> Which of the two versions of newDeck is better? It boils down to personal preference and the environment in which you’re programming. The first version is simpler and perhaps feels more natural. A larger fraction of Java programmers will be able to understand and maintain it, but some programmers will feel more comfortable with the second (stream-based) version. It’s a bit more concise and not too difficult to understand if you’re reasonably well-versed in streams and functional programming. If you’re not sure which version you prefer, the iterative version is probably the safer choice. If you prefer the stream version and you believe that other programmers who will work with the code will share your preference, then you should use it.

这两个版本你的newDeck哪一个更好呢？这就取决于你的个人爱好和编程环境了。第一个版本很简单，看起来也更自然一些，大部分的程序员都能理解和维护它。但是也有一些程序员更喜欢第二个版本，它更加简洁，如果你熟悉stream和函数编程的话，看起来也不难理解。如果你不知道该选择哪个版本，迭代版本大概要更加安全一些。如果你更喜欢stream版本，你也相信后续使用这些代码的程序员也会喜欢这个版本，你就应该使用它。

> In summary, some tasks are best accomplished with streams, and others with iteration. Many tasks are best accomplished by combining the two approaches. There are no hard and fast rules for choosing which approach to use for a task, but there are some useful heuristics. In many cases, it will be clear which approach to use; in some cases, it won’t. **If you’re not sure whether a task is better served by streams or iteration, try both and see which works better.**

总结一下，一些任务最好使用stream完成，而有一些任务最好用迭代完成。很多的任务最好是结合这两种方法来完成。对于一个任务应该选用哪种方法，没有硬性，速成的规则，但是可以参考一些有意义的启发。在大部分情况下，选择哪一种是很清晰的；在一些情况下，就不清晰了。**如果你不能确定这个任务使用stream实现好还是迭代好，你可以都试一下，看看那个工作得好一些。**