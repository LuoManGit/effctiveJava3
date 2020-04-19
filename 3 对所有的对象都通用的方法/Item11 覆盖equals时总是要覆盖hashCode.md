### Item11 覆盖equals时总是要覆盖hashCode

> **You must override** **hashCode** **in every class that overrides equals**. If you fail to do so, your class will violate the general contract for hashCode, which will prevent it from functioning properly in collections such as HashMap and HashSet. Here is the contract, adapted from the Object specification :
>
> - When the hashCode method is invoked on an object repeatedly during an execution of an application, it must consistently return the same value, provided no information used in equals comparisons is modified. This value need not remain consistent from one execution of an application to another.
> - If two objects are equal according to the equals(Object) method,then calling hashCode on the two objects must produce the same integer result.
> - If two objects are unequal according to the equals(Object) method,it is *not* required that calling hashCode on each of the objects must produce distinct results. However, the programmer should be aware that producing distinct results for unequal objects may improve the performance of hash tables.

**一个类如果覆盖了equals方法就必须要覆盖hashCode方法。**如果你不这么做的话，你的类就违反了hashCode的通用约定，从而导致你在使用集合（比如HashMap，HashSet）的时候，会出现问题。下面是Object规范里的hashCode的约定。

- 在一个应用程序里，多次调用一个对象的hashCode方法，只要用于equals比较的信息没有被修改，每次返回的值必须一致。在不同的应用程序中，hashCode放回的值可以不一致。
- 如果两个对象调用equals返回的结果是相等的，那么他们的hashCode方法返回的整形值必须相等。
- 当两个对象用用equals返回的结果是不相等时，不要求他们的hashCode返回的值必须不相等。但是，程序员必须知道，当两个不相等的对象的hashCode不相等时，可以提升hash表的性能。

> **The key provision that is violated when you fail to override** **hashCode** **is the second one: equal objects must have equal hash codes.** Two distinct instances may be logically equal according to a class’s equals method, but to Object’s hashCode method, they’re just two objects with nothing much in common. Therefore, Object’s hashCode method returns two seemingly random numbers instead of two equal numbers as required by the contract.
>
> For example, suppose you attempt to use instances of the PhoneNumber class from Item 10 as keys in a HashMap:

当你没有正确的覆盖hashCode方法的时候，违反的关键约定是上面的第二条：相等的对象必须有相同的hash值。两个不同的对象根据其equals方法可能是逻辑相等的，但对于Object的hashCode方法而言，他们就是两个普普通通的对象，因此，Object的hashCode方法会返回两个看起来随机的数值，而不是这两个相同的数值。

比如，假如你想使用Item10里面的PhoneNumber类的实例作为HashMap里的key，如下：

```java
 Map<PhoneNumber, String> m = new HashMap<>();
 m.put(new PhoneNumber(707, 867, 5309), "Jenny");
```

> At this point, you might expect m.get(new PhoneNumber(707, 867, 5309)) to return "Jenny", but instead, it returns null. Notice that two PhoneNumber instances are involved: one is used for insertion into the HashMap, and a second, equal instance is used for (attempted) retrieval. The PhoneNumber class’s failure to override hashCode causes the two equal instances to have unequal hash codes, in violation of the hashCode contract. Therefore, the get method is likely to look for the phone number in a different hash bucket from the one in which it was stored by the put method. Even if the two instances happen to hash to the same bucket, the get method will almost certainly return null, because HashMap has an optimization that caches the hash code associated with each entry and doesn’t bother checking for object equality if the hash codes don’t match.

在这种情况下，你可能会希望m.get(new PhoneNumber(707, 867, 5309)) 返回"Jenny"，但是其返回值确是null。需要注意这里涉及到两个PhoneNumber实例，一个是用来插入HashMap的，另一个是（企图）用来进行检索的。问题在于PhoneNumber类没有重写hashCode方法导致了两个相等的实例拥有不同的hash值，违反了hashCode约定。因此，这个get方法有可能在另一个hash桶（hash bucket）里去寻找PhoneNumber实例，而不是在之前存储PhoneNumber实例的hash桶里。即便是这个两个实例碰巧被分到一个hash桶里去了，这个get方法也几乎肯定会返回null，因为HashMap做了一个优化，缓存了每个entry里的hashcode，当hashcode不相等的时候，就不用调用equals方法了。

> Fixing this problem is as simple as writing a proper hashCode method for PhoneNumber. So what should a hashCode method look like? It’s trivial to write a bad one. This one, for example, is always legal but should never be used:

要解决这个问题很简单，只要给PhoneNumber类写一个hashCode方法就好了。那hashCode方法该怎么去写呢？写一个合法但是不好用的hashCode方法是没有价值的。比如下面这个，它始终是合法的但是绝不应该这样用：

