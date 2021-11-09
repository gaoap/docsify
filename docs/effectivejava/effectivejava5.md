# Effective Java学习笔记（五）：枚举和注解

## 1、用enum代替int常量

每当需要一组固定常量，并且在编译时就知道其成员的时候，就应该使用枚举。

enum内部避免使用 switch语句.会在enum添加或者删除新成员时带来不必要的麻烦。

这是一种避免switch的写法。这也是策略枚举的一种实现。

```java
enum PayrollDay {
    MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY),
    THURSDAY(WEEKDAY), FRIDAY(WEEKDAY),
    SATURDAY(WEEKEND), SUNDAY(WEEKEND);

    private final PayType payType;
     /**
     * 构造函数，添加支付计算方式，避免通过switch .. case方式判断。
     * 也强制，再添加新的枚举成员时，必须选择支付计算方式。提高了安全性
     */
    PayrollDay(PayType payType) { this.payType = payType; }

    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }

    // The strategy enum type
    enum PayType {
        WEEKDAY {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked <= MINS_PER_SHIFT ? 0 :
                        (minsWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked * payRate / 2;
            }
        };

        abstract int overtimePay(int mins, int payRate);//抽象方法，没有枚举成员，必须强制实现。否则编译器报错。
        private static final int MINS_PER_SHIFT = 8 * 60;

        int pay(int minsWorked, int payRate) {
            int basePay = minsWorked * payRate;
            return basePay + overtimePay(minsWorked, payRate);
        }
    }

    public static void main(String[] args) {
        for (PayrollDay day : values())
            System.out.printf("%-10s%d%n", day, day.pay(8 * 60, 1));
    }
}
```

## 2、用实例域代替序数

如果要使用序数，那就自己保存在实例域中。

```java
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
    SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8),
    NONET(9), DECTET(10), TRIPLE_QUARTET(12);

    private final int numberOfMusicians;
    Ensemble(int size) { this.numberOfMusicians = size; }
    public int numberOfMusicians() { return numberOfMusicians; }

```

## 3、用EnumSet代替位域

```java
public class Text {
    public enum Style {BOLD, ITALIC, UNDERLINE, STRIKETHROUGH}

    // Any Set could be passed in, but EnumSet is clearly best
    public void applyStyles(Set<Style> styles) {
        System.out.printf("Applying styles %s to text%n",
                Objects.requireNonNull(styles));
    }

    // Sample use
    public static void main(String[] args) {
        Text text = new Text();
        text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
    }
}
```

## 4、用EnumMap代替序数

```java
class Plant {
    enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }

    final String name;
    final LifeCycle lifeCycle;

    Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }

    @Override public String toString() {
        return name;
    }

    public static void main(String[] args) {
        Plant[] garden = {
            new Plant("Basil",    LifeCycle.ANNUAL),
            new Plant("Carroway", LifeCycle.BIENNIAL),
            new Plant("Dill",     LifeCycle.ANNUAL),
            new Plant("Lavendar", LifeCycle.PERENNIAL),
            new Plant("Parsley",  LifeCycle.BIENNIAL),
            new Plant("Rosemary", LifeCycle.PERENNIAL)
        };

        //千万不要使用ordinal() 去索引数据  ，这是个错误的示范
        Set<Plant>[] plantsByLifeCycleArr =
                (Set<Plant>[]) new Set[LifeCycle.values().length];
        for (int i = 0; i < plantsByLifeCycleArr.length; i++)
            plantsByLifeCycleArr[i] = new HashSet<>();
        for (Plant p : garden)
            plantsByLifeCycleArr[p.lifeCycle.ordinal()].add(p);
        // Print the results
        for (int i = 0; i < plantsByLifeCycleArr.length; i++) {
            System.out.printf("%s: %s%n",
                    LifeCycle.values()[i], plantsByLifeCycleArr[i]);
        }
        //使用EnumMap去索引数据
        Map<LifeCycle, Set<Plant>> plantsByLifeCycle =
                new EnumMap<>(LifeCycle.class);
        for (LifeCycle lc : LifeCycle.values())
            plantsByLifeCycle.put(lc, new HashSet<>());
        for (Plant p : garden)
            plantsByLifeCycle.get(p.lifeCycle).add(p);
        System.out.println(plantsByLifeCycle);

        //这个调用不会使用 EnumMap!  ，不建议使用
        System.out.println(Arrays.stream(garden)
                .collect(groupingBy(p -> p.lifeCycle)));

        //  这个调用会使用 EnumMap!  ，建议使用
        System.out.println(Arrays.stream(garden)
                .collect(groupingBy(p -> p.lifeCycle,
                        () -> new EnumMap<>(LifeCycle.class), toSet())));
    }
}
```

