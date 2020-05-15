## 9 通用编程

> **T**HIS chapter is devoted to the nuts and bolts of the language. It discusses local variables, control structures, libraries, data types, and two extralinguistic facilities: *reflection* and *native methods*. Finally, it discusses optimization and naming conventions.

本章致力于讨论Java语言的细枝末节。讨论了本地变量，控制结构，类库，数据类型，以及两种语言外的机制：反射和本地方法。最后讨论了优化和命名习惯。

### Item57 局部变量作用域最小化

> This item is similar in nature to Item 15, “Minimize the accessibility of classes and members.” By minimizing the scope of local variables, you increase the readability and maintainability of your code and reduce the likelihood of error.

本节本质上和Item15很像，Item15说“最小化类和成员的可访问性”。通过最小化局部变量的作用域，可以增强代码的可读性和可维护性，并减少出错的可能性。

> Older programming languages, such as C, mandated that local variables must be declared at the head of a block, and some programmers continue to do this out of habit. It’s a habit worth breaking. As a gentle reminder, Java lets you declare variables anywhere a statement is legal (as does C, since C99).

一些老的编程语言，比如C语言，要求局部变量必须声明在代码块的开头，因此有些程序员一直都留有这个习惯。这个习惯应该改变。提醒一下，Java允许你在任何可以语句合法的地方出现变量声明(自C99后，C语言也这样了)

> **The most powerful technique for minimizing the scope of a local variable is to declare it where it is first used.** If a variable is declared before it is used, it’s just clutter—one more thing to distract the reader who is trying to figure out what the program does. By the time the variable is used, the reader might not remember the variable’s type or initial value.

**将局部变量作用域最小化的最有效的方法就是在第一次使用变量的地方，进行变量声明。**如果变量在使用之前声明，只会造成混乱——对于那些企图弄懂程序在干什么的读者来说，又多了一个需要分心的东西。等到用这个变量的时候，读者可能已经忘记了变量的类型和初始值了。

> Declaring a local variable prematurely can cause its scope not only to begin too early but also to end too late. The scope of a local variable extends from the point where it is declared to the end of the enclosing block. If a variable is declared outside of the block in which it is used, it remains visible after the program exits that block. If a variable is used accidentally before or after its region of intended use, the consequences can be disastrous.

提前声明局部变量，会使得变量的作用域不仅仅提前开始了，而且还延后结束了。局部变量的作用域就从声明开始延长到了外围块结束的地方。如果一个变量在使用的块的外面进行了声明，当程序退出这个块的时候，这个变量还是可见的。如果一个变量不小心在目标使用区域之前或者之后使用了，就会造成灾难性的结果。

> **Nearly every local variable declaration should contain an initializer.** If you don’t yet have enough information to initialize a variable sensibly, you should postpone the declaration until you do. One exception to this rule concerns try- catch statements. If a variable is initialized to an expression whose evaluation can throw a checked exception, the variable must be initialized inside a try block (unless the enclosing method can propagate the exception). If the value must be used outside of the try block, then it must be declared before the try block, where it cannot yet be “sensibly initialized.” For an example, see page 283.

**几乎每一个变量的声明都应该包含初始化。**如果你还不能获得确定的信息来将这个变量初始化，那么就应该延迟声明直到你能初始化为止。这个规则有一个例外，是和try-catch语句有关。如果一个变量初始化的表达式会抛出受检异常，那么这个变量就必须在try块 里面进行初始化（除非外围方法可以直接传递异常）。如果这个值必须在try块的外部使用，那么这个变量就必须在try块之前声明，但是在这个时候，这个变量还不能被“有效地初始化”。可以参照Item65中的例子。

> Loops present a special opportunity to minimize the scope of variables. The for loop, in both its traditional and for-each forms, allows you to declare *loop variables*, limiting their scope to the exact region where they’re needed. (This region consists of the body of the loop and the code in parentheses between the for keyword and the body.) Therefore, **prefer** **for** **loops to** **while** **loops**, assuming the contents of the loop variable aren’t needed after the loop terminates.

循环提供了一个特殊的机会来将变量的作用域最小化。在传统的和for-each形式的for循环里，都允许你声明循环变量，限制了变量的作用域就是他们真正需要的区域（这个区域包块循环体，和在for关键字和循环体之间圆括号里面的代码（*也就是循环的初始化，测试，更新部分*））。因此，当循环的变量在循环终止后不在需要的时候，**for循环优先于while循环**。

> For example, here is the preferred idiom for iterating over a collection (Item 58):

比如，下面是一个对集合进行迭代的首选做法：

```java
// Preferred idiom for iterating over a collection or array
   for (Element e : c) {
       ... // Do Something with e
}
```

