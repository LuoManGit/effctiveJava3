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

> This class should have been parameterized to begin with, but since it wasn’t, we can *generify* it after the fact. In other words, we can parameterize it without harming clients of the original non-parameterized version. As it stands, the client has to cast objects that are popped off the stack, and those casts might fail at runtime. The first step in generifying a class is to add one or more type parameters to its declaration. In this case there is one type parameter, representing the element type of the stack, and the conventional name for this type parameter is E (Item 68).
>
> The next step is to replace all the uses of the type Object with the appropriate type parameter and then try to compile the resulting program:

这个类应该一开始就被参数化，但是它没有，在后面将它称为泛型化。换句话说，就是我们现在要将其参数化，并且还不能影响之前的非参数化版本的客户端。按照之前的情况，客户端必须要转换从栈里弹出来的对象的类型，这些转换在运行时还可能失败。泛型化的第一步就是在其声明中添加一个或者多个类型参数。在这个例子里，只有一个类型参数，表示栈内元素的类型，这个类型参数通常的名字是E（Item68）。

第二步就是把所有使用Object类型的地方都换成合适的类型参数,如下，然后尝试编译得到的结果：

```java
// Initial attempt to generify Stack - won't compile! 
public class Stack<E> {
		private E[] elements;
  	private int size = 0;
		private static final int DEFAULT_INITIAL_CAPACITY = 16;
		public Stack() {
				elements = new E[DEFAULT_INITIAL_CAPACITY];
		}
		public void push(E e) { 
      	ensureCapacity(); elements[size++] = e;
		}
		public E pop() { 
      	if (size == 0)
						throw new EmptyStackException();
				E result = elements[--size];
				elements[size] = null; // Eliminate obsolete reference return result;
		}
       ... // no changes in isEmpty or ensureCapacity
}
```

> You’ll generally get at least one error or warning, and this class is no exception. Luckily, this class generates only one error:

通常情况下，不出意外的话，你将得到至少一个error或者warning。幸运地是，这个类只生成了一个error，如下：

```java
Stack.java:8: generic array creation
								elements = new E[DEFAULT_INITIAL_CAPACITY];
														^
```

> As explained in Item 28, you can’t create an array of a non-reifiable type, such as E. This problem arises every time you write a generic type that is backed by an array. There are two reasonable ways to solve it. The first solution directly circumvents the prohibition on generic array creation: create an array of Object and cast it to the generic array type. Now in place of an error, the compiler will emit a warning. This usage is legal, but it’s not (in general) typesafe:

正如Item28里所讲的那样，你不能创建一个不能具体化的类型的数组，比如E。这个问题在你每次编写数组支持的泛型的时候就会出现。针对这个问题，有两种合理的解决方案。第一个就是直接避开泛型数组创建的禁令“创建一个Object数组，然后把它转换成泛型数组。这样做的话，编译器就会生成一个warning而不是error了。这种做法通常来说是类型安全的，但却不是类型安全的。

```java
Stack.java:8: warning: [unchecked] unchecked cast
   found: Object[], required: E[]
				elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY]; 
										^
```

> The compiler may not be able to prove that your program is typesafe, but you can. You must convince yourself that the unchecked cast will not compromise the type safety of the program. The array in question (elements) is stored in a private field and never returned to the client or passed to any other method. The only elements stored in the array are those passed to the push method, which are of type E, so the unchecked cast can do no harm.
>
> Once you’ve proved that an unchecked cast is safe, suppress the warning in as narrow a scope as possible (Item 27). In this case, the constructor contains only the unchecked array creation, so it’s appropriate to suppress the warning in the entire constructor. With the addition of an annotation to do this, Stack compiles cleanly, and you can use it without explicit casts or fear of a ClassCastException:

编译器没办法证明你的程序是类型安全的，但是你却可以。你必须要确信这个非受检的转换不会危害到程序的类型安全。这个问题里的数组（也就是elements）保存在私有域中，也永远都不会返回给客户端，也不会传递给任何其他的方法，唯一往数组里存储的元素是传递给push方法的，它的类型也是E。因此这个非受检的转换不会造成任何危害。

