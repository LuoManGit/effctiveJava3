### Item29 优先考虑泛型

> It is generally not too difficult to parameterize your declarations and make use of the generic types and methods provided by the JDK. Writing your own generic types is a bit more difficult, but it’s worth the effort to learn how.
>
> Consider the simple (toy) stack implementation from Item 7:

通常来说，将声明参数化并使用JDK里的泛型类型和方法都不是很难。而要写一个自己的泛型类型却有点难，但是花时间去学习如何写自己泛型类型是很值得的。

看看Item7里面的简单的堆栈实现：

```java
// Object-based collection - a prime candidate for generics
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
				Object result = elements[--size];
				elements[size] = null; // Eliminate obsolete reference return result;
		}
    public boolean isEmpty() {
        return size == 0;
		}
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
		} 
}
```

> This class should have been parameterized to begin with, but since it wasn’t, we can *generify* it after the fact. In other words, we can parameterize it without harm- ing clients of the original non-parameterized version. As it stands, the client has to cast objects that are popped off the stack, and those casts might fail at runtime. The first step in generifying a class is to add one or more type parameters to its declaration. In this case there is one type parameter, representing the element type of the stack, and the conventional name for this type parameter is E (Item 68).
>
> The next step is to replace all the uses of the type Object with the appropriate type parameter and then try to compile the resulting program:









​	



















