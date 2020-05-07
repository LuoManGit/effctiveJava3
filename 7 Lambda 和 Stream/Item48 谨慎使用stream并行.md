### Item48 谨慎使用stream并行

> Among mainstream languages, Java has always been at the forefront of providing facilities to ease the task of concurrent programming. When Java was released in 1996, it had built-in support for threads, with synchronization and wait/notify. Java 5 introduced the java.util.concurrent library, with concurrent collections and the executor framework. Java 7 introduced the fork-join package, a high- performance framework for parallel decomposition. Java 8 introduced streams, which can be parallelized with a single call to the parallel method. Writing concurrent programs in Java keeps getting easier, but writing concurrent programs that are correct and fast is as difficult as it ever was. Safety and liveness violations are a fact of life in concurrent programming, and parallel stream pipelines are no exception.

在主流的编程语言中，Java一直走在简化并发编程的最前沿。自1996年Java发布是，便通过同步和wait/notify内置了对线程的支持。在Java5中，介绍了java.util.concurrent类库，包含了并发集合，和并发执行框架。在java7中，介绍了fork-join包，一个并行分解的高性能框架。在java8中介绍了stream，可以将一个并行方法的单个调用并行化。虽然在Java中编写并发程序变得越来越简单，但是写一个正确且高效的并发程序还是很难。安全性和活性问题一直都是并发编程需要面对的问题，并行的stream pipeline也不例外。

> Consider this program from Item 45:

回顾一下Item45里面的程序：

```java
// Stream-based program to generate the first 20 Mersenne primes
   public static void main(String[] args) {
       primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
           .filter(mersenne -> mersenne.isProbablePrime(50)
           .limit(20)
           .forEach(System.out::println);
   }
   static Stream<BigInteger> primes() {
       return Stream.iterate(TWO, BigInteger::nextProbablePrime);
   }
```

> On my machine, this program immediately starts printing primes and takes 12.5 seconds to run to completion. Suppose I naively try to speed it up by adding a call to parallel() to the stream pipeline. What do you think will happen to its performance? Will it get a few percent faster? A few percent slower? Sadly, what happens is that it doesn’t print anything, but CPU usage spikes to 90 percent and stays there indefinitely (a *liveness failure*). The program might terminate eventually, but I was unwilling to find out; I stopped it forcibly after half an hour.

在作者的机器上，这个程序会立即开始打印素数，运行完成需要花12.5秒。假如特别天真地通过在这个stream pipeline上添加一个parallel方法调用，来加快计算。你觉得对于其性能会有什么影响？会变得更快吗？还是更慢？遗憾的是，它没有打印任何的东西，而cpu的占用率却飙升到90%，并且一直无限期的保持着（活性失败）。这个程序可能最后还是会终止，但是作者却没有耐心了，在半个小时候，强行终止了它。

> What’s going on here? Simply put, the streams library has no idea how to parallelize this pipeline and the heuristics fail. Even under the best of circumstances, **parallelizing a pipeline is unlikely to increase its performance if the source is from** **Stream.iterate, or the intermediate operation** **limit** **is used.** This pipeline has to contend with *both* of these issues. Worse, the default parallelization strategy deals with the unpredictability of limit by assuming there’s no harm in processing a few extra elements and discarding any unneeded results. In this case, it takes roughly twice as long to find each Mersenne prime as it did to find the previous one. Thus, the cost of computing a single extra element is roughly equal to the cost of computing all previous elements combined, and this innocuous-looking pipeline brings the automatic parallelization algorithm to its knees. The moral of this story is simple: **Do not parallelize stream pipelines indiscriminately.** The performance consequences may be disastrous.

