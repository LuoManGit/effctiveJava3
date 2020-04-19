### Item12 始终要覆盖toString

> While Object provides an implementation of the toString method, the string that it returns is generally not what the user of your class wants to see. It consists of the class name followed by an “at” sign (@) and the unsigned hexadecimal representation of the hash code, for example, PhoneNumber@163b91. The general contract for toString says that the returned string should be “a concise but informative representation that is easy for a person to read.” While it could be argued that PhoneNumber@163b91 is concise and easy to read, it isn’t very informative when compared to 707-867-5309. The toString contract goes on to say, “It is recommended that all subclasses override this method.” Good advice, indeed!

虽然Object为toString方法提供了默认实现，但是其返回的字符串一般情况下都不是类的用户想看到的那样。该字符串包含了这个类的名字，紧跟着一个@符号，和一串hash值的无符号16进制表示，比如PhoneNumber@163b91。toString的通用约定说，返回的字符串应该是简洁但又信息量丰富且便于人们阅读的。虽然PhoneNumber@163b91很简洁也很好读，但是相对于“707-867-5309”而言，显然信息量不太丰富。toString约定也说了：“推荐所有的子类都覆盖这个方法”。这是一个非常好的建议，真的！

> While it isn’t as critical as obeying the equals and hashCode contracts (Items 10 and 11), **providing a good** **toString** **implementation makes your class much more pleasant to use and makes systems using the class easier to debug**. The toString method is automatically invoked when an object is passed to println, printf, the string concatenation operator, or assert, or is printed by a debugger. Even if you never call toString on an object, others may. For example, a component that has a reference to your object may include the string representation of the object in a logged error message. If you fail to override toString, the message may be all but useless.
>
> If you’ve provided a good toString method for PhoneNumber, generating a useful diagnostic message is as easy as this:

虽然遵守toString约定不像遵守equals和hashCode（Item10，Item11）那么重要。**提供一个好的toString实现可以使得我们的类更好用，同时也使得使用这个类进行调试更容易。当一个对象被传递给println，printf，字符串连接操作，assert或者是被调试器打印出来的时候，系统会自动的调用其toString方法。即使你永远都不需要调用这个对象的toString方法，但其他人可能需要。比如，有个组件引用了你的对象，在错误日志信息里会包含这个对象的字符串表示，如果你没有覆盖toString方法的话，这些信息就会没什么用处。

如果你为PhoneNumber提供了一个好的toString方法，要生成一个有用的诊断信息就可以像下面这么容易：

```java
System.out.println("Failed to connect to " + phoneNumber);
```

> Programmers will generate diagnostic messages in this fashion whether or not you override toString, but the messages won’t be useful unless you do. The benefits of providing a good toString method extend beyond instances of the class to objects containing references to these instances, especially collections. Which would you rather see when printing a map, {Jenny=PhoneNumber@163b91} or {Jenny=707-867-5309}?

不管你有没有覆盖toString方法，程序员都习惯用这种方法来生成一个诊断信息，但是如果你没有覆盖toString的话，这个信息就没什么大用。提供一个好的toString受益的不仅仅是这个类的实例们，还包括那些包含这些实例的对象们，尤其是集合。当打印一个map的时候，你想看到{Jenny=PhoneNumber@163b91} 还是 {Jenny=707-867-5309}?

> **When practical, the** **toString** **method should return** **all** **of the interesting information contained in the object**, as shown in the phone number example. It is impractical if the object is large or if it contains state that is not conducive to string representation. Under these circumstances, toString should return a summary such as Manhattan residential phone directory (1487536 listings) or Thread[main,5,main]. Ideally, the string should be self-explanatory. (The Thread example flunks this test.) A particularly annoying penalty for failing to include all of an object’s interesting information in its string representation is test failure reports that look like this:

**在实际应用中，toString方法返回值应该包含这个对象里所有的值得关注的信息**，就像phoneNumber的toString一样。当一个对象太大或者其包含的状态很难用字符串来表达时，这样做就有点不合实际。在这种情况下，toString方法可以返回一个总结，比如Manhattan居民电话本就可以返回“1487536 listings",线程Thread就可以返回”Thread[main,5,main]“。理想情况下，返回的字符串应该是可以自解释的（显然Thread这个例子就不可以）。当返回的字符串中没有包含所有值得关注的信息时，最烦人的就是测试失败时，可能得到的报告像下面这样：

```
Assertion failure: expected {abc, 123}, but was {abc, 123}.
```

> One important decision you’ll have to make when implementing a toString method is whether to specify the format of the return value in the documentation. It is recommended that you do this for *value classes*, such as phone number or matrix. The advantage of specifying the format is that it serves as a standard, unambiguous, human-readable representation of the object. This representation can be used for input and output and in persistent human-readable data objects, such as CSV files. If you specify the format, it’s usually a good idea to provide a matching static factory or constructor so programmers can easily translate back and forth between the object and its string representation. This approach is taken by many value classes in the Java platform libraries, including BigInteger, BigDecimal, and most of the boxed primitive classes

当你要实现一个toString方法的饿时候，需要好好考虑是否需要在文档中确定返回值的格式。如果是值类的话，建议你确定一下，比如PhoneNumber和矩阵。确定格式的优势在于它为对象提供了一个标准的、明确的、适合人类阅读的描述方式。这种描述方式可以用来输入和输出，以及适合人类阅读的永久数据对象，比如CSV文件。如果你明确了格式，就可以提供一个对应的静态工厂方法或者是构造器，然后程序员就可以简单地对对象和其字符串表达进行转换。在Java平台类库中，有很多的值类都采用了这种做法，包括BIgInteger，BigDecimal和绝大多数的原始类型包装类。

