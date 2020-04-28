### Item37 使用EnumMap代替序数索引

> Occasionally you may see code that uses the ordinal method (Item 35) to index into an array or list. For example, consider this simplistic class meant to represent a plant:

有时候，你可能会看都使用ordinal方法（Item35）来索引数组或者列表的代码。比如，下面这个表示植物的简单的类：

```java
class Plant {
       enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }
       
       final String name;
       final LifeCycle lifeCycle;
       
       Plant(String name, LifeCycle lifeCycle) {
           this.name = name;
           this.lifeCycle = lifeCycle;
       }
       @Override public String toString() {
           return name;
       } 
}
```

> Now suppose you have an array of plants representing a garden, and you want to list these plants organized by life cycle (annual, perennial, or biennial). To do this, you construct three sets, one for each life cycle, and iterate through the garden, placing each plant in the appropriate set. Some programmers would do this by putting the sets into an array indexed by the life cycle’s ordinal:

现在，假设你有一个植物数组来表示一个花园，然后你想按照其生命周期类型（一年生、多年生、两年生）来进行组织后，再将这些植物打印出来。为了达到这个效果，你需要构造三个集合，每一个生命周期类型一个集合，然后遍历花园，把每个植物放在对应的集合中。有一些程序员，可能会将这几个集合放在一个数组中，根据LifeCycle的ordinal方法的结果来进行索引。代码如下：

```java
// Using ordinal() to index into an array - DON'T DO THIS!
Set<Plant>[] plantsByLifeCycle = (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];
for (int i = 0; i < plantsByLifeCycle.length; i++)
    plantsByLifeCycle[i] = new HashSet<>();
for (Plant p : garden) 
    plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);
// Print the results
for (int i = 0; i < plantsByLifeCycle.length; i++) {
    System.out.printf("%s: %s%n", Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
}
```

> This technique works, but it is fraught with problems. Because arrays are not compatible with generics (Item 28), the program requires an unchecked cast and will not compile cleanly. Because the array does not know what its index represents, you have to label the output manually. But the most serious problem with this technique is that when you access an array that is indexed by an enum’s ordinal, it is your responsibility to use the correct int value; ints do not provide the type safety of enums. If you use the wrong value, the program will silently do the wrong thing or—if you’re lucky—throw an ArrayIndexOutOfBoundsException.

这个方法可以工作，但是又很多的问题。因为数组不能和泛型兼容（Item28），这个程序就需要一个非受检的转换，因此编译会不干净。又因为数组不知道它的索引表示的是什么，所以你必须要手动标记这些输出。但是这个方法最糟糕的问题还是，当你需要访问一个被枚举的ordinal索引的数组时，使用正确的int值是你的责任，int值不能提供enum的类型安全。如果你使用了错误的值，这个程序可能会默不作声地做着错误的事情，如果幸运的话，可能会掏出ArrayIndexOutOfBoundsException。

> There is a much better way to achieve the same effect. The array is effectively serving as a map from the enum to a value, so you might as well use a Map. More specifically, there is a very fast Map implementation designed for use with enum keys, known as java.util.EnumMap. Here is how the program looks when it is rewritten to use EnumMap:

有一个更好的方法，可以达到同样的效果。这个数组实际上充当着从枚举到值的映射，所以你也可以使用map。更特别地是，有一个专门为使用枚举作为键而设计的快速的map实现，是java.util.EnumMap。下面是使用EnumMap进行重新的程序代码：

```java
// Using an EnumMap to associate data with an enum
   Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle =
       new EnumMap<>(Plant.LifeCycle.class);
   for (Plant.LifeCycle lc : Plant.LifeCycle.values())
       plantsByLifeCycle.put(lc, new HashSet<>());
   for (Plant p : garden)
       plantsByLifeCycle.get(p.lifeCycle).add(p);
   System.out.println(plantsByLifeCycle);
```

