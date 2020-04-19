### Item9 try-with-resources 优先于try-finally

> The Java libraries include many resources that must be closed manually by invoking a close method. Examples include InputStream, OutputStream, and java.sql.Connection. Closing resources is often overlooked by clients, with predictably dire performance consequences. While many of these resources use finalizers as a safety net, finalizers don’t work very well (Item 8).
>
> Historically, a try-finally statement was the best way to guarantee that a resource would be closed properly, even in the face of an exception or return:

在Java类库中包含很多的需要手动调用close方法来关闭的资源，比如InputStream, OutputStream, 和 java.sql.Connection。客户端经常会忘记关闭这些资源，从而导致一些严重的性能问题。虽然大部分这些院都使用了Finalizer来作为安全网，但是Finalizer的效果并不是很理想（Item 8）。

在以前，try-finally语句是保证这些资源被准确关闭的最好的方式，即使出现异常或者返回了也一样。如下：

```java
// try-finally - No longer the best way to close resources!
static String firstLineOfFile(String path) throws IOException { 
  		BufferedReader br = new BufferedReader(new FileReader(path));
  		try {
           return br.readLine();
       } finally {
           br.close();
       }
}
```

> This may not look bad, but it gets worse when you add a second resource:

这个看起来好像挺好的，但是当我们添加第二个资源的时候就会变得糟糕起来。

```java
// try-finally is ugly when used with more than one resource!
static void copy(String src, String dst) throws IOException {
       InputStream in = new FileInputStream(src);
       try {
           OutputStream out = new FileOutputStream(dst);
           try {
               byte[] buf = new byte[BUFFER_SIZE];
               int n;
               while ((n = in.read(buf)) >= 0)
                   out.write(buf, 0, n);
           } finally {
               out.close();
           }
       } finally {
           in.close();
		} 
}
```

> It may be hard to believe, but even good programmers got this wrong most of the time. For starters, I got it wrong on page 88 of *Java Puzzlers* [Bloch05], and no one noticed for years. In fact, two-thirds of the uses of the close method in the Java libraries were wrong in 2007.

这可能有点让人难以置信，但是即使是最优秀的程序员，很多时候都会犯这个错误。一开始，作者在*Java Puzzler* 这本书的88页也犯过这样的错误，但是很多年都没有人发现。事实上，在2007年前，Java类库中的2/3的close方法都用错了。

> Even the correct code for closing resources with try-finally statements, as illustrated in the previous two code examples, has a subtle deficiency. The code in both the try block and the finally block is capable of throwing exceptions. For example, in the firstLineOfFile method, the call to readLine could throw an exception due to a failure in the underlying physical device, and the call to close could then fail for the same reason. Under these circumstances, the second exception completely obliterates the first one. There is no record of the first exception in the exception stack trace, which can greatly complicate debugging in real systems—usually it’s the first exception that you want to see in order to diagnose the problem. While it is possible to write code to suppress the second exception in favor of the first, virtually no one did because it’s just too verbose.
>
> All of these problems were solved in one fell swoop when Java 7 introduced the try-with-resources statement [JLS, 14.20.3]. To be usable with this construct, a resource must implement the AutoCloseable interface, which consists of a single void-returning close method. Many classes and interfaces in the Java libraries and in third-party libraries now implement or extend AutoCloseable. If you write a class that represents a resource that must be closed, your class should implement AutoCloseable too.
>
> Here’s how our first example looks using try-with-resources:

即使是我们前面展示的两种正确的使用try-finally块来关闭资源都方法，都存在一个细微的问题。在try和finally代码块里，都有可能抛出异常，比如，在firstLineOfFile方法里，readLine方法可能会因为底层物理设备问题抛出异常。而close方法的调用同样也可能会因为这些原因失败。在这种情况下，第二个异常会完全掩盖掉第一个异常。在异常打印的栈轨迹上完全没有第一个异常的记录，在真实的系统中，这就会使得debug变得困难起来，因为通常情况下，为了判断问题所在，第一个异常才是程序员想看到的。虽然我们可以通过写代码来禁止第二个异常，以保留第一个异常，但实际上，没有人会这么做，因为代码太冗长了。