> The disadvantage of specifying the format of the toString return value is that once you’ve specified it, you’re stuck with it for life, assuming your class is widely used. Programmers will write code to parse the representation, to generate it, and to embed it into persistent data. If you change the representation in a future release, you’ll break their code and data, and they will yowl. By choosing not to specify a format, you preserve the flexibility to add information or improve the format in a subsequent release.

明确toString方法返回值的一个缺点就是，一旦你确定了这个格式，如果你的类被广泛使用的话，你就一辈子都只能用这个格式了。程序员可能会写代码去解析这个字符串表达，生成它，甚至把它写到持久化的数据里。如果你在后面的发行版本中，修改了这个字符串表达，那么你就会破坏他们的数据和代码，他们就会抓狂。如果你没有确定格式的话，在后面的发行版本中，你就可以拥有添加信息或改进格式的自由。

> **Whether or not you decide to specify the format, you should clearly document your intentions.** If you specify the format, you should do so precisely. For example, here’s a toString method to go with the PhoneNumber class in Item 11:

不管你决定是否确认格式，你都应该在文档里清除地表达你的意图。如果你要确认格式，就应该特别严格的去做，如下，是PhoneNumber（item11）的一个toString方法：

```java
/**
* Returns the string representation of this phone number.
* The string consists of twelve characters whose format is
* "XXX-YYY-ZZZZ", where XXX is the area code, YYY is the
* prefix, and ZZZZ is the line number. Each of the capital
* letters represents a single decimal digit.
*
* If any of the three parts of this phone number is too small
* to fill up its field, the field is padded with leading zeros.
* For example, if the value of the line number is 123, the last 
* four characters of the *string representation will be "0123".
*/
@Override public String toString() {
       return String.format("%03d-%03d-%04d",areaCode, prefix, lineNum);
}
```

> If you decide not to specify a format, the documentation comment should read something like this:

如果你确定不把格式定死，那么文档里也应该有如下的指示信息：

```java
/**
* Returns a brief description of this potion. The exact details 
* of the representation are unspecified and subject to change, 
* but the following may be regarded as typical:
* 
* "[Potion #9: type=love, smell=turpentine, look=india ink]"
*/
@Override public String toString() { ... }
```

> After reading this comment, programmers who produce code or persistent data that depends on the details of the format will have no one but themselves to blame when the format is changed.

读完这个指示后，那些写代码或者永久数据依赖格式的细节的程序员，当格式发生了变化时，就只能怪自己了。

> Whether or not you specify the format, **provide programmatic access to the information contained in the value returned by** **toString** . For example, the PhoneNumber class should contain accessors for the area code, prefix, and line number. If you fail to do this, you *force* programmers who need this information to parse the string. Besides reducing performance and making unnecessary work for programmers, this process is error-prone and results in fragile systems that break if you change the format. By failing to provide accessors, you turn the string format into a defacto API, even if you’ve specified that it’s subject to change.

不管你是否确认格式，都需要为toString返回值里包含的信息提供一个可以编访问的方式。比如，PhoneNumber类就应该包含 area code, prefix, 和 line number的访问方式。如果你没有提供的话，需要这些信息的程序员就不得不去解析这个字符串。除了会降低性能，给程序员带来一些不必要的工作以外，这种方法也很容易出错，系统也不稳定，一旦你修改了格式，整个系统就崩溃了。如果不提供这些访问方式，即使你明确说了这个格式会被修改，但是这个格式也成了事实上的API。

> It makes no sense to write a toString method in a static utility class (Item 4). Nor should you write a toString method in most enum types (Item 34) because Java provides a perfectly good one for you. You should, however, write a toString method in any abstract class whose subclasses share a common string representation. For example, the toString methods on most collection implementations are inherited from the abstract collection classes.

给一个静态工具类写一个toString方法是毫无意义的。同样地，也没有必要为大部分的枚举类型写toString方法，因为java本身就为你提供了一个很好的toString方法。然而， 你应该给每个 其子类共用一种字符串表达 的抽象类都写一个toString方法。比如大部分的集合实现的toString方法都是从抽象集合类哪里继承来的。

> Google’s open source AutoValue facility, discussed in Item 10, will generate a toString method for you, as will most IDEs. These methods are great for telling you the contents of each field but aren’t specialized to the *meaning* of the class. So, for example, it would be inappropriate to use an automatically generated toString method for our PhoneNumber class (as phone numbers have a standard string representation), but it would be perfectly acceptable for our Potion class. That said, an automatically generated toString method is far preferable to the one inherited from Object, which tells you *nothing* about an object’s value.

在第十章中讨论的Google的开源框架AutoValue和大部分的IDE，可以自动生成一个toString方法。这些方法都可以很好的告诉你每一个域的内容是什么，但是不会根据类的意义实现定制。比如，对于我们的PhoneNunber类而言，自动生成的toString方法就不合适，因为phoneNumber有一个标准的字符串表达方式。但是对于Position类，自动生成的toString就很适合。不管怎么说，自动生成的toString方法要比从Object类里继承来的要好得多，因为Object的toString方法里不能告诉你任何关于这个对象的值的信息。

> To recap, override Object’s toString implementation in every instantiable class you write, unless a superclass has already done so. It makes classes much more pleasant to use and aids in debugging. The toString method should return a concise, useful description of the object, in an aesthetically pleasing format.

总结一下，在你写的所有的可以实例化的类里都覆盖toString方法，除非这个类的父类已经覆盖过了。它会使得这个类用起来更舒适，也更便于调试。toStirng方法应该返回一个简洁、有用、美观的对象的描述。

### 