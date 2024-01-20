# Java注解和反射


Java 注解（Annotation）又称 Java 标注，是 JDK5.0 引入的一种机制。Java 语言中的类、方法、变量、参数和包等都可以被标注....

<!--more-->

# 注解

 Java 注解（Annotation）又称 Java 标注，是 JDK5.0 引入的一种机制。Java 语言中的类、方法、变量、参数和包等都可以被标注。

## 1.Annotation定义

例如我们得@Override注解，其定义方式为：

```Java
import java.lang.annotation.*;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}
```

可以看到Override这个注解上还加了@Target和@Retention注解，我们称它们为元注解。进入Target其中：

```Java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Target {
    /**
     * Returns an array of the kinds of elements an annotation interface
     * can be applied to.
     */
    ElementType[] value();
}
```

即在其中可以定义方法。其格式为，例：String  name（）；

自己定义一个注解

```java
public @interface myAnnotation {
    Stirng name();
}
//在其他类、方法、变量、参数上使用只需
@myAnnotation(name = "张三")
public xx (){}
```

上述注解中定义了一个name方法，所以在注解应用过程中必须给他赋值。定义方法的注意事项：

1、定义的格式是：String name();

2、可以有默认值，也可以没有，如果没有默认值在使用的时候必须填写对应的值。默认值使用default添加。例如

`Stirng name() default “张三”;`

3、如果想在使用的时候不指定具体的名字，方法名字定义为value() 即可。（只能定义一个，即使用时直接写xx不用写name = xx）

## 1.Annotation相关组成

```java
 //我们发现字节码中注解其实也是一个接口统一继承自java.lang.annotation.Annotation
```

```java
package java.lang.annotation;
public interface Annotation {

    boolean equals(Object obj);

    int hashCode();

    String toString();

    Class<? extends Annotation> annotationType();
}
```

这个java.lang.annotation包下和我们元注解有关的类：

1.ElementType

ElementType 是 Enum 枚举类型，它用来指定 Annotation 的类型。大白话就是，说明了我的注解将来要放在哪里。

```java
public enum ElementType {
    // 类、接口（包括注释类型）或枚举声明
    TYPE,          
    //  字段声明（包括枚举常量
    FIELD,       
    //  方法声明
    METHOD,       
    //  参数声明
    PARAMETER,      
    //  构造方法声明
    CONSTRUCTOR,     
    //  局部变量声明
    LOCAL_VARIABLE,  
    //   注释类型声明
    ANNOTATION_TYPE,   
    //  包声明
    PACKAGE      
}
```

例如我们在Override中元注解@Target(ElementType.METHOD)，代表此注解只能加在方法上。（可以用逗号同时写多个地方）

2.RetentionPolicy.java

 RetentionPolicy 是 Enum 枚举类型，它用来指定 Annotation 的策略。通俗点说，就是不同 RetentionPolicy 类型的 Annotation 的作用域不同。

1. 若 Annotation 的类型为 SOURCE，则意味着：Annotation 仅存在于编译器处理期间，编译器处理完之后，该 Annotation 就没用了。 例如，" @Override" 标志就是一个 SOURCE。当它修饰一个方法的时候，就意味着该方法覆盖父类的方法；并且在编译期间会进行语法检查！编译器处理完后，"@Override" 就没有任何作用了。
2. 若 Annotation 的类型为 CLASS，则意味着：编译器将 Annotation 存储于类对应的 .class 文件中，它是 Annotation 的默认行为。
3. 若 Annotation 的类型为 RUNTIME，则意味着：编译器将 Annotation 存储于 class 文件中，并且可由JVM读入。

```java
package java.lang.annotation;
public enum RetentionPolicy {
    //Annotation信息仅存在于编译器处理期间，编译器处理完之后就没有该Annotation信息了
    SOURCE,       
    //编译器将Annotation存储于类对应的.class文件中。但不会加载到JVM中。默认行为 
    CLASS,       
    // 编译器将Annotation存储于class文件中，并且可由JVM读入，因此运行时我们可以获取。
    RUNTIME       
}
```

## 3.自带的 Annotation

#### [#](https://ydlclass.com/doc21xnv/javase/reflect/#_1-内置的注解)（1）内置的注解

Java 定义了一套注解，共有10 个，6个在 java.lang 中，剩下 4 个在 java.lang.annotation 中。

（1）作用在代码的注解是

