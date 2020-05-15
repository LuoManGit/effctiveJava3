### Item58 for-each循环优先于传统的for循环

> As discussed in Item 45, some tasks are best accomplished with streams, others with iteration. Here is a traditional for loop to iterate over a collection:

正如Item45里所讨论的那样，有一些任务适合使用stream，而有一些适合使用迭代。下面是一个使用传统的for循环来迭代集合的例子：

```java
 // Not the best way to iterate over a collection!
   for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
       Element e = i.next();
       ... // Do something with e
}
```

> and here is a traditional for loop to iterate over an array:

下面是一个传统的for循环迭代数组的例子：

```java
// Not the best way to iterate over an array!
   for (int i = 0; i < a.length; i++) {
       ... // Do something with a[i]
}
```

> These idioms are better than while loops (Item 57), but they aren’t perfect. The iterator and the index variables are both just clutter—all you need are the elements. Furthermore, they represent opportunities for error. The iterator occurs three times in each loop and the index variable four, which gives you many chances to use the wrong variable. If you do, there is no guarantee that the compiler will catch the problem. Finally, the two loops are quite different, drawing unnecessary attention to the type of the container and adding a (minor) hassle to changing that type.

这些例子都比while循环要好一些，但是，还不够完美。这个iterator以及索引变量都会带啦一些混乱，我们需要的只是元素。并且，还可能出现错误。在每次循环中，iterator出现了3次，索引变量出现了4次，这些出现很容易让人出错。如果你出错了，也无法保证编译器会发现这个问题，最后，这两个循环是完全不一样的，容器的类型转移了不必要的注意力，还给类型转换带来了一些（小小）的麻烦。

> The for-each loop (officially known as the “enhanced for statement”) solves all of these problems. It gets rid of the clutter and the opportunity for error by hid- ing the iterator or index variable. The resulting idiom applies equally to collec- tions and arrays, easing the process of switching the implementation type of a container from one to the other:

for-each循环（官方称之为“增强的for循环”）解决了这些问题。它通过隐藏iterator和索引变量解决了混乱和可能出错的问题。得到的代码应用在集合和数组上都一样，简化了把容器的实现类型从一个转换到另一个的过程。代码如下：

```java
// The preferred idiom for iterating over collections and arrays
   for (Element e : elements) {
       ... // Do something with e
}
```

> When you see the colon (:), read it as “in.” Thus, the loop above reads as “for each element *e* in *elements*.” There is no performance penalty for using for-each loops, even for arrays: the code they generate is essentially identical to the code you would write by hand.

当你看到这个冒号：的时候，可以读作 in（在......里面）。因此这个循环就可以读作“对于在elements里的每一个元素e”。使用for-each循环没有什么性能损失，即使是对数组也一样；生成的代码和你自己手动编写的代码本质上是一样的。

> The advantages of the for-each loop over the traditional for loop are even greater when it comes to nested iteration. Here is a common mistake that people make when doing nested iteration:

对于嵌套式迭代，for-each循环相对于传统for循环的优势还要大一些。下面是人们在做嵌套迭代的时候常犯的错误：

```java
// Can you spot the bug?
   enum Suit { CLUB, DIAMOND, HEART, SPADE }
   enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT,
               NINE, TEN, JACK, QUEEN, KING }
   ...
   static Collection<Suit> suits = Arrays.asList(Suit.values());
   static Collection<Rank> ranks = Arrays.asList(Rank.values());
   List<Card> deck = new ArrayList<>();
   for (Iterator<Suit> i = suits.iterator(); i.hasNext(); )
       for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
           deck.add(new Card(i.next(), j.next()));
```

> Don’t feel bad if you didn’t spot the bug. Many expert programmers have made this mistake at one time or another. The problem is that the next method is called too many times on the iterator for the outer collection (suits). It should be called from the outer loop so that it is called once per suit, but instead it is called from the inner loop, so it is called once per card. After you run out of suits, the loop throws a NoSuchElementException.

如果你没有发现这个问题的话，也不用太难过。很多的专家级别的程序员有时候也会犯这种错。这里的问题在于这个外围集合suits的iterator的next方法调用的次数太多了。它应该在外围循环中调用，这样便是每个花色掉用一次，但是它却在嵌套循环中调用了，因此就成了每个卡片调用一次了。当你运行这个代码的时候，这个循环就会抛出NoSuchElementException。

> If you’re really unlucky and the size of the outer collection is a multiple of the size of the inner collection—perhaps because they’re the same collection—the loop will terminate normally, but it won’t do what you want. For example, consider this ill-conceived attempt to print all the possible rolls of a pair of dice:

如果你运气非常不好，外围集合的大小刚刚好是内部集合大小的倍数（或者他们就是一个集合），这种情况下，这个循环会正常结束，但是得到的结果却不是你想要的。比如下面这个打印一对骰子的所有可能情况，就考虑不周：

