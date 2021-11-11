# Effective Java学习笔记（七）：方法

## 1、检查参数的有效性

例如使用java7中增加的Objects.requireNonNull.

如：

```java
public class NowJava {

    public static void main(String[] args) {
        // Testing requireNonNull(T obj, String message)
        try {
            printName("test");
            printName(null);
        } catch (NullPointerException e) {
            System.out.println(e.getMessage());
        }
        // requireNonNull(T obj, Supplier<String> messageSupplier)
        try {
            Supplier<String> messageSupplier = () -> "Name is required. ";
            printNameWithSuplier("aaa", messageSupplier);
            printNameWithSuplier(null, messageSupplier);
        } catch (NullPointerException e) {
            System.out.println(e.getMessage());

        }

    }

    public static void printName(String name) {
        Objects.requireNonNull(name, "Name is required.");
        System.out.println("Name is " + name);
    }
    public static void printNameWithSuplier(String name, Supplier<String> messageSupplier) {
        Objects.requireNonNull(name, messageSupplier);
        System.out.println("Name is " + name);
    }

}
```

补充知识：参考原文：https://blog.csdn.net/qq_44643122/article/details/107389799

1）checkFromIndexSize(int fromIndex, int size, int length)

检查是否在子范围从 fromIndex （包括）到 fromIndex + size （不包括）是范围界限内 0 （包括）到 length （不包括）。简单理解就是是否越界，意思就是1+（1+size）是不是在区间0-3内；

2）checkFromToIndex(int fromIndex, int toIndex, int length)

检查是否在子范围从 fromIndex （包括）到 toIndex （不包括）是范围界限内 0 （包括）到 length （不包括）。简单理解就是1-2是不是在区间0-3内；

3）checkIndex(int index, int length)

检查 index是否在 0 （含）到 length （不包括）范围内。简单理解就是，就是句面意思；

4）compare(T a, T b, Comparator<? super T> c)

如果参数相同则返回0，否则返回 c.compare(a, b) 。

5）deepEquals(Object a, Object b)

 返回 true如果参数是深层相等，彼此 false其他。 深层相等

6）static boolean equals(Object a, Object b)

 返回 true如果参数相等，彼此 false其他。就是做比较，相等返回true，不等返回false 

7）static int hash(Object... values)

为一系列输入值生成哈希码。 

8）static int hashCode(Object o)

返回非的哈希码 null参数，0为 null的论点。 

9）static boolean isNull(Object obj)

返回 true如果提供的参考是 null ，否则返回 false 。  判断是否是空

10）static boolean nonNull(Object obj)。  

 返回 true如果提供的参考是非 null否则返回 false 。   判断是否不是空

11）static <T> T requireNonNull(T obj)    

检查指定的对象引用是否不是 null 。 

12）static <T> T requireNonNull(T obj, String message)

 检查指定的对象引用是否为null ，如果是，则抛出自定义的NullPointerException 。  如果是空直接上异常

13）static <T> T requireNonNull(T obj, Supplier<String> messageSupplier)

检查指定的对象引用是否为null ，如果是，则抛出自定义的NullPointerException 。 

14）static <T> T requireNonNullElse(T obj, T defaultObj)

 如果它是非 null ，则返回第一个参数，否则返回非 null第二个参数。  字面意思

15）static <T> T requireNonNullElseGet(T obj, Supplier<? extends T> supplier)

 如果它是非 null ，则返回第一个参数，否则返回非 null值 supplier.get() 。 

16）static String toString(Object o)

 返回调用的结果 toString对于非 null参数， "null"为 null的说法。 

17）static String toString(Object o, String nullDefault)

如果第一个参数不是 null ，则返回在第一个参数上调用 toString的结果，否则返回第二个参数。
**非公有的方法，应该使用断言来检查参数的有效性；**

例如：

```java
// Private helper function for a recursive sort 
private static void sort(long a[], int offset, int length) {
    assert a != null;
    assert offset >= 0 && offset <= a.length;
    assert length >= 0 && length <= a.length - offset;
    ... // Do the computation 
}
```

## 2、必要时进行保护性拷贝

例如：

本意是不可改变的一个时间段，实际很容易被破坏掉。