这些问题使用java7中介绍的try-with-resources里一下子就解决了。为了能使用这种结构，资源必须实现AutoCloseable接口，这个接口中只有一个返回值为void的close方法。在java类库和第三方库里的大部分类和接口现在都实现或者扩展了AutoCloseable接口，如果你写了一个类，代表的是必须关闭的资源，那么你的类也应该实现AutoCloseable接口。

以下是我们的第一个例子使用try-with-resources的实现：

```java
// try-with-resources - the the best way to close resources!
   static String firstLineOfFile(String path) throws IOException {
       try (BufferedReader br = new BufferedReader(new FileReader(path))) {
           return br.readLine();
       }
   }
```

> And here’s how our second example looks using try-with-resources:

以下是我们的第二个例子使用try-with-resources的实现：

```java
// try-with-resources on multiple resources - short and sweet
   static void copy(String src, String dst) throws IOException {
       try (InputStream   in = new FileInputStream(src);
 						OutputStream out = new FileOutputStream(dst)) {
						byte[] buf = new byte[BUFFER_SIZE];
						int n;
						while ((n = in.read(buf)) >= 0)
    						out.write(buf, 0, n);
       }
   }
```

> Not only are the try-with-resources versions shorter and more readable than the originals, but they provide far better diagnostics. Consider the firstLineOfFile method. If exceptions are thrown by both the readLine call and the (invisible) close, the latter exception is *suppressed* in favor of the former. In fact, multiple exceptions may be suppressed in order to preserve the exception that you actually want to see. These suppressed exceptions are not merely discarded; they are printed in the stack trace with a notation saying that they were suppressed. You can also access them programmatically with the getSuppressed method, which was added to Throwable in Java 7.
>
> You can put catch clauses on try-with-resources statements, just as you can on regular try-finally statements. This allows you to handle exceptions without sullying your code with another layer of nesting. As a slightly contrived example, here’s a version our firstLineOfFile method that does not throw exceptions, but takes a default value to return if it can’t open the file or read from it:

相对于原始版本，使用try-with-resources的版本不仅仅简洁易懂，出了问题还更容易诊断。比如firstLineOfFile方法，如果在readLine方法和（不可见的）close方法中都抛出了异常，那么close中的异常就会被禁止，保留readLine中的异常。事实上，为了让程序员可以看见真正想看的异常，有很多的异常都被静止了。这些被禁止了的异常不是简单的被抛弃了，它们会被打印在栈轨迹上，并且注明是被禁止了的异常。还可以通过编程调用getSuppressed方法来调用它们，这个方法在java7里被添加到了Throwable里。

在try-with-resources里，也可以像在try-finally语句里一样使用catch字句。这使得你可以处理一些异常，还不用再套一层代码。比如下面这个花了一些心思的例子，这是firstLineOfFile的一个版本，当无法打开或者读取文件的时候，这个方法也不会抛出异常，只是返回一个默认的值。

```java
 // try-with-resources with a catch clause
   static String firstLineOfFile(String path, String defaultVal) {
       try (BufferedReader br = new BufferedReader(
               new FileReader(path))) {
           return br.readLine();
       } catch (IOException e) {
           return defaultVal;
       } 
    }
```

> The lesson is clear: Always use try-with-resources in preference to try- finally when working with resources that must be closed. The resulting code is shorter and clearer, and the exceptions that it generates are more useful. The try- with-resources statement makes it easy to write correct code using resources that must be closed, which was practically impossible using try-finally.

结论很明确：当使用必须要关闭的资源时，总应该优先使用try-with-resources，而不是try-finally。使用try-with-resources的代码简短明了，而且生成的异常信息也更有用。事实证明，使用try-with-resources语句，在使用需要关闭的资源时，可能更加轻松的将代码写正确，而使用try-finally却很难做到。