示例二：

```java
public enum Phase {
    SOLID, LIQUID, GAS;//固体、液体、气体

    public enum Transition {
        /**
         * 熔化（固体，液体），冷冻（液体，固体），
         * <p>
         * 沸腾（液体，气体），冷凝（气体，液体），
         * <p>
         * 升华（固体，气体），沉积（气体，固体）；
         */
        MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);

//        //新增加一种 Phase
//        SOLID, LIQUID, GAS, PLASMA; //离子
//        public enum Transition {
//            MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
//            BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
//            SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID),
//        电离（气体，等离子体），去离子（等离子体，气体）；
//            IONIZE(GAS, PLASMA), DEIONIZE(PLASMA, GAS);

        private final Phase from;
        private final Phase to;

        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }

        // Initialize the phase transition map
        private static final Map<Phase, Map<Phase, Transition>>
                /**
                 * 使用toMap()函数之后，返回的就是一个Map了，自然会需要key和value。
                 * toMap()的第一个参数就是用来生成key值的，第二个参数就是用来生成value值的。
                 * 第三个参数用在key值冲突的情况下：如果新元素产生的key在Map中已经出现过了，第三个参数就会定义解决的办法。
                 *
                 * 在你的例子中
                 *  .collect(Collectors.toMap(UserBo::getUserId, v -> v, (v1, v2) -> v1));
                 * 第一个参数UserBo::getUserId 表示选择UserBo的getUserId作为map的key值；
                 * 第二个参数v -> v表示选择将原来的对象作为map的value值；
                 * 第三个参数(v1, v2) -> v1中，如果v1与v2的key值相同，选择v1作为那个key所对应的value值
                 */
                m = Stream.of(values()).collect(groupingBy(t -> t.from,
                () -> new EnumMap<>(Phase.class),
                toMap(t -> t.to, t -> t,
                        (x, y) -> y, () -> new EnumMap<>(Phase.class))));

        public static Transition from(Phase from, Phase to) {
            return m.get(from).get(to);
        }
    }

    // Simple demo program - prints a sloppy table
    public static void main(String[] args) {
        for (Phase src : Phase.values()) {
            for (Phase dst : Phase.values()) {
                Transition transition = Transition.from(src, dst);
                if (transition != null)
                    System.out.printf("%s to %s : %s %n", src, dst, transition);
            }
        }
    }
}
```

## 5、用接口模拟可扩展的枚举

```java
接口：
public interface Operation {
    double apply(double x, double y);
}
基础实现：
public enum BasicOperation implements Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };

    private final String symbol;

    BasicOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override public String toString() {
        return symbol;
    }
}
扩展实现：
public enum ExtendedOperation implements Operation {
    EXP("^") {
        public double apply(double x, double y) {
            return Math.pow(x, y);
        }
    },
    REMAINDER("%") {
        public double apply(double x, double y) {
            return x % y;
        }
    };
    private final String symbol;
    ExtendedOperation(String symbol) {
        this.symbol = symbol;
    }
    @Override public String toString() {
        return symbol;
    }

//    // Using an enum class object to represent a collection of extended enums (page 178)
//    public static void main(String[] args) {
//        double x = Double.parseDouble(args[0]);
//        double y = Double.parseDouble(args[1]);
//        test(ExtendedOperation.class, x, y);
//    }
//    private static <T extends Enum<T> & Operation> void test(
//            Class<T> opEnumType, double x, double y) {
//        for (Operation op : opEnumType.getEnumConstants())
//            System.out.printf("%f %s %f = %f%n",
//                    x, op, y, op.apply(x, y));
//    }

    // Using a collection instance to represent a collection of extended enums (page 178)
    public static void main(String[] args) {
        double x = Double.parseDouble("2.0");
        double y = Double.parseDouble("2.0");
        test(Arrays.asList(ExtendedOperation.values()), x, y);
    }
    private static void test(Collection<? extends Operation> opSet,
                             double x, double y) {
        for (Operation op : opSet)
            System.out.printf("%f %s %f = %f%n",
                    x, op, y, op.apply(x, y));
    }
}

```





