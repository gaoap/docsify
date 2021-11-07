# Effective Java学习笔记（三）：类和接口

## 第一条：使类和成员的可访问性最小化

尽可能地使每个类或者成员不被外界访问。

确保公有静态final域所引用的对象都不可变。

## 第二条：要在公有类而非公有域中使用访问方法

## 第三条：是可变性最小化

函数式方法，命名建议用介词（如：plus）。过程或者命令式的建议使用动词(如：add)。  

不可变对象本质上是线程安全的，它们不要求同步。

除非有令人信服的理由要使域变成是非 final的，否则要使每个域都是private final的。

## 第四条：复合优先于继承（一个类扩展另一个类的时候）

可以参考修饰者(Decorator)模式。

Guava为所有的集合接口提供了转发类。

违反原则的类： stack 并不是 vector 

Properties不应该扩展Hashtable

错误的扩展示范：

```java
public class InstrumentedHashSet<E> extends HashSet<E> {
    // The number of attempted element insertions
    private int addCount = 0;

    public InstrumentedHashSet() {
    }

    public InstrumentedHashSet(int initCap, float loadFactor) {
        super(initCap, loadFactor);
    }

    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }

    public static void main(String[] args) {
        InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
        s.addAll(List.of("Snap", "Crackle", "Pop"));
        System.out.println(s.getAddCount());
    }
}
```

利用复合转发优化：

```java
转发类：
public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;
    public ForwardingSet(Set<E> s) { this.s = s; }

    public void clear()               { s.clear();            }
    public boolean contains(Object o) { return s.contains(o); }
    public boolean isEmpty()          { return s.isEmpty();   }
    public int size()                 { return s.size();      }
    public Iterator<E> iterator()     { return s.iterator();  }
    public boolean add(E e)           { return s.add(e);      }
    public boolean remove(Object o)   { return s.remove(o);   }
    public boolean containsAll(Collection<?> c)
                                   { return s.containsAll(c); }
    public boolean addAll(Collection<? extends E> c)
                                   { return s.addAll(c);      }
    public boolean removeAll(Collection<?> c)
                                   { return s.removeAll(c);   }
    public boolean retainAll(Collection<?> c)
                                   { return s.retainAll(c);   }
    public Object[] toArray()          { return s.toArray();  }
    public <T> T[] toArray(T[] a)      { return s.toArray(a); }
    @Override public boolean equals(Object o)
                                       { return s.equals(o);  }
    @Override public int hashCode()    { return s.hashCode(); }
    @Override public String toString() { return s.toString(); }
}
复合类：
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;

    public InstrumentedSet(Set<E> s) {
        super(s);
    }

    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }
    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
    public int getAddCount() {
        return addCount;
    }

    public static void main(String[] args) {
        InstrumentedSet<String> s = new InstrumentedSet<>(new HashSet<>());
        s.addAll(List.of("Snap", "Crackle", "Pop"));
        System.out.println(s.getAddCount());
    }
}

```



## 第五条：要么设计继承并提供文档说明，要么禁止继承

注意@implSpec的使用。传入命令行参数: `-tag "apiNote:a:APINote:"`

超类的构造器，在子类的构造器之前运行。

通过构造器调用私有的方法、final方法和静态方法是安全的，这些类都是不可以被覆盖的。

## 第六条：接口优于抽象类

接口是定义混合类的理想选择。

接口的静态方法和默认方法示例：

```java
public interface TestInterface {
    //抽象方法
    void method();
    //接口默认实现方法
    default void defaultMethod(){
        System.out.println("这是接口的默认方法！");
    }
    //接口静态实现方法
    static void staticMethod(){
        System.out.println("这是接口的静态方法！");
    }
}

public class InterfaceImpl implements  TestInterface{
    @Override
    public void method() {
        System.out.println("实现类实现的接口方法！");
    }
    //实现类可以覆盖接口的默认方法
//    @Override
//    public void defaultMethod() {
//        System.out.println("实现类可以覆盖接口的默认方法！");
//    }
    public static void main(String[] args){
        InterfaceImpl impl=new InterfaceImpl();
        impl.method();
        impl.defaultMethod();
        TestInterface.staticMethod();
    }
}
```

骨架实现类的一个示例应用：

```java
public class IntArrays {
    static List<Integer> intArrayAsList(int[] a) {
        Objects.requireNonNull(a);

        // The diamond operator is only legal here in Java 9 and later
        // If you're using an earlier release, specify <Integer>
        return new AbstractList<>() {
            @Override public Integer get(int i) {
                return a[i];  // Autoboxing (Item 6)
            }

            @Override public Integer set(int i, Integer val) {
                int oldVal = a[i];
                a[i] = val;     // Auto-unboxing
                return oldVal;  // Autoboxing
            }

            @Override public int size() {
                return a.length;
            }
        };
    }

    public static void main(String[] args) {
        int[] a = new int[10];
        for (int i = 0; i < a.length; i++)
            a[i] = i;

        List<Integer> list = intArrayAsList(a);
        Collections.shuffle(list);
        System.out.println(list);
    }
}
```

## 第七条：为后代设计接口

谨慎使用接口的默认方法。

## 第八条：接口只用于定义类型

常量接口不是一个值得推荐的使用方式。

补充知识：java 7开始支持，字面量中使用"_"。如下代码所示

```java
  public static void main(String[] args){
    final double AVOGADROS_NUMBER = 6.022_140_857e23;
    int int_ss=1234_1000;
    System.out.println(AVOGADROS_NUMBER==6.022140857e23); //true
    System.out.println(int_ss==12341000);  //true
  }
```



## 第九条：类层次优于标签类

错误示范：

```java
class Figure {
    enum Shape { RECTANGLE, CIRCLE };

    // Tag field - the shape of this figure
    final Shape shape;

    // These fields are used only if shape is RECTANGLE
    double length;
    double width;

    // This field is used only if shape is CIRCLE
    double radius;

    // Constructor for circle
    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    // Constructor for rectangle
    Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    double area() {
        switch(shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
```

改良后示范：

```java
abstract class Figure {
    abstract double area();
}

class Circle extends Figure {
    final double radius;

    Circle(double radius) { this.radius = radius; }

    @Override double area() { return Math.PI * (radius * radius); }
}

class Rectangle extends Figure {
    final double length;
    final double width;

    Rectangle(double length, double width) {
        this.length = length;
        this.width  = width;
    }
    @Override double area() { return length * width; }
}

class Square extends Rectangle {
    Square(double side) {
        super(side, side);
    }
}
```

## 第十条：静态成员类优于非静态成员类

如果声明成员类不要求访问外围实例，就要始终吧修饰符static放在它的声明中。

## 第十一条：限制源文件为单个顶级类

如果有必要，可以用静态成员的方式解决。

如：

```java
public class Test {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }

    private static class Utensil {
        static final String NAME = "pan";
    }

    private static class Dessert {
        static final String NAME = "cake";
    }
}
```