这个地方到底是怎么回事呢？简单地来说，就是streams类库不知道怎么并行化这个pipeline，也不知道怎么处理这个失败。即使是在最好的环境下，**如果来源是Stream.iterate，或者使用了中间操作limit，对pipeline进行并行化都不会对性能有所提升**。这个pipeline刚好两种情况都有。更糟糕的是，在默认的并行策略中，处理limit的不可预知情况的时候，假设多处理几个元素，然后抛弃掉多余的元素是不会有多大影响的。在我们的例子中，找到下一个梅森素数花费的时候是前一个需要的二倍。因此额外多计算一个元素花费的时间，就相当于前面所有的元素花的时间，这个看起来没什么问题的pipeline给自动并发算法带来了致命的打击。这个故事的教训很简单：**不要轻易将stream pipeline并行化**。它造成的性能影响可能是灾难性的。

> as a rule, **performance gains from parallelism are best on streams over** **ArrayList,** **HashMap,** **HashSet, and** **ConcurrentHashMap** **instances; arrays;** **int** **ranges; and** **long** **ranges.** What these data structures have in common is that they can all be accurately and cheaply split into subranges of any desired sizes, which makes it easy to divide work among parallel threads. The abstraction used by the streams library to perform this task is the *spliterator*, which is returned by the spliterator method on Stream and Iterable.

总之，**要想通过并行化来获得性能替身，stream最好是ArrayList，HashMap，HashSet，和ConcurrentHashMap实例；数组；int范围序列；和long范围序列**。它们的数据的共同点是，它们可以准确容易地分成任意大小的子序列，使得把工作分配到并行线程中变得容易。在stream类库中用来执行这个任务的抽象是spliterator，可以通过Stream和Iterable的spliterator方法返回。

>Another important factor that all of these data structures have in common is that they provide good-to-excellent *locality of reference* when processed sequentially: sequential element references are stored together in memory. The objects referred to by those references may not be close to one another in memory, which reduces locality-of-reference. Locality-of-reference turns out to be critically important for parallelizing bulk operations: without it, threads spend much of their time idle, waiting for data to be transferred from memory into the processor’s cache. The data structures with the best locality of reference are primitive arrays because the data itself is stored contiguously in memory.

这些数据还有的一个很重要的共同特征是，在进行序列处理的时候，他们都提供了很好的引用局部性：序列化的元素引用在内存中都保存在一起。这写引用所指的对象可能在内存中隔得比较远，这确实降低了引用局部性。事实证明，局部引用性对于大批量并行操作确实很重要：没有它的话，线程要花很多的时间，空闲等待数据从内存中转移到处理器的缓存中。引用局部性最好的数据结构就是基本类型数组了，因为他们的数据本身在内存中就是保存到一起的。

> The nature of a stream pipeline’s terminal operation also affects the effectiveness of parallel execution. If a significant amount of work is done in the terminal operation compared to the overall work of the pipeline and that operation is inherently sequential, then parallelizing the pipeline will have limited effectiveness. The best terminal operations for parallelism are *reductions*, where all of the elements emerging from the pipeline are combined using one of Stream’s reduce methods, or prepackaged reductions such as min, max, count, and sum. The *shortcircuiting* operations anyMatch, allMatch, and noneMatch are also amenable to parallelism. The operations performed by Stream’s collect method, which are known as *mutable reductions*, are not good candidates for parallelism because the overhead of combining collections is costly.

Stream pipeline的终止操作本质上也会影响并行执行的效率。如果相对于整个pipeline，在终止操作中完成了绝大部分的工作，并且这个操作有固有的顺序，那么并行化pipeline的效率就会受到限制。并行化最好的终止操作就是聚合，pipeline中出现的所有元素都通过一个Stream的reduce方法组合到一起，或者预先设置好的聚合方法比如max，min，count，和sum。这种骤死性的操作，比如anyMatch，allMatch，和noneMatch，也很适合并行化。通过Stream的collect方法执行的操作，称为“可变聚合”，并不是并行化的最好的选择，因为合并集合的成本很高。

> If you write your own Stream, Iterable, or Collection implementation and you want decent parallel performance, you must override the spliterator method and test the parallel performance of the resulting streams extensively. Writing high-quality spliterators is difficult and beyond the scope of this book.

