### Item68 遵守普遍接受的命名规范

> The Java platform has a well-established set of *naming conventions*, many of which are contained in *The Java Language Specification* [JLS, 6.1]. Loosely speaking, naming conventions fall into two categories: typographical and grammatical.

Java平台有一套很好的命名规范，有很多都包含在了“Java语言规范”里面[JLS, 6.1]。粗略来说，命名规范可以分为两大类：字面上的和语法的。

> There are only a handful of typographical naming conventions, covering packages, classes, interfaces, methods, fields, and type variables. You should rarely violate them and never without a very good reason. If an API violates these conventions, it may be difficult to use. If an implementation violates them, it may be difficult to maintain. In both cases, violations have the potential to confuse and irritate other programmers who work with the code and can cause faulty assumptions that lead to errors. The conventions are summarized in this item.

有一堆的字面上的命名规范，覆盖了包、类、接口、方法、域和类型变量。不到万不得已，千万不要违反这些规范。如果一个API违反了这些规范，那么它用起来就很难。如果一个实现违反了这些规范，那么就会很难维护。在这两种情况下，违反规范都会潜在地造成其他和这些代码一起工作的程序员带来苦恼，可能造成一些会导致问题的有缺陷的假设。在本节中对这些规范进行了总结。

> Package and module names should be hierarchical with the components separated by periods. Components should consist of lowercase alphabetic characters and, rarely, digits. The name of any package that will be used outside your organization should begin with your organization’s Internet domain name with the components reversed, for example, edu.cmu, com.google, org.eff. The standard libraries and optional packages, whose names begin with java and javax, are exceptions to this rule. Users must not create packages or modules whose names begin with java or javax. Detailed rules for converting Internet domain names to package name prefixes can be found in the JLS [JLS, 6.1].

包和模块名字应该是层次结构的，各个部分使用句号进行分隔。每部分应该是由小写的字符组成的，或者是数字。任何要在你的组织之外使用的包，其名字应该是你的组织的互联网域名倒过来组成每一部分，比如 edu.cmu, com.google, org.eff。标准类库和可选择的包使用java和javax开头，是这个规则的一个例外。用户决不能使用java和javax作为自己创建的包或者模块的开头。关于把互联网域名转换为包名前缀的细节请查看JLS [JLS, 6.1]。

> The remainder of a package name should consist of one or more components describing the package. Components should be short, generally eight or fewer characters. Meaningful abbreviations are encouraged, for example, util rather than utilities. Acronyms are acceptable, for example, awt. Components should generally consist of a single word or abbreviation.

包名中剩下的一个或多个部分是用来描述这个包的，这些部分应该竟可能的简洁，通常来说包含8个或者更少的字符。建议使用一些有意义的缩略词，比如，util而不是utilities。首字母缩略词也是可以接受的，比如，awt。每个部分通常都包含一个单词或者一个缩略词。

> Many packages have names with just one component in addition to the Internet domain name. Additional components are appropriate for large facilities whose size demands that they be broken up into an informal hierarchy. For example, the javax.util package has a rich hierarchy of packages with names such as java.util.concurrent.atomic. Such packages are known as *subpackages*, although there is almost no linguistic support for package hierarchies.

有很多的包，除了互联网域名以外，就只有一个部分。对于大型的项目而言，增加一些部分是很有必要的，因为项目的规模要求它们分割成一个非正式的层次结构。比如 javax.util 包就有一个基于名字的丰富的层次结构，比如java.util.concurrent.atomic。这种包被称为子包，但是Java语言中对于包层次并没有什么语言层面的支持。

> Class and interface names, including enum and annotation type names, should consist of one or more words, with the first letter of each word capitalized, for example, List or FutureTask. Abbreviations are to be avoided, except for acronyms and certain common abbreviations like max and min. There is some disagreement as to whether acronyms should be uppercase or have only their first letter capitalized. While some programmers still use uppercase, a strong argument can be made in favor of capitalizing only the first letter: even if multiple acronyms occur back-to-back, you can still tell where one word starts and the next word ends. Which class name would you rather see, HTTPURL or HttpUrl?

类和接口的名称，包括枚举和注解类型的名称，都应该包含一个或多个单词，每个单词的第一个字母都大写。比如List和FutureTask。除了首字母缩写和一些常见的缩写（比如max和min）以外，应该要避免使用缩写。对于首字母缩写的词是应该全部大写还是其首字母大写，是有一些争议的。虽然很多程序员一致使用全部大写，但是还是建议只大写首字母：因为即使有几个首字母缩写挨着，你也能很清晰分辨出一个单词的开始和结束。比如HttpUrl和HTTPURL，你更喜欢哪种类名？