```java
// The worst possible legal hashCode implementation - never use!
   @Override public int hashCode() { return 42; }
```

> It’s legal because it ensures that equal objects have the same hash code. It’s atrocious because it ensures that *every* object has the same hash code. Therefore, every object hashes to the same bucket, and hash tables degenerate to linked lists. Programs that should run in linear time instead run in quadratic time. For large hash tables, this is the difference between working and not working.

这个方法是合法的，因为它保证了所有相等的对象都有相同的hash值。但是它非常糟糕，因为它还保证了每个对象都有相同的hash值。在使用hash表的时候，这些对象就会散列到同一个桶里，然后hash表就会退化成一个链表。本来应该是线性时间复杂度的程序，就变成了平方级的时间复杂度。对于大型的hash表，就会关系到hash表能不能正常的工作了。

> A good hash function tends to produce unequal hash codes for unequal instances. This is exactly what is meant by the third part of the hashCode contract. Ideally, a hash function should distribute any reasonable collection of unequal instances uniformly across all int values. Achieving this ideal can be difficult. Luckily it’s not too hard to achieve a fair approximation. Here is a simple recipe:

一个好的hash函数应该倾向于“为不同的实例生成不同的hash值”。这也就是hashCode约定里第三条的真实含义。一个理想的hash函数应用为不同的实例，生成不同的hash值，均匀分布在所有的int值上。要实现一个理想的hash函数很难，但是实现一个差不多的接近理想情况的hash函数并不那么难。这里有一种简单的实现方法：

> 1. Declare an int variable named result, and initialize it to the hash code c for the first significant field in your object, as computed in step 2.a. (Recall from Item 10 that a significant field is a field that affects equals comparisons.)
>
> 2. For every remaining significant field f in your object, do the following: 
>
>    a. Compute an int hash code c for the field:
>
>    1. If the field is of a primitive type, compute Type.hashCode(f), where Type is the boxed primitive class corresponding to f’ s type.
>    2. If the field is an object reference and this class’s equals method compares the field by recursively invoking equals, recursively invoke hashCode on the field. If a more complex comparison is required, compute a “canonical representation” for this field and invoke hashCode on the canonical representation. If the value of the field is null, use 0 (or some other constant, but 0 is traditional).
>    3. If the field is an array, treat it as if each significant element were a separate field. That is, compute a hash code for each significant element by applying these rules recursively, and combine the values per step 2.b. If the array has no significant elements, use a constant, preferably not 0. If all elements are significant, use Arrays.hashCode.
>
>    b. Combine the hash code c computed in step 2.a into result as follows: result = 31 * result + c;
>
> 3. Return result.

1. 声明一个叫result的int变量，然后将它初始化为第一个重要域的hash值，具体hash值计算见步骤2.a（在Item10里已经说过了，重要域是指equals方法里用来比较的域）。

2. 对于对象里的每一个剩下的重要域f ，都进行如下操作：

   a. 计算这些域的hash值：

   	1. 如果这个域的类型是原始类型，那么调用Type.hashCode(f)方法来获得hash值，其中Type是指f的类型对应的包装类型。

      	2. 如果这个域是一个对象引用，并且这个类的equals方法递归地调用了这个域的equals方法，那么我们就可以递归地调用这个域的hashCode方法来计算hash值。如果需要很复杂的比较，我们可以为这个域计算一个“范式（canonical representation）”，然后调用这个范式的hashCode方法。如果该域的值为null，其hash值就定为0（也可以是其他的常数，只是通常都用0）
      	3. 若果这个域是一个数组，则要把每一个重要元素当做一个单独的域来计算。也就是，递归的应用这些规则计算每一个重要元素的hashcode，然后使用步骤2.b里的方法结合起来。如果这个数组一个重要的元素都没有，那就使用一个常数，最好不要是0。如果所有的元素都很重要，那就使用Arrays.hashCode计算hash值。

   b. 按照公式将步骤2.a里计算得到的hash值和result结合到一起：

   ```java
   result = 31 * result + c
   ```

3. 返回result。

> When you are finished writing the hashCode method, ask yourself whether equal instances have equal hash codes. Write unit tests to verify your intuition (unless you used AutoValue to generate your equals and hashCode methods, in which case you can safely omit these tests). If equal instances have unequal hash codes, figure out why and fix the problem.

当你写完一个hashCode方法后，就问一下自己，相同的实例能够确保有相同的hash值吗？并写一些单元测试来验证你的直觉（当你使用AutoValue来生成equals和hashCode方法的时候，你可以放心地不做这些测试）。如果相同的实例，却有不同的hash值，找到问题所在，改掉它。