如果你想写一个自己的Stream，Iterable 或者Collection实现，并且你希望并行时性能不错，你就必须覆盖spliterator方法，并且广泛测试得到的stream的并行性能。写一个高质量的spliterator太负责了，不在本书的讨论范围内。

> **Not only can parallelizing a stream lead to poor performance, including liveness failures; it can lead to incorrect results and unpredictable behavior** (*safety failures*). Safety failures may result from parallelizing a pipeline that uses mappers, filters, and other programmer-supplied function objects that fail to adhere to their specifications. The Stream specification places stringent requirements on these function objects. For example, the accumulator and combiner functions passed to Stream’s reduce operation must be associative, non-interfering, and stateless. If you violate these requirements (some of which are discussed in Item 46) but run your pipeline sequentially, it will likely yield correct results; if you parallelize it, it will likely fail, perhaps catastrophically.

**对Stream并行化不仅可能导致性能更差，比如活性失败；还肯能会出现不正确的结果和不可预期的行为（即安全性失败）。安全性失败可有是因为并行化的pipeline使用了没有遵守规定的映射器、过滤器、或者是其他的函数对象。stream的规范在这些函数对象上，有严格的规定。比如，传递给reduce操作的收集器和组合器函数，必须是有关联，互不干扰，和无状态的。如果你违背了这些要求(在Item46里面讨论了一些)，但是你串行地执行pipeline的话，可能会得到正确的结果。如果你并行执行的话，它就可能会失败，甚至还是灾难性的。

> Along these lines, it’s worth noting that even if the parallelized Mersenne primes program had run to completion, it would not have printed the primes in the correct (ascending) order. To preserve the order displayed by the sequential version, you’d have to replace the forEach terminal operation with forEachOrdered, which is guaranteed to traverse parallel streams in *encounter order*.

在上面的介绍中，有一点需要注意的是，即使这个并行化的梅森素数程序可以允许结束，它也不会按正确（升序）的顺序打印这些素数。为了要获得和串行允许一样的结果，你必须要把终止操作forEach修改为forEachOrdered。这个方法可以保证按照encounter顺序遍历并行的stream。

> Even assuming that you’re using an efficiently splittable source stream, a parallelizable or cheap terminal operation, and non-interfering function objects, you won’t get a good speedup from parallelization unless the pipeline is doing enough real work to offset the costs associated with parallelism. As a *very* rough estimate, the number of elements in the stream times the number of lines of code executed per element should be at least a hundred thousand [Lea14].

即使你使用的是有效的可分割的stream源，一个可并行化或者简单的终止操作，以及互不干扰的函数对象；除非pipeline做的工作足够地多可以抵消并行相关的消耗，否则你也不能通过并行化获得速度提升。有一个粗略的估计，stream中元素的数量是每个元素执行的代码的行数的至少10万倍的时候 [Lea14].，才可能有速度提升。

> It’s important to remember that parallelizing a stream is strictly a performance optimization. As is the case for any optimization, you must test the performance before and after the change to ensure that it is worth doing (Item 67). Ideally, you should perform the test in a realistic system setting. Normally, all parallel stream pipelines in a program run in a common fork-join pool. A single misbehaving pipeline can harm the performance of others in unrelated parts of the system.

需要切记的是：并行化stream是一项严格的性能优化。只要和优化相关，你都必须先测试其性能，然后在改变后再次测试性能，以确定确实值得优化（Item67）。理想情况下，你应该在实际系统环境下执行测试。通常来说，所有的程序里的并行化stream pipeline都在一个fork-join池里运行。一个行为异常的pipeline，会损害系统中其他不相关模块的性能，

> If it sounds like the odds are stacked against you when parallelizing stream pipelines, it’s because they are. An acquaintance who maintains a multimillion- line codebase that makes heavy use of streams found only a handful of places where parallel streams were effective. This does *not* mean that you should refrain from parallelizing streams. **Under the right circumstances, it** **is** **possible to achieve near-linear speedup in the number of processor cores simply by adding a** **parallel** **call to a stream pipeline.** Certain domains, such as machine learning and data processing, are particularly amenable to these speedups.

