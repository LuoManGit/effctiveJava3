### Item45 谨慎地使用Stream

> The streams API was added in Java 8 to ease the task of performing bulk operations, sequentially or in parallel. This API provides two key abstractions: the *stream*, which represents a finite or infinite sequence of data elements, and the *stream pipeline*, which represents a multistage computation on these elements. The elements in a stream can come from anywhere. Common sources include collections, arrays, files, regular expression pattern matchers, pseudorandom number generators, and other streams. The data elements in a stream can be object references or primitive values. Three primitive types are supported: int, long, and double.

在Java8中，添加了流API来简化串行和并行的大批量计算任务。这个API提供了两个关键的抽象：流，表示有限或者无限数据元素序列；流管道，表示在这些元素上的多步计算。stream中的元素可以来自任何位置，常见的来源包括集合，数组，文件，正则表达式模式匹配器，伪随机数生成器，或者其他的流。在流里的数据原始可以是对象引用，也可以是基本类型值，支持的三种基本类型是：int ,long, 和 double。

>A stream pipeline consists of a source stream followed by zero or more *intermediate operations* and one *terminal operation*. Each intermediate operation transforms the stream in some way, such as mapping each element to a function of that element or filtering out all elements that do not satisfy some condition. Intermediate operations all transform one stream into another, whose element type may be the same as the input stream or different from it. The terminal operation performs a final computation on the stream resulting from the last intermediate operation, such as storing its elements into a collection, returning a certain element, or printing all of its elements.

一个Stream Pipeline中包括一个源Stream，紧跟着0个或多个中间操作，和一个终止操作。每一个中间操作都使用一些方式对Stream进行转换，比如把每个元素映射到这个元素的函数上，或者过滤掉不满足条件的所有元素。中间操作总是把一个Stream转换成另一个Stream，它的元素类型可能和输入Stream相同，也可能不同。终止操作会对从最后一个中间操作得到的Stream进行最后的计算，比如把元素保存在集合中，返回一个特定的袁术，或者打印所有的元素。

> Stream pipelines are evaluated *lazily*: evaluation doesn’t start until the terminal operation is invoked, and data elements that aren’t required in order to complete the terminal operation are never computed. This lazy evaluation is what makes it possible to work with infinite streams. Note that a stream pipeline without a terminal operation is a silent no-op, so don’t forget to include one.

Stream pipelines的计算都是lazy的：要知道终止操作被调用的时候，计算才会开始。完成终止操作不需要的数据元素永远都不会被计算到。这种lazy计算机制使得无限Stream成为可能。需要注意的是，一个没有终止计算的Stream pipeline就是一个静默的无操作指令，因此千万不要忘记写终止操作。

> The streams API is *fluent*: it is designed to allow all of the calls that comprise a pipeline to be chained into a single expression. In fact, multiple pipelines can be chained together into a single expression.

Stream的API是流式的：这样的设计使得pipeline中的所有的调用可以链接成一个表达式。事实上，多个pipeline也能链接成一个表达式。

> By default, stream pipelines run sequentially. Making a pipeline execute in parallel is as simple as invoking the parallel method on any stream in the pipeline, but it is seldom appropriate to do so (Item 48).

默认情况下，Stream pipeline都是串行地计算的。要让一个pipeline并行地计算，只需要在pipeline中的每一个Stream上都调用并行方法就可以了，但是只有很少的情况才适合这么做（Item48）。

> The streams API is sufficiently versatile that practically any computation can be performed using streams, but just because you can doesn’t mean you should. When used appropriately, streams can make programs shorter and clearer; when used inappropriately, they can make programs difficult to read and maintain. There are no hard and fast rules for when to use streams, but there are heuristics.

Stream 的API多种多样，足够用Stream进行所有的计算，但是可以做到，并不意味着你应该这做。当使用得当的时候，Stream可以让程序更加简洁明了；但是使用不当的时候，Stream可以让程序很难阅读和维护。使用Stream没有捷径，但是还是可以有所启发。

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

> As shown in the programs in this item, stream pipelines express repeated com- putation using function objects (typically lambdas or method references), while iterative code expresses repeated computation using code blocks. There are some things you can do from code blocks that you can’t do from function objects: 
>
> - Fromacodeblock,youcanreadormodifyanylocalvariableinscope;froma lambda, you can only read final or effectively final variables [JLS 4.12.4], and you can’t modify any local variables.
> - From a code block, you can return from the enclosing method, break or continue an enclosing loop, or throw any checked exception that this method is declared to throw; from a lambda you can do none of these things.





















