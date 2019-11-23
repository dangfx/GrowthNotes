## 设计模式

>设计模式（Design pattern）是一套被反复使用、多数人知晓的、经过分类编目的、代码设计经验的总结。
>
>总体来说设计模式分为三大类：
>
>创建型模式，共五种：工厂方法模式、抽象工厂模式、单例模式、建造者模式、原型模式。
>
>结构型模式，共七种：适配器模式、装饰器模式、代理模式、外观模式、桥接模式、组合模式、享元模式。
>
>行为型模式，共十一种：策略模式、模板方法模式、观察者模式、迭代子模式、责任链模式、命令模式、备忘录模式、状态模式、访问者模式、中介者模式、解释器模式。

参考资料：

[1.设计模式示例](https://www.journaldev.com/1827/java-design-patterns-example-tutorial)

### 1.singleton

>1.饿汉式，非延迟加载，线程安全，影响系统性能

```java
// 饿汉式
public class Singleton {
    private static final Singleton singleton = new Singleton();

    private Singleton() {}

    public static Singleton getInstance(){
        return singleton;
    }
}
```

>2.懒汉式，延迟加载，非线程安全，不影响系统性能

```java
// 懒汉式
public class Singleton {
    private static Singleton singleton;

    private Singleton() {}

    public static Singleton getInstance(){
        if(singleton == null){
            singleton = new Singleton();
        }
        return singleton;
    }
}
```

> 3.懒汉式，延迟加载，线程安全，影响系统性能

```java
// 懒汉式
public class Singleton {

    private static Singleton singleton;

    private Singleton() {}

    public static synchronized Singleton getInstance(){
        if(singleton == null){
            singleton = new Singleton();
        }
        return singleton;
    }
}
```

> 4.双重检查锁 延迟加载，线程安全，不影响系统性能
>
> 在java字节码中会有4个步骤：
>
> 1、申请内存空间
> 2、初始化默认值（区别于构造器方法的初始化）
> 3、执行构造器方法
> 4、连接引用和实例
>
> 这4个步骤后两个有可能会重排序，1234或1243都有可能，造成未初始化完全的对象。volatile可以禁止指令重排序，从而避免这个问题。

```java
// 双重检查
public class Singleton {

    private volatile static Singleton singleton;

    private Singleton() {}

    public static Singleton getInstance(){
        if(singleton == null){
            synchronized (Singleton.class){
                if(singleton == null){
                    singleton = new Singleton(); // 注意：非原子操作
                }
            }
        }
        return singleton;
    }
}
```

> 5.静态内部类  延迟加载，线程安全，不影响系统性能

```java
// 静态内部类
public class Singleton {

    private static final class SingletonHolder {
        private static final Singleton INSTANCE = new Singleton();
    }

    private Singleton() {}

    public static Singleton getInstance(){
        return SingletonHolder.INSTANCE;
    }
}
```

> 6.枚举

```java
// 枚举
public enum Singleton {
    SINGLETON;
}

// 测试
public class PatternTest {
    @Test
    public void test_01(){
        Singleton singleton1 = Singleton.SINGLETON;
        Singleton singleton2 = Singleton.SINGLETON;
        System.out.println(singleton1 == singleton2);
    }
}
```

### 2.builder

>建造者模式：创建复杂对象时，对象的组成与装配方式独立；
>
>Product：创建的复杂对象；
>
>AbstractBuilder：实现复杂对象各个部分的创建；
>
>Director：调用具体建造者来创建复杂对象的各个部分；只负责保证对象各部分完整创建或按某种顺序创建
>
>构建不可变对象

```java
// builder class
public class ImmutableClass {
    //required fields
    private int id;
    private String name;

    //optional fields
    private HashMap properties;
    private String company;

    public int getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public HashMap getProperties() {
        //return cloned object to avoid changing it by the client application
        return (HashMap) properties.clone();
    }

    public String getCompany() {
        return company;
    }

    private ImmutableClass(ImmutableClassBuilder builder) {
        this.id = builder.id;
        this.name = builder.name;
        this.properties = builder.properties;
        this.company = builder.company;
    }

    //Builder class
    public static class ImmutableClassBuilder{
        //required fields
        private int id;
        private String name;

        //optional fields
        private HashMap properties;
        private String company;

        public ImmutableClassBuilder(int i, String nm){
            this.id=i;
            this.name=nm;
        }

        public ImmutableClassBuilder setProperties(HashMap hm){
            this.properties = (HashMap) hm.clone();
            return this;
        }

        public ImmutableClassBuilder setCompany(String comp){
            this.company = comp;
            return this;
        }

        public ImmutableClass build(){
            return new ImmutableClass(this);
        }
    }
}

// test class
@Test
public void test_02(){
    // Getting immutable class only with required parameters
    ImmutableClass immutableClass = new ImmutableClass.ImmutableClassBuilder(1, "Pankaj").build();

    HashMap hm = new HashMap();
    hm.put("Name", "Pankaj");
    hm.put("City", "San Jose");
    // Getting immutable class with optional parameters
    ImmutableClass immutableClass1 = new ImmutableClass.ImmutableClassBuilder(1, "Pankaj")
        .setCompany("Apple").setProperties(hm).build();

    //Testing immutability
    HashMap hm1 = immutableClass1.getProperties();

    //lets modify the Object passed as argument or get from the Object
    hm1.put("test", "test");

    //check that immutable class properties are not changed
    System.out.println(immutableClass1.getProperties());
}
```

### 3.Prototype

>prototype：给出一个原型对象来指明所有创建的对象的类型，然后用复制这个原型对象创建出更多同类型的对象
>
>克隆满足条件：
>
>​	1.克隆对象与原对象不是同一个对象
>
>​	2.克隆对象与原对象的类型一样
>
>​	3.克隆对象与原对象成员应该一致(可选)
>
>浅克隆：成员变量是值类型（八大基本类型，byte,short,int,long,char,double,float,boolean）.直接复制，如果是复杂的类型，（枚举，String,对象）复制对应的内存地址。
>
>深克隆：把要复制的对象所引用的对象都复制了一遍，这种对被引用到的对象的复制叫做间接复制[注意深度]
>
>优点：始化步骤繁琐，资源耗损严重的对象，简化对象创建过程，构建效率更高
>
>缺点：违背了开闭原则；克隆深度问题
>
>注意：克隆与单例冲突，clone是直接通过内存拷贝的方式，绕过构造方法

```java
import java.io.*;

public class Prototype implements Cloneable, Serializable {

    private Integer id;
    private String name;
    private Address address;

    public Prototype(Integer id, String name, Address address) {
        this.id = id;
        this.name = name;
        this.address = address;
    }

    public Address getAddress() {
        return address;
    }

    // java 默认是浅克隆
    protected Prototype clone(){
        try {
            return (Prototype) super.clone();
        } catch (CloneNotSupportedException e) {
            return null;
        }
    }
    
    // 通过序列化实现深客隆
    protected Prototype deepClone() throws Exception {
        // 序列化：将对象写入流中
        ByteArrayOutputStream bao = new ByteArrayOutputStream();
        ObjectOutputStream oos = new ObjectOutputStream(bao);
        oos.writeObject(this);
        //反序列化：将对象从流中取出来
        ByteArrayInputStream bi = new ByteArrayInputStream(bao.toByteArray());
        ObjectInputStream ois = new ObjectInputStream(bi);
        return (Prototype)ois.readObject();
    }

    @Override
    public String toString() {
        return "Prototype{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", address=" + address +
                '}';
    }
}

class Address implements Serializable {
    private Integer id;
    private String addr;

    public Address(Integer id, String addr) {
        this.id = id;
        this.addr = addr;
    }

    @Override
    public String toString() {
        return "Address{" +
                "id=" + id +
                ", addr='" + addr + '\'' +
                '}';
    }
}
```

测试：

```java
@Test
public void test_03() throws Exception {
    Address testAddr = new Address(11, "test_addr");
    Prototype prototype1 = new Prototype(1, "test", testAddr);
    // 浅克隆
    Prototype prototype2 = prototype1.clone();
    System.out.println(prototype1 == prototype2);// false
    System.out.println(prototype1.getAddress() == prototype2.getAddress());// true
    /*
    // 深克隆
    Prototype prototype2 = prototype1.deepClone();
    System.out.println(prototype1 == prototype2);// false
    System.out.println(prototype1.getAddress() == prototype2.getAddress());// false
    */
    System.out.println(prototype1);
    System.out.println(prototype2);
}
```

### 4.template

>执行顺序已经确认，各执行步骤的具体实现可能待确定。

```java
public abstract class HouseTemplate {

    // 构建顺序已经确定[final]
    public final void buildHouse(){
        buildFoundation();
        buildPillars();
        buildWalls();
        buildWindows();
        System.out.println("build house finished.");
    }
	// 已确定步骤，不支持修改[private]
    private void buildFoundation(){
        System.out.println("build house foundation");
    }

    private void buildWindows(){
        System.out.println("build house window.");
    }
	// 待确定步骤，具体类实现[public]
    public abstract void buildPillars();
    public abstract void buildWalls();
}

class WoodenHouse extends HouseTemplate {

    @Override
    public void buildPillars() {
        System.out.println("build wooden house pillars.");
    }

    @Override
    public void buildWalls() {
        System.out.println("build wooden house walls.");
    }
}

class GlassHouse extends HouseTemplate {

    @Override
    public void buildPillars() {
        System.out.println("build glass house pillars.");
    }

    @Override
    public void buildWalls() {
        System.out.println("build glass house walls.");
    }
}
```



## 基本语法

### java 关键字

>static：属于类，在类加载过程中，JVM为static修饰的内容分配一次内存空间。
>
>作用：变量，方法，代码块，内部类(调用的时候加载)，静态导入(静态属性，静态方法)
>
>final：最终，不可变
>
>作用：
>
>​	基本数据类型(常量)，值不可修改；引用数据类型，地址不可修改；
>
>​	方法不能被子类重写，可以被继承
>
>​	类不能被继承
>
>transient：被修饰的成员变量不被序列化(对象级别)，节省存储空间 
>
>​	序列化：Java中对象的序列化指的是将对象转换成以字节序列的形式来表示；
>
>​	序列化方式：
>
>​		1.implements Serializable 
>
>​		2.implements Externalizable
>
>​	失效场景：
>
>​		1.Externalizable，重写writeExternal和readExternal方法，效率比Serializable高一些，可以决定哪些属性需要序列化，transient 失效；
>
>​		2.静态变量不管是不是transient关键字修饰，transient 失效；
>
>​	使用场景：HashMap，transient int modCount; //HashMap被改变的次数
>
>