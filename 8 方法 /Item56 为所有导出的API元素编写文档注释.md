### Item56 为所有导出的API元素编写文档注释

> If an API is to be usable, it must be documented. Traditionally, API documentation was generated manually, and keeping it in sync with code was a chore. The Java programming environment eases this task with the *Javadoc* utility. Javadoc generates API documentation automatically from source code with specially formatted *documentation comments*, more commonly known as *doc comments*.

如果要使一个API真正可用，就必须要编写对应的文档。传统意义上的API文档是手动生成的，然后让文档和代码同步是一个非常繁琐的事情。现在的Java编程环境使用Javadoc技术简化了这个任务。Javadoc可以自动地从包含特殊格式的文档注释（通常被称为doc comments）的源代码中，生成API文档。

> While the doc comment conventions are not officially part of the language, they constitute a de facto API that every Java programmer should know. These conventions are described in the *How to Write Doc Comments* web page [Javadoc- guide]. While this page has not been updated since Java 4 was released, it is still an invaluable resource. One important doc tag was added in Java 9, {@index}; one in Java 8, {@implSpec}; and two in Java 5, {@literal} and {@code}. These tags are missing from the aforementioned web page, but are discussed in this item.

虽然这个文档注释约定还不是Java语言中的一部分。但是他们构成了所有程序员必须知道的事实API。这些约定的内容在How to Write Doc Comments网页上进行了说明 [Javadoc- guide]。虽然这些页面在自Java4以后就没有更新过了，但是还是非常宝贵的资源。在Java9中新增了一个重要的文档标签@index；Java8中新增了一个@implSpec；Java5中新增了两个@literal 和 @code。这些标签在前面提到的网页中没有介绍，但是在本节中进行了讨论。

> **To document your API properly, you must precede** **every** **exported class, interface, constructor, method, and field declaration with a doc comment.** If a class is serializable, you should also document its serialized form (Item 87). In the absence of a doc comment, the best that Javadoc can do is to reproduce the declaration as the sole documentation for the affected API element. It is frustrating and error-prone to use an API with missing documentation comments. Public classes should not use default constructors because there is no way to provide doc comments for them. To write maintainable code, you should also write doc comments for most unexported classes, interfaces, constructors, methods, and fields, though these comments needn’t be as thorough as those for exported API elements.

**为了对API进行合适的文档注释，你必须在每一个导出的类，接口，构造器，方法，域声明前面，写一个文档注释。**如果这个类是可序列化的，你也应该为它的序列化形式编写文档（Item87）。在缺少文档注释的情况下，Javadoc能做的就是也就是重新生成这个声明作为这个API的唯一的文档。使用没有文档的API让人很烦，而且也容易出错。公有的类不应该使用默认的构造器，因为没有办法为它们提供文档说明。为了编写的代码可维护，也应该为大部分的未导出的类，接口，构造器，方法，和域编写文档注释，即使它们不需要被导出作为API的一部分。

> **The doc comment for a method should describe succinctly the contract between the method and its client.** With the exception of methods in classes designed for inheritance (Item 19), the contract should say *what* the method does rather than *how* it does its job. The doc comment should enumerate all of the method’s *preconditions*, which are the things that have to be true in order for a client to invoke it, and its *postconditions*, which are the things that will be true after the invocation has completed successfully. Typically, preconditions are described implicitly by the @throws tags for unchecked exceptions; each unchecked exception corresponds to a precondition violation. Also, preconditions can be specified along with the affected parameters in their @param tags.

**一个方法的文档注释需要简要地描述方法和客户端之间的约定**。除了设计用于继承的类的方法以外，约定里必须说明这个方法做了什么，而不是怎么做的。文档注释必须要枚举所有的前提条件和后置条件。所谓前提条件就是要调用这个方法，客户端必须满足为true的条件；后置条件是成功调用了这个方法后，就会为true的条件。通常情况下，通过@throws标签抛出的非受检异常就含蓄地表示了前提条件，每一个非受检异常都对应一个前提违例。而且@param标签的受影响的参数也指定了前提条件。

