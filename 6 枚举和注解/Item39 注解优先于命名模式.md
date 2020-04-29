### Item39 注解优先于命名模式

> Historically, it was common to use *naming patterns* to indicate that some program elements demanded special treatment by a tool or framework. For example, prior to release 4, the JUnit testing framework required its users to designate test methods by beginning their names with the characters test [Beck04]. This technique works, but it has several big disadvantages. First, typographical errors result in silent failures. For example, suppose you accidentally named a test method tsetSafetyOverride instead of testSafetyOverride. JUnit 3 wouldn’t complain, but it wouldn’t execute the test either, leading to a false sense of security.

按照经验，一般会使用“命名模式”来表示一些程序元素，需要工具或者框架进行特殊处理。比如，在JUnit4版本发行之前，JUnit测试框架要求用户使用test开头来命名测试方法[Beck04]。这个方法虽然可以工作，但是存在几个很大的缺点。首先，书写错误可能会导致出错，但是又没有任何提示。比如，你不小心把一个测试方法命名成了tsetSafetyOverride，而不是testSafetyOverride。JUnit3 不会提示，也不会执行这个测试，会给人带来错误的安全感。

> A second disadvantage of naming patterns is that there is no way to ensure that they are used only on appropriate program elements. For example, suppose you called a class TestSafetyMechanisms in hopes that JUnit 3 would automatically test all of its methods, regardless of their names. Again, JUnit 3 wouldn’t complain, but it wouldn’t execute the tests either.

命名模式的第二个缺点是没有办法保证它们只是应用于合适的程序元素上。比如，你把一个类命名为TestSafetyMechanisms，希望JUnit3可以自动的测试所有的方法，而忽略了方法的名字。同样地，JUnit3不会提示，也不会执行这些测试。

> A third disadvantage of naming patterns is that they provide no good way to associate parameter values with program elements. For example, suppose you want to support a category of test that succeeds only if it throws a particular exception. The exception type is essentially a parameter of the test. You could encode the exception type name into the test method name using some elaborate naming pattern, but this would be ugly and fragile (Item 62). The compiler would have no way of knowing to check that the string that was supposed to name an exception actually did. If the named class didn’t exist or wasn’t an exception, you wouldn’t find out until you tried to run the test.

命名模式的第三个缺点是没有很好的方法来给程序元素提供相关的参数。比如，你想支持这样一种测试模式，只有当方法抛出特定的异常的时候才算成功。这个异常的类型就是测试的一个参数。你可以使用特别精细的命名模式来把这个异常类型的名字编在测试方法的名字中，但是这种方法很丑也很脆弱（Item62）。编译器没有办法知道怎么去测试 这个用来命名异常的字符串。如果这个命名异常类不存在，或者根本就不是一个异常，你也只能在运行测试的时候才能发现。

> Annotations [JLS, 9.7] solve all of these problems nicely, and JUnit adopted them starting with release 4. In this item, we’ll write our own toy testing framework to show how annotations work. Suppose you want to define an annotation type to designate simple tests that are run automatically and fail if they throw an exception. Here’s how such an annotation type, named Test, might look:

注解可以很好的解决上面所有的问题，JUnit自版本4中开始使用注解。在本节中，我们将写一个自己的简单的测试框架，假如你想定义一个注解来指定简单的，自动运行的，在抛出异常时失败的测试。下面就是名为Test的注解可能的样子：

```java
// Marker annotation type declaration
   import java.lang.annotation.*;
   /**
    * Indicates that the annotated method is a test method.
    * Use only on parameterless static methods.
    */
   @Retention(RetentionPolicy.RUNTIME)
   @Target(ElementType.METHOD)
   public @interface Test {
   }
```

> The declaration for the Test annotation type is itself annotated with Retention and Target annotations. Such annotations on annotation type declarations are known as *meta-annotations*. The @Retention(RetentionPolicy.RUNTIME) meta-annotation indicates that Test annotations should be retained at runtime. Without it, Test annotations would be invisible to the test tool. The @Target.get(ElementType.METHOD) meta-annotation indicates that the Test annotation is legal only on method declarations: it cannot be applied to class declarations, field declarations, or other program elements.

这个Test注解的声明上上也使用了Retention和Target注解进行注释。在注解类型声明上的这种注解被称为元注解。元注解@Retention(RetentionPolicy.RUNTIME)表示这个Test注解在运行时应该被保留。如果没有这个注解的话，Test注解对于测试工具来说就不可见了。元注解@Target.get(ElementType.METHOD)表示Test注解只有在方法声明上使用时合法的，它不能被用在类声明，域声明，或者其他的程序元素上。

