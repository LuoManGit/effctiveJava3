### Item66 谨慎地使用本地方法

> The Java Native Interface (JNI) allows Java programs to call *native methods*, which are methods written in *native programming languages* such as C or C++. Historically, native methods have had three main uses. They provide access to platform-specific facilities such as registries. They provide access to existing libraries of native code, including legacy libraries that provide access to legacy data. Finally, native methods are used to write performance-critical parts of applications in native languages for improved performance.

Java本地接口( Java Native Interface, JNI) 允许Java程序员调用本地方法，本地方法即使用本地编程语言比如C或者C++编写的方法。他们提供了对特定于平台的机制提供了访问方法，比如注册表。他们提供了对遗留本地代码类库的访问方法，从而可以访问遗留的数据。最后本地方法还可以使用本地语言用来编写一些程序中注重性能的部分，以提高性能。

> It is legitimate to use native methods to access platform-specific facilities, but it is seldom necessary: as the Java platform matured, it provided access to many features previously found only in host platforms. For example, the process API, added in Java 9, provides access to OS processes. It is also legitimate to use native methods to use native libraries when no equivalent libraries are available in Java.

使用本地方法来访问特定于平台的机制是合法的，但是只有在很少的情况下才有必要：随着java平台的完善，已经提供了很多之前只有宿主平台能提供的特性。比如，在Java9中新增的进程API，提供了访问操作系统进程的方法。使用本地方法来使用一些在Java中没有的本地库，也是合法的。

> **It is rarely advisable to use native methods for improved performance.** In early releases (prior to Java 3), it was often necessary, but JVMs have gotten *much* faster since then. For most tasks, it is now possible to obtain comparable performance in Java. For example, when java.math was added in release 1.1, BigInteger relied on a then-fast multiprecision arithmetic library written in C. In Java 3, BigInteger was reimplemented in Java, and carefully tuned to the point where it ran faster than the original native implementation.

**一般不建议使用本地方法来提高性能**。在早期的版本中（Java3之前），还是很有必要的。但是随着JVM变得越来越快，对于大部分的任务，使用Java也可能获得与之相当的性能。比如，在1.1版本中加入java.math的时候，BigInteger就依赖一个C语言编写的快速多精度算术库。在Java3中，BigInteger使用Java进行了重新实现，并且进行了细致地调优，运行得比原来的本地方法实现还要快。

> A sad coda to this story is that BigInteger has changed little since then, with the exception of faster multiplication for large numbers in Java 8. In that time, work continued apace on native libraries, notably GNU Multiple Precision arithmetic library (GMP). Java programmers in need of truly high-performance multiprecision arithmetic are now justified in using GMP via native methods [Blum14].

遗憾的是，自那以后，除了Java8里的对于大整数的快速乘法以外，BIgInteger就没怎么改变了。在那个时候，相关工作和本地代码库齐头并进，尤其是GNU 高精度算术运算库（GMP）。需要更高性能的高精度运算的程序员，就可以通过本地方法使用GMP。

> The use of native methods has *serious* disadvantages. Because native languages are not *safe* (Item 50), applications using native methods are no longer immune to memory corruption errors. Because native languages are more platform-dependent than Java, programs using native methods are less portable. They are also harder to debug. If you aren’t careful, native methods can *decrease* performance because the garbage collector can’t automate, or even track, native memory usage (Item 8), and there is a cost associated with going into and out of native code. Finally, native methods require “glue code” that is difficult to read and tedious to write.

本地方法的使用有一些严重的缺点。因为本地语言是不安全的（Item50）,使用本地方法的语言就会受到内存破坏错误的影响。由于本地方法是对平台的依赖比java更高，因此使用本地方法的程序可移植性也就不高。使用本地方法还很难debug。如果你不小心，本地方法可能会损害性能，因为垃圾回收期无法自动回收，甚至跟踪本地方法的内存使用情况（Item8）。并且在进入和退出本地方法的时候还需要一些开销。最后调用本地方法需要的“胶水代码”很难阅读，编写起来也单调乏味。

> In summary, think twice before using native methods. It is rare that you need to use them for improved performance. If you must use native methods to access low-level resources or native libraries, use as little native code as possible and test it thoroughly. A single bug in the native code can corrupt your entire application.

总结一下，在使用本地方法之前要反复思量。只有在极少数情况下，你才需要使用他们来提高性能。如果你必须要使用本地方法来访问底层资源，或者遗留的代码库，尽可能少地使用本地代码，并且进行全面的测试。本地代码中的一个bug就可以让你的整个应用崩盘。