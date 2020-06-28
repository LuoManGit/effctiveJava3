### Item77 不要忽略异常

> While this advice may seem obvious, it is violated often enough that it bears repeating. When the designers of an API declare a method to throw an exception, they are trying to tell you something. Don’t ignore it! It is easy to ignore exceptions by surrounding a method invocation with a try statement whose catch block is empty:

虽然这个建议很显然，但是它却常常被违反，因此值得再次提出来。当一个API的设计者声明一个方法会抛出异常，它就是想告诉你一些事情。不要忽略它！可以在一个方法调用的周围用try块包围起来，然后在catch块里什么也不做，就可以很简单地忽略掉这个异常。如下：

```java
// Empty catch block ignores exception - Highly suspect!
try { ...
   } catch (SomeException e) {
   }

```

> **An empty** **catch** **block defeats the purpose of exceptions,** which is to force you to handle exceptional conditions. Ignoring an exception is analogous to ignoring a fire alarm—and turning it off so no one else gets a chance to see if there’s a real fire. You may get away with it, or the results may be disastrous. Whenever you see an empty catch block, alarm bells should go off in your head.

**一个空的catch块，违反了异常设计的目的**，其目的就是为了强迫你处理异常。忽略异常就像忽略火警一样——如果你关掉了火警，就没有人有机会去检查是不是发生了火灾了。你可能会侥幸平安无事，也可能其结果是灾难性的。任何时候当你看到空的异常块的时候，在你的脑袋里，应该拉响警报。

> There are situations where it is appropriate to ignore an exception. For example, it might be appropriate when closing a FileInputStream. You haven’t changed the state of the file, so there’s no need to perform any recovery action, and you’ve already read the information that you need from the file, so there’s no reason to abort the operation in progress. It may be wise to log the exception, so that you can investigate the matter if these exceptions happen often. **If you choose to ignore an exception, the** **catch** **block should contain a comment explaining why it is appropriate to do so, and the variable should be named** **ignored:**

在一些场景下，忽略异常也是合适的。比如，当我们关闭一个FileInputStream。如果你还没有改变文件的状态，那么就没有必要执行任何恢复操作，如果你已经从文件里读取了你需要的信息，那么也就没有必要终止正在进行的操作。最明智的方法就是记录这个异常，所以你就可以调查这些异常是否发生。**如果你选择忽略了异常，那么这个catch块包含一句注释，解释为什么可以忽略，并且这个变量的名字应该命名为ignored。**如下：

```java
Future<Integer> f = exec.submit(planarMap::chromaticNumber); int numColors = 4; 
// Default; guaranteed sufficient for any map 
try {
	numColors = f.get(1L, TimeUnit.SECONDS);
} catch (TimeoutException | ExecutionException ignored) {
  // Use default: minimal coloring is desirable, not required
}
```

> The advice in this item applies equally to checked and unchecked exceptions. Whether an exception represents a predictable exceptional condition or a programming error, ignoring it with an empty catch block will result in a program that continues silently in the face of error. The program might then fail at an arbitrary time in the future, at a point in the code that bears no apparent relation to the source of the problem. Properly handling an exception can avert failure entirely. Merely letting an exception propagate outward can at least cause the program to fail swiftly, preserving information to aid in debugging the failure.

本节中的这条建议，对于受检异常和非受检异常，都同样使用。不管异常代表一个可预测的异常条件，还是程序错误，使用一个空的catch块来忽略异常，就会导致程序在遇到错误的时候，继续执行下去。这个程序可能会在一个未来某个不确定的时候失败，在某个代码看起来和问题一点关联都没有的地方失败。适当地处理异常可以彻底避免失败；把异常传播给外界，也至少可以让程序快速失败，从而保留了有助于调试失败的信息。