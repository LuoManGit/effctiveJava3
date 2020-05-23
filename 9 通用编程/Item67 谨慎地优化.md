### Item67 谨慎地优化

> There are three aphorisms concerning optimization that everyone should know:
>
> More computing sins are committed in the name of efficiency (without necessarily achieving it) than for any other single reason—including blind stupidity. —William A. Wulf [Wulf72]
>
> We *should* forget about small efficiencies, say about 97% of the time: premature optimization is the root of all evil.
>
> —Donald E. Knuth [Knuth74]
>
> We follow two rules in the matter of optimization:
>  Rule 1. Don’t do it.
>  Rule 2 (for experts only). Don’t do it yet—that is, not until you have a perfectly clear and unoptimized solution.
>  —M. A. Jackson [Jackson75]

关于优化，这里有三条每个人都应该知道的关于优点格言：

各个计算的问题都被归咎于效率（虽然很多效率都没必要），而不是一些其他的原因——甚至一些盲目傻傻的原因—William A. Wulf [Wulf72]

我们不要计较一些小小的效率问题，97%的情况下：过度优化才是万恶之源。—Donald E. Knuth [Knuth74]

当我们在进行优化的时候，要遵守的两条规则：

规则1：不要优化。

规则2（仅仅针对专家）：还是不要优化——也就是说，除非你有一个非常清晰的优化方法，否则不要优化。—M. A. Jackson [Jackson75]

> All of these aphorisms predate the Java programming language by two decades. They tell a deep truth about optimization: it is easy to do more harm than good, especially if you optimize prematurely. In the process, you may produce software that is neither fast nor correct and cannot easily be fixed.

这些格言的出现比Java编程语言还要早20年。它们说明了关于优化的真理：优化的弊大于利，尤其是那些不成熟的优化。在优化的过程中，你可以得到一个既不快还不正确的程序，而且还很难修正。

> Don’t sacrifice sound architectural principles for performance. **Strive to write good programs rather than fast ones.** If a good program is not fast enough, its architecture will allow it to be optimized. Good programs embody the principle of *information hiding*: where possible, they localize design decisions within individual components, so individual decisions can be changed without affecting the remainder of the system (Item 15).

不要为了性能牺牲了合理的结构。**坚持编写好的程序而不是快的程序**。如果一个好的程序不够快，它的结构也让它可以优化。好的程序满足“信息隐藏”原则：只要有可能，他们会把设计决策集中在单个组件中，因此单个决策的修改不会影响到系统的其他部分（Item15）。

> This does *not* mean that you can ignore performance concerns until your program is complete. Implementation problems can be fixed by later optimization, but pervasive architectural flaws that limit performance can be impossible to fix without rewriting the system. Changing a fundamental facet of your design after the fact can result in an ill-structured system that is difficult to maintain and evolve. Therefore you must think about performance during the design process.

这并不意味着在程序完成之前，你就可以忽略性能问题。实现问题咋以下一次优化中可以修复，但是普遍的限制性能的架构方面的问题，如果不重写系统的话，就很难修复。在系统完成之后改变某个基础的方面，可能会破坏系统的结构，导致其很难维护和升级。因此在设计过程中，你必须要好好考虑性能。

> **Strive to avoid design decisions that limit performance.** The components of a design that are most difficult to change after the fact are those specifying interactions between components and with the outside world. Chief among these design components are APIs, wire-level protocols, and persistent data formats. Not only are these design components difficult or impossible to change after the fact, but all of them can place significant limitations on the performance that a system can ever achieve.

**坚持避免限制性能的设计决策。**在设计完成后最难以改变的组件是那些需要和外部世界以及组件间进行交互的指定的组件。这些设计组件主要包括API，交互层协议，和永久数据格式。在设计完成后，这些设计不仅仅很难或者不可能修改，并且他们还可能对系统可以达到的性能有一定的限制。

> **Consider the performance consequences of your API design decisions.** Making a public type mutable may require a lot of needless defensive copying (Item 50). Similarly, using inheritance in a public class where composition would have been appropriate ties the class forever to its superclass, which can place artificial limits on the performance of the subclass (Item 18). As a final example, using an implementation type rather than an interface in an API ties you to a specific implementation, even though faster implementations may be written in the future (Item 64).