> You may exclude *derived fields* from the hash code computation. In other words, you may ignore any field whose value can be computed from fields included in the computation. You *must* exclude any fields that are not used in equals comparisons, or you risk violating the second provision of the hashCode contract.
>
> The multiplication in step 2.b makes the result depend on the order of the fields, yielding a much better hash function if the class has multiple similar fields. For example, if the multiplication were omitted from a String hash function, all anagrams would have identical hash codes. The value 31 was chosen because it is an odd prime. If it were even and the multiplication overflowed, information would be lost, because multiplication by 2 is equivalent to shifting. The advantage of using a prime is less clear, but it is traditional. A nice property of 31 is that the multiplication can be replaced by a shift and a subtraction for better performance on some architectures: 31 * i == (i << 5) - i. Modern VMs do this sort of optimization automatically.
>
> Let’s apply the previous recipe to the PhoneNumber class:

在散列码的计算中，你可以把衍生域排除在外。也就是说，你可以不用计算那些值会在其他的域里被计算的域。必须把equals里面没有用来比较的域排除在外，否则你就有可能违反hashCode的第二条约定。

步骤2.b里面的乘法计算使得结果和域参与计算的顺序有关，如果一个类包含很多相似的域，那么包含这种的计算hash函数就会更好一些。例如，假如在String的hash函数里没有这样的乘法计算，就会使得所有的同字母异构的字符串都拥有相同的hash值。之所以选择31，是因为他是一个素数，如果是一个偶数的话，当乘法计算溢出的时候就会丢失信息，因为和2相乘相当于移位操作。虽然使用素数的优势不是很明显，但是这就是传统。31的一个非常好的特性是，乘以31操作可以用一个移位操作和一个减法计算来替代，可以获得更好的性能：31 * i == (i << 5) - i。在现代的虚拟机里， 会自动进行这种优化。

让我们根据前面的步骤，写一个PhoneNumber的hashCode函数，如下：

```java
// Typical hashCode method
   @Override public int hashCode() {
       int result = Short.hashCode(areaCode);
       result = 31 * result + Short.hashCode(prefix);
       result = 31 * result + Short.hashCode(lineNum);
       return result;
}
```

> Because this method returns the result of a simple deterministic computation whose only inputs are the three significant fields in a PhoneNumber instance, it is clear that equal PhoneNumber instances have equal hash codes. This method is, in fact, a perfectly good hashCode implementation for PhoneNumber, on par with those in the Java platform libraries. It is simple, is reasonably fast, and does a reasonable job of dispersing unequal phone numbers into different hash buckets.
>
> While the recipe in this item yields reasonably good hash functions, they are not state-of-the-art. They are comparable in quality to the hash functions found in the Java platform libraries’ value types and are adequate for most uses. If you have a bona fide need for hash functions less likely to produce collisions, see Guava’s com.google.common.hash.Hashing [Guava].

因为这个方法返回的只是PhoneNumber实例中三个重要域的简单明确的计算结果，因此很明显，相同的PhoneNumber实例拥有相同的hash值。实际上，PhoneNumber的hashCode的实现是非常好的，可以和java平台类库中的实现媲美。它很简单，快捷，也能很好的将不同的phoneNumber实例分发到不同的桶里去。

虽然前面给出的步骤可以实现很好的hash函数，但是还不是最最好的。它们的质量可以和java平台类库中可以找到的值类型的hash函数相比，对于大部分实际情况，也足够了。如果你非要让hash函数尽可能的减少hash冲突，可以参看Guava的com.google.common.hash.Hashing [Guava].

> The Objects class has a static method that takes an arbitrary number of objects and returns a hash code for them. This method, named hash, lets you write one-line hashCode methods whose quality is comparable to those written according to the recipe in this item. Unfortunately, they run more slowly because they entail array creation to pass a variable number of arguments, as well as boxing and unboxing if any of the arguments are of primitive type. This style of hash function is recommended for use only in situations where performance is not critical. Here is a hash function for PhoneNumber written using this technique:

在Objects类里有一个静态方法，它的参数是任意数量的对象，返回一个它们的hash值。这个方法的名字叫hash，使得我们可以在hashCode里只写一行代码，其质量和我们根据前面的步骤写出来的代码差不多。不幸的是，它们运行得很慢，因为为了传入可变数量的参数，需要创建一个数组，同时如果参数里有原始类型的话，还需要装箱和拆箱。因此这种风格的hash函数，只推荐在性能要求不严格的情况下使用。下面是一个使用这种方式写得PhoneNumber的hash函数：

```java
// One-line hashCode method - mediocre performance
   @Override public int hashCode() {
      return Objects.hash(lineNum, prefix, areaCode);
   }
```

