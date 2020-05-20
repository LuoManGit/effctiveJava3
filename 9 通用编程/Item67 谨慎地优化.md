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





















