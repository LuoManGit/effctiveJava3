### Item35 用实例代替序数

> Many enums are naturally associated with a single int value. All enums have an ordinal method, which returns the numerical position of each enum constant in its type. You may be tempted to derive an associated int value from the ordinal:

许多枚举天生就和一个int值关联。所有的枚举都有一个ordinal方法，可以返回每个枚举实例在其类型中的数字位置。你可能会通过下面这样的代码来从ordinal里获取与常量关联的int值：

```java
// Abuse of ordinal to derive an associated value - DON'T DO THIS
   public enum Ensemble {
       SOLO,   DUET,   TRIO, QUARTET, QUINTET,
       SEXTET, SEPTET, OCTET, NONET,  DECTET;
			 public int numberOfMusicians() { return ordinal() + 1; } 
   }
```

> While this enum works, it is a maintenance nightmare. If the constants are reordered, the numberOfMusicians method will break. If you want to add a second enum constant associated with an int value that you’ve already used, you’re out of luck. For example, it might be nice to add a constant for *double quartet*, which, like an octet, consists of eight musicians, but there is no way to do it.

虽然这个枚举可以工作，但是维护起来就像一场噩梦一样。如果这些常量被重新排序了，这个numberOfMusicians方法就被破坏了。如果你想添加一个枚举常量，然后和一个已经用过的int值关联，就做不到了。比如，要在其中添加一个常量，双四重奏（像八重奏一样包含8个音乐家），但是却没有办法做到。

> Also, you can’t add a constant for an int value without adding constants for all intervening int values. For example, suppose you want to add a constant representing a *triple quartet*, which consists of twelve musicians. There is no standard term for an ensemble consisting of eleven musicians, so you are forced to add a dummy constant for the unused int value (11). At best, this is ugly. If many int values are unused, it’s impractical.

而且，如果没有为某个int值以下的所有int值设置常量，就不能添加该int值的常量。比如，你想添加一个常量值表示“三 四重奏”（包换12个音乐家）。但是对于包含11个音乐家的组合，没有一个标准的术语，但是你还是必须要为没用的int值11添加一个虚拟的常量。最好的情况就是代码有点丑，如果有很多的int中都没有用，那就是不切实际的。

> Luckily, there is a simple solution to these problems. **Never derive a value associated with an enum from its ordinal; store it in an instance field instead:**

幸运的是，这些问题有一个简单的解决方法。**不要从枚举的序数（ordinal）中获取关联的值；应该把它保存在一个实例域中**。如下：

```java
public enum Ensemble {
       SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
       SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8),
       NONET(9), DECTET(10), TRIPLE_QUARTET(12);
  
			 private final int numberOfMusicians;
			 Ensemble(int size) { this.numberOfMusicians = size; } 
  
  		 public int numberOfMusicians(){
         return numberOfMusicians; 
       }
}
```

> The Enum specification has this to say about ordinal: “Most programmers will have no use for this method. It is designed for use by general-purpose enumbased data structures such as EnumSet and EnumMap.” Unless you are writing code with this character, you are best off avoiding the ordinal method entirely.

Enum的规范里关于ordinal是这么说的：“大部分的程序员都不需要使用这个方法，这个方式是为基于枚举的通用数据结构比如EnumSet和EnumMap设计的。”除非你需要写这种类型的代码，否则你最好的完全避免使用ordinal方法。