## 6、注解优先于命名模式

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
    ExceptionTest[] value();
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
/**
 * @Repeatable 注解是用于声明其它类型注解的元注解，来表示这个声明的注解是可重复的。
 * @Repeatable的值是另一个注解，其可以通过这个另一个注解的值来包含这个可重复的注解。
 */
@Repeatable(ExceptionTestContainer.class)
/**
 * 其中，@ExceptionTest注解上的元注解@Repeatable中的值，使用了@ExceptionTestContainer注解，
 * @ExceptionTestContainer注解中包含的值类型是一个@ExceptionTest注解的数组！
 * 这就解释了官方文档中@Repeatable中值的使用。
 */
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}

ublic class Sample4 {
    @ExceptionTest(ArithmeticException.class)
    public static void m1() {  // Test should pass
        int i = 0;
        i = i / i;
    }

    @ExceptionTest(ArithmeticException.class)
    public static void m2() {  // Should fail (wrong exception)
        int[] a = new int[0];
        int i = a[1];
    }

    @ExceptionTest(ArithmeticException.class)
    public static void m3() { }  // Should fail (no exception)

    @ExceptionTest(IndexOutOfBoundsException.class)
    @ExceptionTest(NullPointerException.class)
    public static void doublyBad() {
        List<String> list = new ArrayList<>();
        // The spec permits this staticfactory to throw either
        // IndexOutOfBoundsException or NullPointerException
        list.addAll(5, null);
    }
}

public class RunTests {
    public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;
        Class testClass = Class.forName(args[0]);
        for (Method m : testClass.getDeclaredMethods()) {
        //重复注解要注意这里的写法
            if (m.isAnnotationPresent(ExceptionTest.class)
                    || m.isAnnotationPresent(ExceptionTestContainer.class)) {
                tests++;
                try {
                    m.invoke(null);
                    System.out.printf("Test %s failed: no exception%n", m);
                } catch (Throwable wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    int oldPassed = passed;
                    ExceptionTest[] excTests =
                            m.getAnnotationsByType(ExceptionTest.class);
                    for (ExceptionTest excTest : excTests) {
                        if (excTest.value().isInstance(exc)) {
                            passed++;
                            break;
                        }
                    }
                    if (passed == oldPassed)
                        System.out.printf("Test %s failed: %s %n", m, exc);
                }
            }
        }
        System.out.printf("Passed: %d, Failed: %d%n",
                          passed, tests - passed);
    }
}
```

补充知识：

instanceof和isInstance长的非常像，用法也很类似，先看看这两个的用法：

obj.instanceof(class)

也就是说这个对象是不是这种类型，

1.一个对象是本身类的一个对象

2.一个对象是本身类父类（父类的父类）和接口（接口的接口）的一个对象

3.所有对象都是Object

4.凡是null有关的都是false  null.instanceof(class)

class.inInstance(obj)

这个对象能不能被转化为这个类

1.一个对象是本身类的一个对象

2.一个对象能被转化为本身类所继承类（父类的父类等）和实现的接口（接口的父接口）强转

3.所有对象都能被Object的强转

4.凡是null有关的都是false   class.inInstance(null)

类名.class和对象.getClass()几乎没有区别，因为一个类被类加载器加载后，就是唯一的一个类。



## 7、坚持使用override注解

```java
public class Bigram {
    private final char first;
    private final char second;

    public Bigram(char first, char second) {
        this.first  = first;
        this.second = second;
    }
    /**
     * 这里本来是想覆盖equals(Object o)方法，但是参数类型使用错误，由覆写变成了重载。
     * 如果使用override注解，编译器就会帮助检查，提前发现错误。
     */
    public boolean equals(Bigram b) {
        return b.first == first && b.second == second;
    }