> This program is shorter, clearer, safer, and comparable in speed to the original version. There is no unsafe cast; no need to label the output manually because the map keys are enums that know how to translate themselves to printable strings; and no possibility for error in computing array indices. The reason that EnumMap is comparable in speed to an ordinal-indexed array is that EnumMap uses such an array internally, but it hides this implementation detail from the programmer, combining the richness and type safety of a Map with the speed of an array. Note that the EnumMap constructor takes the Class object of the key type: this is a *bounded type token*, which provides runtime generic type information (Item 33).

这个程序更简短，速度上也可以和之前的版本媲美。也没有不安全的转化，也不需要再输出的时候手动打标签，因为map的key就是枚举，它知道该怎么把自己转换成可打印的字符串，也不可能在计算数组索引时出现error。EnumMap的速度可以和“序数索引的数组”相媲美的原因在于，EnumMap内部也使用了类似的数组，但是它对程序员隐藏了这些实现细节。集map的丰富功能、类型安全 和 数组的速度于一身。需要注意的是，EnumMap凑早起需要一个其键类型的Class对象作为参数：这是一个无限制的类型令牌，提供了运行时的泛型类型信息（Item33）。

> The previous program can be further shortened by using a stream (Item 45) to manage the map. Here is the simplest stream-based code that largely duplicates the behavior of the previous example:

上面的程序，使用Stream来管理map的话，还可以变得简短得多。下面是基于Stream的最简单的代码，基本复制了前面的例子的大部分行为：

```java
// Naive stream-based approach - unlikely to produce an EnumMap! System.out.println(Arrays.stream(garden).collect(groupingBy(p -> p.lifeCycle)));
```

> The problem with this code is that it chooses its own map implementation, and in practice it won’t be an EnumMap, so it won’t match the space and time performance of the version with the explicit EnumMap. To rectify this problem, use the three-parameter form of Collectors.groupingBy, which allows the caller to specify the map implementation using the mapFactory parameter:

这个代码有一个问题就是他无法选择自己的map实现，并且实际上，它也不会使用EnumMap。所以它的时间和空间性能就不能和由明确的EnumMap的类型媲美了。为了解决这个问题，我们可以使用Collectors.groupingBy有三个参数的版本，这个版本允许调用者通过mapFactory参数来明确指定map实现。代码如下：

```java
// Using a stream and an EnumMap to associate data with an enum
   System.out.println(Arrays.stream(garden)
           .collect(groupingBy(p -> p.lifeCycle,
					 		() -> new EnumMap<>(LifeCycle.class), toSet())));
```

> This optimization would not be worth doing in a toy program like this one but could be critical in a program that made heavy use of the map.

在这种玩具程序中，不值得使用这种优化，但是在大量使用map的程序中，就很有必要了。

> The behavior of the stream-based versions differs slightly from that of the EmumMap version. The EnumMap version always makes a nested map for each plant lifecycle, while the stream-based versions only make a nested map if the garden contains one or more plants with that lifecycle. So, for example, if the garden contains annuals and perennials but no biennials, the size of plantsByLifeCycle will be three in the EnumMap version and two in both of the stream-based versions.

这个基于Stream的版本和EnumMap的版本的行为有一点点不同。EnumMap版本会为每一种植物的生命周期生成一个嵌套映射，而Stream版本只会当garden数组中包含一个或多个该生命周期的植物是，才生成一个嵌套映射。举个例子，如果这个garden数组中只包含 一年生 和 多年生的植物，没有两年生的植物，那么EnumMap中的plantsByLifeCycle的大小将会是3，而在两个Stream版本中，map的大小都是2。

> You may see an array of arrays indexed (twice!) by ordinals used to represent a mapping from two enum values. For example, this program uses such an array to map two phases to a phase transition (liquid to solid is freezing, liquid to gas is boiling, and so forth):

你可能还看到使用（两次）ordinal进行索引的数组的数组，用来表示两个枚举值到其他值的映射。举个例子，下面这段程序，使用一个数组来表示两个状态到一个状态转化的映射（液体到固体是凝固，液体到气体是沸腾，等等）：