一旦你证明了一个非受检转换是安全的，就可以在尽可能小的作用域范围内（Item27）禁止这个警告。在这个例子中，构造器只包含非受检数组创建，因此，可以在整个构造器上禁止warning。添加了这个注解以后，如下，这个Stack就可以干干净净的编译了，你可以不再需要显式的转换，直接使用了，也不用担心ClassCastException。

```java
// The elements array will contain only E instances from push(E). 
// This is sufficient to ensure type safety, but the runtime
// type of the array won't be E[]; it will always be Object[]! @SuppressWarnings("unchecked")
public Stack() {
       elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
}
```

> The second way to eliminate the generic array creation error in Stack is to change the type of the field elements from E[] to Object[]. If you do this, you’ll get a different error:

消除泛型数组创建error的第二个方法就是把elements域的类型从E[]改为Object[]。如果你这个做了的话，你将得到另外一个不同的error，如下：

```java
Stack.java:19: incompatible types
   found: Object, required: E
           E result = elements[--size];
														^
```

> You can change this error into a warning by casting the element retrieved from the array to E, but you will get a warning:

你可以通过把从数组中获得的元素转换为E，把这个error改为warning。如果你这样做的话，得到waring如下：

```java
Stack.java:19: warning: [unchecked] unchecked cast
   	found: Object, required: E
		E result = (E) elements[--size]; 
								^
```

> Because E is a non-reifiable type, there’s no way the compiler can check the cast at runtime. Again, you can easily prove to yourself that the unchecked cast is safe, so it’s appropriate to suppress the warning. In line with the advice of Item 27, we suppress the warning only on the assignment that contains the unchecked cast, not on the entire pop method:

因为E是一个不可具体化的类型，编译器在运行时不能检验这个转换。同样地，你还是可以很容易证明这个非受检转换是安全的，因此按照Item27中的建议，禁止这个warning是合适的。我们在包含这个非受检的转换上禁止这个warning，而不是在整个pop方法上，如下：

```java
// Appropriate suppression of unchecked warning
   public E pop() {
       if (size == 0)
           throw new EmptyStackException();
			// push requires elements to be of type E, so cast is correct 
			 @SuppressWarnings("unchecked") E result = (E) elements[--size];
       elements[size] = null; // Eliminate obsolete reference
       return result;
   }
```

> Both techniques for eliminating the generic array creation have their adherents. The first is more readable: the array is declared to be of type E[], clearly indicating that it contains only E instances. It is also more concise: in a typical generic class, you read from the array at many points in the code; the first technique requires only a single cast (where the array is created), while the second requires a separate cast each time an array element is read. Thus, the first technique is preferable and more commonly used in practice. It does, however, cause *heap pollution* (Item 32): the runtime type of the array does not match its compile-time type (unless E happens to be Object). This makes some programmers sufficiently queasy that they opt for the second technique, though the heap pollution is harmless in this situation.
>
> The following program demonstrates the use of our generic Stack class. The program prints its command line arguments in reverse order and converted to uppercase. No explicit cast is necessary to invoke String’s toUpperCase method on the elements popped from the stack, and the automatically generated cast is guaranteed to succeed:

这两种避免泛型数组创建的方法，都有其拥护者。第一种方法可读性更强：数组被声明为E[]，清晰地表示了它只包含E实例；并且更加简洁：在一个典型的泛型类中，你可以以在代码里看到很多次数组，第一中方法只需要进行一次转换（当数组创建的时候）；而第二种方法需要在每次读取数组元素的时候都进行转换。因此第一种方法要更受欢迎一些，在实际情况中，也用得更多些。但是第一种方法确实造成了”堆泄露“（详见Item32）：因为数组的运行时类型和编译时类型不一致（除非E刚刚也是Object）。有一些程序员对这点深恶痛绝，选择使用第二种方法，虽然在这个例子中，堆泄露的危害很小。

