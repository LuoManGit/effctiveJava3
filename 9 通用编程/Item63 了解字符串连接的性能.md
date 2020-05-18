### Item63 了解字符串连接的性能

> The string concatenation operator (+) is a convenient way to combine a few strings into one. It is fine for generating a single line of output or constructing the string representation of a small, fixed-size object, but it does not scale. **Using the string concatenation operator repeatedly to concatenate** **n** **strings requires time quadratic in** **n**. This is an unfortunate consequence of the fact that strings are *immutable* (Item 17). When two strings are concatenated, the contents of both are copied.

使用字符串连接操作符（+）来把少量的字符串连接起来是很方便的。用来生成单行输出、或者构造字符串来表达大小固定的小对象，是很合适的，但是它不适合用在大规模的运算中。**使用字符串连接操作符来把n个字符串重复连接到一起，需要的时间是n的平方级的**。造成这种结果的原因的字符串是不可变的（Item17）。当两个字符串连接的时候，他们的内容都需要复制一下。

> For example, consider this method, which constructs the string representation of a billing statement by repeatedly concatenating a line for each item:

比如，下面这个方法，通过把每个项目连接成一行，来构造账单声明的字符串表达：

```java
// Inappropriate use of string concatenation - Performs poorly!
   public String statement() {
       String result = "";
       for (int i = 0; i < numItems(); i++)
           result += lineForItem(i);  // String concatenation
       return result;
}
```

> The method performs abysmally if the number of items is large. **To achieve acceptable performance, use a** **StringBuilder** **in place of a** **String** to store the statement under construction:

如果项目的数量很大，那这个方法的性能就会极其地差。**为了达到可以接受的性能，可以使用StringBuilder替代String** 来保存构造过程中的声明。代码如下：

```java
public String statement() {
		StringBuilder b = new StringBuilder(numItems() * LINE_WIDTH); 
  	for (int i = 0; i < numItems(); i++)
				b.append(lineForItem(i)); 
  	return b.toString();
}
```

> A lot of work has gone into making string concatenation faster since Java 6, but the difference in the performance of the two methods is still dramatic: If numItems returns 100 and lineForItem returns an 80-character string, the second method runs 6.5 times faster than the first on my machine. Because the first method is quadratic in the number of items and the second is linear, the performance difference gets much larger as the number of items grows. Note that the second method preallocates a StringBuilder large enough to hold the entire result, eliminating the need for automatic growth. Even if it is detuned to use a default-sized StringBuilder, it is still 5.5 times faster than the first method.

虽然在Java6以后，做了很多的工作来使得字符串连接更快一些，但是这两个方法之间的性能区别还是很大的。如果numItems()返回100，lineForItem(i)返回的字符串的有80个字符，那么在我的机器上，第二个方法运行速度比第一个快6.5倍。因为第一个方法需要的时间是项目数量的平方级，而第二个方法却是线性级的，随着项目数量的增加，这个性能差别就会更大。需要注意的是，在第二个方法中，提前创建了一个能够装下整个结果的StringBuilder，这样就不需要自动增长了。即使使用默认大小的StringBuilder，第二个方法也比第一个方法快5.5倍。

> The moral is simple: **Don’t use the string concatenation operator to combine more than a few strings** unless performance is irrelevant. Use StringBuilder’s append method instead. Alternatively, use a character array, or process the strings one at a time instead of combining them.

这个原则很简单：**不要使用字符串连接操作符来组合多个字符串**，除非性能无关紧要。应该使用StringBuilder的append方法来替代。也可以使用字符串数组，或者直接处理整个字符串，而不是把她们组合起来。