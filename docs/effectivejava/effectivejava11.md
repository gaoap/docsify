Effective Java学习笔记（十一）：序列化

## 1、其他方法优先于java序列化

不要反序列化不被信任的数据。新构建的需要跨平台的结构化数据考虑使用JSON或者protobuf。

## 2、谨慎地实现Serializable接口

## 3、考虑使用自定义的序列化形式

## 4、保护性的编写readObject方法

## 5、对于实例控制，枚举类型优先于readResolve

补充知识：原文地址 https://blog.xungong68.com/146.html

**序列化与readResolve()方法（一）**

1. 什么是Java对象序列化

  Java平台允许我们在内存中创建可复用的Java对象，但一般情况下，只有当JVM处于运行时，这些对象才可能存在，即，这些对象的生命周期不会比JVM的生命周期更长。但在现实应用中，就可能要求在JVM停止运行之后能够保存(持久化)指定的对象，并在将来重新读取被保存的对象。Java对象序列化就能够帮助我们实现该功能。

  使用Java对象序列化，在保存对象时，会把其状态保存为一组字节，在未来，再将这些字节组装成对象。必须注意地是，对象序列化保存的是对象的"状态"，即它的成员变量。由此可知，对象序列化不会关注类中的静态变量。

  除了在持久化对象时会用到对象序列化之外，当使用RMI(远程方法调用)，或在网络中传递对象时，都会用到对象序列化。Java序列化API为处理对象序列化提供了一个标准机制，该API简单易用，在本文的后续章节中将会陆续讲到。

2. 简单示例

在Java中，只要一个类实现了java.io.Serializable接口，那么它就可以被序列化。此处将创建一个可序列化的类Person，本文中的所有示例将围绕着该类或其修改版。

  Gender类，是一个枚举类型，表示性别

```java
public enum Gender {
  MALE, FEMALE
}
```



如果熟悉Java枚举类型的话，应该知道每个枚举类型都会默认继承类java.lang.Enum，而该类实现了Serializable接口，所以枚举类型对象都是默认可以被序列化的。

  Person类，实现了Serializable接口，它包含三个字段：name，String类型；age，Integer类型；gender，Gender类型。另外，还重写该类的toString()方法，以方便打印Person实例中的内容。

```java
public class Person implements Serializable {
    private String name = null;
    private Integer age = null;
    private Gender gender = null;

    public Person() {
        System.out.println("none-arg constructor");
    }

    public Person(String name, Integer age, Gender gender) {
        System.out.println("arg constructor");
        this.name = name;
        this.age = age;
        this.gender = gender;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public Gender getGender() {
        return gender;
    }

    public void setGender(Gender gender) {
        this.gender = gender;
    }

    @Override
    public String toString() {
        return "[" + name + ", " + age + ", " + gender + "]";
    }
}
```



SimpleSerial，是一个简单的序列化程序，它先将一个Person对象保存到文件person.out中，然后再从该文件中读出被存储的Person对象，并打印该对象。

```java
public class SimpleSerial {
    public static void main(String[] args) throws Exception {
        File file = new File("person.out");
        ObjectOutputStream oout = new ObjectOutputStream(new FileOutputStream(file));
        Person person = new Person("John", 101, Gender.MALE);
        oout.writeObject(person);
        oout.close();

        ObjectInputStream oin = new ObjectInputStream(new FileInputStream(file));
        Object newPerson = oin.readObject(); // 没有强制转换到Person类型
        oin.close();
        System.out.println(newPerson);
    }

}
```



上述程序的输出的结果为：

arg constructor

[John, 31, MALE]

  此时必须注意的是，当重新读取被保存的Person对象时，并没有调用Person的任何构造器，看起来就像是直接使用字节将Person对象还原出来的。

当Person对象被保存到person.out文件中之后，我们可以在其它地方去读取该文件以还原对象，但必须确保该读取程序的CLASSPATH中包含有Person.class(哪怕在读取Person对象时并没有显示地使用Person类，如上例所示)，否则会抛出ClassNotFoundException。

