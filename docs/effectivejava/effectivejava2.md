# Effective Java学习笔记（二）：所有对象都通用的方法

## 第一条：覆盖equals时应遵守的通用约定

不应该覆盖equals的条件：

1. 类的每个实例本质上都是唯一的。
2. 类没必要提供“逻辑相等”的功能。
3. 超类易经覆盖了equals，超类的行为对于这个类也是合适的。
4. 类是私有的，或者是包级私有的，可以确定它的equals方法永远不会被调用。

可以覆盖equals的条件：

1. 如果类具有“逻辑相等”概念，而且超类还没有覆盖equals时。

equals的约定规范：

1. 自反性、对称性、传递性、一致性（多次调用，结果不变）、对于任何非null的引用值x,x.equals（null）必须返回false。

注意事项：

无法在扩展可实例化的类的同时，既增加新的值组件，同时又保留equals约定。

Date和Timestamp不要混合使用。原因是违法传递性。

对于一个抽象类的子类中可以增加新的值组件且不违反equals约定，因为不能直接创建超类的实例。

java.net.URLd的equals违反了一致性。

**通用约定不允许抛出NullPointerException异常。**

o instanceof MyType 类型检查时，如果 o为null，则instanceof 操作符都会返回false

**高质量实现equals 的方法诀窍：**

1. 使用==操作符检查"参数是否为这个对象的引用"
2. 使用 instanceof 操作符检查"参数是否为正确的类型"
3. 把参数转换成正确的类型
4. 对于该类中的每个“关键”域，检查参数中的域是否与该对象中对应的预想匹配。
5. 覆盖equals时总要覆盖hashCode
6. 不要企图让equals方法过于智能
7. 不要将equals声明中的Object对象替换为其他的类型

Google开源的AutoValue框架是个不错的选择，可以试试。本人习惯用Lombok

## 第二条：覆盖equals时总要覆盖hashCode方法

每个覆盖了equals方法的类中，都必须覆盖hashCode方法。否则影响在集合中的使用。例如： HashMap HashSet 

规范：

1、在应用程序执行期间，只要对equals方法的比较操作所用到的信息没有被修改，那么对同一个对象的多次调用，Hashcode方法都必须始终返回同一个值。两个程序中的hashCode 可以不一致。

2、两个对象根据equals方法比较相等，那么调用两个对象中的hashCode方法都必须产生相同的效果。

3、如果两个对象根据Equals方法比较不相同，那么调用两个对象的HashCode方法，则不一定要求产生不同结果。建议也不同，这样可以提高性能。

4、计算hashCode 之所以用31是因为乘法计算31比较高效：31*i==（i  << 5）- i 

5、不要试图从散列码计算中排除一个对象的关键域来提高性能。

6、不要对hashCode方法的返回值作出具体的规定，因为客户算无法理所当然的依赖它

## 第三条：始终要覆写toString

静态工具类、枚举类型没有必要覆写toString .

## 第四条：谨慎的覆盖clone

对象拷贝更好的办法是提供拷贝构造器或者拷贝工厂。

例外，最好用clone复制数组

# 第五条：考虑实现Comparable接口

compareTo 方法不但允许简单的等同性比较，而且允许执行顺序比较。

实现Comparable接口的对象数组排序：Arrays.sort(a);

将这个对象与指定的对象进行比较。当该对象小于、等于或大于指定的对象的时候，分别返回负整数、零或者正整数。

BIgDecima类，它的compareTo方法与euals不一致。如果创建了一个空的HashSet实例，并且添加new BigDecimal("1.0") 和 new BigDecimal("1.00").这个集合就将包含两个元素，因为新增集合中的两个BigDecimal实例通过equals方法比较时不相等。然而，如果使用TreeSet执行同样的过程，集合中将只包含一个元素，因为两个BigDecimal实例通过compareTo方法比较时是相等的。

对象引用类型比较，糟糕的写法：

```java

#错误示范：容易出现整数溢出等问题
static Comparator<Object> hashCodeOrder =new Comparator<>(){
	public int compare(Object o1,Object o2){
	return 01.hashCode()-02.hashCode();
	}
}
#推荐方式一：
static Comparator<Object> hashCodeOrder =new Comparator<>(){
	public int compare(Object o1,Object o2){
	return Integer.compare(01.hashCode(),02.hashCode());
	}
}
#推荐方式二：
static Comparator<Object> hashCodeOrder =new Comparator.comparintInt(o -> o.hashCode());
 
```