> The comment before the Test annotation declaration says, “Use only on parameterless static methods.” It would be nice if the compiler could enforce this, but it can’t, unless you write an *annotation processor* to do so. For more on this topic, see the documentation for javax.annotation.processing. In the absence of such an annotation processor, if you put a Test annotation on the declaration of an instance method or on a method with one or more parameters, the test program will still compile, leaving it to the testing tool to deal with the problem at runtime.

在Test上的注释声明说：“只能用在无参数的静态方法上。“如果编译器可以强制实现这个功能的话，是非常好的，但是它并不能，除非你写一个注解处理器去进行限制。关于这个话题的更多的内容，可以参见javax.annotation.processing的文档。在没有这样一个注解处理器的情况下，如果你在实例方法或者有一个或者多个参数的方法声明上，使用这个注解，测试程序还是可以编译，把这个问题留给运行时的测试工具来处理。

> Here is how the Test annotation looks in practice. It is called a *marker annotation* because it has no parameters but simply “marks” the annotated element. If the programmer were to misspell Test or to apply the Test annotation to a program element other than a method declaration, the program wouldn’t compile:

下面是Test注解在实际应用中的样子。它被称为”标记注解“，因为它没有参数，只是简单的给被注解元素做了个标记。如果程序员把Test拼写错了，或者把Test注解写在了不是方法声明的其他程序元素的时候，这个程序就不会编译。代码如下;

```java
// Program containing marker annotations
public class Sample {
	@Test public static void m1() { } // Test should pass
  public static void m2() { }
	@Test public static void m3() { // Test should fail
           throw new RuntimeException("Boom");
       }
	public static void m4() { }
	@Test public void m5() { } // INVALID USE: nonstatic method 
  public static void m6() { }
	@Test public static void m7() { // Test should fail
           throw new RuntimeException("Crash");
       }
  public static void m8() { }
}
```

> The Sample class has seven static methods, four of which are annotated as tests. Two of these, m3 and m7, throw exceptions, and two, m1 and m5, do not. But one of the annotated methods that does not throw an exception, m5, is an instance method, so it is not a valid use of the annotation. In sum, Sample contains four tests: one will pass, two will fail, and one is invalid. The four methods that are not annotated with the Test annotation will be ignored by the testing tool.

这个Sample类有7个静态方法，4个方法被注解为测试。其中m3和m7这两个方法会抛出异常，m1和m5不会抛出异常。有一个被注解也不会抛出异常的方法m5，是一个实例方法，因此这不是该方法的一个合法的使用。总结一下，Sample中有4个测试：1个会通过，2个会失败，1个是非法的。这4个没有使用Test注解的方法会被测试工具忽略掉。

> The Test annotations have no direct effect on the semantics of the Sample class. They serve only to provide information for use by interested programs. More generally, annotations don’t change the semantics of the annotated code but enable it for special treatment by tools such as this simple test runner:

Test注解对于Sample类的语义没有直接的影响。他们只是给那些对此感兴趣的程序提供一写信息。更通俗一点来说，注解不会改变被注解代码的语义，但是可以使这段代码可以被一些工具特殊处理，比如下面这个简单的测试运行类：

```java
// Program to process marker annotations
   import java.lang.reflect.*;
   public class RunTests {
       public static void main(String[] args) throws Exception {
           int tests = 0;
           int passed = 0;
           Class<?> testClass = Class.forName(args[0]);
           for (Method m : testClass.getDeclaredMethods()) {
             if (m.isAnnotationPresent(Test.class)) { 
               tests++;
               try {
                 m.invoke(null);
                 passed++;
               } catch (InvocationTargetException wrappedExc) {
                 Throwable exc = wrappedExc.getCause();
                 System.out.println(m + " failed: " + exc);
               } catch (Exception exc) {
                 System.out.println("Invalid @Test: " + m);
               }
             } 
           }
					 System.out.printf("Passed: %d, Failed: %d%n",passed, tests - passed);
       }
   }
```

> The test runner tool takes a fully qualified class name on the command line and runs all of the class’s Test-annotated methods reflectively, by calling Method.invoke. The isAnnotationPresent method tells the tool which methods to run. If a test method throws an exception, the reflection facility wraps it in an InvocationTargetException. The tool catches this exception and prints a failure report containing the original exception thrown by the test method, which is extracted from the InvocationTargetException with the getCause method.















