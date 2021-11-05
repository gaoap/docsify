# effectivejava学习笔记（一）

## 第一条：用静态工厂方法代替构造器

优点：

1. 静态工厂有名称，方便通过名称理解。预防语义理解上的错误。
2. 不必每次调用时都新建一个新对象。
3. 可以返回原类型，任意子类型的对象。
4. 返回对象可以随着每次调用而发生变化。这个取决于静态工厂方法的参数值。
5. 静态工厂方法返回的对象所属类，可以在编写包含该静态工厂的类时不存在。

缺点:

1. 类如果不含有公有的或者受保护的构造器，就不能被子类化（不能被继承）。
2. 在API文档中，没有像构造器一样明显的标注出来。比较难发现静态工厂方法。

一般的静态工厂方法名称：一般惯例如下（并非绝对）：

1. from : 类型转换
2. of：聚合方法
3. valueOf: 
4. instance 、 getInstance、create、newInstance、getType 、newType 、type



## 第二条：遇到多个构造器参数时要考虑使用建造者模式

1. 避免使用重叠构造器，当参数过多时，难于理解。
2. 避免使用JavaBean模式，缺点：处于不一致状态、把类做成不可变的可能性不复存在。

## 第三条：用私有构造器或者枚举类型强化Singleton属性

使用构造器私有的方式，构建出的Singleton都存在风险：

1. 反序列化导致出现多实例：需要覆写readResolve()方法，来保证反序列化的实例唯一情况。
2. 反射出现多个实例：AccessibleObject上的setAccessible()方法可以控制访问私有构造器。
3. 单元素枚举类型经常成为实现Singleton的最佳方法。但是如果Singleton必须扩展一个超类，而不是扩展Enum的时候，则不宜使用这个方法。

## 第四条：通过私有构造器强化不可实例化的能力



## 第五条：优先考虑依赖注入来引用资源

当创建一个新的实例时，就将该资源传到构造器中。

依赖注入的框架：Dagger、Guice、Spring

## 第六条：避免创建不必要的对象

1、千万不要：String s=new String("hello");

2、可以这样：string s="hello"; //此方式可以保证包含“hello”的字符串常量被重用。

正则的使用：

```java
    // 不会复用Pattern
    static boolean isRomanNumeralSlow(String s) {
        return s.matches("^(?=.)M*(C[MD]|D?C{0,3})"
                + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
    }

    //复用Pattern
    private static final Pattern ROMAN = Pattern.compile(
            "^(?=.)M*(C[MD]|D?C{0,3})"
                    + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

    static boolean isRomanNumeralFast(String s) {
        return ROMAN.matcher(s).matches();
    }
```

3、优先使用基本类型，而不是装箱类型。

```java
private static long sum() {
        Long sum = 0L; //这个地方使用了装箱类型。导致每一次循环，都要新生成一个Long对象
        for (long i = 0; i <= Integer.MAX_VALUE; i++)
            sum += i;
        return sum;
    }
```

## 第七条：消除过期的对象引用

1、只要内存是类自己来管理的，就要小心内存泄漏：如下示例

```java
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
        //elements[size] = null; // 清空过期引用。这个注释不能去，否则会导致内存溢出的风险
        return result;
    }

    /**
     *  这个地方存在内存泄漏，一个栈先增长，在收缩。那么栈中弹出"elements[--size]"的类，永远不会被回收
     *  这些类被栈维护者过期引用。
     */
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
```

2、内存泄漏常见来源是缓存。

对于缓存的数据，可以考虑WeakHashMap。

WeakHashMap补充知识：WeakHashMap 继承于AbstractMap，实现了Map接口。 和HashMap一样，WeakHashMap 也是一个散列表，它存储的内容也是键值对(key-value)映射，而且键和值都可以是null。 不过WeakHashMap的键是“弱键”。在 WeakHashMap 中，当某个键不再正常使用时，会被从WeakHashMap中被自动移除。这个“弱键”的原理，大致上就是，通过WeakReference和ReferenceQueue实现的。 WeakHashMap的key是“弱键”，即是WeakReference类型的；ReferenceQueue是一个队列，它会保存被GC回收的“弱键”。实现步骤是：
   a、新建WeakHashMap，将“键值对”添加到WeakHashMap中。
      实际上，WeakHashMap是通过数组table保存Entry(键值对)；每一个Entry实际上是一个单向链表，即Entry是键值对链表。
   b 、当某“弱键”不再被其它对象引用，并被GC回收时。在GC回收该“弱键”时，这个“弱键”也同时会被添加到ReferenceQueue(queue)队列中。
   c、当下一次我们需要操作WeakHashMap时，会先同步table和queue。table中保存了全部的键值对，而queue中保存被GC回收的键值对；同步它们，就是删除table中被GC回收的键值对。