> Method and field names follow the same typographical conventions as class and interface names, except that the first letter of a method or field name should be lowercase, for example, remove or ensureCapacity. If an acronym occurs as the first word of a method or field name, it should be lowercase.

方法和域名的字面上的命名规范与类和接口的一样，只是方法和域的名字的第一个字母需要小写。比如remove和ensureCapacity。如果有一个首字母缩略词出现在方法或者域名字的第一个单词的问题，那么它应该小写。

> The sole exception to the previous rule concerns “constant fields,” whose names should consist of one or more uppercase words separated by the underscore character, for example, VALUES or NEGATIVE_INFINITY. A constant field is a static final field whose value is immutable. If a static final field has a primitive type or an immutable reference type (Item 17), then it is a constant field. For example, enum constants are constant fields. If a static final field has a mutable reference type, it can still be a constant field if the referenced object is immutable. Note that constant fields constitute the *only* recommended use of underscores.

前面这一规则的一个单独的例外，是关于”常量域“的，它的名字应该是包含一个或者多个使用下划线隔开的大写单词的。比如 VALUES 或者NEGATIVE_INFINITY。一个常量域是一个静态final域的话，它的值是不可变的。如果静态final域持有的类型是基本类型或者不可变引用类型（Item17），那么它就是一个常量域。比如枚举常量都是常量域。如果一个静态final域持有的类型是可变引用类型，如果它引用的对象不可变的话，它还是一个常量域。注意，常量域是唯一推荐使用下划线的场景。

> Local variable names have similar typographical naming conventions to member names, except that abbreviations are permitted, as are individual characters and short sequences of characters whose meaning depends on the context in which they occur, for example, i, denom, houseNum. Input parameters are a special kind of local variable. They should be named much more carefully than ordinary local variables, as their names are an integral part of their method’s documentation.

本地变量名字的字面命令规范会成员变量相似，只不过它允许缩写，比如含义取决于出现的上下文的单个字符和短字符序列，比如，i，denom，houseNum。输入参数也是一种特殊的本地变量。他们的命名必须要比普通的本地变量细心一些，因为他们的名字组成了他们的方法的文档的一部分。

> Type parameter names usually consist of a single letter. Most commonly it is one of these five: T for an arbitrary type, E for the element type of a collection, K and V for the key and value types of a map, and X for an exception. The return type of a function is usually R. A sequence of arbitrary types can be T, U, V or T1, T2, T3.

类型参数名字通常是由单个字母组成的，这些字母 通常是下面这些5个中的其中一个：T表示任意的类型，E表示集合中的元素，K和V表示map的键和值，X表示异常。一个函数的返回类型通常是R。一系列的任意类型可以是T，U，V 或者T1，T2，T3。

> For quick reference, the following table shows examples of typographical conventions.

为了可以快速地查询，下面这个表格展示了这些字面规范的例子：

| 标识符类型 | 示例                                             |
| ---------- | ------------------------------------------------ |
| 包或者模块 | org.junit.jupiter.api, com.google.common.collect |
| 类或者接口 | Stream, FutureTask, LinkedHashMap, HttpClient    |
| 方法或者域 | remove, groupingBy, getCrc                       |
| 常量域     | MIN_VALUE, NEGATIVE_INFINITY                     |
| 本地变量   | i, denom, houseNum                               |
| 类型参数   | T,E,K,V,X,R,U,V,T1,T2                            |

> Grammatical naming conventions are more flexible and more controversial than typographical conventions. There are no grammatical naming conventions to speak of for packages. Instantiable classes, including enum types, are generally named with a singular noun or noun phrase, such as Thread, PriorityQueue, or ChessPiece. Non-instantiable utility classes (Item 4) are often named with a plural noun, such as Collectors or Collections. Interfaces are named like classes, for example, Collection or Comparator, or with an adjective ending in able or ible, for example, Runnable, Iterable, or Accessible. Because annotation types have so many uses, no part of speech predominates. Nouns, verbs, prepositions, and adjectives are all common, for example, BindingAnnotation, Inject, ImplementedBy, or Singleton.

语法上的命名规范要比字面量命名规范要灵活一些，也更有争议一些。对于包，没有语法上的命名规范。可实例化的类，包括枚举类型，通常是使用单数名词或者名词短语进行命名，比如Thread，PriorityQueue，ChessPiece。不可实例化的工具类（Item4）通常使用复数名词进行命名，比如Collectors，Collections。接口的命名和类相似，比如，Collection或者Comparator。或者使用一个以able或者ible结尾的形容词，比如Runnable，Iterable或者Accessible。由于注解类型有很多种使用，没有那种占了绝大多数，因此名词，动词，介词，和形容词都很常见，比如BindingAnnotation, Inject, ImplementedBy, 或者 Singleton。