> If a class is immutable and the cost of computing the hash code is significant, you might consider caching the hash code in the object rather than recalculating it each time it is requested. If you believe that most objects of this type will be used as hash keys, then you should calculate the hash code when the instance is created. Otherwise, you might choose to *lazily initialize* the hash code the first time hashCode is invoked. Some care is required to ensure that the class remains thread-safe in the presence of a lazily initialized field (Item 83). Our PhoneNumber class does not merit this treatment, but just to show you how it’s done, here it is. Note that the initial value for the hashCode field (in this case, 0) should not be the hash code of a commonly created instance:

如果一个类是不可变的，并且计算hash值开销比较大，为了避免每次需要的时候都算一遍，可以考虑将hash值缓存起来。如果你认定这个类的绝大部分实体都将被当做hash值使用，当这个实例创建的时候你就可以计算它的hash值。否则，你可以选择使用“延迟初始化”的方法在第一次调用hash值的时候初始化hash值。在使用延迟初始化的时候，要注意保证这个类是线程安全的（Item83）。虽然我们的PhoneNumber类不值得这样做，但还是可以用它来说明这种方法具体如何实现，如下。需要注意的是，hashCode的初始值（本例中为0）通常不能是创建的实例的hash值。

```java
// hashCode method with lazily initialized cached hash code
   private int hashCode; // Automatically initialized to 0
   @Override public int hashCode() {
       int result = hashCode;
       if (result == 0) {
           result = Short.hashCode(areaCode);
           result = 31 * result + Short.hashCode(prefix);
           result = 31 * result + Short.hashCode(lineNum);
           hashCode = result;
			 }
       return result;
   }
```

> **Do not be tempted to exclude significant fields from the hash code computation to improve performance.** While the resulting hash function may run faster, its poor quality may degrade hash tables’ performance to the point where they become unusable. In particular, the hash function may be confronted with a large collection of instances that differ mainly in regions you’ve chosen to ignore. If this happens, the hash function will map all these instances to a few hash codes, and programs that should run in linear time will instead run in quadratic time.
>
> This is not just a theoretical problem. Prior to Java 2, the String hash function used at most sixteen characters evenly spaced throughout the string, starting with the first character. For large collections of hierarchical names, such as URLs, this function displayed exactly the pathological behavior described earlier.

**不要为了提升性能就企图把一些重要的域排除在hash值的计算范围之外。**虽然hash函数可能会运行得快一些，但是它可能会因为质量太差导致hash表会慢到根本无法使用。尤其是，hash函数可能会面对一大堆的实例，它们可能主要的区别就在于被你排除在外的那些域上。在这种情况下，hash函数会为这些对象生成少量的hash值，导致程序的时间复杂度从线性复杂度变成了平方级复杂度。

这不仅仅是一个理论的问题。在java2发行之前，String的hash函数最多使用16个字符，对于长度大于16个字符的，就从第一个字符开始进行均分间隔取样，进行计算。对于一大堆的层次状名称，比如URL，这个hash函数就刚好表现出了前面说到的问题（*很多不同的String的hash值相同*）。

> **Don’t provide a detailed specification for the value returned by** **hashCode, so clients can’t reasonably depend on it; this gives you the flexibility to change it.** Many classes in the Java libraries, such as String and Integer, specify the exact value returned by their hashCode method as a function of the instance value. This is *not* a good idea but a mistake that we’re forced to live with: It impedes the ability to improve the hash function in future releases. If you leave the details unspecified and a flaw is found in the hash function or a better hash function is discovered, you can change it in a subsequent release.

**不要通过hashCode的返回值提供一些详细的规定，因此客户端也不能理所当然地依靠这些规定，也就给了你修改hash函数的灵活性**。在Java类库中的很多类，比如String，Integer，把hashCode方法返回的具体的值规定为实例值的一个函数。这是一个不好的主意，甚至是错误的，但是我们必须面对它。它限制了在未来的版本中优化hash函数的能力。如果你没有进行详细的规定，当hash函数有点问题，或者你发现了一个更好的hash函数时，就可以在后面的发行版本里面修改hash函数。

> In summary, you *must* override hashCode every time you override equals, or your program will not run correctly. Your hashCode method must obey the general contract specified in Object and must do a reasonable job assigning unequal hash codes to unequal instances. This is easy to achieve, if slightly tedious, using the recipe on page 51. As mentioned in Item 10, the AutoValue framework provides a fine alternative to writing equals and hashCode methods manually, and IDEs also provide some of this functionality.

总结一下，当你每次覆盖equals方法的时候，就必须要覆盖hashCode方法，否则你的程序就会出错。写的hashCode方法必须要遵守Object规定里的通用约定，对于不同的实例，必须要合理的分配不同的hash值。只要你老老实实地按照前面的步骤写的话，这些目标是很容易达到的。正如Item10里提到的那样，不想手写equals和hashCode方法的话，可以使用AutoValue框架，同时一些IDE也提供了一些类似的功能。

### 