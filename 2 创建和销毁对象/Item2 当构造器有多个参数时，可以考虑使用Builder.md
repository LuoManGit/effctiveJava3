### Item2 当构造器有多个参数时，可以考虑使用Builder

> Static factories and constructors share a limitation: they do not scale well to large numbers of optional parameters. Consider the case of a class representing the Nutrition Facts label that appears on packaged foods. These labels have a few required fields—serving size, servings per container, and calories per serving—and more than twenty optional fields—total fat, saturated fat, trans fat, cholesterol, sodium, and so on. Most products have nonzero values for only a few of these optional fields.

静态工厂方法和构造器有一个缺点：他们都不能很好的扩展到大量的可选参数。比如，用一个类来表示食品外面的营养成分标签，这些标签里有部分域是必须有的：比如每份的大小、每罐的份数、每份的卡路里含量。还有20多种的可选域：总脂肪含量、饱和脂肪含量、胆固醇含量、钠含量等等。对于这些可选参数，大部分食品只有几项是非零值。

> What sort of constructors or static factories should you write for such a class? Traditionally, programmers have used the telescoping constructor pattern, in which you provide a constructor with only the required parameters, another with a single optional parameter, a third with two optional parameters, and so on, culminating in a constructor with all the optional parameters. Here’s how it looks in practice. For brevity’s sake, only four optional fields are shown:

针对这样的类，我们应该编写什么样的构造器或者静态工厂方法呢？一般情况下，程序员会采用伸缩构造器模式，在这种模式下，将提供一个只包含必要参数的构造器，然后提供一个包含必要参数和一个可选参数的构造器，然后提供一个包含必要参数喝两个可选参数的构造器，以此类推，最后一个构造器包含了所有的必要参数和可选参数。下面有一个例子，为了简单起见，他只显示4个可选的参数。

```java
// Telescoping constructor pattern - does not scale well!
// 收缩构造器模式 - 不能很好的扩展
public class NutritionFacts {
    private final int servingSize; // (mL) required 
    private final int servings;    // (per container) required
    private final int calories;    // (per serving) optional    
    private final int fat;         // (g/serving) optional
    private final int sodium;      // (mg/serving) optional
    private final int carbohydrate; // (g/serving) optional
    public NutritionFacts(int servingSize, int servings) { 
        this(servingSize, servings, 0);
    }
    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0); 
    }
    public NutritionFacts(int servingSize, int servings, int calories, int fat) {
        this(servingSize, servings, calories, fat, 0); 
    }
    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0); 
    }
    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, 
        int carbohydrate) {
        this.servingSize = servingSize; this.servings = servings;
        this.calories = calories
        this.fat = fat
        this.sodium = sodium
        this.carbohydrate = carbohydrate;
    } 
}
```

> When you want to create an instance, you use the constructor with the shortest parameter list containing all the parameters you want to set:

当你想创建一个实例时，你就使用哪个包含了你想设置的所有参数的且参数列表最短的构造器。比如：

```java
NutritionFacts cocaCola = new NutritionFacts(240,8,100,0,35,27)
```



> Typically this constructor invocation will require many parameters that you don’t want to set, but you’re forced to pass a value for them anyway. In this case, we passed a value of 0 for fat. With “only” six parameters this may not seem so bad, but it quickly gets out of hand as the number of parameters increases
>
> In short, **the telescoping constructor pattern works, but it is hard to write client code when there are many parameters, and harder still to read it.** The reader is left wondering what all those values mean and must carefully count parameters to find out. Long sequences of identically typed parameters can cause subtle bugs. If the client accidentally reverses two such parameters, the compiler won’t complain, but the program will misbehave at runtime (Item 51).

通常这个构造器还包括需要一些你本来不想设置的参数，但是又不得不给他传一个值。在上述的例子中，我们给fat参数传了一个0、现在只有6个人参数，因此看起来还不是那么的糟糕，但是随着参数的增加，情况就会变得不可控制。

**简而言之，伸缩构造器模式是可行的，但是当有很多参数的时候，客户端代码就会变得很难写，也很难读**。阅读的人要是想弄明白这些参数都是什么意思，就必须要小心翼翼地去数着这个参数。一长串的相同类型的参数还会导致一些难以察觉的bug，比如，客户端将两个相同类型的参数写烦了，编译器也不会报错，但在程序就不会按照我们预想的运行了。

> A second alternative when you’re faced with many optional parameters in a constructor is the JavaBeans pattern, in which you call a parameterless constructor to create the object and then call setter methods to set each required parameter and each optional parameter of interest:

对于这种多参数的构造器问题，我们还可以使用JavaBeans模式。在这种模式里，我们调用午餐构造器去创建对象，然后调用setter方法去设置必要的参数和感兴趣的可选参数。如下。

```java
// JavaBeans Pattern - allows inconsistency, mandates mutability
// JavaBeans模式 - 允许不一致，要求可变
public class NutritionFacts {
    // Parameters initialized to default values (if any)
    private int servingSize = -1; // Required; no default value 
    private int servings = -1; // Required; no default value
    private int calories = 0;
    private int fat = 0;
    private int sodium = 0;
    private int carbohydrate = 0;
    public NutritionFacts() { }
    // Setters
    public void setServingSize(int val) { 
        servingSize = val; 
    } 
    public void setServings(int val) { 
        servings = val; 
    }
    public void setCalories(int val) {
        calories = val;
    }
    public void setFat(int val) {
        fat = val;
    }
    public void setSodium(int val) {
        sodium = val;
    }
    public void setCarbohydrate(int val) { 
        carbohydrate = val; 
    }
}
```

> This pattern has none of the disadvantages of the telescoping constructor pattern. It is easy, if a bit wordy, to create instances, and easy to read the resulting code:

这种模式克服了伸缩构造器模式的所有缺点。也就是说，JavaBeans模式创建实例简单，且阅读方便。创建实例代码如下:

```java
NutritionFacts cocaCola = new NutritionFacts(); 
cocaCola.setServingSize(240); 
cocaCola.setServings(8); 
cocaCola.setCalories(100); 
cocaCola.setSodium(35); 
cocaCola.setCarbohydrate(27);
```

> Unfortunately, the JavaBeans pattern has serious disadvantages of its own. Because construction is split across multiple calls, **a JavaBean may be in an inconsistent state partway through its construction.**The class does not have the option of enforcing consistency merely by checking the validity of the constructor parameters. Attempting to use an object when it’s in an inconsistent state may cause failures that are far removed from the code containing the bug and hence difficult to debug. A related disadvantage is that **the JavaBeans pattern precludes the possibility of making a class immutable**(Item 17) and requires added effort on the part of the programmer to ensure thread safety.
>
> It is possible to reduce these disadvantages by manually “freezing” the object when its construction is complete and not allowing it to be used until frozen, but this variant is unwieldy and rarely used in practice. Moreover, it can cause errors at runtime because the compiler cannot ensure that the programmer calls the freeze method on an object before using it.

不幸的是，JavaBeans模式也有一些严重的缺陷。因为对象的构造过程本分在了几个方法调用里，**因此一个JavaBean在创建过程中，可能处于不一致状态。** 类无法仅仅通过检测构造器参数的有效性来保证一致性。当我们企图去使用一个处于不一致状态的对象的时候会出错，而且这种错误比代码中包含bug更难察觉和修改。和这个相关的另一个缺点是**JavaBeans模式不能创建一个不可变类**（Item17），也就需要程序员付出很大的努力去保证线程安全。

为了解决不一致的问题，可以在对象构造完成之后手动将对象冻结，并且只允许使用冻结后的对象。但是这种方式很不灵活，在实际中，也很少使用。除此之外，这种方式在运行时也很容易出现问题，因为编译器无法保证程序员在使用一个对象之前一定会调用冻结方法。

> Luckily, there is a third alternative that combines the safety of the telescoping constructor pattern with the readability of the JavaBeans pattern. It is a form of the _Builder _pattern [Gamma95]. 
>
> Instead of making the desired object directly, the client calls a constructor (or static factory) with all of the required parameters and gets a builder object. Then the client calls setter-like methods on the builder object to set each optional parameter of interest. Finally, the client calls a parameterless build method to generate the object, which is typically immutable. The builder is typically a static member class (Item 24) of the class it builds. Here’s how it looks in practice:

幸运的是，还有第三种选择。它是Builder模式的一种形式，集伸缩构造器模式的安全性和JavaBeans模式的可阅读性于一身。在Builder模式下，客户端并不直接创建对象，而是调用包含所有必要参数的构造器（或者静态工厂方法）获得一个builder对象，然后调用builder对象的类似setter方法的方法去设置感兴趣的可选择参数，最后，客户端调用一个无参的build方法生成目标对象，这个对象通常是不可变的。通常情况下，这个Builder是它构建的目标类里的一个静态成员类。如下：