> Methods that perform some action are generally named with a verb or verb phrase (including object), for example, append or drawImage. Methods that return a boolean value usually have names that begin with the word is or, less com- monly, has, followed by a noun, noun phrase, or any word or phrase that functions as an adjective, for example, isDigit, isProbablePrime, isEmpty, isEnabled, or hasSiblings.

执行一些动作的方法通常使用一个动词或者动词短语（包括对象）来进行命名，比如，append或者drawImage。返回一个boolean值的方法的名字通常以单词is，或者has（比较少）开头，然后紧跟着一个名词，名词短语或者有形容词功能的名词或者短语。比如isDigit, isProbablePrime, isEmpty, isEnabled, 或者hasSiblings。

> Methods that return a non-boolean function or attribute of the object on which they’re invoked are usually named with a noun, a noun phrase, or a verb phrase beginning with the verb get, for example, size, hashCode, or getTime. There is a vocal contingent that claims that only the third form (beginning with get) is acceptable, but there is little basis for this claim. The first two forms usu- ally lead to more readable code, for example:

如果方法返回本调用对象的非boolean的函数或者属性，通常使用名词，名词短语或者以get开头的动词短语来命名。比如size, hashCode, 或者getTime。有一个组织声称只有第三种形式（即以get开头）才能被接受，但是这个声明并没有被接受。前两种得到的代码的可读性通常来说要高一些，比如：

```java
 if (car.speed() > 2 * SPEED_LIMIT)
       generateAudibleAlert("Watch out for cops!");
```

> The form beginning with get has its roots in the largely obsolete *Java Beans* specification, which formed the basis of an early reusable component architecture. There are modern tools that continue to rely on the Beans naming convention, and you should feel free to use it in any code that is to be used in conjunction with these tools. There is also a strong precedent for following this naming convention if a class contains both a setter and a getter for the same attribute. In this case, the two methods are typically named get*Attribute* and set*Attribute*.

这种以get开头的命名形式一开始出现在过时的Java Beans的规范里，它组成了一些早期可重用组件框架的基础。有一些现代工具也依赖Beans的命名规范，你可以在需要和这些工具结合的代码中使用这种形式。当类中针对同一个属性既有setter也有getter的时候，也强烈建议遵循这种命名规范。在这种情况下，这两个方法被命名为getAttribute和setAttribute。

> A few method names deserve special mention. Instance methods that convert the type of an object, returning an independent object of a different type, are often called to*Type*, for example, toString or toArray. Methods that return a *view* (Item 6) whose type differs from that of the receiving object are often called as*Type*, for example, asList. Methods that return a primitive with the same value as the object on which they’re invoked are often called *type*Value, for example, intValue. Common names for static factories include from, of, valueOf, instance, getInstance, newInstance, get*Type*, and new*Type* (Item 1, page 9).

一些方法的名字需要特殊的关注。转换对象的类型，返回一个独立的不同类型对象的实例方法，通常命名为toType，比如toString，toArray。返回一个和传入对象不同类型的试图的方法，通常命名为asType，比如，asList。返回一个对象的值相等的基本类型的方法，通常命名为typeValue，比如intValue。静态工厂的常见的命名有 from, of, valueOf, instance, getInstance, newInstance, get*Type*, 和 new*Type* (Item 1).

> Grammatical conventions for field names are less well established and less important than those for class, interface, and method names because well- designed APIs contain few if any exposed fields. Fields of type boolean are often named like boolean accessor methods with the initial is omitted, for example, initialized, composite. Fields of other types are usually named with nouns or noun phrases, such as height, digits, or bodyStyle. Grammatical conventions for local variables are similar to those for fields but even weaker.

相对于类，接口和方法的名字，域的命名语法规范并没有很好地建立起来，也不那么重要，因为一个设计良好的API中包含的暴露的域很少。boolean类型的域命名和boolean访问方法很像，省略了前面的is，比如initialized, composite。其他类型的域通常是使用名词或者名词短语进行命名的，比如height, digits, or bodyStyle。本地变量的语法命名规范和域相似，但是要更弱一些。

> To summarize, internalize the standard naming conventions and learn to use them as second nature. The typographical conventions are straightforward and largely unambiguous; the grammatical conventions are more complex and looser. To quote from *The Java Language Specification* [JLS, 6.1], “These conventions should not be followed slavishly if long-held conventional usage dictates other- wise.” Use common sense.

总结一下，把这些标准的命名规范当做是一种内在机制，并且当做第二特性还学习使用它们。字面规范是直接且明确的；而语法规范要更复杂和宽松一些。引用*The Java Language Specification* [JLS, 6.1]里的一句话，”当长期养成的使用习惯和这些相悖的时候，请不要盲目地遵守这些规范。“应该使用大家公认的。