```java
// Using ordinal() to index array of arrays - DON'T DO THIS!
   public enum Phase {
       SOLID, LIQUID, GAS;
       public enum Transition {
           MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;
           // Rows indexed by from-ordinal, cols by to-ordinal
           private static final Transition[][] TRANSITIONS = {
               { null,    MELT,     SUBLIME },
               { FREEZE,  null,     BOIL    },
               { DEPOSIT, CONDENSE, null    }
           };
				 // Returns the phase transition from one phase to another 
         public static Transition from(Phase from, Phase to) { 
           		return TRANSITIONS[from.ordinal()][to.ordinal()];
         }
      }
	}
```

> This program works and may even appear elegant, but appearances can be deceiving. Like the simpler garden example shown earlier, the compiler has no way of knowing the relationship between ordinals and array indices. If you make a mistake in the transition table or forget to update it when you modify the Phase or Phase.Transition enum type, your program will fail at runtime. The failure may be an ArrayIndexOutOfBoundsException, a NullPointerException, or (worse) silent erroneous behavior. And the size of the table is quadratic in the number of phases, even if the number of non-null entries is smaller.

这个程序可以工作，甚至看起来还有点优雅，但不要被它的外表欺骗了。和前面的花园的简单例子类似。编译器无法知道数组索引和ordinal（序数）之间的关系。如果你在这个转换表中出现了错误，或者当你修改Phase或者Phase.Transition的枚举类型的时候，忘了更新转换表，程序在运行时就会出错。这个错误可能是ArrayIndexOutOfBoundsException，NullPointerException，或者，更糟糕的，就没有任何提示的错误的行为。并且这个列表的带下是phases数量的平方倍，即使非空的实例的数量要少一些。

> Again, you can do much better with EnumMap. Because each phase transition is indexed by a *pair* of phase enums, you are best off representing the relationship as a map from one enum (the “from” phase) to a map from the second enum (the “to” phase) to the result (the phase transition). The two phases associated with a phase transition are best captured by associating them with the phase transition enum, which can then be used to initialize the nested EnumMap:

同样地，还是可以使用EnumMap来优化。由于每一个状态转化是和一对状态枚举相关联的，因此，你最好是使用这样一个map来表示这个关系：其键为一个枚举类型（表示from状态），其值为一个map（其键为第二个枚举（表示to状态），其值为结果（也就是状态转换））。两个状态关联一个状态转换，最好通过通过状态转换枚举上关联的数据来获取，以用来初始化嵌套的EnumMap。代码如下（*语言是苍白的，show me the code*）：

```java
// Using a nested EnumMap to associate data with enum pairs
   public enum Phase {
      SOLID, LIQUID, GAS;
      public enum Transition {
         MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
         BOIL(LIQUID, GAS),   CONDENSE(GAS, LIQUID),
         SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);
         private final Phase from;
         private final Phase to;
         Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
         }
         // Initialize the phase transition map
         private static final Map<Phase, Map<Phase, Transition>>
           m = Stream.of(values()).collect(groupingBy(t -> t.from,
            () -> new EnumMap<>(Phase.class),
            toMap(t -> t.to, t -> t,
               (x, y) -> y, () -> new EnumMap<>(Phase.class))));
         public static Transition from(Phase from, Phase to) {
            return m.get(from).get(to);
         } 
      }
}
```

> The code to initialize the phase transition map is a bit complicated. The type of the map is Map<Phase, Map<Phase, Transition>>, which means “map from (source) phase to map from (destination) phase to transition.” This map-of-maps is initialized using a cascaded sequence of two collectors. The first collector groups the transitions by source phase, and the second creates an EnumMap with mappings from destination phase to transition. The merge function in the second collector ((x, y) -> y)) is unused; it is required only because we need to specify a map factory in order to get an EnumMap, and Collectors provides telescoping factories. The previous edition of this book used explicit iteration to initialize the phase transition map. The code was more verbose but arguably easier to understand.