- @Override - 检查该方法是否是重写方法。如果发现其父类，或者是引用的接口中并没有该方法时，会报编译错误。
- @Deprecated - 标记过时方法。如果使用该方法，会报编译警告。
- @SuppressWarnings - 指示编译器去忽略注解中声明的警告。
- @SafeVarargs - Java 7 开始支持，忽略任何使用参数为泛型变量的方法或构造函数调用产生的警告。
- @FunctionalInterface - Java 8 开始支持，标识一个匿名函数或函数式接口。
- @Repeatable - Java 8 开始支持，标识某注解可以在同一个声明上使用多次。

（2）作用在其他注解的注解(或者说 元注解)是:

- @Retention - 标识这个注解怎么保存，是只在代码中，还是编入class文件中，或者是在运行时可以通过反射访问。
- @Documented - 标记这些注解是否包含在用户文档中。
- @Target - 标记这个注解可以修饰哪些 Java 成员。
- @Inherited - 如果一个类用上了@Inherited修饰的注解，那么其子类也会继承这个注解

#### [#](https://ydlclass.com/doc21xnv/javase/reflect/#_2-常用注解)（2）常用注解

 通过上面的示例，我们能理解：@interface 用来声明 Annotation，@Documented 用来表示该 Annotation 是否会出现在 javadoc 中， @Target 用来指定 Annotation 的类型，@Retention 用来指定 Annotation 的策略。

> @Documented 标记这些注解是否包含在用户文档中。

> @Inherited

@Inherited 的定义如下：加有该注解的注解会被子类继承，注意，仅针对**类，成员属性**、方法并不受此注释的影响。

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Inherited {
}
```

> @SuppressWarnings

@SuppressWarnings 的定义如下：

```text
@Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE})
@Retention(RetentionPolicy.SOURCE)
public @interface SuppressWarnings {
    String[] value();
}
```

 SuppressWarnings 的作用是，让编译器对"它所标注的内容"的某些警告保持静默，用于抑制编译器产生警告信息。。例如，"@SuppressWarnings(value={"deprecation", "unchecked"})" 表示对"它所标注的内容"中的 "SuppressWarnings 不再建议使用警告"和"未检查的转换时的警告"保持沉默。

| **关键字**  | **用途**                                           |
| ----------- | -------------------------------------------------- |
| all         | 抑制所有警告                                       |
| boxing      | 抑制装箱、拆箱操作时候的警告                       |
| fallthrough | 抑制在switch中缺失breaks的警告                     |
| finally     | 抑制finally模块没有返回的警告                      |
| rawtypes    | 使用generics时忽略没有指定相应的类型               |
| serial      | 忽略在serializable类中没有声明serialVersionUID变量 |
| unchecked   | 抑制没有进行类型检查操作的警告                     |
| unused      | 抑制没被使用过的代码的警告                         |

## Annotation 的作用

（1）Annotation 具有"让编译器进行编译检查的作用“。

（2）利用反射，和反射配合使用能产生奇妙的化学反应。



# 反射

在方法区存在这么一些对象，叫做类对象，他们表述了我们写的所有的类，当我们new对象时会根据这类对象，并调用其构造方法为我们创建实例。

JAVA反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为 java语言的反射机制。

功能：

- 在运行时判断任意一个对象所属的类；
-  在运行时构造任意一个类的对象； 
- 在运行时判断任意一个类所具有的成员变量和方法；
-  在运行时调用任意一个对象的方法；生成动态代理

## 反射API

Java反射所需要的类并不多，主要有java.lang.Class类和java.lang.reflect包中的Field、Constructor、 Method、Annotation类。

每个类【有且仅有】一个Class类，也叫类对象。

Class类是Java反射的起源，我们只有先获得一个类对象才能进行后续操作。

### 1.获取类对象方法

```java
1、使用类
Class clazz = Dog.class;   //用的少，没有体现反射的动态性，在编译期就会
2、使用全类名
Class aClass = Class.forName("com.xinzhi.Day");     //体现反射的动态性/编译时看是不是一个字符串，是，编译通过，运行时就可以动态生成对象
3、使用对象
Dog dog = new Dog();
Class clazz = dog.getClass();  //com.xx.xx.Dog@..

4.使用类加载器：ClassLoader
 ClassLoader classLoader = Test.class.getClassLoader();