3. Serializable的作用

  为什么一个类实现了Serializable接口，它就可以被序列化呢？在上节的示例中，使用ObjectOutputStream来持久化对象，在该类中有如下代码：

```java
private void writeObject0(Object obj, boolean unshared) throws IOException {
    if (obj instanceof String) {
        writeString((String) obj, unshared);
    } else if (cl.isArray()) {
        writeArray(obj, desc, unshared);
    } else if (obj instanceof Enum) {
        writeEnum((Enum) obj, desc, unshared);
    } else if (obj instanceof Serializable) {
        writeOrdinaryObject(obj, desc, unshared);
    } else {
        if (extendedDebugInfo) {
            throw new NotSerializableException(cl.getName() + "\n"
                    + debugInfoStack.toString());
        } else {
            throw new NotSerializableException(cl.getName());
        }
    }
}
```

 

4. 默认序列化机制

  如果仅仅只是让某个类实现Serializable接口，而没有其它任何处理的话，则就是使用默认序列化机制。使用默认机制，在序列化对象时，不仅会序列化当前对象本身，还会对该对象引用的其它对象也进行序列化，同样地，这些其它对象引用的另外对象也将被序列化，以此类推。所以，如果一个对象包含的成员变量是容器类对象，而这些容器所含有的元素也是容器类对象，那么这个序列化的过程就会较复杂，开销也较大。

5. 影响序列化

  在现实应用中，有些时候不能使用默认序列化机制。比如，希望在序列化过程中忽略掉敏感数据，或者简化序列化过程。下面将介绍若干影响序列化的方法。

5.1 transient关键字

  当某个字段被声明为transient后，默认序列化机制就会忽略该字段。此处将Person类中的age字段声明为transient，如下所示，

```java
public class Person implements Serializable {
    transient private Integer age = null;
}
```

再执行SimpleSerial应用程序，会有如下输出：

arg constructor

[John, null, MALE]

可见，age字段未被序列化。

5.2 writeObject()方法与readObject()方法

  对于上述已被声明为transitive的字段age，除了将transitive关键字去掉之外，是否还有其它方法能使它再次可被序列化？方法之一就是在Person类中添加两个方法：writeObject()与readObject()，如下所示：

```java
public class Person implements Serializable {
    
    transient private Integer age = null;
    

    private void writeObject(ObjectOutputStream out) throws IOException {
        out.defaultWriteObject();
        out.writeInt(age);
    }

    private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {
        in.defaultReadObject();
        age = in.readInt();
    }
}
```

在writeObject()方法中会先调用ObjectOutputStream中的defaultWriteObject()方法，该方法会执行默认的序列化机制，如5.1节所述，此时会忽略掉age字段。然后再调用writeInt()方法显示地将age字段写入到ObjectOutputStream中。readObject()的作用则是针对对象的读取，其原理与writeObject()方法相同。

  再次执行SimpleSerial应用程序，则又会有如下输出：

arg constructor

[John, 31, MALE]

必须注意地是，writeObject()与readObject()都是private方法，那么它们是如何被调用的呢？毫无疑问，是使用反射。详情可见ObjectOutputStream中的writeSerialData方法，以及ObjectInputStream中的readSerialData方法。

5.3 Externalizable接口

  无论是使用transient关键字，还是使用writeObject()和readObject()方法，其实都是基于Serializable接口的序列化。JDK中提供了另一个序列化接口--Externalizable，使用该接口之后，之前基于Serializable接口的序列化机制就将失效。此时将Person类修改成如下，

```java
public class Person implements Externalizable {

    private String name = null;

    transient private Integer age = null;

    private Gender gender = null;

    public Person() {
        System.out.println("none-arg constructor");
    }

    public Person(String name, Integer age, Gender gender) {
        System.out.println("arg constructor");
        this.name = name;
        this.age = age;
        this.gender = gender;
    }

    private void writeObject(ObjectOutputStream out) throws IOException {
        out.defaultWriteObject();
        out.writeInt(age);
    }

    private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {
        in.defaultReadObject();
        age = in.readInt();
    }

    @Override
    public void writeExternal(ObjectOutput out) throws IOException {

    }

    @Override
    public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {

    }  
}
```