```java
public final class Period {
    private final Date start;
    private final Date end;

    /**
     * @param  start the beginning of the period
     * @param  end the end of the period; must not precede start
     * @throws IllegalArgumentException if start is after end
     * @throws NullPointerException if start or end is null
     */
    public Period(Date start, Date end) {
        if (start.compareTo(end) > 0)
            throw new IllegalArgumentException(
                    start + " after " + end);
        this.start = start;
        this.end   = end;
    }

    public Date start() {
        return start;
    }
    public Date end() {
        return end;
    }
}

//可以通过如下方式，破坏上述类：
 public static void main(String[] args) {
        Date start = new Date();
        Date end = new Date();
        Period p = new Period(start, end);
        end.setYear(78);  
        System.out.println(p);   
}

//如果要避免上述潜在的风险，可以修改构造器，进行保护性拷贝处理。
    public final class Period {
    private final Date start;
    private final Date end;


    public String toString() {
        return start + " - " + end;
    }

    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end = new Date(end.getTime());
        if (this.start.compareTo(this.end) > 0)
            throw new IllegalArgumentException(
                    this.start + " after " + this.end);
    }
    public Date start() {
        return new Date(start.getTime());
    }

    public Date end() {
        return new Date(end.getTime());
    }
}
```

补充知识：  原文参考：https://blog.csdn.net/qq_21845263/article/details/106203865

Java8中的时间类主要有：Date、Instant、LocalDateTime（LocalDate、LocalTime）、ZonedDateTime，期中Date，java.time包下的时间类都是可变类，其它是不可变类。

在这里,要分清楚包含时区信息的类、以及不包含时区信息的类。不包含时区信息的类实际上就类似于一个yyyy-MM-dd HH:mm:ss字符串，需要额外的时区信息才能表达一个时刻，即LocalDateTime、LocalDate、LocalTime。而Date（0时区）、Instant（0时区）、ZonedDateTime都包含有时区信息。

```java
import java.util.Date;
import java.util.TimeZone;

class Scratch {
    public static void main(String[] args) {
        TimeZone.setDefault(TimeZone.getTimeZone("GMT"));
        Date d1 = new Date();
        Instant i1 = Instant.now();
        ZonedDateTime z1 = ZonedDateTime.now();
        LocalDateTime l1 = LocalDateTime.now();

        TimeZone.setDefault(TimeZone.getTimeZone("Asia/Shanghai"));
        Date d2 = new Date();
        Instant i2 = Instant.now();
        ZonedDateTime z2 = ZonedDateTime.now();
        LocalDateTime l2 = LocalDateTime.now();

        TimeZone.setDefault(TimeZone.getTimeZone("Australia/Darwin"));
        Date d3 = new Date();
        Instant i3 = Instant.now();
        ZonedDateTime z3 = ZonedDateTime.now();
        LocalDateTime l3 = LocalDateTime.now();
    }
}

```

 **Date**：

```java
private transient long fastTime;

private transient BaseCalendar.Date cdate;
```

本质是记录了0时区的时间，不同时区的人在同一时刻new Date()时，其对象内存放的毫秒数是一样的（都是0时区）。

例如：

```java
Date date = new Date(1503544630000L);  // 对应的北京时间是2017-08-24 11:17:10
 
SimpleDateFormat bjSdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");     // 北京
bjSdf.setTimeZone(TimeZone.getTimeZone("Asia/Shanghai"));  // 设置北京时区
 
SimpleDateFormat tokyoSdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");  // 东京
tokyoSdf.setTimeZone(TimeZone.getTimeZone("Asia/Tokyo"));  // 设置东京时区
 
SimpleDateFormat londonSdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"); // 伦敦
londonSdf.setTimeZone(TimeZone.getTimeZone("Europe/London"));  // 设置伦敦时区
 
System.out.println("毫秒数:" + date.getTime() + ", 北京时间:" + bjSdf.format(date));
System.out.println("毫秒数:" + date.getTime() + ", 东京时间:" + tokyoSdf.format(date));
System.out.println("毫秒数:" + date.getTime() + ", 伦敦时间:" + londonSdf.format(date));

// 输出
// 毫秒数:1503544630000, 北京时间:2017-08-24 11:17:10
// 毫秒数:1503544630000, 东京时间:2017-08-24 12:17:10
// 毫秒数:1503544630000, 伦敦时间:2017-08-24 04:17:10

String timeStr = "2017-8-24 11:17:10"; // 字面时间
SimpleDateFormat bjSdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
bjSdf.setTimeZone(TimeZone.getTimeZone("Asia/Shanghai"));
Date bjDate = bjSdf.parse(timeStr);  // 解析
System.out.println("字面时间: " + timeStr +",按北京时间来解释:" + bjSdf.format(bjDate) + ", " + bjDate.getTime());
 
SimpleDateFormat tokyoSdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");  // 东京
tokyoSdf.setTimeZone(TimeZone.getTimeZone("Asia/Tokyo"));  // 设置东京时区
Date tokyoDate = tokyoSdf.parse(timeStr); // 解析
System.out.println("字面时间: " + timeStr +",按东京时间来解释:"  + tokyoSdf.format(tokyoDate) + ", " + tokyoDate.getTime());

// 输出为：
// 字面时间: 2017-8-24 11:17:10,按北京时间来解释:2017-08-24 11:17:10, 1503544630000
// 字面时间: 2017-8-24 11:17:10,按东京时间来解释:2017-08-24 11:17:10, 1503541030000

```