> In addition to preconditions and postconditions, methods should document any *side effects*. A side effect is an observable change in the state of the system that is not obviously required in order to achieve the postcondition. For example, if a method starts a background thread, the documentation should make note of it.

除了前提条件和后置条件以外，方法的文档注释还必须说明所有的副作用。副作用是指系统的状态的变化，它不是为了达到后置条件的明显的要求。比如，如果一个方法启动了一个后台线程，文档中就必须说明这一点。

> To describe a method’s contract fully, the doc comment should have an @param tag for every parameter, an @return tag unless the method has a void return type, and an @throws tag for every exception thrown by the method, whether checked or unchecked (Item 74). If the text in the @return tag would be identical to the description of the method, it may be permissible to omit it, depending on the coding standards you are following.

为了完整地描述方法的约定，文档注释中，针对每个参数，都要有一个@param标签；非void的返回类型应该有一个@retrun标签；每个方法抛出的受检或非受检异常，都有一个@throw标签。如果@return中的文本和方法描述中的一致，就可以忽略这个描述，这取决于你使用的编码规范。

> By convention, the text following an @param tag or @return tag should be a noun phrase describing the value represented by the parameter or return value. Rarely, arithmetic expressions are used in place of noun phrases; see BigInteger for examples. The text following an @throws tag should consist of the word “if,” followed by a clause describing the conditions under which the exception is thrown. By convention, the phrase or clause following an @param, @return, or @throws tag is not terminated by a period. All of these conventions are illustrated by the following doc comment:

按照约定，这个@param和@return标签后面的文本应该是一个名词短语，用来描述参数和返回值表示的值。在极少数情况下，也可能使用数学表达式来代替名词短语。比如BigInteger，@throw标签后面的文本就包含了if，紧跟着异常会抛出的条件描述分句。按照约定，紧跟着@param，@return，@throws的短语和分句都不用句号作为结果。下面这个例子展示了上面所有的约定：

```java
/**
* Returns the element at the specified position in this list. *
* <p>This method is <i>not</i> guaranteed to run in constant
* time. In some implementations it may run in time proportional * to the element position.
*
* @param index index of element to return; must be
* non-negative and less than the size of this list
* @return the element at the specified position in this list
* @throws IndexOutOfBoundsException if the index is out of range
*         ({@code index < 0 || index >= this.size()})
*/
E get(int index);
```

> Notice the use of HTML tags in this doc comment (<p> and <i>). The Javadoc utility translates doc comments into HTML, and arbitrary HTML elements in doc comments end up in the resulting HTML document. Occasionally, programmers go so far as to embed HTML tables in their doc comments, although this is rare.

注意，在这个文档注释中使用了HTML标签(<p> 和 <i>)。javadoc工具会把文档注释转化为HTML，并且，任何的在文档注释中的任何的HTML元素最终都会变成HTML文档。有的时候，程序员还会使用HTML的表格在文档注释中，虽然这比较少见。

> Also notice the use of the Javadoc {@code} tag around the code fragment in the @throws clause. This tag serves two purposes: it causes the code fragment to be rendered in code font, and it suppresses processing of HTML markup and nested Javadoc tags in the code fragment. The latter property is what allows us to use the less-than sign (<) in the code fragment even though it’s an HTML metacharacter. To include a multiline code example in a doc comment, use a Javadoc {@code} tag wrapped inside an HTML <pre> tag. In other words, precede the code example with the characters <pre>{@code and follow it with }</pre>. This preserves line breaks in the code, and eliminates the need to escape HTML metacharacters, but *not* the at sign (@), which must be escaped if the code sample uses annotations.

还需要注意，在@throw语句的代码片段中使用的javadoc标签@code。这个标签有两个作用：首先，让这个代码以代码字体的形式展示，并限制在代码片段的HTML标记和嵌套的Javadoc标签的执行；另一个作用是允许代码片段中使用小于符号（<），即使<是HTML元字符。为了在文档注释中包含几行代码，可以在一个HTML标签<pre>中使用Javadoc标签@code。换句话说，就是要在代码例子前面使用<pre>{@code ，后面使用}</pre>。这样就可以在代码中保留换行，也不需要对HTML元字符进行转换，但是@字符还是不行，如果代码中使用了注释就必须进行转换。