**在设计API的时候要考虑性能后果。**使用公有可变类型可能需要大量的不必要的保护性拷贝（Item50）。同样地，在公有类中使用继承，其构成可能永远和它的父类绑定在一起了，这样可能会给子类的性能带来人为的限制（Item18）。最后一个例子，在API中使用实现类型而不是接口类型，会让你和某个特定的实现绑定在一起了，即使在未来已经写了更加快速的实现（Item64）。

> The effects of API design on performance are very real. Consider the getSize method in the java.awt.Component class. The decision that this performance- critical method was to return a Dimension instance, coupled with the decision that Dimension instances are mutable, forces any implementation of this method to allocate a new Dimension instance on every invocation. Even though allocating small objects is inexpensive on a modern VM, allocating millions of objects needlessly can do real harm to performance.

API设计对性能的影响是真实存在的。比如java.awt.Component类的getSize方法，这个对性能要求很高的方法返回的是一个Dimension实例，而Dimension实例又是可变的，因此导致这个方法的每个实现，在调用这个方法的时候都需要创建一个新的Dimension实例。即使对于现代虚拟机来说，创建一个小对象的开销并不大，但是创建上百万的无用的对象也还是会造成性能损害的。

> Several API design alternatives existed. Ideally, Dimension should have been immutable (Item 17); alternatively, getSize could have been replaced by two methods returning the individual primitive components of a Dimension object. In fact, two such methods were added to Component in Java 2 for performance reasons. Preexisting client code, however, still uses the getSize method and still suffers the performance consequences of the original API design decisions.

有几个可以替代的API设计方法。最理想的是，Dimension应该是不可变的（Item17）；或者，使用两个分别返回Dimension的私有组件的方法来提到getSize方法。事实上，在Java2中，出于性能考虑，已经在Component中添加了这样的两个方法。然而，那么已经存在的客户端代码，仍然使用getSize方法，也就仍然需要收到原始API设计带来的性能的影响。

> Luckily, it is generally the case that good API design is consistent with good performance. **It is a very bad idea to warp an API to achieve good performance.** The performance issue that caused you to warp the API may go away in a future release of the platform or other underlying software, but the warped API and the support headaches that come with it will be with you forever.
>
> Once you’ve carefully designed your program and produced a clear, concise, and well-structured implementation, *then* it may be time to consider optimization, assuming you’re not already satisfied with the performance of the program.

幸运的是，通常来说，好的API就会有好的性能。**为了获得好的性能而扭曲API是非常不可取的**。因为让你扭曲APi的性能问题，可能会随着平台版本或者底层软件的版本升级而消失。但是扭曲的API，以及它带来的让人头疼的问题，会一直伴随着你。

一旦你好好地设计了这个程序，并且生成了一个清晰、简洁、结构良好的实现。然后如果你对程序的性能不满意的话，现在就是你考虑优化的时候了。

> Recall that Jackson’s two rules of optimization were “Don’t do it,” and “(for experts only). Don’t do it yet.” He could have added one more: **measure performance before and after each attempted optimization.** You may be surprised by what you find. Often, attempted optimizations have no measurable effect on performance; sometimes, they make it worse. The main reason is that it’s difficult to guess where your program is spending its time. The part of the program that you think is slow may not be at fault, in which case you’d be wasting your time trying to optimize it. Common wisdom says that programs spend 90 percent of their time in 10 percent of their code.

回顾一下Jackson的两条关于优化的规则：“不要优化” 和“（只针对专家），还是不要优化”。它还可以增加一条：**在每次进行优化的前后，都进行性能测试。**你可能会被得到的结果吓到。通常，试图做的优化并不能给性能带来可测量的影响；有的还是，还会更糟糕。主要的原因是，很难才出程序把它的时间都花在哪里了。你认为程序中比较慢的部分，可能并不慢，而你还浪费了时间去试图优化它。大多数人认为“程序在10%的代码上花了90%的时间”。