**Instant**

```java
private final long seconds;

private final int nanos;
```

本质是记录了0时区的时间,精确到纳秒。

 **LocalDateTime**

```java
private final LocalDate date;

private final LocalTime time;
```

**LocalDate

```java
private final int year;

private final short month;

private final short day;
```

**LocalTime**

```java
private final byte hour;

private final byte minute;

private final byte second;

private final int nano;
```

根据当前设置的时区获取当前时区的日期、时间，但不会记录时区信息。Local*这些时间类实际上就相当于一个字符串，只不过把年月日、时分秒解析出来了而已。

**ZonedDateTime**

```java
private final LocalDateTime dateTime;

private final ZoneOffset offset;

private final ZoneId zone;
```

在LocalDateTime的基础上同时保存了当前的时区，即：根据当前设置的时区获取当前时区的日期、时间，同时保存当前时区信息。

自带时区信息，默认获取当前时区。

**记录时间原理**
new Date()和Instant.now本质上都是记录了0时区的时间，即使当前时区不同，其存储的都是距离1970-01-01 00:00:00所经过的时间。

Local*这些时间类实际上就相当于一个字符串，只不过把年月日、时分秒解析出来了而已。

**打印时间**
打印时设置的时区信息会对原时间解析出来的字符串有影响，打印过程会转换原时区时间到现在时区时间，这点一定要注意。

Instant打印时要给DateTimeFormatter设置时区才能打印，否则会报错。

```java
DateTimeFormatter.ofPattern(pattern).withZone(ZoneId.of("Asia/Shanghai"))
```

**注意点**
 DateTimeFormatter.with*()会返回一个新的对象
正确的做法：

```java
DateTimeFormatter.ofLocalizedDate(FormatStyle.FULL).withZone(ZoneId.of("GMT")).withLocale(Locale.UK)
```

错误的做法

```java
DateTimeFormatter formatter = DateTimeFormatter.ofLocalizedDate(FormatStyle.FULL);
formatter.withZone(ZoneId.of("GMT"));
formatter.withLocale(Locale.UK);
```

因为DateTimeFormatter是一个不可变类，所以不可以修改其属性，同时也就是个线程安全类了。其灵活的创建过程是利用DateTimeFormatterBuilder来实现的。DateTimeFormatter.ofPattern()没有设置打印时的时区
需要设置时区需要`DateTimeFormatter.ofPattern(“yyyy-MM-dd HH:mm:ss”).withZone(ZoneId.of(“GMT”))`这样设置。

**SimpleDateFormat并非线程安全**其format()与parse()方法都不是线程安全。

**Clock类**
Clock是带有时区信息的时间处理类，用它可以获取不同时区的时间，也可以配合**Duration**，对时间进行时、分、秒级别的修改

**Duration 和 Period**
Duration 和 Period 都是用来表示两个时间量之间的差值，不同点在于Duration 是基于时间值，而 Period 是基于日期值。

**java.time包下的5个包组成：**
java.time – 包含值对象的基础包
java.time.chrono – 提供对不同的日历系统的访问
java.time.format – 格式化和解析时间和日期
java.time.temporal – 包括底层框架和扩展特性
java.time.zone – 包含时区支持的类
**JDBC映射**
最新JDBC映射将把数据库的日期类型和Java 8的新类型关联起来：

```
date -> LocalDate
time -> LocalTime
timestamp -> LocalDateTime
```

**SimpleDateFormat**

SimpleDateFormat只能格式化Date。而DateTimeFormatter可以格式化TemporalAccessor（不包括Date，但是包括LocalDate*、Instant、ZonedDateTime等）。

## 3、谨慎设计方法签名

1. 谨慎的选择方法的名称
2. 不要过于追求提供便利的方法
3. 避免过长的参数列表
4. 对于参数类型，要优先使用接口而不是类
5. 对于boolean参数，要优先使用两个元素的枚举类型

## 4、慎用重载

安全而保守的策略是：永远不要导出两个具有相同参数数目的重载方法。

错误示范1：