> Finally, notice the use of the words “this list” in the doc comment. By convention, the word “this” refers to the object on which a method is invoked when it is used in the doc comment for an instance method.

最后，注意文档注释中的“this list”，根据约定，如果是在实例方法上的文档注释，这个“this”就是指调用方法的对象。

> As mentioned in Item 15, when you design a class for inheritance, you must document its *self-use patterns,* so programmers know the semantics of overriding its methods. These self-use patterns should be documented using the @implSpec tag, added in Java 8. Recall that ordinary doc comments describe the contract between a method and its client; @implSpec comments, by contrast, describe the contract between a method and its subclass, allowing subclasses to rely on implementation behavior if they inherit the method or call it via super. Here's how it looks in practice:

正如Item15里面提到的那样，当你设计一个类用于继承的时候，你必须用文档说明它的自用模式，因此程序员才能知道覆盖的方法的语义。这写自用模式需要用Java8里新增的@implSpec 标签进行文档说明。前面说了，普通的文档注释是描述方法和其客户端之间的约定的，而@implSpec注释是描述方法和子类之间的关系的，如果它继承了这个方法或者通过super调用了这个方法，子类就可以依赖其实现。下面是一个用法范例：

```java
/**
* Returns true if this collection is empty.
*
* @implSpec
* This implementation returns {@code this.size() == 0}. *
* @return true if this collection is empty
*/
   public boolean isEmpty() { ... }

```

> As of Java 9, the Javadoc utility still ignores the @implSpec tag unless you pass the command line switch -tag "implSpec：a：Implementation Requirements:". Hopefully this will be remedied in a subsequent release.

在Java9中，javadoc工具依然忽略了 @implSpec标签，除非你在命令行中使用参数 -tag "implSpec：a：Implementation Requirements:"。希望这个在后面的版本中可以改进。

> Don’t forget that you must take special action to generate documentation that contains HTML metacharacters, such as the less-than sign (<), the greater-than sign (>), and the ampersand (&). The best way to get these characters into documentation is to surround them with the {@literal} tag, which suppress processing of HTML markup and nested Javadoc tags. It is like the {@code} tag, except that it doesn’t render the text in code font. For example, this Javadoc fragment:

不要忘记，对于那些生成的文档中包括HTML元符号的，需要特殊处理。比如小于符号<，大于符号>,还有&符号。在文档中获得这些符号的最佳的方法是使用@literal标签。这个标签也能限制html标记和嵌入javadoc标签的处理，除了不会以代码字体展示文本以外，它的功能和@code标签很像，比如下面这个Javadoc片段：

```java
* A geometric series converges if {@literal |r| < 1}.
```

> generates the documentation: “A geometric series converges if |r| < 1.” The {@literal} tag could have been placed around just the less-than sign rather than the entire inequality with the same resulting documentation, but the doc comment would have been less readable in the source code. This illustrates the general principle that **doc comments should be readable both in the source code and in the generated documentation.** If you can’t achieve both, the readability of the generated documentation trumps that of the source code.

这个代码片段生成的文档是“A geometric series converges if |r| < 1.”。这个{@literal} 标签也可以就用在小于符号周围，而不是整个不等式之前，也可以得到同样的文档。但是这个文档注释的源代码的可读性就不高了。这说明了一条通用的规则：**“文档注释要使得源代码和生成的文档都具有可读性“**。如果两个不可兼得，那么生成的文档的可读性要优先于源代码。

> The first “sentence” of each doc comment (as defined below) becomes the *summary description* of the element to which the comment pertains. For example, the summary description in the doc comment on page 255 is “Returns the element at the specified position in this list.” The summary description must stand on its own to describe the functionality of the element it summarizes. To avoid confusion, **no two members or constructors in a class or interface should have the same summary description.** Pay particular attention to overloadings, for which it is often natural to use the same first sentence (but unacceptable in doc comments).

每一个文档注释中的第一句（如下所示）都应该是注释所在元素的一个总结描述。如，在前面的一个例子中，总结描述就是 “Returns the element at the specified position in this list.” 这个总结描述必须独立地描述其元素的功能。为了避免造成困惑，**一个类或接口中的任何两个成员或者构造器不应该有相同的总结描述。**尤其需要注意重载方法，很容易自然地使用同样的第一句话（但是，在文档注释中，这是无法接受的）。

