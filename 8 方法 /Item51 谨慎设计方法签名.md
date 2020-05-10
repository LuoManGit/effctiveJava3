### Item51 谨慎设计方法签名

> This item is a grab bag of API design hints that don’t quite deserve items of their own. Taken together, they’ll help make your API easier to learn and use and less prone to errors.

本节是API设计技巧的一个总结，他们都还不足以单独讲一节。把它们放到一起，可以帮助你把API设计得容易看懂，使用时不容易犯错。

> **Choose method names carefully.** Names should always obey the standard naming conventions (Item 68). Your primary goal should be to choose names that are understandable and consistent with other names in the same package. Your secondary goal should be to choose names consistent with the broader consensus, where it exists. Avoid long method names. When in doubt, look to the Java library APIs for guidance. While there are plenty of inconsistencies—inevitable, given the size and scope of these libraries—there is also a fair amount of consensus.

**认真的选择方法的名称**。名字始终应该遵守标准命名习惯（Item68）。选择名字的主要目标应该是好理解且和同一个包里的其他名字保持一致。选择的次要目标应该是和大众习惯（如果有的话）保持一致。毫无疑问，应该以Java类库的API为指导，虽然这里面很多的前后不一致，但由于这个类库的规模和范围很大，因此这个问题难以避免，但是其中还是有很多得到了大家的认可。

> **Don’t go overboard in providing convenience methods.** Every method should “pull its weight.” Too many methods make a class difficult to learn, use, document, test, and maintain. This is doubly true for interfaces, where too many methods complicate life for implementors as well as users. For each action supported by your class or interface, provide a fully functional method. Consider providing a “shorthand” only if it will be used often. **When in doubt, leave it out.**

**不要过于追求提供便利的方法**。每一个方法都应该尽其所能。一个类里有太多方法，会使得这个类很难学习，使用，文档化，测试和维护。毫无疑问，对于接口也是这样，太多方法会使得接口的使用和实现都变得复杂。对于方法和类支持的每一个方法，都提供功能齐全的方法，如果这个方法使用非常频繁的话，就为之提供一个标准的实现。**如果不能确定，就不要提供标准的实现**。

> **Avoid long parameter lists.** Aim for four parameters or fewer. Most programmers can’t remember longer parameter lists. If many of your methods exceed this limit, your API won’t be usable without constant reference to its documentation. Modern IDEs help, but you are still much better off with short parameter lists. **Long sequences of identically typed parameters are especially harmful.** Not only won’t users be able to remember the order of the parameters, but when they transpose parameters accidentally, their programs will still compile and run. They just won’t do what their authors intended.

**避免参数列表过长**。最多4个参数。大部分程序员都记不住更长的参数列表了。如果你的方法超过了这个限制，那么程序员必须不同的看文档才能使用你的API。虽然现代IDE可以对此有所帮助，但是你还是最好提供短一点的参数列表。**多个相同的类型参数，尤其危险**。不仅仅是用户记不住这些参数的顺序，而且，当他们不小心弄错了顺序，这个程序还是可以编译和运行，只是得到的结果不是作者所期望的。

> There are three techniques for shortening overly long parameter lists. One is to break the method up into multiple methods, each of which requires only a subset of the parameters. If done carelessly, this can lead to too many methods, but it can also help *reduce* the method count by increasing orthogonality. For example, consider the java.util.List interface. It does not provide methods to find the first or last index of an element in a sublist, both of which would require three parameters. Instead it provides the subList method, which takes two parameters and returns a *view* of a sublist. This method can be combined with the indexOf or lastIndexOf method, each of which has a single parameter, to yield the desired functionality. Moreover, the subList method can be combined with *any* method that operates on a List instance to perform arbitrary computations on sublists. The resulting API has a very high power-to-weight ratio.

有三个技术可以缩短参数列表的长度。第一种技术是把方法拆分为几个方法，每一个方法的参数列表都是原参数列表的自己。如果不小心，可能会导致特别多的方法，但是它确实可以通过增加方法的正交性来减少方法的数目。比如，java.util.List接口，它确实没有提供方法，来寻找子集内某个元素第一次或者最后一次出现的索引位置，这两种方法都需要三个参数。对应地，它提供了一个subList方法，有两个参数，返回一个子列表视图。这个方法可以和indexOf和lastIndexOf方法（这两种方法都只有一个参数）一起用，就可以得到期望的功能。此外，subLast方法还可以和任何”对List实例上进行任意操作“的方法合作。这样得到的API就有很高的功能-权重比。