听起来好像并行化stream有很多的问题，它也确实是这样的。有一个朋友维护一个大量使用stream的几百万行的代码库，发现其中只有少数几个并行stream是有效的。但是这也并不意味着你应该避免使用并行化stream。**在正确的情景下，只在stream pipeline上简单地添加一个parallel调用，还是有可能将速度提升 处理器个数的倍数。在某些确定的领域，比如及其学习和数据处理上，特别适用这些速度提升。

> As a simple example of a stream pipeline where parallelism is effective, consider this function for computing π(*n*), the number of primes less than or equal to *n*:

举一个简单的stream pipeline的例子，其并行化是有效的。下面这个函数是用来计算π(n)的，表示小于或等于n的素数的数量：

```java
// Prime-counting stream pipeline - benefits from parallelization
   static long pi(long n) {
       return LongStream.rangeClosed(2, n)
           .mapToObj(BigInteger::valueOf)
           .filter(i -> i.isProbablePrime(50))
           .count();
}
```

> On my machine, it takes 31 seconds to compute π(10^8) using this function. Simply adding a parallel() call reduces the time to 9.2 seconds:

在作者的机器上，使用这个函数计算π(10^8) 花了31秒。而简单地添加了parallel()以后，花费时间减少到了9.2秒：

```java
// Prime-counting stream pipeline - parallel version
   static long pi(long n) {
       return LongStream.rangeClosed(2, n)
         .parallel()
         .mapToObj(BigInteger::valueOf)
         .filter(i -> i.isProbablePrime(50))
         .count();
   }
```

> In other words, parallelizing the computation speeds it up by a factor of 3.7 on my quad-core machine. It’s worth noting that this is *not* how you’d compute π(*n*) for large values of *n* in practice. There are far more efficient algorithms, notably Lehmer’s formula.

换句话说，在作者的四核机器上，对这个计算的并行化提升了3.7倍的速度。需要注意的是，这并不是实际应用中，计算n很大时的 π(*n*)的方法。有一些更有效的算法，尤其是Lehmer公式。

> If you are going to parallelize a stream of random numbers, start with a SplittableRandom instance rather than a ThreadLocalRandom (or the essentially obsolete Random). SplittableRandom is designed for precisely this use, and has the potential for linear speedup. ThreadLocalRandom is designed for use by a single thread, and will adapt itself to function as a parallel stream source, but won’t be as fast as SplittableRandom. Random synchronizes on every operation, so it will result in excessive, parallelism-killing contention.

如果你想要并行化一个随机数的stream，应该从SplittableRandom实例开始，而不是从ThreadLocalRandom（或者更过时的Random）开始。SplittableRandom是专门为此设计的，可能带来线性的速度提升。ThreadLocalRandom是设计用来单线程中的，它也可能把自己当做一个并行stream源运用到函数中，但是速度没有SplittableRandom快。Random在每次操作上都要进行同步，同步可能被滥用，因此扼杀了并行的优势。

> In summary, do not even attempt to parallelize a stream pipeline unless you have good reason to believe that it will preserve the correctness of the computation and increase its speed. The cost of inappropriately parallelizing a stream can be a program failure or performance disaster. If you believe that parallelism may be justified, ensure that your code remains correct when run in parallel, and do careful performance measurements under realistic conditions. If your code remains correct and these experiments bear out your suspicion of increased performance, then and only then parallelize the stream in production code.

总结一下，除非你有很好的的理由相信并行化stream pipeline会得到正确的结果以及提升速度，否则不要并行化stream pipeline。不合适的并行化stream的袋盖可能是程序错误，或者性能受损。如果你相信并行化是合适的，确定在并行化运行的时候，其他的代码也能保持正确，并且在现实条件下，做仔细的性能测试。如果你的代码是正确的，实验也证明这有助于性能提升，这个时候，也只有这个时候，才可以在编写代码的时候并行化stream。