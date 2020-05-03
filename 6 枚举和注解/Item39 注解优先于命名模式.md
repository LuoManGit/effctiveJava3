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

Test注解对于Sample类的语义没有直接的影响。他们只是给那些对此感兴趣的程序提供一写信息。更通俗一点来说，注解不会改变被注解代码的语义，但是可以使这段代码可以被一些工具特殊处理，比如下面这个简单的测试运行工具：

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

这个测试工具使用一个完全匹配的类名作为命令行参数，并使用Method.invoke反射地调用该类的所有的使用Test注解的方法。isAnnotationPresent方法告诉工具应该运行那个方法。如果这个方法抛出了异常，反射机制会将这个异常包装在一个InvocationTargetException里。然后测试工具会捕获这个异常，并打印一个失败报告，该报告中包含测试方法的原始异常，这个原始异常是通过InvocationTargetException的getCause方法获取到的。

> If an attempt to invoke a test method by reflection throws any exception other than InvocationTargetException, it indicates an invalid use of the Test annotation that was not caught at compile time. Such uses include annotation of an instance method, of a method with one or more parameters, or of an inaccessible method. The second catch block in the test runner catches these Test usage errors and prints an appropriate error message. Here is the output that is printed if RunTests is run on Sample:

如果通过反射调用的方法抛出了不是InvocationTargetException的异常，那就表明这是一个Test注解在编译时没有发现的非法的使用。这些的使用包括在一个实例方法，或者有一个或多个参数的方法，或者不可达的方法上进行注解。在测试运行工具中，第二个catch块就捕获了这类Test使用错误，并且打印了一个合适的错误信息。这个在Sample上运行RunTests的答应结果:

```java
public static void Sample.m3() failed: RuntimeException: Boom
Invalid @Test: public void Sample.m5()
public static void Sample.m7() failed: RuntimeException: Crash
Passed: 1, Failed: 3
```

> Now let’s add support for tests that succeed only if they throw a particular exception. We’ll need a new annotation type for this:

我们现在要给测试框架增加，只在抛出特定异常才算成功的功能，我们需要一个新的注解类型，如下：

```java
   // Annotation type with a parameter
   import java.lang.annotation.*;
   /**
    * Indicates that the annotated method is a test method that
    * must throw the designated exception to succeed.
    */
   @Retention(RetentionPolicy.RUNTIME)
   @Target(ElementType.METHOD)
   public @interface ExceptionTest {
       Class<? extends Throwable> value();
	 }
```

> The type of the parameter for this annotation is Class<? extends Throwable>. This wildcard type is, admittedly, a mouthful. In English, it means “the Class object for some class that extends Throwable,” and it allows the user of the annotation to specify any exception (or error) type. This usage is an exam- ple of a *bounded type token* (Item 33). Here’s how the annotation looks in practice. Note that class literals are used as the values for the annotation parameter:

这个注解的参数类型是Class<? extends Throwable>，这个通配符类型读起来确实有点复杂，它的意思是：一些继承了Throwable的类的Class对象。允许这个注解的用户指定任何一个异常或者错误类型。这也是一个有限制的类型令牌的例子（Item33）。下面是这个注解在实际应用中的使用方法，注意，这些类字面量都是用作注解的参数值的：

```java
// Program containing annotations with a parameter
   public class Sample2 {
       @ExceptionTest(ArithmeticException.class)
       public static void m1() {  // Test should pass
           int i = 0;
					 i = i / i; 
       }
       @ExceptionTest(ArithmeticException.class)
       public static void m2() {  // Should fail (wrong exception)
           int[] a = new int[0];
           int i = a[1];
				}
       @ExceptionTest(ArithmeticException.class)
       public static void m3() { }  // Should fail (no exception)
   }
```

> Now let’s modify the test runner tool to process the new annotation. Doing so consists of adding the following code to the main method:

现在让我们来修改一些测试运行工具，以处理这个新的注解。为了达到这个目的，需要将下面这个代码添加到main方法里：

```java
if (m.isAnnotationPresent(ExceptionTest.class)) {
       tests++;
       try {
           m.invoke(null);
           System.out.printf("Test %s failed: no exception%n", m);
       } catch (InvocationTargetException wrappedEx) {
           Throwable exc = wrappedEx.getCause();
           Class<? extends Throwable> excType =
               m.getAnnotation(ExceptionTest.class).value();
           if (excType.isInstance(exc)) {
               passed++;
           } else {
               System.out.printf(
                   "Test %s failed: expected %s, got %s%n",
                   m, excType.getName(), exc);
           }
       } catch (Exception exc) {
           System.out.println("Invalid @Test: " + m);
       }
}
```

> This code is similar to the code we used to process Test annotations, with one exception: this code extracts the value of the annotation parameter and uses it to check if the exception thrown by the test is of the right type. There are no explicit casts, and hence no danger of a ClassCastException. The fact that the test program compiled guarantees that its annotation parameters represent valid exception types, with one caveat: if the annotation parameters were valid at compile time but the class file representing a specified exception type is no longer present at runtime, the test runner will throw TypeNotPresentException.

