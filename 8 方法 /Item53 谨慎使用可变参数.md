### Item53 谨慎使用可变参数

> Varargs methods, formally known as *variable arity* methods [JLS, 8.4.1], accept zero or more arguments of a specified type. The varargs facility works by first creating an array whose size is the number of arguments passed at the call site, then putting the argument values into the array, and finally passing the array to the method.

可变参数方法，一般称作 *variable arity*方法[JLS, 8.4.1]。接受0个或者多个特定类型的参数。可变参数机制首先会创建一个大小是传入参数个数的数组，然后把这些参数值放在数组中，最后把这个数组传到方法里。

> For example, here is a varargs method that takes a sequence of int arguments and returns their sum. As you would expect, the value of sum(1, 2, 3) is 6, and the value of sum() is 0:

比如下面就是一个参数为一系列int值，返回其和的可变参数方法。正如你所想的那样，sum(1, 2, 3)的值为6，sum() 的值为0。

```java
// Simple use of varargs
   static int sum(int... args) {
       int sum = 0;
       for (int arg : args)
           sum += arg;
     	 return sum; 
   }
```

> Sometimes it’s appropriate to write a method that requires *one* or more arguments of some type, rather than *zero* or more. For example, suppose you want to write a function that computes the minimum of its arguments. This function is not well defined if the client passes no arguments. You could check the array length at runtime:

有的时候，相比0个或者多个，需要一个或者多个特定类型的参数的方法，可能更合适一些。比如，你想写一个函数，计算传入参数的大小，当客户端没有传入参数的时候，这个函数不是很好定义，在运行时，你需要检查数组的长度。代码如下：

```java
// The WRONG way to use varargs to pass one or more arguments!
   static int min(int... args) {
       if (args.length == 0)
         throw new IllegalArgumentException("Too few arguments"); 
     int min = args[0];
     for (int i = 1; i < args.length; i++)
           if (args[i] < min)
               min = args[i];
     return min; 
   }
```

> This solution has several problems. The most serious is that if the client invokes this method with no arguments, it fails at runtime rather than compile time. Another problem is that it is ugly. You have to include an explicit validity check on args, and you can’t use a for-each loop unless you initialize min to Integer.MAX_VALUE, which is also ugly.

这个解决方法有几个问题。最严重的时候，如果这个客户端调用这个方法时，没有提供参数，它就会再运行时失败，而不是编译时失败。另一个问题就是有点丑。首先必须要对参数进行一个显式的有效性检查，如果你不使用Integer.MAX_VALUE来初始化min的话，你就不能使用for-each循环，且Integer.MAX_VALUE也有点丑。

> Luckily there’s a much better way to achieve the desired effect. Declare the method to take two parameters, one normal parameter of the specified type and one varargs parameter of this type. This solution corrects all the deficiencies of the previous one:

幸运地是，有一个很好的方法可以达到想要的效果。声明这个方法有两个参数，一个就是特定类型的普通参数，另一个是该类型的可变参数。这个解决方法可以解决前面那种方法的所有的问题，代码如下：

```java
// The right way to use varargs to pass one or more arguments
   static int min(int firstArg, int... remainingArgs) {
       int min = firstArg;
       for (int arg : remainingArgs)
           if (arg < min)
               min = arg;
     	 return min; 
   }
```

> As you can see from this example, varargs are effective in circumstances where you want a method with a variable number of arguments. Varargs were designed for printf, which was added to the platform at the same time as varargs, and for the core reflection facility (Item 65), which was retrofitted. Both printf and reflection benefited enormously from varargs.

正如你从这个例子中看到的那样，当你想要一个方法的参数数量可变的时候，可变参数是非常有效的。可变参数是专门为printf方法设计的，可变参数和printf方法一起加入到Java平台中，为了核心的反射机制（Item65），可变参数进行了增强。printf和反射都从可变参数中获得了很大的好处。

> Exercise care when using varargs in performance-critical situations. Every invocation of a varargs method causes an array allocation and initialization. If you have determined empirically that you can’t afford this cost but you need the flexibility of varargs, there is a pattern that lets you have your cake and eat it too. Suppose you’ve determined that 95 percent of the calls to a method have three or fewer parameters. Then declare five overloadings of the method, one each with zero through three ordinary parameters, and a single varargs method for use when the number of arguments exceeds three:

在对性能要求特别严格的场景下，使用可变参数需要很小心。因为每次调用可变参数方法，都会分配和初始化一个数组。如果根据经验你确定你不能承受这个代价，但是你还是需要可变参数的灵活性，有一个模式可以让你如愿以偿。假如你确定95的方法调用的参数都只有3个或者更少，你就可以声明5个该方法的重载，从0个到3个参数分别对应一个方法，还有一个可变参数的方法，当参数的数量超过3个 的时候使用。如下：

```java
	 public void foo() { }
   public void foo(int a1) { }
   public void foo(int a1, int a2) { }
   public void foo(int a1, int a2, int a3) { }
   public void foo(int a1, int a2, int a3, int... rest) { }
```

> Now you know that you’ll pay the cost of the array creation only in the 5 percent of all invocations where the number of parameters exceeds three. Like most performance optimizations, this technique usually isn’t appropriate, but when it is, it’s a lifesaver.

现在，你知道，在所有的调用中，只有5%时候参数会超过3个，从而需要付出数组创建的代价。和大部分的性能优化一样，这个技术并不总是合适的，但是，当它需要的时候，它可以帮上很大的忙。

> The static factories for EnumSet use this technique to reduce the cost of creating enum sets to a minimum. This was appropriate because it was critical that enum sets provide a performance-competitive replacement for bit fields (Item 36).

EnumSet的静态工厂方法也使用了这个技术来把创建枚举集合的代价降到最小。这是很合适的，因为枚举集合是位域一个性能比较好的替代，因此对性能要求比较严格（Item36）。

> In summary, varargs are invaluable when you need to define methods with a variable number of arguments. Precede the varargs parameter with any required parameters, and be aware of the performance consequences of using varargs.

总结一下，当你需要定义一个参数数量可变的方法的时候，可变参数是很有用的。可以在可变参数前面定义必须要有的参数，并且要注意使用可变参数带来的性能问题。