> A second technique for shortening long parameter lists is to create *helper classes* to hold groups of parameters. Typically these helper classes are static member classes (Item 24). This technique is recommended if a frequently occurring sequence of parameters is seen to represent some distinct entity. For example, suppose you are writing a class representing a card game, and you find yourself constantly passing a sequence of two parameters representing a card’s rank and its suit. Your API, as well as the internals of your class, would probably benefit if you added a helper class to represent a card and replaced every occurrence of the parameter sequence with a single parameter of the helper class.

第二种缩短列表长度的技术，是创建一个辅助类来持有一组参数。通常情况下，这些辅助类是静态成员类（Item24）。当一组可以被看做是一个实体的参数频繁出现的时候，可以使用这种技术。比如，你现在写一个类来表示卡牌游戏，你发现你总是会传两个参数来表示牌的点数和花色。如果你添加一个辅助类来表示一张牌，把所有出现这两个参数的地方都使用单个的辅助类参数来替换，你的API，以及类的内部表示都将从中受益。

> A third technique that combines aspects of the first two is to adapt the Builder pattern (Item 2) from object construction to method invocation. If you have a method with many parameters, especially if some of them are optional, it can be beneficial to define an object that represents all of the parameters and to allow the client to make multiple “setter” calls on this object, each of which sets a single parameter or a small, related group. Once the desired parameters have been set, the client invokes the object’s “execute” method, which does any final validity checks on the parameters and performs the actual computation.

第三种技术结合了前面两种技术的特征，从对象构造到方法调用都采用Builder模式（Item2）。如果你有一个方法有很多参数，尤其其中还有一些参数是可选择的时候，最好定义一个对象来表示这些参数，并允许客户端调用多次该对象的setter方法，每次设计一个参数或者一组较少的相关的参数。一旦这些需要的参数都设置玩了，客户端调用这个对象的execute方法，进行最后的参数有效性检验，并执行真正的计算。

> **For parameter types, favor interfaces over classes** (Item 64). If there is an appropriate interface to define a parameter, use it in favor of a class that implements the interface. For example, there is no reason to ever write a method that takes HashMap on input—use Map instead. This lets you pass in a HashMap, a TreeMap, a ConcurrentHashMap, a submap of a TreeMap, or any Map implementation yet to be written. By using a class instead of an interface, you restrict your client to a particular implementation and force an unnecessary and potentially expensive copy operation if the input data happens to exist in some other form.

**对于参数类型，优先选择接口，而不是类（Item64)**。如果有一个合适的接口来定义某一个参数，就使用这个接口，而不是使用实现这个接口的类。比如，没有任何理由写一个方法使用HashMap类作为输入，应该使用Map。使用Map的话，就可以传入一个HashMap，TreeMap，ConcurentHashMap，TreeMap的子map，或者其他将来要实现的Map类型。如果使用类来代替接口的话，就限制了客户端只能传入特定的实现类型，当输入参数不是这个类型的时候，还必须要进行没有必要，且代价非常高的复制计算来做类型转换。

> **Prefer two-element enum types to** **boolean** **parameters,** unless the meaning of the boolean is clear from the method name. Enums make your code easier to read and to write. Also, they make it easy to add more options later. For example, you might have a Thermometer type with a static factory that takes this enum:

**对于boolean类型的参数，优先使用两个元素的枚举类型**，除非这个boolean的含义从方法名中可以清晰地得到。枚举类型可以使你的代码更容易阅读以及编写。此外，后面添加更多的选择也很方便。比如，你可能有一个这样的Thermometer(温度计）类型，有一个静态工厂方法需要下面这样的枚举作为参数类型：

```java
public enum TemperatureScale { FAHRENHEIT, CELSIUS } //FAHRENHEIT——华氏 ，CELSIUS——摄氏度
```

> Not only does Thermometer.newInstance(TemperatureScale.CELSIUS) make a lot more sense than Thermometer.newInstance(true), but you can add KELVIN to TemperatureScale in a future release without having to add a new static factory to Thermometer. Also, you can refactor temperature-scale dependencies into methods on the enum constants (Item 34). For example, each scale constant could have a method that took a double value and converted it to Celsius.

不仅仅Thermometer.newInstance(TemperatureScale.CELSIUS) 方法要比Thermometer.newInstance(true)有意义，而且你还可以在后面的版本中添加一个KELVIN（开尔文）到TemperatureScale里，不需要给Thermometer增加一个新的静态工厂方法。同样地，你还可以基于枚举常量重构温度维度的方法（Item34）。比如，每一个维度常量都可以有一个方法，以double为参数，把单位转化为Celsius。