### Item7 清除过期的对象引用

> If you switched from a language with manual memory management, such as C or C++, to a garbage-collected language such as Java, your job as a programmer was made much easier by the fact that your objects are automatically reclaimed when you’re through with them. It seems almost like magic when you first experience it. It can easily lead to the impression that you don’t have to think about memory management, but this isn’t quite true.
>
> Consider the following simple stack implementation:

当你从一个需要手动管理内存的语言，比如C或者C++，切换到一个具有垃圾回收功能的语言时，比如java。程序员的工作会变得更加容易一些，因为当你使用完一个对象后，这个对象将会自动被回收。当你第一次经历的时候，会感觉很神奇。因此很容易给人留下这样的印象：不需要再考虑内存管理了。但这种想法是不正确的。

考虑下面这个简单的栈的实现：

```java
// Can you spot the "memory leak"?
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    } 
    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    } 
    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size];
    } 
    /**
     * Ensure space for at least one more element, roughly
     * doubling the capacity each time the array needs to grow.
     */
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

> There’s nothing obviously wrong with this program (but see Item 29 for a generic version). You could test it exhaustively, and it would pass every test with flying colors, but there’s a problem lurking. Loosely speaking, the program has a “memory leak,” which can silently manifest itself as reduced performance due to increased garbage collector activity or increased memory footprint. In extreme cases, such memory leaks can cause disk paging and even program failure with an OutOfMemoryError, but such failures are relatively rare.
>
> So where is the memory leak? If a stack grows and then shrinks, the objects that were popped off the stack will not be garbage collected, even if the program using the stack has no more references to them. This is because the stack maintains *obsolete references* to these objects. An obsolete reference is simply a reference that will never be dereferenced again. In this case, any references outside of the “active portion” of the element array are obsolete. The active portion consists of the elements whose index is less than size.

这段程序看起来没什么明显的问题（其反向版本见Item29）。它可以通过任何的测试，但是存在一个潜伏的问题。不严格地说，这个程序存在“内存泄露”的问题，内存泄漏将导致垃圾回收活动频繁，内存占用增多，从而导致性能降低。在极端的情况下，内存泄露还可能导致出现“磁盘交换”*（个人理解：内存放不下了，操作系统会在磁盘中开辟一块临时存储空间，将内存里的数据放在这里，即磁盘交换）*，甚至会抛出OOM异常导致程序终止。当然这种情况还是比较少见的。

那么，这段程序哪里出现了内存泄露呢？考虑一个栈先增长，然后再收缩，但是这些弹出的对象并没有被垃圾回收，即便是使用栈的程序不再引用这些对象，这些对象也不会被回收。这是因为栈有这些对象的过期引用（obsolute reference)，所谓过期引用是指永远不会被解除的应用。在本例中，element数组活动区域以外的引用都是过期的，活动区域是指element数组中索引小于size的区域。

> Memory leaks in garbage-collected languages (more properly known as *unintentional object retentions*) are insidious. If an object reference is unintentionally retained, not only is that object excluded from garbage collection, but so too are any objects referenced by that object, and so on. Even if only a few object references are unintentionally retained, many, many objects may be prevented from being garbage collected, with potentially large effects on performance.
>
> The fix for this sort of problem is simple: null out references once they become obsolete. In the case of our Stack class, the reference to an item becomes obsolete as soon as it’s popped off the stack. The corrected version of the pop method looks like this:

在支持垃圾回收的语言中的内存泄露（更多时候被称为”无意识的对象保持“）是非常隐蔽的。如果一个对象引用被无意识的保持，那么不仅仅这个对象不会被垃圾回收，那些被这个对象引用的对象们也不会被垃圾回收，以此递推。因此即使只有非常非常少的对象被无意识的保持，也会导致大量的对象不会被垃圾回收，从而对性能造成较大的潜在影响。

这类问题的解决方案很简单：一旦引用过期，就将其置为null。比如前面提到的Stack类，一旦节点从栈里被弹出，这个节点的引用就过期了。改进后的pop方法如下：

```java
public Object pop() {
    if (size == 0)
        throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null; // Eliminate obsolete reference
    return result;
}
```

> An added benefit of nulling out obsolete references is that if they are subsequently dereferenced by mistake, the program will immediately fail with a NullPointerException, rather than quietly doing the wrong thing. It is always beneficial to detect programming errors as quickly as possible.
>
> When programmers are first stung by this problem, they may overcompensate by nulling out every object reference as soon as the program is finished using it. This is neither necessary nor desirable; it clutters up the program unnecessarily. **Nulling out object references should be the exception rather than the norm.** The best way to eliminate an obsolete reference is to let the variable that contained the reference fall out of scope. This occurs naturally if you define each variable in the narrowest possible scope (Item 57).

使用null来清除过期引用的另外一个好处是，如果这个引用随后又被错误地解除引用，程序将会立马报NullPointerException异常，而不是悄悄地做一些错误的事情。尽可能块的发现程序中的问题总是有好处的。

当程序员第一次被这种问题困扰过以后，可能会出现过度反应，每当程序一旦结束使用一个对象，都将对象得引用置为null。这种操作没有必要，也不明智，还把程序搞得很乱。**清空对象的引用应该是一种例外行为，而不是日常行为。**消除过期引用的最好的方式是让持有引用的变量结束其生命周期。当你将每个变量都定义在最紧凑的作用域里时，这种情况就会自然而然地发生。

> So when should you null out a reference? What aspect of the Stack class makes it susceptible to memory leaks? Simply put, it *manages its own memory*. The *storage pool* consists of the elements of the elements array (the object reference cells, not the objects themselves). The elements in the active portion of the array (as defined earlier) are *allocated*, and those in the remainder of the array are *free*. The garbage collector has no way of knowing this; to the garbage collector, all of the object references in the elements array are equally valid. Only the programmer knows that the inactive portion of the array is unimportant. The programmer effectively communicates this fact to the garbage collector by manually nulling out array elements as soon as they become part of the inactive portion.
>
> Generally speaking, **whenever a class manages its own memory, the programmer should be alert for memory leaks**. Whenever an element is freed, any object references contained in the element should be nulled out.

那么什么时候需要将引用置为null呢？例子中的Stack类是怎么出现内存泄露问题的呢？简单的来说，就是Stack类他自己管理了内存。存储池（storage pool)包括了elements数组里的元素们（元素是对象引用，不是对象本身）。数组活动区域的元素是已经分配了的，而其余部分则是自由的（free)。但是垃圾回收期却不知道这些，对于垃圾回收期而言，在elements数组里的所有的对象引用都是相同的，只有程序员知道数组的非活动区域是不重要的。程序员可以通过将这些变成非活动区域的元素手动置为null，以便垃圾回收期进行回收。

总的来说，**当一个类自己管理内存的时候，程序员就要当心内存泄露问题**。当一个元素被释放是，应该将这个元素保持的对象应用置为null。

> **Another common source of memory leaks is caches.** Once you put an object reference into a cache, it’s easy to forget that it’s there and leave it in the cache long after it becomes irrelevant. There are several solutions to this problem. If you’re lucky enough to implement a cache for which an entry is relevant exactly so long as there are references to its key outside of the cache, represent the cache as a WeakHashMap; entries will be removed automatically after they become obsolete. Remember that WeakHashMap is useful only if the desired lifetime of cache entries is determined by external references to the key, not the value.
>
> More commonly, the useful lifetime of a cache entry is less well defined, with entries becoming less valuable over time. Under these circumstances, the cache should occasionally be cleansed of entries that have fallen into disuse. This can be done by a background thread (perhaps a ScheduledThreadPoolExecutor) or as a side effect of adding new entries to the cache. The LinkedHashMap class facilitates the latter approach with its removeEldestEntry method. For more sophisticated caches, you may need to use java.lang.ref directly.

另一个内存泄露的常见来源就是缓存。一旦把一个对象的引用放在了缓存里，就很容易忘记它，就算是它已经不需要了也一直被保存在缓存里。针对这个问题，有几种解决方案。如果你要实现一个这样的缓存：只要缓存外部对这项的键有引用，那么这项就有意义，这种情况就可以使用WeakHashMap，当缓存项过期以后，这项就会被自动清除掉。需要注意的是，只有当缓存项的生命周期根据外部对键（key）而不是值（value）的引用来决定的时候，WeakHashMap才有用。

更常见的情况是，“缓存项的生命周期是否有意义”很难去定义，随着时间的推移，缓存项变得越来越没有价值。在这种情况下，缓存应该时不时地清除没用的项。这个清除的操作可以放在一个后台线程里（比如ScheduledHashMap里)，或者在往缓存里放新的元素时进行清除操作。LinkedHashMap提供了一个removeEldestEntry方法可以很容易的实现这一操作。对于一些更加复杂的缓存，你可以直接使用java.lang.ref。

> **A third common source of memory leaks is listeners and other callbacks.**If you implement an API where clients register callbacks but don’t deregister them explicitly, they will accumulate unless you take some action. One way to ensure that callbacks are garbage collected promptly is to store only *weak references* to them, for instance, by storing them only as keys in a WeakHashMap.
>
> Because memory leaks typically do not manifest themselves as obvious failures, they may remain present in a system for years. They are typically discovered only as a result of careful code inspection or with the aid of a debugging tool known as a *heap profiler*. Therefore, it is very desirable to learn to anticipate problems like this before they occur and prevent them from happening.

**第三种常见的内存泄露的来源是监听器和其他回调。**如果你实现了一个用于客户端注册回调的api，但是又没有显示地注销掉，那么除非你采取一些办法，否则这些回调就会堆积起来。一种保证这些回调会被及时垃圾回收的方法是值使用弱引用（weak reference）进行保存，比如，使用WeakHashMap的key进行保存。

内存泄漏不会表现为明显的失败，因此他们可以在一个系统里存在很多年。往往只能通过特别细致的代码检查，或者借助一个叫堆分析器（heap profiler）的工具才能发现内存泄露问题。因此，学会在内存泄露之前就预测到问题并解决问题是非常好的。