下面这个程序表示了我们的泛型Stack类的用法。这个成语把自己的命令行参数逆序，并转换为大写打印出来。在从stack中弹出来的每个元素上调用Stirng的toUpperCase方法的时候，不需要进行显式的类型转换，并且这个自动转换是保证会成功的：

```java
// Little program to exercise our generic Stack
   public static void main(String[] args) {
       Stack<String> stack = new Stack<>();
       for (String arg : args)
           stack.push(arg);
       while (!stack.isEmpty())
					System.out.println(stack.pop().toUpperCase());
   }
```

> The foregoing example may appear to contradict Item 28, which encourages the use of lists in preference to arrays. It is not always possible or desirable to use lists inside your generic types. Java doesn’t support lists natively, so some generic types, such as ArrayList, *must* be implemented atop arrays. Other generic types, such as HashMap, are implemented atop arrays for performance.

上面这个例子好像违反了Item28的列表优先于数组这一规则。但是在泛型中使用数组不可能总是明智的，因为java不是生来就支持列表的，一些泛型，比如ArrayList，也必须要基于数组来实现。一些其他的泛型，比如HashMap为了更好的性能，也是基于数组实现的。

> The great majority of generic types are like our Stack example in that their type parameters have no restrictions: you can create a Stack<Object>, Stack<int[]>, Stack<List<String>>, or Stack of any other object reference type. Note that you can’t create a Stack of a primitive type: trying to create a Stack<int> or Stack<double> will result in a compile-time error. This is a fundamental limitation of Java’s generic type system. You can work around this restriction by using boxed primitive types (Item 61).
>
> There are some generic types that restrict the permissible values of their type parameters. For example, consider java.util.concurrent.DelayQueue, whose declaration looks like this:

绝大多数的泛型都和我们例子中的Stack一样，其类型参数没有什么限制。你可以创建Stack<Object>，Stack<int[]>，Stack<List<String>>，或者 任何对象引用类型的Stack。需要注意的是，你不能创建一个基本类型的堆栈：试图创建Stack<int> 或者 Stack<double> 都会带来编译期的error。这个Java的泛型系统的基本限制，你可以通过使用基本类型的封装类来绕开这个限制（Item61）。

还有一些泛型，限制了其类型参数许可的值。比如， java.util.concurrent.DelayQueue，它的声明如下：

```java
class DelayQueue<E extends Delayed> implements BlockingQueue<E>
```

> The type parameter list (<E extends Delayed>) requires that the actual type parameter E be a subtype of java.util.concurrent.Delayed. This allows the DelayQueue implementation and its clients to take advantage of Delayed methods on the elements of a DelayQueue, without the need for explicit casting or the risk of a ClassCastException. The type parameter E is known as a *bounded type parameter*. Note that the subtype relation is defined so that every type is a subtype of itself [JLS, 4.10], so it is legal to create a DelayQueue<Delayed>.

类型参数列表(<E extends Delayed>)需要E的真实类型参数是java.util.concurrent.Delayed.的子类型。这样使得 DelayQueue的实现和其客户端可以利用DelayQueue中元素上的Delayed方法，又不需要进行明确的类型转换，也不用冒着出现ClassCastException的风险。这个类型参数E被称为”有边界的类型参数“。需要注意的是，根据子类型关系的定义，每一个类型都是其自己的子类型，因此创建一个DelayQueue<Delayed>也是合法的。

> In summary, generic types are safer and easier to use than types that require casts in client code. When you design new types, make sure that they can be used without such casts. This will often mean making the types generic. If you have any existing types that should be generic but aren’t, generify them. This will make life easier for new users of these types without breaking existing clients (Item 26).

总的来说，相对于需要客户端写代码进行类型转换的类型，泛型更加安全，也更加容易使用。当你在设计一个新的类型的时候，确保他们可以不需要这类转换，可以直接使用。这通常意味着需要将类型泛型化。如果你已经存在一些应该被泛型化，但又没有泛型化的类型，应该把它们都泛型化。因为这样会使得使用这些类型的新手更轻松，也不会破坏已经存在的客户端。(Item26)