WeakHashMap是不同步的。可以使用 Collections.synchronizedMap 方法来构造同步的 WeakHashMap

补充：HashMap、Hashtable、WeakHashMap比较

Map是用于保存具有映射关系的数据（key-vlaue）。Map供给key到value的映射，Map的key不允许重复，每个key只能映射一个value。即同一个Map对象的任何两个key通过equals方法比较总是返回false，Map中包含了一个keySet()方法，用于返回Map所以key组成的Set集合。笼统地说Map接口其实可以说包含了三个集合，一组key-value集合、一组key集合以及value集合。 Map是键到值的映射，Java集合框架在实现上采用一个个Map.Entry内部类来封装每一个键值对，这样，Map中的元素就变成了Map.Entry的集合。

**HashMap类**
HashMap允许键（key）和值（value）为空（null），null可以作为键，这样的键只有一个，但可以有一个或多个键所对应的值为null，这里有个需要注意的是，但我们用get()方法返回null值时，即可以表示HashMap中没有该键，也可以表示该键所对应的值为null。因此，在HashMap中不能由get()方法来判断HashMap中是否存在某个键，而应该用containsKey()方法来判断。HashMap是非synchronized（非线程安全的实现），如果涉及到多个线程，那就不应该考虑使用HashMap。但是Java 5提供了ConcurrentHashMap，它比HashTable的扩展性更好、性能更高。HashMap可以通过下面的语句进行同步：Map map = Collections.synchronizeMap(hashMap);

**Hashtable类**
HashTable不允许有null的键（key）和值（value），HashTable是synchronized（线程安全实现），这意味着HashTable是线程安全的，多个线程可以共享一个HashTable。上面提到ConcurrentHashMap到底和Hashtable有啥不一样，笔者也查阅下相关知识，主要Hashtable的实现方式是锁整个hash表，而ConcurrentHashMap分段式封锁，分层多个节点进行关键部位加锁，避免大锁，允许多个修改操作并发进行。具体可以了解Java集合—ConcurrentHashMap原理分析

**WeakHashMap类**
WeakHashMap是一种改进的HashMap，它对key实验“弱引用”，若是一个key不再被外部所引用，那么该key可以被GC收受接管。

3、内存泄漏的第三个常见来源是监听器和其他回调。 

4、可以利用Heap工具（如：JProfiler）分析内存泄漏

## 第八条：避免使用终结方法（finalize）和清除方法（java9 Cleaner）

1、注重时间的任务不应该由终结方法或者清除方法来完成。

2、为了防止非final类受到终结方法攻击，要编写一个空的final的finalize方法。

3、可以考虑类实现AutoCloseable

使用AutoCloseable的一个示例：

```java
public class Room implements AutoCloseable {
    //创建一个清除处理
    private static final Cleaner cleaner = Cleaner.create();
    //需要清除或者释放的资源
    private static class State implements Runnable {
        int numJunkPiles; // 房间的脏乱程度
        State(int numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }
        //执行清除的时候执行的是此操作
        @Override public void run() {
            System.out.println("Cleaning room");
            numJunkPiles = 0;
        }
    }
    private final State state;
    private final Cleaner.Cleanable cleanable;
    public Room(int numJunkPiles) {
        state = new State(numJunkPiles);
        //注册使用的对象
        cleanable = cleaner.register(this, state);
    }

    @Override public void close() {
        cleanable.clean();
    }
}
//客户端调用
public class Adult {
    public static void main(String[] args) {
        try (Room myRoom = new Room(7)) {
           //执行业务代码
            System.out.println("Goodbye");
        }
    }
}
```

## 第九条：try-with-resources优先于try-finally

必须先实现：AutoCloseable 才能使用try-with-resources。

try-with-resources也可以使用catch子句：

```java
static String firstLineOfFile(String path, String defaultVal) {
        try (BufferedReader br = new BufferedReader(
                new FileReader(path))) {
            return br.readLine();
        } catch (IOException e) {
            return defaultVal;
        }
    }
```