此时再执行SimpleSerial程序之后会得到如下结果：

arg constructor

none-arg constructor

[null, null, null]

从该结果，一方面可以看出Person对象中任何一个字段都没有被序列化。另一方面，如果细心的话，还可以发现这此次序列化过程调用了Person类的无参构造器。

  Externalizable继承于Serializable，当使用该接口时，序列化的细节需要由程序员去完成。如上所示的代码，由于writeExternal()与readExternal()方法未作任何处理，那么该序列化行为将不会保存/读取任何一个字段。这也就是为什么输出结果中所有字段的值均为空。

  另外，若使用Externalizable进行序列化，当读取对象时，会调用被序列化类的无参构造器去创建一个新的对象，然后再将被保存对象的字段的值分别填充到新对象中。这就是为什么在此次序列化过程中Person类的无参构造器会被调用。由于这个原因，实现Externalizable接口的类必须要提供一个无参的构造器，且它的访问权限为public。

  对上述Person类作进一步的修改，使其能够对name与age字段进行序列化，但要忽略掉gender字段，如下代码所示：

```java
public class Person implements Externalizable {

    private String name = null;

    transient private Integer age = null;

    private Gender gender = null;

    public Person() {
        System.out.println("none-arg constructor");
    }

    public Person(String name, Integer age, Gender gender) {
        System.out.println("arg constructor");
        this.name = name;
        this.age = age;
        this.gender = gender;
    }

    private void writeObject(ObjectOutputStream out) throws IOException {
        out.defaultWriteObject();
        out.writeInt(age);
    }

    private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {
        in.defaultReadObject();
        age = in.readInt();
    }

    @Override
    public void writeExternal(ObjectOutput out) throws IOException {
        out.writeObject(name);
        out.writeInt(age);
    }

    @Override
    public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
        name = (String) in.readObject();
        age = in.readInt();
    }
}
```

执行SimpleSerial之后会有如下结果：

arg constructor

none-arg constructor

[John, 31, null]

5.4 readResolve()方法

  当我们使用Singleton模式时，应该是期望某个类的实例应该是唯一的，但如果该类是可序列化的，那么情况可能会略有不同。此时对第2节使用的Person类进行修改，使其实现Singleton模式，如下所示：

```java
public class Person implements Serializable {

    private static class InstanceHolder {
        private static final Person instatnce = new Person("John", 31, Gender.MALE);
    }

    public static Person getInstance() {
        return InstanceHolder.instatnce;
    }

    private String name = null;

    private Integer age = null;

    private Gender gender = null;

    private Person() {
        System.out.println("none-arg constructor");
    }

    private Person(String name, Integer age, Gender gender) {
        System.out.println("arg constructor");
        this.name = name;
        this.age = age;
        this.gender = gender;
    } 
}
```

同时要修改SimpleSerial应用，使得能够保存/获取上述单例对象，并进行对象相等性比较，如下代码所示：

```java
public class SimpleSerial {

    public static void main(String[] args) throws Exception {
        File file = new File("person.out");
        ObjectOutputStream oout = new ObjectOutputStream(new FileOutputStream(file));
        oout.writeObject(Person.getInstance()); // 保存单例对象
        oout.close();

        ObjectInputStream oin = new ObjectInputStream(new FileInputStream(file));
        Object newPerson = oin.readObject();
        oin.close();
        System.out.println(newPerson);

        System.out.println(Person.getInstance() == newPerson); // 将获取的对象与Person类中的单例对象进行相等性比较
    }
}
```

执行上述应用程序后会得到如下结果：

arg constructor

[John, 31, MALE]

false

值得注意的是，从文件person.out中获取的Person对象与Person类中的单例对象并不相等。为了能在序列化过程仍能保持单例的特性，可以在Person类中添加一个readResolve()方法，在该方法中直接返回Person的单例对象，如下所示：