> Be careful if the intended summary description contains a period, because the period can prematurely terminate the description. For example, a doc comment that begins with the phrase “A college degree, such as B.S., M.S. or Ph.D.” will result in the summary description “A college degree, such as B.S., M.S.” The problem is that the summary description ends at the first period that is followed by a space, tab, or line terminator (or at the first block tag) [Javadoc-ref]. Here, the second period in the abbreviation “M.S.” is followed by a space. The best solution is to surround the offending period and any associated text with an {@literal} tag, so the period is no longer followed by a space in the source code:

当想要的总结描述中包含句号的时候，需要格外小心，因为句号会造成描述提前结束。比如，”A college degree, such as B.S., M.S. or Ph.D.”文档注释，生成的总结描述是“A college degree, such as B.S., M.S.” 。问题是总结描述或在第一个紧跟着空格，tab，或者换行符的句号（或者第一个块标签）处终止[Javadoc-ref]。在这里，第二段缩写“M.S.”后面跟着一个空格。最好的解决方法就是将所有麻烦的句号和相关的文本，使用{@literal}标签包围起来，使得这个句号在源代码中就不在跟着一个空格了，注释如下：

```java
/**
* A college degree, such as B.S., {@literal M.S.} or Ph.D. 
*/
public class Degree { ... }
```

> It is a bit misleading to say that the summary description is the first *sentence* in a doc comment. Convention dictates that it should seldom be a complete sentence. For methods and constructors, the summary description should be a verb phrase (including any object) describing the action performed by the method. For example:
>
> - ArrayList(int initialCapacity)—Constructs an empty list with the specified initial capacity.
> - Collection.size()—Returns the number of elements in this collection.

说文档注释中的第一个句子（sentence）是总结描述，有点误导人。约定指出，总结描述很少是一个句子。对于方法和构造器而言，总结藐视就是一个动词断续（包含任何对象）描述了这个方法执行的动作。比如：

- ArrayList(int initialCapacity)—通过指定的初始化容量来构造一个空的list。
- Collection.size()—返回集合中元素的个数。

> As shown in these examples, use the third person declarative tense (“returns the number”) rather than the second person imperative (“return the number”).

正如前面的例子展示的，使用第三人称时态 (“returns the number”)比使用第二人称时态 (“return the number”)要好一些。

> For classes, interfaces, and fields, the summary description should be a noun phrase describing the thing represented by an instance of the class or interface or by the field itself. For example:
>
> - Instant—An instantaneous point on the time-line.
> - Math.PI—The double value that is closer than any other to pi, the ratio of the circumference of a circle to its diameter.

对于类，接口，和域，总结描述应该是一个名词短语，描述了类和接口的实例或者这个域所表示的东西。比如：

- Instant -- 时间线上的一个瞬时点。
- Math.PI -- 最接近pi的double值，pi是圆周周长和直径的比值。

> In Java 9, a client-side index was added to the HTML generated by Javadoc. This index, which eases the task of navigating large API documentation sets, takes the form of a search box in the upper-right corner of the page. When you type into the box, you get a drop-down menu of matching pages. API elements, such as classes, methods, and fields, are indexed automatically. Occasionally you may wish to index additional terms that are important to your API. The {@index} tag was added for this purpose. Indexing a term that appears in a doc comment is as simple as wrapping it in this tag, as shown in this fragment:

在Java9中，在javadoc生成的HTML中添加了客户端索引。这个索引简化了大型API文档的导航任务，在页面的右上角采用了搜索框的形式。当你往搜索框里输入的时候，你会得到一个下拉的匹配页面。相关的API元素，比如类，方法，域都会自动的索引。有时候，你可能想在重要的API中，添加额外的索引条目。为此，增加了 {@index} 标签，在注释文档中简单地用这个标签包装起来，就可以为该条目添加索引。如下面这个片段所示：

```java
* This method complies with the {@index IEEE 754} standard.
```