    public int hashCode() {
        return 31 * first + second;
    }

    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for (int i = 0; i < 10; i++)
            for (char ch = 'a'; ch <= 'z'; ch++)
                s.add(new Bigram(ch, ch));
        System.out.println(s.size());
    }
}
```



## 8、用标记接口定义类型

1.
标记接口定义了一种由标记的类的实例实现的类型。标记注释没有。这种类型的存在使您可以在编译时捕获错误，而如果使用标记注释，则这些错误在运行时才捕获。

2.
标记接口相对于标记注释的另一个优点是可以更精确地定位它们。如果使用target声明了注释类型`ElementType.TYPE`，则可以将其应用于任何类或接口。假设您有一个仅适用于特定接口的实现的标记。如果将其定义为标记接口，则可以使其扩展适用于其的唯一接口，从而确保所有标记的类型也是适用于其的唯一接口的子类型。

**补充知识：**原文链接：https://blog.csdn.net/qq_43390895/article/details/100175330

@Inherited是一个标识，用来修饰注解
在注解上使用@Inherited 表示该注解会被子类继承，注意，仅针对**类，成员属性**、方法并不受此注释的影响。

注意：

得到以下结论：

**类继承关系中@Inherited的作用**

类继承关系中，子类会继承父类使用的注解中被@Inherited修饰的注解

**接口继承关系中@Inherited的作用**

接口继承关系中，子接口不会继承父接口中的任何注解，不管父接口中使用的注解有没有被@Inherited修饰

**类实现接口关系中@Inherited的作用**

类实现接口时不会继承任何接口中定义的注解
**父类的方法用了@Inherited修饰的注解，子类也不会继承这个注解**
当用了@Inherited修饰的注解的@Retention是RetentionPolicy.RUNTIME，则增强了继承性，在反射中可以获取得到

代码演示：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Inherited
public @interface ATable {
    public String name() default "";
}

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface BTable {
    public String name() default "";
}

//作为父类
@ATable
public class Super {
    private int superx;
    public int supery;

    public Super() {
    }
    //私有
    private int superX(){
        return 0;
    }
    //公有
    public int superY(){
        return 0;
    }

}

@BTable
public class Sub extends Super {
    private int subx;
    public int suby;

    private Sub() {
    }

    public Sub(int i) {
    }

    //私有
    private int subX() {
        return 0;
    }
    //公有
    public int subY() {
        return 0;
    }

}

class TestMain {
    public static void main(String[] args) { 
    	Class<Sub> clazz = Sub.class;
    	System.out.println("============================AnnotatedElement===========================");
        System.out.println(Arrays.toString(clazz.getAnnotations()));    //获取自身和父亲的注解。如果@ATable未加@Inherited修饰，则获取的只是自身的注解而无法获取父亲的注解。
        System.out.println("------------------");
    }
}

class TestMain {
    public static void main(String[] args) {
        Class<Sub> clazz = Sub.class;
        System.out.println("============================Field===========================");
        System.out.println(Arrays.toString(clazz.getFields())); // 自身和父亲的公有字段
        System.out.println("------------------");
        System.out.println(Arrays.toString(clazz.getDeclaredFields()));  //自身所有字段
        System.out.println("============================Method===========================");
        System.out.println(Arrays.toString(clazz.getMethods()));   //自身和父亲的公有方法
        System.out.println("------------------");
        System.out.println(Arrays.toString(clazz.getDeclaredMethods()));// 自身所有方法
        System.out.println("============================Constructor===========================");
        System.out.println(Arrays.toString(clazz.getConstructors()));   //自身公有的构造方法
        System.out.println("------------------");
        System.out.println(Arrays.toString(clazz.getDeclaredConstructors()));   //自身的所有构造方法
        System.out.println("============================AnnotatedElement===========================");
        System.out.println(Arrays.toString(clazz.getAnnotations()));    //获取自身和父亲的注解
        System.out.println("------------------");
        System.out.println(Arrays.toString(clazz.getDeclaredAnnotations()));  //只获取自身的注解
        System.out.println("------------------");
    }
}


```

通过代码的结果得知：子类继承了父类（由于继承特性，子类会拥有父类的公有一切），在通过反射获取子类所有公有字段/方法/构造器的时候，会获取得到自身和父亲的所有public字段/方法/构造器，而通过反射获取所有任何字段/方法/构造器的时候，只能得到自身的所有任何访问权限修饰符的字段/方法/构造器，不会得到父类的任何字段/方法/构造器。然注解不一样，只有当父类的注解中用@Inherited修饰，子类的getAnnotations()才能获取得到父亲的注解以及自身的注解，而getDeclaredAnnotations()只会获取自身的注解，无论如何都不会获取父亲的注解。
 