```java
// Builder Pattern
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // Required parameters
        private final int servingSize;
        private final int servings;
        // Optional parameters - initialized to default values
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;
        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        } 
        public Builder calories(int val){ 
            calories = val; return this; 
        }
        public Builder fat(int val){ 
            fat = val; return this; 
        }
        public Builder sodium(int val){ 
            sodium = val; return this; 
        }
        public Builder carbohydrate(int val){ 
            carbohydrate = val; return this; 
        }
        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    } 
    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}
```

> The *NutritionFacts* class is immutable, and all parameter default values are in one place. The builder’s setter methods return the builder itself so that invocations can be chained, resulting in a fluent API. Here’s how the client code looks:

值得注意的是，NutritionFacts类是不可变的额，所有的参数默认值都放在一个地方。builder的setter方法返回值是builder本身，这样方便把所有的setter方法的调用连起来，写成流式的api形式，如下：

```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8).calories(100).sodium(35).carbohydrate(27).build();
```

> This client code is easy to write and, more importantly, easy to read. **The Builder pattern simulates named optional parameters as found in Python and Scala.**
>
> Validity checks were omitted for brevity. To detect invalid parameters as soon as possible, check parameter validity in the builder’s constructor and methods. Check invariants involving multiple parameters in the constructor invoked by the build method. To ensure these invariants against attack, do the checks on object fields after copying parameters from the builder (Item 50). If a check fails, throw an *IllegalArgumentException* (Item72) whose detail message indicates which parameters are invalid (Item 75).
>
> **The Builder pattern is well suited to class hierarchies.** Use a parallel hierarchy of builders, each nested in the corresponding class. Abstract classes have abstract builders; concrete classes have concrete builders. For example, consider an abstract class at the root of a hierarchy representing various kinds of pizza:

从上面的例子可以看出，客户端代码很容易写，也很容易读。Builder模式模仿了Python和Scala中的具名的可选参数。

为了简洁起见，例子中省略了有效性检查。要想尽快的检查出无效的参数，可以在builder的构造器方法和setter方法里进行参数的有效性检测。为了避免这些不可变量被攻击，在从builder里复制参数的时候应该进行检测（Item50)，如果检测到异常，抛出一个*IllegalArgumentException*（Item72）异常，在其详细信息里说明那个参数异常（Item 75）。

**Builder模式也很适合类的层级结构。**对于多个平行类，可以平行的使用各个类对应的builder。抽象类有抽象的builder，非抽象类有非抽象的builder。比如，层级结构的一个底层抽象类如下，用于展示不同种类的pizza。

```java
// Builder pattern for class hierarchies
public abstract class Pizza {
    public enum Topping { 
        HAM, MUSHROOM, ONION, PEPPER,SAUSAGE 
    }
    final Set<Topping> toppings;
    abstract static class Builder<T extends Builder<T>> {
        EnumSet<Topping> toppings =
        EnumSet.noneOf(Topping.class);
        public T addTopping(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            return self();
        } 
        abstract Pizza build();
        // Subclasses must override this method to return "this"
        protected abstract T self();
    } 
    Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone(); // See Item 50
    }
}
```

> Note that *Pizza.Builder* is a generic type with a recursive type parameter (Item 30). This, along with the abstract *self* method, allows method chaining to work properly in subclasses, without the need for casts. This work around for the fact that Java lacks a self type is known as the simulated self-type idiom. Here are two concrete subclasses of Pizza, one of which represents a standard New-York-style pizza, the other a calzone. The former has a required size parameter, while the latter lets you specify whether sauce should be inside or out:

代码中需要注意的是，Pizza.Builder 类是一个带有递归类型参数的泛型。泛型和self方法一起使得方法链在子类中也能很的运行，也需要进行类型转换。这就解决了Java缺少self类型这一事实，也是模拟self类型的一种习惯用法。下面是两个具体的pizza的子类：一个是标准纽约风味的pizza，需要有一个表示大小的参数size；另一个是半圆形pizza，需要指定酱汁是内置还是外置。

```java
  public class NyPizza extends Pizza {
    public enum Size { 
        SMALL, MEDIUM, LARGE 
    }
    private final Size size;

    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;
        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        } 
        @Override 
        public NyPizza build() {
            return new NyPizza(this);
        } 
        @Override 
        protected Builder self() { 
            return this; 
        }
    } 

    private NyPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }
} 

public class Calzone extends Pizza {
    private final boolean sauceInside;

    public static class Builder extends Pizza.Builder<Builder> {
        private boolean sauceInside = false; // Default
        public Builder sauceInside() {
            sauceInside = true;
            return this;
        } 
        @Override 
        public Calzone build() {
            return new Calzone(this);
        } 
        @Override 
        protected Builder self() { 
            return this; 
        }
    }
    private Calzone(Builder builder) {
        super(builder);
        sauceInside = builder.sauceInside;
    }
}
```