> Generics, enums, and annotations require special care in doc comments.**When documenting a generic type or method, be sure to document all type parameters:**

在文档注释中，泛型，枚举和注解都需要特别小心。当给一个泛型类型或者方法写注释文档的时候，也必须要为所有的类型参数写文档说明，如下：

```java
/**
* An object that maps keys to values. A map cannot contain * duplicate keys; each key can map to at most one value.
*
* (Remainder omitted)
*
* @param <K> the type of keys maintained by this map
* @param <V> the type of mapped values
*/
public interface Map<K, V> { ... }
```

> **When documenting an enum type, be sure to document the constants** as well as the type and any public methods. Note that you can put an entire doc comment on one line if it’s short:

**在给枚举类型写文档说明的时候，也要确保给常量、类型、和所有公有方法写文档说明**。注意，如果这个文档注释很短的话，你可以把他们都放到一行上。枚举类型文档化，示例如下：

```java
		/**
    * An instrument section of a symphony orchestra.
    */
   public enum OrchestraSection {
       /** Woodwinds, such as flute, clarinet, and oboe. */
				WOODWIND,
       /** Brass instruments, such as french horn and trumpet. */
				BRASS,
       /** Percussion instruments, such as timpani and cymbals. */
				PERCUSSION,
       /** Stringed instruments, such as violin and cello. */
				STRING;
   }
```

> **When documenting an annotation type, be sure to document any members** as well as the type itself. Document members with noun phrases, as if they were fields. For the summary description of the type, use a verb phrase that says what it means when a program element has an annotation of this type:

**在给注解类型写文档说明的时候，确保给这个类型和每一个成员写文档说明**。如果它们是域的话，就用名词短语进行注释。对于大部分类型的总结描述，都使用动词短语来说明，当一个程序元素拥有一个这个类型的注解的时候，意味着什么。示例如下：

```java
/**
    * Indicates that the annotated method is a test method that
    * must throw the designated exception to pass.
    */
   @Retention(RetentionPolicy.RUNTIME)
   @Target(ElementType.METHOD)
   public @interface ExceptionTest {
       /**
        * The exception that the annotated test method must throw
        * in order to pass. (The test is permitted to throw any
        * subtype of the type described by this class object.)
        */
       Class<? extends Throwable> value();
   }
```

> Package-level doc comments should be placed in a file named package-info.java. In addition to these comments, package-info.java must contain a package declaration and may contain annotations on this declaration. Similarly, if you elect to use the module system (Item 15), module-level comments should be placed in the module-info.java file.

包级的文档说明，应该放在一个名为package- info.java的文件中。除了这些注释以外，package- info.java中还必须包含一个包声明，以及可能在声明上有一些注解。同样地，如果你使用模块系统 (Item 15)，模块级别的注释应该放在module-info.java文件里。

> Two aspects of APIs that are often neglected in documentation are thread- safety and serializability. **Whether or not a class or static method is thread- safe, you should document its thread-safety** level, as described in Item 82. If a class is serializable, you should document its serialized form, as described in Item 87.

在文档中比较容易忽视的API的两个方面是线程安全性和可序列化性。**不管这个类或者静态方法是否是线程安全的，你都应该在文档中说明其线程安全级别**，如Item82所述。如果一个类是可序列画的，你必须要文档说明它的序列化形式，如Item87所述。

> Javadoc has the ability to “inherit” method comments. If an API element does not have a doc comment, Javadoc searches for the most specific applicable doc comment, giving preference to interfaces over superclasses. The details of the search algorithm can be found in *The Javadoc Reference Guide* [Javadoc-ref]. You can also inherit *parts* of doc comments from supertypes using the {@inheritDoc} tag. This means, among other things, that classes can reuse doc comments from interfaces they implement, rather than copying these comments. This facility has the potential to reduce the burden of maintaining multiple sets of nearly identical doc comments, but it is tricky to use and has some limitations. The details are beyond the scope of this book.