```java
public class CollectionClassifier {
    public static String classify(Set<?> s) {
        return "Set";
    }

    public static String classify(List<?> lst) {
        return "List";
    }

    public static String classify(Collection<?> c) {
        return "Unknown Collection";
    }

    public static void main(String[] args) {
        Collection<?>[] collections = {
                new HashSet<String>(),
                new ArrayList<BigInteger>(),
                new HashMap<String, String>().values()
        };
      、/**
         * 输出结果为：
         * Unknown Collection
         * Unknown Collection
         * Unknown Collection
         */
        for (Collection<?> c : collections)
            System.out.println(classify(c));
    }
}
```

修正后的类：

```java
public class FixedCollectionClassifier {
    public static String classify(Collection<?> c) {
        return c instanceof Set  ? "Set" :
                c instanceof List ? "List" : "Unknown Collection";
    }

    public static void main(String[] args) {
        Collection<?>[] collections = {
                new HashSet<String>(),
                new ArrayList<BigInteger>(),
                new HashMap<String, String>().values()
        };

        for (Collection<?> c : collections)
            System.out.println(classify(c));
    }
}
```

再来一个错误示范：**自动装箱类引出的意外结果**

```java
public class SetList {
    public static void main(String[] args) {
        Set<Integer> set = new TreeSet<>();
        List<Integer> list = new ArrayList<>();

        for (int i = -3; i < 3; i++) {
            set.add(i);
            list.add(i);
        }
        for (int i = 0; i < 3; i++) {
            set.remove(i);
            list.remove(i);
        }
        System.out.println(set + " " + list);
    }
}
```

## 5、慎用可变参数



## 6、返回零长度的数组或者集合，而不是null



## 7、谨慎返回optinal

容器类型包括集合、映射、Stream、数组和optional，都不应该被包装在optional中。

如果无法返回结果并且当没有返回结果时客户端必须执行特殊的处理，那么就应该声名该方法返回Optional<T>。

永远不应该返回基本包装类型的optional。(Boolean、Byte、Character、Short、Float是例外)。

几乎永远都不适合用optional作为键、值或者集合或数组中的元素。

optional会有性能开销

## 8、为所有导出的API元素编写文档注释

示例：

```java
// Documentation comment examples (Pages 255-9)
public class DocExamples<E> {
    // Method comment (Page 255)
    /**
     * Returns the element at the specified position in this list.
     *
     * <p>This method is <i>not</i> guaranteed to run in constant
     * time. In some implementations it may run in time proportional
     * to the element position.
     *
     * @param  index index of element to return; must be
     *         non-negative and less than the size of this list
     * @return the element at the specified position in this list
     * @throws IndexOutOfBoundsException if the index is out of range
     *         ({@code index < 0 || index >= this.size()})
     */
    E get(int index) {
        return null;
    }

    // Use of @implSpec to describe self-use patterns & other visible implementation details. (Page 256)
    /**
     * Returns true if this collection is empty.
     *
     * @implSpec This implementation returns {@code this.size() == 0}.
     *
     * @return true if this collection is empty
     */
    public boolean isEmpty() {
        return false;
    }

    // Use of the @literal tag to include HTML and javadoc metacharacters in javadoc comments. (Page 256)
    /**
     * A geometric series converges if {@literal |r| < 1}.
     */
    public void fragment() {
    }

    // Controlling summary description when there is a period in the first "sentence" of doc comment. (Page 257)
    /**
     * A suspect, such as Colonel Mustard or {@literal Mrs. Peacock}.
     */
    public enum FixedSuspect {
        MISS_SCARLETT, PROFESSOR_PLUM, MRS_PEACOCK, MR_GREEN, COLONEL_MUSTARD, MRS_WHITE
    }


    // Generating a javadoc index entry in Java 9 and later releases. (Page 258)
    /**
     * This method complies with the {@index IEEE 754} standard.
     */
    public void fragment2() {
    }

    // Documenting enum constants (Page 258)
    /**
     * An instrument section of a symphony orchestra.
     */
    public enum OrchestraSection {
        /** Woodwinds, such as flute, clarinet, and oboe. */
        WOODWIND,

        /** Brass instruments, such as french horn and trumpet. */
        BRASS,

        /** Percussion instruments, such as timpani and cymbals. */
        PERCUSSION,

        /** Stringed instruments, such as violin and cello. */
        STRING;
    }

    // Documenting an annotation type (Page 259)
    /**
     * Indicates that the annotated method is a test method that
     * must throw the designated exception to pass.
     */
    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.METHOD)
    public @interface ExceptionTest {
        /**
         * The exception that the annotated test method must throw
         * in order to pass. (The test is permitted to throw any
         * subtype of the type described by this class object.)
         */
        Class<? extends Throwable> value();
    }
}
```