Class clazz = classLoader.loadClass("com.lu.Dog");
```

Class实例对应着加载到内存中的一个运行时类（经过编译后加载到内存中的类称为运行时类）

### 2.对类对象的一些操作

```java
//获取类名字
String name = clazz.getName();
//获取类加载器
ClassLoader classLoader = clazz.getClassLoader();
//获取资源
URL resource = clazz.getResource("");
//得到父类
Class superclass = clazz.getSuperclass();
//判断一个类是不是接口，数组等等
boolean array = clazz.isArray();
boolean anInterface = clazz.isInterface();
//重点，使用class对象实例化一个对象
Object instance = clazz.newInstance();//必须要有空参构造器，权限修饰符的权限要够。
```

### 3、获取相关属性并操作

> 字段 Field

#### 1.获取成员变量

```java
//获取字段，只能获取公共的字段（public）
Field name = clazz.getField("color");
//打印结果public java.lang.String com.lu.cglib.Dog.color
Field[] fields = clazz.getFields(); //获取所有字段，
//能获取所有的字段包括private
Field color = clazz.getDeclaredField("color");
Field[] fields = clazz.getDeclaredFields();
System.out.println(color.getType());
```

#### 2.获取属性

```Java
Dog dog = new Dog(); 
dog.setColor("red");
Class clazz = Dog.class; 
Field color = clazz.getDeclaredField("color"); 
System.out.println(color.get(dog));

//当然你要是明确类型你还能用以下方法：
Int i = age.getInt(dog);
xxx.getDouble(dog);
xxx.getFloat(dog);
xxx.getBoolean(dog);
xxx.getChar(dog);
//每一种基本类型都有对应方法
```

#### 3.获取方法

```java
Dog dog = new Dog();
dog.setColor("red");
Class clazz = Dog.class;
//获取某个方法，第一个参数是方法名字，后边是参数类型，
Method method = clazz.getMethod("eat",String.class);
//拿到参数的个数
int parameterCount = method.getParameterCount();
//拿到方法的名字
String name = method.getName();
//拿到参数的类型数组
Class<?>[] parameterTypes = method.getParameterTypes();
//拿到返回值类型
Class<?> returnType = method.getReturnType();
//重点。反射调用方法，传一个实例，和参数
method.invoke(dog,"热狗");

一般调用invoke之前会有method.setAccessible(true)//保证当前的方法是可访问的。
    
//如何调用静态方法
Method method = clazz.getMethod("staticmethodName");
method.setAccessible(true)
 Object returnVal = method.invoke(Dog.class);//参数里的DOg.class应该也不用写本来这个class实例就是通过这个类.出来的。而且静态方法也不需要对象。
```

#### 4.构造函数

（1）获取并构建对象

```java
Constructor[] constructors = clazz.getConstructors();
Constructor constructor = clazz.getConstructor();
Constructor[] declaredConstructors = clazz.getDeclaredConstructors();
Constructor declaredConstructor = clazz.getDeclaredConstructor();


Object obj = constructor.newInstance();

/*Class.newInstance() 只能够调用无参的构造函数，即默认的构造函数； 
Constructor.newInstance() 可以根据传入的参数，调用任意构造构造函数。 
Class.newInstance() 抛出所有由被调用构造函数抛出的异常。 
Class.newInstance() 要求被调用的构造函数是可见的，也即必须是public类型的; 
Constructor.newInstance() 在特定的情况下，可以调用私有的构造函数。 
```

#### 5.注解

从方法、字段、类上获取注解

```java
//元注解 要加上runtime
//类上
Annotation annotation = clazz.getAnnotation(Bean.class);
Annotation[] annotations = clazz.getAnnotations();
//字段上
Annotation annotation = field.getAnnotation(Bean.class);
Annotation[] annotations = field.getAnnotations();
//方法上
Annotation annotation = method.getAnnotation(Bean.class);
Annotation[] annotations = method.getAnnotations();
```

### 加载配置文件

```java
//加载在当前module配置文件的两种方式：`

Properties pros = new Properties ();`

//读取方式之一：`
FileInputStream fis = new FileInputStream("jdbc.properties");`

pros.load(fis)`



//2.使用ClassLoader,如配置文件在当前moudle的src下`

ClassLoader   classloader = Test.class.getClassLoader();`

InputStream is = classloader.getResourceAsSteram("jdbc.properties");`

pros.load(is);


String user = pros.getProperty("user")
```