```java
public class Person implements Serializable {

    private static class InstanceHolder {
        private static final Person instatnce = new Person("John", 31, Gender.MALE);
    }

    public static Person getInstance() {
        return InstanceHolder.instatnce;
    }

    private String name = null;

    private Integer age = null;

    private Gender gender = null;

    private Person() {
        System.out.println("none-arg constructor");
    }

    private Person(String name, Integer age, Gender gender) {
        System.out.println("arg constructor");
        this.name = name;
        this.age = age;
        this.gender = gender;
    }

    private Object readResolve() throws ObjectStreamException {
        return InstanceHolder.instatnce;
    }
}
```

再次执行本节的SimpleSerial应用后将有如下输出：

arg constructor

[John, 31, MALE]

true

  无论是实现Serializable接口，或是Externalizable接口，当从I/O流中读取对象时，readResolve()方法都会被调用到。实际上就是用readResolve()中返回的对象直接替换在反序列化过程中创建的对象，而被创建的对象则会被垃圾回收掉。

## 6、考虑用序列化代理代替序列化实例

代理序列化举例：参考：https://www.icode9.com/content-1-632693.html

```java
/**
 *  序列化代理模式
 * 1. 序列化Person时, 会调用调用writeReplace()生成一个PersonProxy对象, 然后对此对象进行序列化
 * 2. 反序列化时, 会调用PersonProxy的readResolve()方法生成一个Person对象,
 *      最后返回此对象的拷贝 (通过PersonProxy类的readResolve方法和main方法中的输出可以看出)
 * 3. 因此, Person类的序列化工作完全交给PersonProxy类, 正如此模式的名称所表达的一样
 */
public class Person implements Serializable {
    private final String name;
    private final String hobby;
    private final int age;
    public Person(String name, String hobby, int age) {
        System.out.println("Person(String name, String hobby, int age)");
        //约束条件
        if(age < 0 || age > 200) {
            throw new IllegalArgumentException("非法年龄");
        }
        this.name = name;
        this.hobby = hobby;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public String getHobby() {
        return hobby;
    }

    public int getAge() {
        return age;
    }


    private static class PersonProxy implements Serializable {
        private final String name;
        private final String hobby;
        private final int age;

        public PersonProxy(Person original) {
            System.out.println("PersonProxy(Person original)");
            this.name = original.getName();
            this.hobby = original.getHobby();
            this.age = original.getAge();
        }
        //反序列化时将序列化代理转变回为外围类的实例
        private Object readResolve() {
            System.out.println("PersonProxy.readResolve()");
            Person person = new Person(name, hobby, age);
            System.out.println("resolveObject: " + person);
            return person;
        }
    }

    private Object writeReplace() {
        System.out.println("Person.writeReplace()");
        return new PersonProxy(this); //readObject的时候是调用, PersonProxy的readResolve()
    }

    //此方法不会执行,为何？
    //实现writeReplace就不要实现writeObject了，因为writeReplace的返回值会被自动写入输出流中，
    // 就相当于自动这样调用：writeObject(writeReplace());
    private void writeObject(ObjectOutputStream out) {
        System.out.println("Person.writeObject()");
    }

    //防止攻击者伪造数据, 企图违反约束条件 (如: 违反年龄约束)
    private Object readObject(ObjectInputStream in) throws InvalidObjectException {
        System.out.println("Person.readObject()");
        throw new InvalidObjectException("Proxy required");
    }

    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        Person person = new Person("张三", "足球" ,25);
        ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("serial.ser"));
        out.writeObject(person);
        out.flush();
        out.close();

        Thread.sleep(1000);

        ObjectInputStream in = new ObjectInputStream(new FileInputStream("serial.ser"));
        Person deserPerson = (Person) in.readObject();
        System.out.println("main: " + person);
        in.close();

        if(person == deserPerson) {
            System.out.println("序列化前后是同一个对象");
        } else {
            //程序会走这一段, 反序列化会创建对象, 但是不会执行类的构造方法, 而是使用输入流构造对象
            System.out.println("序列化前后不是同一个对象, 哈哈哈");
        }
    }
}
```