javadoc可以继承方法注释。如果一个API元素没有文档注释，javadoc就会为它寻找最适合的文档注释，接口优先于父类。这个搜索算法的细节可以在*The Javadoc Reference Guide* [Javadoc-ref]里面找到。你也可以使用 {@inheritDoc} 标签，来从超类型中继承部分文档注释。这意味着，这些类可以重用他们实现的接口的文档注释，而不是复制这些注释。这个技术可能可以减轻维护几个差不多相同的文档注释的负担。但是使用起来需要一些技巧，而且有一些限制。这些细节超出了本书的范围，就不详细介绍了。

> One caveat should be added concerning documentation comments. While it is necessary to provide documentation comments for all exported API elements, it is not always sufficient. For complex APIs consisting of multiple interrelated classes, it is often necessary to supplement the documentation comments with an external document describing the overall architecture of the API. If such a document exists, the relevant class or package documentation comments should include a link to it.

关于文档注释还有一点需要注意。虽然有必要为所有导出的API元素提供文档注释，但是这还不够。对于一些复杂的包括几个相关的类的API，提供一个额外的文档来描述这个API的总体的结构是很有必要的。如果有这样的文档，那么相关的类或者包文档注释，都应该提供一个指向这个文档的链接。

> Javadoc automatically checks for adherence to many of the recommendations in this item. In Java 7, the command line switch -Xdoclint was required to get this behavior. In Java 8 and 9, checking is enabled by default. IDE plug-ins such as checkstyle go further in checking for adherence to these recommendations [Burn01]. You can also reduce the likelihood of errors in doc comments by running the HTML files generated by Javadoc through an *HTML validity checker*. This will detect many incorrect uses of HTML tags. Several such checkers are available for download, and you can validate HTML on the web using the W3C markup validation service [W3C-validator]. When validating generated HTML, keep in mind that as of Java 9, Javadoc is capable of generating HTML5 as well as HTML 4.01, though it still generates HTML 4.01 by default. Use the -html5 command line switch if you want Javadoc to generate HTML5.

javadoc可以对本节提出的许多的建议进行自动检测。在Java7里，启用命令行参数-Xdoclint就可以获得这个功能。在Java8和java9里，这个功能是默认开启的。一些IDE的插件，比如checkstyle，会进一步根据这些建议完成检测[Burn01]。你也可以通过HTML有效性检查器来运行Javadoc生成的HTML文件，以减少文档注释中出现错误的可能性。这个可以检查出很多的HTML标签的错误使用。有很多这种检查器都可以下载，你也可以在网页上使用W3C markup validation service [W3C-validator]来在线检查HTML。在检查生成的HTML的时候，需要注意的是，在Java9中，javadoc既可以生成HTML5，也可以生成HTML 4.01，默认情况下还是HTML 4.01，可以使用命令行参数-html5来生成HTML5。

> The conventions described in this item cover the basics. Though it is fifteen years old at the time of this writing, the definitive guide to writing doc comments is still *How to Write Doc Comments* [Javadoc-guide].

本节的规则只介绍了最基本的规则，虽然已经过去了15年，最权威的文档注释的指导仍然是*How to Write Doc Comments* [Javadoc-guide]。

> If you adhere to the guidelines in this item, the generated documentation should provide a clear description of your API. The only way to know for sure, however, is to **read the web pages generated by the Javadoc utility.** It is worth doing this for every API that will be used by others. Just as testing a program almost inevitably results in some changes to the code, reading the documentation generally results in at least a few minor changes to the doc comments.

如果你遵守了本节中的指导，那么生成的文档就会对你的API提供一个清晰的描述。但是唯一能确定文档是否清晰的方法，就是**阅读javadoc工具生成的网页**。对于每一个需要被别人使用的API，这样多是很值得的。就像基本所有的测试都会带来代码修改一样，阅读文档，通常也会需要对文档注释做一些修改。

> To summarize, documentation comments are the best, most effective way to document your API. Their use should be considered mandatory for all exported API elements. Adopt a consistent style that adheres to standard conventions. Remember that arbitrary HTML is permissible in documentation comments and that HTML metacharacters must be escaped

总结一下，文档注释是将API文档化的最好、且最有效的方法。对于所有导出的API元素，应该强制使用文档注释。采用一贯的风格来遵守标准的约定。记住，在文档注释中可以出现任何的HTML标签，但是那些HTML元字符必须进行转义。