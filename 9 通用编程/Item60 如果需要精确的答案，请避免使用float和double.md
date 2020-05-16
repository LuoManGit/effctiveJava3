### Item60 如果需要精确的答案，请避免使用float和double

> The float and double types are designed primarily for scientific and engineering calculations. They perform *binary floating-point arithmetic*, which was carefully designed to furnish accurate approximations quickly over a broad range of magnitudes. They do not, however, provide exact results and should not be used where exact results are required. **The** **float** **and** **double** **types are particularly ill-suited for monetary calculations** because it is impossible to represent 0.1 (or any other negative power of ten) as a float or double exactly.

这个float和double 是专门设计用来做科学计算和工程计算的。他们执行二进制浮点数运算，这种运算是为了在广泛的数值范围上提供近似准确的快递运行而设计的。然而，他们确实不能提供精确的结果，因此在需要精确的结果的地方，就不应该使用他们。**float和double类型尤其不适合用于货币计算**，因为他们都不能准确的表示0.1（或者其他的10的负数次方）。

> For example, suppose you have $1.03 in your pocket, and you spend 42¢. How much money do you have left? Here’s a naive program fragment that attempts to answer this question:

比如，假如你现在的口袋里有$1.03，然后你花了42¢。那么你还剩下多少钱？下面这个简单的程序片段企图回答这个问题：

```java
System.out.println(1.03 - 0.42);
```

> Unfortunately, it prints out 0.6100000000000001. This is not an isolated case. Suppose you have a dollar in your pocket, and you buy nine washers priced at ten cents each. How much change do you get?

遗憾地是，它打印的结果是0.6100000000000001。这不是个别的例子，假如你现在口袋里1美元，然后你买了9个washer，每个washer为10美分，那么应该找多少钱的零头呢？

```java
System.out.println(1.00 - 9 * 0.10);
```

> According to this program fragment, you get $0.09999999999999998.
>  You might think that the problem could be solved merely by rounding results prior to printing, but unfortunately this does not always work. For example, suppose you have a dollar in your pocket, and you see a shelf with a row of delicious candies priced at 10¢, 20¢, 30¢, and so forth, up to a dollar. You buy one of each candy, starting with the one that costs 10¢, until you can’t afford to buy the next candy on the shelf. How many candies do you buy, and how much change do you get? Here’s a naive program designed to solve this problem:

根据这个程序片段，应该给你找$0.09999999999999998。

你可能会认为这种问题只要做一下舍入就可以解决，但是不幸的事，这并不是每次都有用。比如，假如你的口袋里有1美元，然后你看到一排美味的糖果，标价分别为10美分，20美分，一直到1美元，你打算从10美分的开始买，直到买不起了为止，那么你可以买多少个糖果呢，以及会给你找多少钱呢？下面是一个简单的程序，用来解决这个问题：

```java
// Broken - uses floating point for monetary calculation!
   public static void main(String[] args) {
       double funds = 1.00;
       int itemsBought = 0;
       for (double price = 0.10; funds >= price; price += 0.10) {
           funds -= price;
           itemsBought++;
       }
       System.out.println(itemsBought + " items bought.");
       System.out.println("Change: $" + funds);
   }
```

> If you run the program, you’ll find that you can afford three pieces of candy, and you have $0.3999999999999999 left. This is the wrong answer! The right way to solve this problem is to **use** **BigDecimal**, **int, or long** **for monetary calculations**.

如果你运行这个程序，你会发现你只能买3个糖果，会给你找零$0.3999999999999999。这个答案是错误的，这个问题的正确解决方法是使用**BigDecimal，int，或者long 来做货币计算。

> Here’s a straightforward transformation of the previous program to use the BigDecimal type in place of double. Note that BigDecimal’s String constructor is used rather than its double constructor. This is required in order to avoid introducing inaccurate values into the computation [Bloch05, Puzzle 2]:

下面是前面的程序直接使用BigDecimal替换double的版本。注意BigDecimal使用的是String的构造器，而不是double的构造器。这样做是为了避免把不准确的值带入到计算中[Bloch05, Puzzle 2]:

```java
public static void main(String[] args) {
       final BigDecimal TEN_CENTS = new BigDecimal(".10");
			 int itemsBought = 0;
			 BigDecimal funds = new BigDecimal("1.00"); 
       for (BigDecimal price = TEN_CENTS; funds.compareTo(price) >= 0;
            price = price.add(TEN_CENTS)) {
         
           funds = funds.subtract(price);
           itemsBought++;
       }
       System.out.println(itemsBought + " items bought.");
       System.out.println("Money left over: $" + funds);
}
```

> If you run the revised program, you’ll find that you can afford four pieces of candy, with $0.00 left over. This is the correct answer.

如果你运行这个修改后的程序的话，你就会发现你能买得起4块糖果，然后剩下$0.00 。这就是正确的答案。

> There are, however, two disadvantages to using BigDecimal: it’s a lot less convenient than using a primitive arithmetic type, and it’s a lot slower. The latter disadvantage is irrelevant if you’re solving a single short problem, but the former may annoy you.

然而使用BigDecimal有两个缺点：相对于使用基本类型，BigDecimal使用起来不是很方便，并且要慢得多。对于这种简单的问题，可能第二个缺点并不重要，第一个缺点可能会让你更烦一些。

> An alternative to using BigDecimal is to use int or long, depending on the amounts involved, and to keep track of the decimal point yourself. In this example, the obvious approach is to do all computation in cents instead of dollars. Here’s a straightforward transformation that takes this approach:

BIgDecimal的另一个替代方法是使用int或者long（这取决涉及的数组的大小），然后自己记录处理十进制小数点。在这个例子中，最明显的方法就是把所有的计算都以美分为单位，而不是美元，下面是这个采用这种方法的简单的版本：

```java
public static void main(String[] args) {
       int itemsBought = 0;
       int funds = 100;
       for (int price = 10; funds >= price; price += 10) {
           funds -= price;
           itemsBought++;
       }
       System.out.println(itemsBought + " items bought.");
       System.out.println("Cash left over: " + funds + " cents");
   }
```

> In summary, don’t use float or double for any calculations that require an exact answer. Use BigDecimal if you want the system to keep track of the decimal point and you don’t mind the inconvenience and cost of not using a primitive type. Using BigDecimal has the added advantage that it gives you full control over rounding, letting you select from eight rounding modes whenever an operation that entails rounding is performed. This comes in handy if you’re performing business calculations with legally mandated rounding behavior. If performance is of the essence, you don’t mind keeping track of the decimal point yourself, and the quantities aren’t too big, use int or long. If the quantities don’t exceed nine decimal digits, you can use int; if they don’t exceed eighteen digits, you can use long. If the quantities might exceed eighteen digits, use BigDecimal.

总结一下，对于任何需要准确结果的计算，都不要使用float和double。如果你希望系统记住十进制小数点，而且也不介意因为没有使用基本类型带来的麻烦和代价，就使用BIgDecimal。使用BIgDecimal还有一个优点是，你可以完全控住舍入，当一个操作需要执行舍入的时候，你可以从8种舍入模式中选择一种。当你执行的商业计算允许合法的强制舍入极端的时候，这个就可以派上用场了。如果性能很重要，你也不介意自己记录十进制小数点，数值范围也不太大，就使用int或者long。如果数值范围不超过9位十进制数，就使用int；如果不超过18位十进制数，就可以使用long。如果数值方位超过了18位十进制数，还是要使用BIgDecimal。