> Note that the build method in each subclass’s builder is declared to return the correct subclass: the *build* method of *NyPizza.Builder* returns *NyPizza*, while the one in *Calzone.Builder* returns *Calzone*. This technique, where in a subclass method is declared to return a subtype of the return type declared in the super-class, is known as *covariant return typing*. It allows clients to use these builders without the need for casting. 
>
> The client code for these “hierarchical builders” is essentially identical to the code for the simple *NutritionFacts* builder. The example client code shown next assumes static imports on enum constants for brevity:

代码中需要注意的是，每一个子类builder里的build方法的返回类型都是对应的子类类型，比如NyPizza.Builder里的build方法返回类型是NyPizza，而Calzone.Builder里的build方法返回的则是Calzone。我们将这种子类方法的返回类型 声明为 父类方法返回类型的子类 的技术称为协变返回类型(covariant return type)。基于这种技术，客户端在使用这些builder的时候就不需要进行类型转换了。

这些具有层级结构的builder的客户端代码本质上和简单的NutritionFacts的builder一样。客户端代码如下，为了简洁起见，其中的枚举常量假设为静态导入。

```java
NyPizza pizza = new NyPizza.Builder(SMALL).addTopping(SAUSAGE).addTopping(ONION).build();
Calzone calzone = new Calzone.Builder().addTopping(HAM).sauceInside().build();
```

> A minor advantage of builders over constructors is that builders can have multiple varargs parameters because each parameter is specified in its own method. Alternatively, builders can aggregate the parameters passed into multiple calls to a method into a single field, as demonstrated in the *addTopping* method earlier.
>
> The Builder pattern is quite flexible. A single builder can be used repeatedly to build multiple objects. The parameters of the builder can be tweaked between invocations of the build method to vary the objects that are created. A builder can fill in some fields automatically upon object creation, such as a serial number that increases each time an object is created.

相对于构造器，Builder还有一个小小的优势：builder可以有多个可变（varargs)的参数，因为builder里每个参数都是通过单独的方法来设置的。此外builder可以通过多次调用同一个方法来将参数聚合到一个数据域中，比如前面的Pizza.Builder里的addTopping方法。

Builder模式非常灵活，一个builder对象可以反复使用创建多个对象。在每次创建对象之间可以调整builder参数的值来使得每次创建的对象都不同。builder还可以在创建对象时自动为一些域设置值，比如每次创建对象时自动增加序号值。

> The Builder pattern has disadvantages as well. In order to create an object, you must first create its builder. While the cost of creating this builder is unlikely to be noticeable in practice, it could be a problem in performance-critical situations. Also, the Builder pattern is more verbose than the telescoping constructor pattern, so it should be used only if there are enough parameters to make it worthwhile, say four or more. But keep in mind that you may want to add more parameters in the future. But if you start out with constructors or static factories and switch to a builder when the class evolves to the point where the number of parameters gets out of hand, the obsolete constructors or static factories will stick out like a sore thumb. Therefore, it’s often better to start with a builder in the first place.

当然，Builder模式也有缺点。为了创建一个对象，必须先创建其builder对象。虽然大多数情况下，创建一个构造器的消耗不足为虑，但是在非常注重性能的情况下，这就可能是一个问题了。同时，Builder模式比伸缩构造器模式更加冗长，因此，只有当参数的数量多到值得编写很长的代码的时候才使用Builder模式，比如4个及其以上。但是，还应该注意到以后可能会添加更多的参数。假如我们一开始使用的构造器或者静态工厂方法，然后随着餐宿个数的增多，想转用Builder模式，那么过时的构造器和builder就会显得格格不入，因此最好是一开始就使用Builder模式。

> In summary, **the Builder pattern is a good choice when designing classes whose constructors or static factories would have more than a handful of parameters**, especially if many of the parameters are optional or of identical type. Client code is much easier to read and write with builders than with telescoping constructors, and builders are much safer than JavaBeans.

总的来说，**在设计一个类的时候，当该类的构造器或者静态工厂方法有一定数量的参数时，Builder模式是一个不错的选择。**尤其是当一些参数是可选的，或者一些参数具有相同的类型的时候。Builder模式相对于伸缩构造器模式而言，客户端代码更容易编写和阅读；相对于JavaBeans模式而言，更安全。

### 