> If you need access to the iterator, perhaps to call its remove method, the preferred idiom uses a traditional for loop in place of the for-each loop:

如果你需要访问集合的迭代器，你可能需要调用它的remove方法，优先使用传统for循环来代替for-each循环：

```java
// Idiom for iterating when you need the iterator
   for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
       Element e = i.next();
       ... // Do something with e and i
}
```

> To see why these for loops are preferable to a while loop, consider the following code fragment, which contains two while loops and one bug:

为了弄清楚为什么for循环优先于while循环，我们来看看下面这个包含两个while循环和一个bug的代码片段：

```java
	 Iterator<Element> i = c.iterator();
   while (i.hasNext()) {
       doSomething(i.next());
   }
...
   Iterator<Element> i2 = c2.iterator(); 
	 while (i.hasNext()) { // BUG!
       doSomethingElse(i2.next());
   }
```

> The second loop contains a copy-and-paste error: it initializes a new loop variable, i2, but uses the old one, i, which is, unfortunately, still in scope. The resulting code compiles without error and runs without throwing an exception, but it does the wrong thing. Instead of iterating over c2, the second loop terminates immediately, giving the false impression that c2 is empty. Because the program errs silently, the error can remain undetected for a long time.

第二个循环中包含一个复制粘贴错误：它初始化了一个新的循环变量i2，但是不幸的是，在循环域中还是使用了老的变量i。这样生成的代码在编译的时候不会出错，在运行的时候也不会有异常，但是它做的事情确实是错误的。第二个循环立马终止了，而不是c2进行了迭代，给人一种c2是空的 的错觉。由于这个程序的错误时悄悄发生的，因此这个错误可能存在很长时间都不会被发现。

> If a similar copy-and-paste error were made in conjunction with either of the for loops (for-each or traditional), the resulting code wouldn’t even compile. The element (or iterator) variable from the first loop would not be in scope in the second loop. Here’s how it looks with the traditional for loop:

如果这个复制粘贴错误发生在任何一种循环形式（for-each或者传统）里。得到的 代码根本就不能编译。这个第一个循环里的元素（或者迭代器）变量的作用域根本就不包括第二个循环。下面是使用传统的for循环的样子：

```java
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
       Element e = i.next();
       ... // Do something with e and i
} ...
// Compile-time error - cannot find symbol i
for (Iterator<Element> i2 = c2.iterator(); i.hasNext(); ) {
       Element e2 = i2.next();
       ... // Do something with e2 and i2
   }
```

> Moreover, if you use a for loop, it’s much less likely that you’ll make the copy-and-paste error because there’s no incentive to use different variable names in the two loops. The loops are completely independent, so there’s no harm in reusing the element (or iterator) variable name. In fact, it’s often stylish to do so.
>
> The for loop has one more advantage over the while loop: it is shorter, which enhances readability. Here is another loop idiom that minimizes the scope of local variables:

并且，如果你使用for循环，就基本不可能发生这种复制粘贴的错误，因为没有必要在两个for循环中使用不同的变量名字。for循环之间是完全独立的，因此重用元素（或者迭代器）变量面子，是不会有任何问题的。实际上，这也是一种很流行的做法。

相对于while循环，for循环还有一个优势：就是for循环要更短一些，可读性也更高。下面是另一个对局部变量作用域最小化的for循环示例：

```java
for (int i = 0, n = expensiveComputation(); i < n; i++) {
       ... // Do something with i;
}
```

> The important thing to notice about this idiom is that it has *two* loop variables, i and n, both of which have exactly the right scope. The second variable, n, is used to store the limit of the first, thus avoiding the cost of a redundant computation in every iteration. As a rule, you should use this idiom if the loop test involves a method invocation that is guaranteed to return the same result on each iteration.

其中有一个很重要的事情，这个例子有两个循环变量， i和n，这两个变量都有正确的作用域。第二个变量n用来保存第一个变量i的极限值，避免了在每次迭代中都进行多余的计算。通常来说，如果循环测试中包含了一个方法调用，并且这个方法调用在每次迭代中都会返回相同的值，那么就应该使用这种做法。

> A final technique to minimize the scope of local variables is to **keep methods small and focused.** If you combine two activities in the same method, local variables relevant to one activity may be in the scope of the code performing the other activity. To prevent this from happening, simply separate the method into two: one for each activity.

最后一个把局部变量作用域最小化的方法是**保持方法小而集中**。如果你把两个操作放在了一个方法里，那么一个操作的局部变量的作用域 可能会包括第另一个操作的代码。为了避免这种情况发生，可以简单地把这个方法拆分成两个：一个操作对应一个方法。