初始化状态转换map的代码有点复杂。这个map的类型是Map<Phase, Map<Phase, Transition>>，表示从（from）源状态 到  从（to）目标状态到转换的映射 的映射。这个map的map是通过两个集合的级联顺序来初始化的。第一个集合按照源状态（from）对transitions进行分组，第二个集合使用目标状态（to)到transitions的映射来创建EnumMap。第二个集合中的合并函数 ((x, y) -> y)) 并没有用到，它只有当需要为了获取EnumMap而指定map工厂的时候才会被用到，同时Collectors提供了重叠工厂（*？？？*）。在本书的第二版中使用的是显式迭代来初始化状态转移map，那个代码虽然要长一些，但是确实要好理解一些。

> Now suppose you want to add a new phase to the system: *plasma*, or ionized gas. There are only two transitions associated with this phase: *ionization*, which takes a gas to a plasma; and *deionization*, which takes a plasma to a gas. To update the array-based program, you would have to add one new constant to Phase and two to Phase.Transition, and replace the original nine-element array of arrays with a new sixteen-element version. If you add too many or too few elements to the array or place an element out of order, you are out of luck: the program will compile, but it will fail at runtime. To update the EnumMap-based version, all you have to do is add PLASMA to the list of phases, and IONIZE(GAS, PLASMA) and DEIONIZE(PLASMA, GAS) to the list of phase transitions:

假如你现在想添加一个新的状态：plasma（离子），或者电离气体。和这个状态相关的转换只有两个：电离化（ionization），从气体到离子状态；消电离化（deionization），从离子状态带气体状态。要更新那个基于数组的程序的话，你需要给Phase新增一个常量，给Phase.Transition新增两个常量，然后用一个新的16个元素的数组替换之前的9个元素的数组。如果你不小心在数组中添加的元素多了，或者少了，或者有个元素顺序放错了。这个程序可以编译，按时在运行时会失败。而要更新基于EnumMap的版本的话，你需要做的就是在Phase列表中添加一个PLASMA，然后在Phase.Transition列表中添加IONIZE(GAS, PLASMA) 和DEIONIZE(PLASMA, GAS) 就可以了。代码如下：

```java
// Adding a new phase using the nested EnumMap implementation
public enum Phase {
	SOLID, LIQUID, GAS, PLASMA;
	public enum Transition {
		MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID), 
    BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID), 			
    SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID), 
    IONIZE(GAS, PLASMA), DEIONIZE(PLASMA, GAS); 
    ...  // Remainder unchanged
  }
}
```

> The program takes care of everything else and leaves you virtually no opportunity for error. Internally, the map of maps is implemented with an array of arrays, so you pay little in space or time cost for the added clarity, safety, and ease of maintenance.

这个程序会为处理好其他的事情，然后你基本就没有机会犯错。map的map本质上就是通过数组的数来实现的，因此，在提升了简洁性，安全性，和易维护性的同时，也没有付出任何空间和时间的代价。

> In the interest of brevity, the above examples use null to indicate the absence of a state change (wherein to and from are identical). This is not good practice and is likely to result in a NullPointerException at runtime. Designing a clean, elegant solution to this problem is surprisingly tricky, and the resulting programs are sufficiently long that they would detract from the primary material in this item.

为了简洁起见，上面的程序都是使用null来表示没有的状态转变（这里的to和from是一致的）。这样做是不好的，可能会导致运行时出现NullPointerException。但是要设计一个整洁、优雅的方案来解决这个问题，非常地复杂。得到的代码也会有点长，会妨碍表达本条的主要精神。

> In summary, **it is rarely appropriate to use ordinals to index into arrays: use** **EnumMap** **instead.** If the relationship you are representing is multidimensional, use EnumMap<..., EnumMap<...>>. This is a special case of the general principle that application programmers should rarely, if ever, use Enum.ordinal (Item 35).

总结一下，**最好不要使用序数（ordinal）来索引数组，最好使用EnumMap。**当你要表达的关系是多维的时候，可以使用EnumMap<..., EnumMap<...>>结构。应用程序的程序员一般都不需要使用Enum.ordinal，只有在Item35中介绍的特殊情况下才会使用。