这段代码和前面用来运行Test注解的代码很像，有一处不同：这个代码获取注解参数的值，然后使用它来检查所测试的方法抛出的异常的类型是否正确。代码中没有显式类型转换，因此不会有出现ClassCastException的可能。事实是测试程序在编译时保证了，这个注解参数表示的就是一种合法的异常类型，有一点需要注意的是：如果这个注解参数在编译器是合法的，但是它所表示的特定的异常类型的类型文件在运行时不存在了，这个测试运行工具就会抛出TypeNotPresentException。

> Taking our exception testing example one step further, it is possible to envision a test that passes if it throws any one of several specified exceptions. The annotation mechanism has a facility that makes it easy to support this usage. Suppose we change the parameter type of the ExceptionTest annotation to be an array of Class objects:

将上面的异常测试例子再更进一步，想象这样一个测试：当抛出几个特定类型中的任何一个的时候，测试通过。在注解机制中有一个工具，可以很容易支持这种使用方法，假设我们将ExceptionTest注解的类型参数改为一个类对象的数组：

```java
// Annotation type with an array parameter
   @Retention(RetentionPolicy.RUNTIME)
   @Target(ElementType.METHOD)
   public @interface ExceptionTest {
			 Class<? extends Exception>[] value(); 
   }
```

> The syntax for array parameters in annotations is flexible. It is optimized for single-element arrays. All of the previous ExceptionTest annotations are still valid with the new array-parameter version of ExceptionTest and result in single-element arrays. To specify a multiple-element array, surround the elements with curly braces and separate them with commas:

这个注解里的数组参数的语法是很灵活的。它为单个元素的数组做了优化，前面的所有的ExceptionTest注解在这个新的数组参数版本的ExceptionTest中都是合法的，会生成一个单个元素的数组。为了制定一个多个元素的数组，要用花括号把元素包围起来，并且使用逗号进行分隔。如下：

```java
// Code containing an annotation with an array parameter 
@ExceptionTest({ IndexOutOfBoundsException.class,NullPointerException.class }) 
public static void doublyBad() {
       List<String> list = new ArrayList<>();
       // The spec permits this method to throw either
       // IndexOutOfBoundsException or NullPointerException
       list.addAll(5, null);
}
```

> It is reasonably straightforward to modify the test runner tool to process the new version of ExceptionTest. This code replaces the original version:

要处理这种新版本的ExceptionTest，只需要对测试运行器进行很简单的修改。使用下面的代码代替原来的版本：

```java
if (m.isAnnotationPresent(ExceptionTest.class)) {
       tests++;
       try {
           m.invoke(null);
           System.out.printf("Test %s failed: no exception%n", m);
       } catch (Throwable wrappedExc) {
						Throwable exc = wrappedExc.getCause();
						int oldPassed = passed;
						Class<? extends Exception>[] excTypes = 
              m.getAnnotation(ExceptionTest.class).value();
						for (Class<? extends Exception> excType : excTypes) {
    						if (excType.isInstance(exc)) {
        					passed++;
									break; 
                }
            }
						if (passed == oldPassed)
    						System.out.printf("Test %s failed: %s %n", m, exc);
       }
}
```

> As of Java 8, there is another way to do multivalued annotations. Instead of declaring an annotation type with an array parameter, you can annotate the declaration of an annotation with the @Repeatable meta-annotation, to indicate that the annotation may be applied repeatedly to a single element. This meta-annotation takes a single parameter, which is the class object of a *containing annotation type*, whose sole parameter is an array of the annotation type [JLS, 9.6.3]. Here’s how the annotation declarations look if we take this approach with our ExceptionTest annotation. Note that the containing annotation type must be annotated with an appropriate retention policy and target, or the declarations won’t compile:

在Java8中，还有一种方法可以处理多值注解。不是用使用一个数组参数来声明注解类型，你可以在注解的声明上使用元注解@Repeatable 来进行注解，来表示这个注解可以在一个单个元素上多次应用。@Repeatable 这个元注解有一个参数，该参数是一个包含注解类型的class对象，这个包含注解类型的唯一的参数就是，被@Repeatable所注解的注解类型 数组 [JLS, 9.6.3]。需要注意的是，这个包含注解类型也必须要使用合适的@Retention和@Target进行注解，否则这个声明将无法进行编译。代码如下：

```java
// Repeatable annotation type
   @Retention(RetentionPolicy.RUNTIME)
   @Target(ElementType.METHOD)
   @Repeatable(ExceptionTestContainer.class)
   public @interface ExceptionTest {
       Class<? extends Exception> value();
   }
   @Retention(RetentionPolicy.RUNTIME)
   @Target(ElementType.METHOD)
   public @interface ExceptionTestContainer {
       ExceptionTest[] value();
   }
```

> Here’s how our doublyBad test looks with a repeated annotation in place of an array-valued annotation:

下面是我们的doublyBad测试使用 可重复注解 代替 数组值注解 的样子：

```java
// Code containing a repeated annotation
   @ExceptionTest(IndexOutOfBoundsException.class)
   @ExceptionTest(NullPointerException.class)
   public static void doublyBad() { ... }
```

> Processing repeatable annotations requires care. A repeated annotation generates a synthetic annotation of the containing annotation type. The getAnnotationsByType method glosses over this fact, and can be used to access both repeated and non-repeated annotations of a repeatable annotation type. But isAnnotationPresent makes it explicit that repeated annotations are not of the annotation type, but of the containing annotation type. If an element has a repeated annotation of some type and you use the isAnnotationPresent method to check if the element has an annotation of that type, you’ll find that it does not. Using this method to check for the presence of an annotation type will therefore cause your program to silently ignore repeated annotations. Similarly, using this method to check for the containing annotation type will cause the program to silently ignore non-repeated annotations. To detect repeated and non-repeated annotations with isAnnotationPresent, you much check for both the annotation type and its containing annotation type. Here’s how the relevant part of our RunTests program looks when modified to use the repeatable version of the ExceptionTest annotation:

处理可重复注解的代码需要非常小心。一个重复的注解会生成一个包含注解类型的合成注解。getAnnotationsByType方法忽略了这个事实，因此可以被用来获取可重复注解类型的重复的和非重复的注解。但是isAnnotationPresent方法却让这个行为变成了显式的，也就是说，重复的注解不是这个注解类型，而是包含注解类型。如果一个元素有一个类型的重复注解，然后你使用isAnnotationPresent方法来检查这个元素是否有这个类型的注解，你会发现它没有。使用isAnnotationPresent方法来检测一个注解类型是否存在，会导致你的程序忽略掉重复的注解。同样的，当你使用isAnnotationPresent方法来检测 包含注解类型 是否存在，也会导致程序忽略掉非重复的注解。为了能使用isAnnotationPresent方法来检测重复的和非重复的注解，你必须要对注解类型和它的包含注解类型都进行检查。下面是运行测试程序，改成使用ExceptionTest的可重复版本的相关代码：

```java
// Processing repeatable annotations
   if (m.isAnnotationPresent(ExceptionTest.class)
       || m.isAnnotationPresent(ExceptionTestContainer.class)) {
       tests++;
       try {
           m.invoke(null);
           System.out.printf("Test %s failed: no exception%n", m);
       } catch (Throwable wrappedExc) {
						Throwable exc = wrappedExc.getCause(); 
         		int oldPassed = passed;
         		ExceptionTest[] excTests = m.getAnnotationsByType(ExceptionTest.class);
						for (ExceptionTest excTest : excTests) {
              if (excTest.value().isInstance(exc)) { 
                passed++;
                break; 
              }
            }
						if (passed == oldPassed)
    						System.out.printf("Test %s failed: %s %n", m, exc);
       }
   }
```

> Repeatable annotations were added to improve the readability of source code that logically applies multiple instances of the same annotation type to a given program element. If you feel they enhance the readability of your source code, use them, but remember that there is more boilerplate in declaring and processing repeatable annotations, and that processing repeatable annotations is error-prone.

可重复的注解加入提升了源代码的可读性，逻辑上，把一个注解类型的多个实例应用到了一个给定的程序元素上。如果你觉得它们增加了源代码的可读性，就使用它们。但是在声明和处理可重复注解的时候有更多的样板代码，并且处理可重复注解也更容易出错。

> The testing framework in this item is just a toy, but it clearly demonstrates the superiority of annotations over naming patterns, and it only scratches the surface of what you can do with them. If you write a tool that requires programmers to add information to source code, define appropriate annotation types. **There is simply no reason to use naming patterns when you can use annotations instead.**

虽然这个测试框架只是一个小玩具，但是它清晰地展示了注解相较于命名模式的优越性，并且这只是展示了注解功能的冰山一角。如果你写一个工具需要程序员往源代码中添加信息，就可以定义一些合适的注解类型。**当你能使用注解的时候，就完全没有理由再使用命名模式了。**

> That said, with the exception of toolsmiths, most programmers will have no need to define annotation types. But **all programmers should use the predefined annotation types that Java provides** (Items 40, 27). Also, consider using the annotations provided by your IDE or static analysis tools. Such annotations can improve the quality of the diagnostic information provided by these tools. Note, however, that these annotations have yet to be standardized, so you may have some work to do if you switch tools or if a standard emerges.

也就是说，除了”工具铁匠(平台框架程序员）“以外，大部分的程序员都不需要定义注解类型。但是**所有的程序员都应该使用Java提供的定义好的注解（Item40，27）**。同样的，考虑使用你的IDE或者静态分析工具提供的注解。这些注解可以提升这些工具提供的诊断信息的质量。但是需要注意的是，这些注意还没有标准化，如果你要换一个工具，或者生成一个标准，你就有很多的工作需要做了。