```java
// Same bug, different symptom!
   enum Face { ONE, TWO, THREE, FOUR, FIVE, SIX }
   ...
   Collection<Face> faces = EnumSet.allOf(Face.class);
   for (Iterator<Face> i = faces.iterator(); i.hasNext(); )
       for (Iterator<Face> j = faces.iterator(); j.hasNext(); )
           System.out.println(i.next() + " " + j.next());
```

> The program doesn’t throw an exception, but it prints only the six “doubles” (from “ONE ONE” to “SIX SIX”), instead of the expected thirty-six combinations.
>
> To fix the bugs in these examples, you must add a variable in the scope of the outer loop to hold the outer element:

这个程序不会抛出异常，但是只会打印6对（从“ONE ONE”到“SIX SIX"），而不是像期待的那样打印36对。

为了解决这个问题，你必须在外围循环中添加一个变量来保存外围的元素。代码如下：

```java
// Fixed, but ugly - you can do better!
   for (Iterator<Suit> i = suits.iterator(); i.hasNext(); ) {
       Suit suit = i.next();
       for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
						deck.add(new Card(suit, j.next()));
   }
```

> If instead you use a nested for-each loop, the problem simply disappears. The resulting code is as succinct as you could wish for:

如果你使用嵌套的for-each循环，这个问题就自然消失了。得到的代码也正如你所期望的一样简洁明了。代码如下：

```java
// Preferred idiom for nested iteration on collections and arrays
   for (Suit suit : suits)
       for (Rank rank : ranks)
           deck.add(new Card(suit, rank));
```

> Unfortunately, there are three common situations where you *can’ t* use for-each:
>
> • **Destructive filtering**—If you need to traverse a collection removing selected elements, then you need to use an explicit iterator so that you can call its remove method. You can often avoid explicit traversal by using Collection’ s removeIf method, added in Java 8.
>
> • **Transforming**—If you need to traverse a list or array and replace some or all of the values of its elements, then you need the list iterator or array index in order to replace the value of an element.
>
> • **Parallel iteration**—If you need to traverse multiple collections in parallel, then you need explicit control over the iterator or index variable so that all iterators or index variables can be advanced in lockstep (as demonstrated unintentionally in the buggy card and dice examples above).

遗憾的是，下面有三种情况，不能使用for-each：

- **破坏性的过滤**——如果你需要遍历一个集合，然后删除其中选定的元素，你就必须要使用一个显示的iterator，因此你可以调用它的remove方法。你还可以使用java8里新增的Collection的removeIf方法，来避免显示的遍历。
- **转换**——如果你需要遍历一个列表或者数组，并且替换掉其部分或者全部元素的值，你就必须使用列表iterator或者数组索引，以便替换元素的值。
- **并行迭代**——如果你需要并行地遍历几个集合，你就需要显示地控制迭代器或者索引变量，以便所有的迭代器或者索引变量可以同步（就像前面展示的有问题的牌和骰子例子里，示范的那样）

> If you find yourself in any of these situations, use an ordinary for loop and be wary of the traps mentioned in this item.
>
> Not only does the for-each loop let you iterate over collections and arrays, it lets you iterate over any object that implements the Iterable interface, which consists of a single method. Here is how the interface looks:

如果你发现你自己处在以上这些情况中，就需要使用传统的循环，并且警惕本节中介绍过的陷阱。

for-each循环不仅仅可以然你迭代集合和数组，任何实现了Iterable接口的对象，for-each循环都可以进行迭代。Iterable接口只包含一个方法，下面是这个接口:

```java
public interface Iterable<E> {
       // Returns an iterator over the elements in this iterable
       Iterator<E> iterator();
}
```

> It is a bit tricky to implement Iterable if you have to write your own Iterator implementation from scratch, but if you are writing a type that represents a group of elements, you should strongly consider having it implement Iterable, even if you choose not to have it implement Collection. This will allow your users to iterate over your type using the for-each loop, and they will be forever grateful.

如果你必须从头实现自己的Iterator的话，要实现Iterable还是有点困难的。但是如果你写一个表示一组元素的类型是，你应该好好考虑一下，让它实现Iterable接口，即使你选择不实现集合接口。这样可以允许你的用户对你的类型使用for-each循环进行迭代，他们会非常感谢你的。

> In summary, the for-each loop provides compelling advantages over the traditional for loop in clarity, flexibility, and bug prevention, with no performance penalty. Use for-each loops in preference to for loops wherever you can.

总结一下，for-each循环相对于传统的for循环，在简洁性、灵活性、预防出错方面都有很大的优势，还没有性能问题。在你可以使用for-each循环的时候，都优先使用for-each循环。

