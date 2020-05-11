### Item54 返回empty的集合或者数组，而不是null

> It is not uncommon to see methods that look something like this:

我们经常会看到下面这样的方法：

```java
// Returns null to indicate an empty collection. Don’t do this!
   private final List<Cheese> cheesesInStock = ...;
   /**
    * @return a list containing all of the cheeses in the shop,
    *     or null if no cheeses are available for purchase.
    */
   public List<Cheese> getCheeses() {
       return cheesesInStock.isEmpty() ? null
           : new ArrayList<>(cheesesInStock);
}
```

> There is no reason to special-case the situation where no cheeses are available for purchase. Doing so requires extra code in the client to handle the possibly null return value, for example:

完全没有理由把没有奶酪可买当做是一种特殊的情况。这样做，会使得客户端需要额外的代码来处理返回值为null的情况，如下：

```java
List<Cheese> cheeses = shop.getCheeses();
if (cheeses != null && cheeses.contains(Cheese.STILTON))
       System.out.println("Jolly good, just the thing.");
```

> This sort of circumlocution is required in nearly every use of a method that returns null in place of an empty collection or array. It is error-prone, because the programmer writing the client might forget to write the special-case code to handle a null return. Such an error may go unnoticed for years because such methods usually return one or more objects. Also, returning null in place of an empty container complicates the implementation of the method returning the container.

当我们使用null来代替空的集合和数组来作为方法的返回值的时候，就需要使用这种曲折的方法。它很容易出错，因为编写客户端的程序员可能会忘记特殊的代码来处理这个null返回，这种错误可能好几年都不会发现，因为这种方法总是会返回一个或者几个对象。并且，使用null来代替空的容器会使得返回容器的方法实现也很复杂。

> It is sometimes argued that a null return value is preferable to an empty collection or array because it avoids the expense of allocating the empty container. This argument fails on two counts. First, it is inadvisable to worry about performance at this level unless measurements have shown that the allocation in question is a real contributor to performance problems (Item 67). Second, it *is* possible to return empty collections and arrays without allocating them. Here is the typical code to return a possibly empty collection. Usually, this is all you need:

有时候，有人认为返回一个null比返回空的集合和数组要好，因为null避免了创建空的容器的开销。这个说法是站不住脚的，原因有二。第一是担心这种级别的性能是不明智的，除非分析证明这个问题里的创建容器是造成性能问题的真正原因（Item67）。第二，我们可以在不创建的情况下，返回一个空的集合或数组。下面使我们常见的用来返回空的集合的代码，一般情况下，也正是你所需要的：

```java
//The right way to return a possibly empty collection
   public List<Cheese> getCheeses() {
       return new ArrayList<>(cheesesInStock);
}
```

> In the unlikely event that you have evidence suggesting that allocating empty collections is harming performance, you can avoid the allocations by returning the same *immutable* empty collection repeatedly, as immutable objects may be shared freely (Item 17). Here is the code to do it, using the Collections.emptyList method. If you were returning a set, you’d use Collections.emptySet; if you were returning a map, you’d use Collections.emptyMap. But remember, this is an optimization, and it’s seldom called for. If you think you need it, measure performance before and after, to ensure that it’s actually helping:

如果你真的有证据证明，创建空的集合会影响到性能，你可以通过重复返回一个相同的不可变的空集合来避免这个创建，因为不可变的对象可以自由共享（Item17）。下面是使用Collections.emptyList方法的代码。如果你返回一个set，你就使用Collections.emptySet，如果你返回一个map，就使用Collections.emptyMap。但是记住，这只是一个优化，很少使用。如果你觉得你需要它，在使用前后需要进行性能测试，以确保它确实有用：

```java
// Optimization - avoids allocating empty collections
   public List<Cheese> getCheeses() {
       return cheesesInStock.isEmpty() ? Collections.emptyList()
           : new ArrayList<>(cheesesInStock);
}
```

> The situation for arrays is identical to that for collections. Never return null instead of a zero-length array. Normally, you should simply return an array of the correct length, which may be zero. Note that we’re passing a zero-length array into the toArray method to indicate the desired return type, which is Cheese[]:

对于数组，这种情况是一样的。永远都不要返回null来代替长度为0的数组。通常情况下，你应该简单返回一种正确长度的数组，它的长度可能是0。注意我们给toArray方法传入一个长度为0的数组，来表示我们希望的返回类型是Cheese[]。

> If you believe that allocating zero-length arrays is harming performance, you can return the same zero-length array repeatedly because all zero-length arrays are immutable:

如果你认为创建一个长度为0的数组会影响性能，那么你可以重复地返回同一个长度为0的数组，因为所有的长度为0的数组都是不可变的。代码如下：

```java
// Optimization - avoids allocating empty arrays
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];
public Cheese[] getCheeses() {
    return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```

> In the optimized version, we pass *the same* empty array into every toArray call, and this array will be returned from getCheeses whenever cheesesInStock is empty. Do *not* preallocate the array passed to toArray in hopes of improving performance. Studies have shown that it is counterproductive [Shipilëv16]:

在上面这个优化的版本中，在每次调用toArray的时候，我们都会传递一个相同的空数组，并且，当cheesesInStock为空的时候，就会直接返回这个数组。不要指望在传入toArray之前创建数组，代码如下，可以提升性能，研究表明，这样只会适得其反[Shipilëv16]:

```java
// Don’t do this - preallocating the array harms performance!
   return cheesesInStock.toArray(new Cheese[cheesesInStock.size()]);
```

> In summary, **never return** **null** **in place of an empty array or collection.** It makes your API more difficult to use and more prone to error, and it has no performance advantages.

总结一下，**永远不要返回null来代替空的数组或集合。**塔湖使得你的API很难使用，容易出错，还没有什么性能优势。