> Profiling tools can help you decide where to focus your optimization efforts. These tools give you runtime information, such as roughly how much time each method is consuming and how many times it is invoked. In addition to focusing your tuning efforts, this can alert you to the need for algorithmic changes. If a quadratic (or worse) algorithm lurks inside your program, no amount of tuning will fix the problem. You must replace the algorithm with one that is more efficient. The more code in the system, the more important it is to use a profiler. It’s like looking for a needle in a haystack: the bigger the haystack, the more useful it is to have a metal detector. Another tool that deserves special mention is jmh, which is not a profiler but a *microbenchmarking framework* that provides unparalleled visibility into the detailed performance of Java code [JMH].

性能剖析工具可以很好的帮助你选择优化的重点。这些工具可以给你提供运行时信息，大概就是每个方法执行需要的时间和方法调用的次数。除了确定优化的重点以外，还可以提示你可能有换算法的必要。如果你的程序隐藏着一个平方级别（甚至更糟糕）的算法，进行调整就没什么用了，你必须要使用另外一个高效的算法来替代它。一个系统的代码越多，你就越有必要使用性能剖析攻击。就像是在一个干草堆里找一根针一样：这个干草堆越大，金属探测仪就越有用。另外一个需要特别关注的工具是jmh，它不是一个性能剖析工具，而是一个微基准测试框架，可以给Java代码的性能细节提供无比的可见性。

> The need to measure the effects of attempted optimization is even greater in Java than in more traditional languages such as C and C++, because Java has a weaker *performance model*: The relative cost of the various primitive operations is less well defined. The “abstraction gap” between what the programmer writes and what the CPU executes is greater, which makes it even more difficult to reliably predict the performance consequences of optimizations. There are plenty of performance myths floating around that turn out to be half-truths or outright lies.

对于Java语言，对优化结果进行测量的需求，比其他传统语言比如C和C++，都更有必要，因为Java是弱性能模型：很多基本操作的开销很难定义。程序员编写的代码和CPU执行的代码之间的“语义沟”（*差别*）很大，这使得很难去预测优化对性能的影响。有很多流传的性能说法，最后证明是真假参半、或者就是错误的。

> Not only is Java’s performance model ill-defined, but it varies from implementation to implementation, from release to release, and from processor to processor. If you will be running your program on multiple implementations or multiple hardware platforms, it is important that you measure the effects of your optimization on each. Occasionally you may be forced to make trade-offs between performance on different implementations or hardware platforms.

不仅仅是Java的性能模型没有很好的定义，而且它还会根据不同的实现、不同的版本、不同的处理器而有所不同。如果你的程序需要运行在多个实现或者多个硬件平台上，那么在每个平台上都测试优化的影响就很重要了。有的时候，你必须在不同实现或者硬件平台中做性能权衡。

> In the nearly two decades since this item was first written, every component of the Java software stack has grown in complexity, from processors to VMs to libraries, and the variety of hardware on which Java runs has grown immensely. All of this has combined to make the performance of Java programs even less predictable now than it was in 2001, with a corresponding increase in the need to measure it.

自本节第一次编写已经过去了将近20年了，Java软件栈中的每一个组件，从处理器到虚拟机到类库，都成长得更加复杂了，运行Java的硬件平台也越来越丰富了。所有的这些都使得Java程序的性能比2001年更难预测了，因此对于性能测试的需求也就更重要了。

> To summarize, do not strive to write fast programs—strive to write good ones; speed will follow. But do think about performance while you’re designing systems, especially while you’re designing APIs, wire-level protocols, and persistent data formats. When you’ve finished building the system, measure its performance. If it’s fast enough, you’re done. If not, locate the source of the problem with the aid of a profiler and go to work optimizing the relevant parts of the system. The first step is to examine your choice of algorithms: no amount of low-level optimization can make up for a poor choice of algorithm. Repeat this process as necessary, measuring the performance after every change, until you’re satisfied.

总结一下，不要致力于写快的程序——应该致力于写好的程序，速度在其次。但是当你在设计系统，尤其是设计API，交互层协议和永久数据格式的时候，一定要考虑到性能。当你完成性能构建的时候，测试一下它的性能。如果已经够快了的话，你的任务就已经完成了。如果还不够的话，使用性能剖析工具来定位问题的源头，然后优化系统中相关的部分。第一步是检查你选择的算法：再多的底层优化，也无法弥补不好的算法选择。有必要的话，重复这个过程，在每次修改后都测试一下性能，直到满意为止。

