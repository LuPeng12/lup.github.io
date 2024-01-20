# lambda表达式和四大接口


匿名内部类就是没有名字的内部类，必须在创建时使用 new 语句来声明类。正因为没有名字，所以匿名内部类只能使用一次。使用匿名内部类还有个前提...

<!--more-->

# 1.lambda表达式

### 1.1 匿名内部类和函数式接口

##### 1.1.1 匿名内部类

匿名内部类就是没有名字的内部类，必须在创建时使用 new 语句来声明类。正因为没有名字，所以匿名内部类只能使用一次。使用匿名内部类还有个前提条件：必须继承一个父类或实现一个接口。

不使用匿名内部类实现：

```java
abstract class Person {
   public abstract void eat();
}
class Child extends Person {
   public void eat() {
       System.out.println( "eat something" );
   }
}
public class Demo {
   public static void main(String[] args) {
     Person p =  new Child();
       p.eat();
   }
}
```

 使用匿名内部类实现：

```java
abstract class Person {
    public abstract void eat();
}
 
public class Demo {
    public static void main(String[] args) {
        Person p = new Person() {
            public void eat() {
                System.out.println("eat something");
            }
        };
        p.eat();
    }
}
```

可以看到，我们直接将抽象类Person中的方法在大括号中实现了

这样便可以省略一个类的书写

并且，匿名内部类还能用于接口上

```java
@FunctionalInterface
public interface Runnable {
    /**
     * When an object implementing interface <code>Runnable</code> is used
     * to create a thread, starting the thread causes the object's
     * <code>run</code> method to be called in that separately executing
     * thread.
     * <p>
     * The general contract of the method <code>run</code> is that it may
     * take any action whatsoever.
     *
     * @see     java.lang.Thread#run()
     */
    public abstract void run();
}
```

```java
//匿名内部类   实现一个接口，实现其方法
Runnable r = new Runnable() {
    public void run() {
        for (int i = 1; i <= 5; i++) {
            System.out.print(i + " ");
        }
    }
};
Thread t = new Thread(r);
t.start();
```

匿名内部类以关键字new开头，后面接接口名以及英文小括号，表示定义一个是实现该接口但没有显示名的类。大括号中的代码重写run方法。



##### 1.1.2 函数式接口

函数式接口是一种*包含单一抽象方法的接口*，但不一定只有一个方法，类通过为接口中的方法提供实现来实现接口。（函数式接口中，抽象方法只能有一个，但是可以有默认方法、静态方法等非抽象方法），可以使⽤@FunctionalInterface标记该接⼝为函数式接⼝。

```java
@FunctionalInterface
public interface FilenameFilter {
    /**
     * Tests if a specified file should be included in a file list.
     *
     * @param   dir    the directory in which the file was found.
     * @param   name   the name of the file.
     * @return  <code>true</code> if and only if the name should be
     * included in the file list; <code>false</code> otherwise.
     */
    boolean accept(File dir, String name);
}
```

java8之前，接口中的所有方法都被默认为抽象方法，可以省略abstract关键字，由于接口中的所有方法都是public方法，可以省略访问修饰符。

@FunctionalInterface注解的优点：

​	@FunctionalInterface注解会触发编译时校验，有助于确保接口符合要求。如果接口不包含或包含多个抽象方法，程序将提示编译错误。

```java
@FunctionalInterface
public interface MyInterface {
    void test1();
    //int test2(); 
    
    default String worldDefault(){
        return "This is default method !";
    }

    static String worldStatic(){
        return "This is static method !";
    }
}
```

test1和test2只能存在一个，因为加了@FunctionalInterface注解，而test1和test2都默认为抽象方法，触发了@FunctionalInterface的校验机制。

```java
@FunctionalInterface
public interface MyChildyInterface extends MyInterface{
   int test2();
}
```

一个接口可以继承多个接口，如果一个接口继承现有的函数式接口后，又添加了其他抽象方法后，该接口就不再是函数式接口。因此MyChildyInterface不属于函数式接口，因为它包含两个抽象方法父接口的test1()和现有的test2()，添加@FunctionalInterface注解后会提示编译错误。

**特点：**

**（1）有且仅有一个抽象方法(接口是隐式抽象的，当声明一个接口的时候，不必使用abstract关键字)** 

**（2）但可以有default、static方法，由于这两种方法都有相应的实现，所以与“有且仅有一个抽象方法”并不矛盾**

**（3）可继承Object类中的public方法（因为任何一个类都继承Object类，包含了来自 java.lang.Object里对这些抽象方法的实现，也不属于抽象方法）** 

**（4）函数式接口里允许子接口继承多个父接口，但每个父接口中都只能存在一个抽象方法，且必须是相同的抽象方法。**



### 1.2 Lambda表达式

lambda可以理解为是jdk8中⼀个语法糖，它可以对某些匿名内部类的写法进⾏简化，它是函数式编程思想的⼀个重要体现。lambda表达式不关注是什么对象，更关注对数据进⾏了什么操作。lambda允许把函数作为参数传⼊到另⼀个⽅法中。简而⾔之，可以看成是对匿名内部类的简写，使⽤lambda表达式时，接⼝必须是函数式接⼝。 

```java
//匿名内部类   实现一个接口，实现其方法
Runnable r = new Runnable() {
    public void run() {
        for (int i = 1; i <= 5; i++) {
            System.out.print(i + " ");
        }
    }
};
Thread t = new Thread(r);
t.start();

//Lambda表达式
Runnable r1 = () ->{
    for (int i = 1; i <= 5; i++) {
        System.out.print(i + " ");
    }
};
Thread t1 = new Thread(r);
t1.start();
```

lambda表达式必须匹配接口中单一抽象方法的参数类型和返回类型。lambda表达式属于函数式接口中单一抽象方法的实现，可以将lambda表达式视为实现接口的匿名内部类的主体，这就是lambda表达式必须与抽象方法兼容的原因，其参数类型和返回类型必须匹配该方法的签名。所实现的方法的名称并不重要，它不会作为lambda表达式语法的一部分。

比如：在上面lambda表达式中，并没有看到实现的Runnable接口的抽象方法run()的方法名称，但是该lambda表达式的参数类型和返回类型和抽象方法run()是相同的，该表达式小括号里面没有参数，表示是无参，同时print()方法返回的是void，和Runnable接口中抽象方法run()的参数类型和返回值一样。

FilenameFilter的匿名内部类实现及Lambda表达式：

```java
//FilenameFilter的匿名内部类实现
File file = new File("D:\\Study\\project\\LambdaAndFunction\\src\\main\\java\\com\\zzy\\lambdaandfunction\\LambdaDemo");
String[] names = file.list(new FilenameFilter() {
    @Override
    public boolean accept(File dir, String name) {
        return name.endsWith(".java");
    }
});
System.out.println(Arrays.asList(names));


//Lambda表达式实现FilenameFilter接口
File file1 = new File("D:\\Study\\project\\LambdaAndFunction\\src\\main\\java\\com\\zzy\\lambdaandfunction\\LambdaDemo");
String[] names1 = file1.list( (dir, name) -> name.endsWith(".java")  );
System.out.println(Arrays.asList(names1));
```

参数包含在小括号中，但并未声明类型。在编译时，编译器发现list方法传入一个FilenameFilter类型的参数，从而获知其单一抽象方法accept()的签名，进而了解accept()的参数为File和String，因此兼容lambda表达式匹配的类型。由于accept()方法的返回类型是布尔值，所以lambda表达式返回值也是布尔值。

lambda表达式的优化：

```java
//1.无参无返回值
void test1();  //抽象方法

MyInterface my1 = () -> {
    System.out.println("lambda优化1");
};
//lambda优化2 如果{}中执⾏语句只有1句，且⽆返回值，{}可以省略
MyInterface my2 = () -> System.out.println("lambda优化2");


//2.无参有返回值
int test2();  //抽象方法

MyInterface my3 = () -> {
    return 1;
};
//lambda优化2 如果{}中执⾏语句只有1句，且有返回值，{}可以省略，但是必须同时省略return
MyInterface my4 = () -> 2;



//3.有参无返回值
void test3(int a);  //抽象方法

MyInterface my5 = (int a) -> {
    System.out.println("lambda优化"+ a);
};
//lambda优化2 如果形参列表只有⼀个参数可以省略，数据类型和()
MyInterface my6 = a -> System.out.println("lambda优化"+ a);


 //4.有参有返回值
int test4(int a, int b, int c);  //抽象方法

MyInterface my7 = (int a, int b) -> {
    return a + b ;
};
//lambda优化2 形参列表数据类型可以省略，程序可以⾃⼰推断
 MyInterface my8 = (a, b) -> a + b ;
```

小括号中参数类型可以省略，如果lambda表达式中，函数体只有一个语句，可以不需要使用{}包裹，可以省略return。如果有多个语句，则必须使用{}包裹起来，并且不能省略return关键字。

**特点：**

**无参数，无返回值 （）-> System.out.println("aaaaaaaa");** 

**一个参数，并无返回值 （x）-> System.out.println(x);**

**只有一个参数，小括号可以不写 x -> System.out.println(x);** 

**有两个以上的参数，并且Lambda体中有多条语句 （x,y）-> x+y ;**

**如果Lambda体中只有一条语句，那么return 和大括号都可以省略不写;** 

**Lambda表达式的参数列表的数据类型可以省略不写，因为JVM编译器通过上下文推断出数据 类型，即”类型推断”**



总结：lambda表达式属于函数式接口中单一抽象方法的实现。它返回一个指定接口类型的对象实例。

lambda表达式既可以是方法的参数，也可以是方法的返回类型，还可以被赋给引用。

```java
//1、lambda表达式作为方法的参数
File file1 = new File("D:\\Study\\project\\LambdaAndFunction\\src\\main\\java\\com\\zzy\\lambdaandfunction\\LambdaDemo");
String[] names1 = file1.list((dir, name) -> name.endsWith(".java"));
System.out.println(Arrays.asList(names1));

//2、lambda表达式作为方法的返回类型
public static void main(String[] args) {
   String[] array={"abc", "jk", "aghjghj", "u"};
   System.out.println(Arrays.toString(array));
   //排序,长度由大到小输出
   //Arrays.sort(array, newComparator());
   Arrays.sort(array, (a, b) -> b.length() - a.length());
   System.out.println(Arrays.toString(array));
}

// 函数式接口 作为返回值类型  返回的是 其实例 实现类对象
private static Comparator<String> newComparator() {
    //return 的是函数式接口的实例
    return (a, b) -> b.length() - a.length();
}

//3、lambda表达式赋给引用
Runnable r1 = () ->{
    for (int i = 1; i <= 5; i++) {
        System.out.print(i + " ");
    }
};
```



### 1.3 方法引用

方法引用（ Method References ）是一种语法糖 ，它**本质上就是Lambda 表达式**，Lambda 表达式是函数式接口的实例，所以说方法引用也是函数式接口的实例。

若lambda体中的内容有方法已经实现了，我们可以使用“方法引用”,方法引用是Lambda表达式 的另外一种表现形式,属于lambda表达式的一种简化语法。（lambda表达式和方法引用在任何情 况下都不能脱离上下文存在，上下文指定了将表达式赋给哪个函数式接口。）

如果说lambda表达式本质上是将方法作为对象进行处理，那么方法引用就是将现有方法作为lambda表达式进行处理。

方法引用的一般语法是:    Qualifier::MethodName

使用双冒号（::）将实例引用或类名与方法分开。

`MethodName `是方法的名称。

`Qualifier`告诉哪里的方法。

```java
List<Integer> list1 = Arrays.asList(82,22,34,50,9);
List<Integer> list2 = Arrays.asList(82,22,34,50,9);
//使用lambda表达式
list1.sort( (a,b) -> a-b );
//使用方法引用：对一个Integer列表进行排序，因为Integer中已经存在静态的比较方法compare()，因此可以直接用静态方法引用的方式来调用
list2.sort(Integer::compare);
System.out.println(list1);
System.out.println(list2);

//[9, 22, 34, 50, 82]
//[9, 22, 34, 50, 82]
```

list1.sort( (a,b) -> a-b )是对Comparator接口的compare方法进行实现，而list2.sort(Integer::compare)是引用现有的Integer的compare方法。

##### 1.3.1 静态方法的引用

静态方法的方法引用：Class::staticMethod

```java
Stream.generate(()->Math.random())
        .limit(10)
        .forEach(f->System.out.println(f));
```

```java
public static double random() {
        return RandomNumberGeneratorHolder.randomNumberGenerator.nextDouble();
}
```

generate(Supplier<T> s)方法传入Supplier【供给型接口】这个函数式接口作为参数，get()方法不传入参数，产生一个结果，和Math类的random()签名兼容。

```java
Stream.generate(Math::random)
      .limit(10)
      .forEach(System.out::println);
```



因此方法引用  Math::random  表示该方法是Supplier接口的实现。



##### 1.3.2 特定类型的任意对象的实例方法的方法引用

实例方法的方法引用：Class::instanceMethod

（1）String.length()

通过类名调用实例方法，当上下文提供s值时，它将用作方法的目标而不是方法的参数。

```java
Stream.of("i","have","an" ,"apple")
        .map( s->s.length()  )
        .forEach( f->System.out.println(f) );
        
Stream.of("i","have","an" ,"apple")
        .map(String::length)
        .forEach(System.out::println);
```



（2）多参数实例方法

如果通过类名引用一个传入多个参数的方法，则上下文提供的第一个参数将作为方法的目标，第二个参数将作为方法的参数。

```java
//方法引用语法调用第一个元素的compareTo（）方法，将第二个参数作为该方法的参数
List<String> collect1 = Arrays.asList("i", "have", "an", "apple").stream()
        .sorted((s1, s2) -> s1.compareTo(s2))
        .collect(Collectors.toList());
System.out.println(collect1);

List<String> collect2 = Arrays.asList("i", "have", "an", "apple").stream()
        .sorted(String::compareTo)
        .collect(Collectors.toList());
System.out.println(collect2);
```



#### 1.3.3 引用特定对象的实例方法

引用特定对象的实例方法: Object::instanceMethod

如果通过对象传入参数，则上下文提供的参数用作方法的参数。

```java
//特定对象的实例方法引用适用于lambda表达式的主体中仅仅调用了某个对象的某个实例方法的场景。
Arrays.asList(new String[] {"a", "c", "b"}).stream().forEach(s -> System.out.println(s));

Arrays.asList(new String[] {"a", "c", "b"}).stream().forEach(System.out::println);
```



##### 1.3.4 构造函数的引用

构造函数的引用： ClassName::new

根据字符串列表来创建相应的Person引用列表  -->  将每一个字符串映射到Person类

```java
List<String> names =
        Arrays.asList("Grace Hopper", "Barbara Liskov", "Ada Lovelace",
                "Karen Spärck Jones");
List<Person> people1 = names.stream()
        .map(name -> new Person(name))
        .collect(Collectors.toList());

List<Person> people2 = names.stream()
        .map(Person::new)
        .collect(Collectors.toList());
System.out.println(people1);
System.out.println(people2);

//[Person{name='Grace Hopper'}, Person{name='Barbara Liskov'}, Person{name='Ada Lovelace'}, Person{name='Karen Spärck Jones'}]
//[Person{name='Grace Hopper'}, Person{name='Barbara Liskov'}, Person{name='Ada Lovelace'}, Person{name='Karen Spärck Jones'}]
```

需要调用的构造器的参数列表要与函数式接口中抽象方法的参数列表保持一致。

Person::new 的作用是引用Person类中的构造函数，执行Person类中的哪个构造函数，由上下文决定，由于上下文提供一个字符串，所以执行由一个参数传入的String构造函数。



##### 1.3.5 数组构造方法引用

数组构造方法引用：数据类型[]::new

```java
List<String> names =Arrays.asList("Grace Hopper", "Barbara Liskov", "Ada Lovelace","Karen Spärck Jones");
Person[] array = names.stream()
        .map(Person::new)
        .toArray(Person[]::new);
for (Person person:array) {
    System.out.println(person);
}
```

toArray方法传入的参数为数组对象，

toArray 方法参数创建了一个大小合适的 Person 引用数组，并采用经过实例化的 Person 实例进行填充。



##### 1.3.6 调用父类或本类方法引用

调用父类或本类方法引用：this::方法名   super::方法名

```java
//接口
public interface MyInterface2 {
    void testMethod();
}

//父类
public class father {
    public void say(){
        System.out.println("father");
    }
}

//子类
public class son extends father{
    @Override
    public void say() {
        System.out.println("son");
    }

    public void test(){
        //lambda常规写法
        MyInterface2 my3 = ()->this.say();
        my3.testMethod();
        MyInterface2 my4 = ()->super.say();
        my4.testMethod();
        //调⽤⽗类或本类⽅法引⽤
        MyInterface2 my5 = this::say;
        my5.testMethod();
        MyInterface2 my6 = super::say;
        my6.testMethod();
    }
}

//调用父类或本类方法引用
son son = new son();
son.test();

```

子类的test()方法中，创建一个MyInterface2对象，要实现MyInterface2接口中的testMethod()方法，该方法没有参数传入，也没有返回值。this.say()调用子类的say()方法，super.say()调用父类的say()方法。



# 2.四大接口

Java标准库中的许多接口仅包含一个抽象方法，他们属于函数式接口，因此，Java8专门定义了java.util.function包。java.util.function包中的接口分为四类：分别是Consumer（消费型接口）、Supplier（供给型接口）、Predicate（谓词型接口）、Function（函数型接口）。

### 2.1 consumer接口（消费型接口）

单一抽象方法为：void accept(T t);

默认方法： andThen(Consumer after)

| 方法                                     | 描述                             |
| ---------------------------------------- | -------------------------------- |
| void accept(T t)                         | 对给定的参数执行此操作           |
| default Consumer andThen(Consumer after) | 当前Consumer执行后, 要执行的操作 |



```java
@FunctionalInterface
public interface Consumer<T> {

    void accept(T t);

    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
}
```

##### 2.1.1 accept()

方法传入一个泛型参数并返回void

```java
Consumer<String> con = str -> System.out.println(str);
con.accept("Consumer是一个消费型函数式接口");
```

##### 2.1.2 andThen()

在当前Consumer后添加操作的方法，该方法生成一个组合的Consumer, 它依次执行此Consumer和after参数提供的操作。andThen()所用组合中的Consumer都共用同一个输入。andThen接收一个Consumer接口，返回的也是一个Consumer接口

```java
Consumer<String> question = q1->System.out.println("A: 请问您是 " + q1 + " 吗?");

Consumer<String> answer = a1-> System.out.println("B: 是的, 我是 " + a1);

Consumer<String> confirm = c1-> System.out.println("A: 啊? 真的嘛? 您真的是 " + c1 + " 吗?");

Consumer<String> angery = a2-> System.out.println("B: 当然了, 我就是 " + a2 + ". 你要我说几遍?");

//组合的 Consumer 公用了相同的输入
question.andThen(answer).andThen(confirm).andThen(angery).accept("张三");

运行结果：
A: 请问您是 张三 吗?
B: 是的, 我是 张三
A: 啊? 真的嘛? 您真的是 张三 吗?
B: 当然了, 我就是 张三. 你要我说几遍?
```

当调用 andThen方法的时候，并不是执行 `(T t) -> { accept(t); after.accept(t); }` 这段代码，而是返回了一个 Consumer 接口，注意它的结构是给一个对象 t，然后大括号中消费 t。处理逻辑是当前 Consumer 执行 accept 方法，然后再让 after这个 Consumer 执行 accept方法。理解 andThen 方法返回的结构特别重要。

相当于分别执行了一次  `question.accept()； answer.accept()； confirm.accept() ； angery.accept()；`。

如果执行任一操作引发异常, 则将其转发给组合操作的调用者，如果执行当前Consumer抛出异常, 则不会执行after操作。如果after参数为null抛出异常NullPointerException。



##### 2.1.3 其他Consumer接口

java.util.function包定义了三种Consumer<T>的基本变体以及一种双参数形式。

| 接口           | 单一抽象方法          |
| -------------- | --------------------- |
| IntConsumer    | void accept(int x)    |
| DoubleConsumer | void accept(double x) |
| LongConsumer   | void accept(long x)   |
| BiConsumer     | void accept(T t,U u)  |

基本变体的Consumer接口，指定了accept方法的参数类型。

```java
//声明函数对象 IntConsumer
IntConsumer intConsumer = i -> System.out.println(i);
intConsumer.accept(123);
```

IntConsumer接口的accept方法 `void accept(int value);` 指定了输入参数为int型，如果输入参数为其他类型，则编译会报错。

BiConsumer是一种双参数形式接口，BiConsumer接口的accept()方法传入两个泛型参数，这两个泛型参数应为不同的类型。

```java
// 声明函数对象 biconsumer
BiConsumer<Integer,String> biconsumer = (int1, str2) -> System.out.println("参数1为整型，值为: "+int1+",参数2为字符串型，值为："+str2);

// BiConsumer.accept()方法接收参数
biconsumer.accept(20,"BiConsumer.accept()方法接收参数");
```

java.util.function 包定义了`BiConsumer`接口的三种变体，每种变体的第二个参数为基本数据类型。以 ObjIntConsumer 接口为例，accept方法传入两个参数，分别为泛型参数和int参数。ObjLongConsumer和ObjDoubleConsumer接口的定义与ObjIntConsumer类似。

```java
// 声明函数对象 consumer1
ObjIntConsumer<String> consumer1 = (str1,int1) -> System.out.println("参数为字符串型，值为："+str1+",第二个参数为整型值："+int1);

// BiConsumer.accept()方法接收参数
consumer1.accept("ObjIntConsumer.accept()方法接收参数",20);
```



### 2.2 Supplier接口（供给型接口）

单一抽象方法为： T get();

| ***方法*** | 描述     |
| ---------- | -------- |
| T get()    | 获得结果 |



##### 2.2.1 get()

get()方法不接受任何参数，只返回泛型类型的值。

调用Supplier接口时，不要求每次都返回一个新的或不同的结果。

```java
Supplier<Double> randomValue = () -> Math.random();
System.out.println("获取随机数：" + randomValue.get());
System.out.println("再次获取随机数：" + randomValue.get());

运行结果：
获取随机数：0.3172168289009717
再次获取随机数：0.31141812585953
```



##### 2.2.3 其他Supplier接口

| 接口            | 单一抽象方法           |
| --------------- | ---------------------- |
| IntSupplier     | int getAsInt()         |
| DoubleSupplier  | double getAsDouble()   |
| LongSupplier    | long getAsLong()       |
| BooleanSupplier | boolean getAsBoolean() |

IntSupplier接口包含的单一抽象方法为 getAsInt，它返回一个 Integer 型数据。其他接口和IntSupplier类似。

```java
IntSupplier randomIntValue = ()-> (int) (Math.random()*10);
System.out.println("获取数：" + randomIntValue.getAsInt());
```

### 2.3 Predicate接口（断言型接口）

单一抽象方法： boolean test(T t)

默认方法： and(Predicate other) 

​		  or(Predicate other) 

​		  negate(Predicate other) 

静态方法：isEqual()

用途 判断对象是否符合某个条件（传入一个参数，返回一个布尔值）

| 方法                                       | 描述                                                         |
| ------------------------------------------ | ------------------------------------------------------------ |
| boolean test(T t)                          | 在给定的参数上评估这个谓词                                   |
| default Predicate and(Predicate other)     | 接收一个Predicate类型，也就是将传入的条件和当 前条件以并且的关系过滤数据 |
| default Predicate negate()                 | 返回表示此谓词的逻辑否定的谓词。                             |
| default Predicate or(Predicate other)      | 接收一个Predicate类型，将传入的条件和当前的条 件以或者的关系过滤数据。 |
| static Predicate isEqual(Object targetRef) | 根据 Objects.equals(Object) 测试两个参数是否相等。           |



##### 2.3.1 test()

```
boolean test(T t);
```

test()方法传入一个泛型参数，返回一个布尔值。

```java
Predicate<String> predicate = str -> str.length() > 7;
System.out.println("长度是否大于7："+predicate.test("scbjacbskldjcnosd"));//true
System.out.println("长度是否大于7："+predicate.test("uhsdk"));//false
```

##### 2.3.2 and()

```java
default Predicate<T> and(Predicate<? super T> other) {
    Objects.requireNonNull(other);
    return (t) -> test(t) && other.test(t);
}
```

and() 生成一个组合断言, 表示该断言和另一个断言的短路逻辑与(AND).

在评估组合断言时, 如果当前断言为 `false`, 则不再对另一个断言进行评估.

在评估任一断言期间抛出的任何异常都将转发给调用者; 如果在前断言的求值过程中发生了异常, 则不再对另一个断言求值.

```java
//and()
Predicate<String> p1 = p -> p.contains("H");
Predicate<String> p2 = p -> p.length() > 5;
boolean isValid = p1.and(p2).test("Hello world");//true&&true -> true
System.out.println("是否有效："+isValid);
```



##### 2.3.3 or()

```java
default Predicate<T> or(Predicate<? super T> other) {
    Objects.requireNonNull(other);
    return (t) -> test(t) || other.test(t);
}
```

or()生成一个组合断言, 表示该断言和另一个断言的短路逻辑或(OR).

在评估组合断言时, 如果当前断言为 `true`, 则不再对另一个断言进行评估.

在评估任一断言期间抛出的任何异常都将转发给调用者; 如果在前断言的求值过程中发生了异常, 则不再对另一个断言求值.

```java
//or()
Predicate<String> o1 = p -> p.contains("k");
Predicate<String> o2 = p -> p.length() > 20;
boolean isValid2 = o1.or(o2).test("Hello world");//等同于 return p1.test(s) || p2.test(s) -> false;
System.out.println("是否有效2："+isValid2);

```

##### 2.3.4 negate()

```java
default Predicate<T> negate() {
    return (t) -> !test(t);
}
```

negate()获取一个否定当前断言的断言

```java
//negate()
Predicate<String> n1 = p -> p.length() > 20;
boolean isValid3 = n1.negate().test("Hello world");//等同于 return !p1.test(s) ;
System.out.println("是否有效3："+isValid3);
```

##### 2.3.5 isEqual()

```java
static <T> Predicate<T> isEqual(Object targetRef) {
    return (null == targetRef)
            ? Objects::isNull
            : object -> targetRef.equals(object);
}
```

生成一个断言, 该断言根据 `Objects.equals(Object)` 测试两个参数是否相等。

```java
//isEqual()
Predicate<String> e = Predicate.isEqual("abc");
System.out.println("abc isEqual abc:"+e.test("abc"));
System.out.println("abc isEqual bcd:"+e.test("bcd"));
```



### 2.4 Function接口（功能型接口）

单一抽象方法： R apply(T t)

默认方法： compose(Function before) 

​		  andThen(Function after)

静态方法：identity() 

| 方法                                      | 描述                                                         |
| ----------------------------------------- | ------------------------------------------------------------ |
| R apply(T t)                              | 将此函数应用于给定的参数。                                   |
| default Function andThen(Function after)  | 返回一个组合函数，首先将该函数应用于其输入，然后将 after函数应用于结果（先执行当前对象的apply方法，再执行after对象的方法）。 |
| default Function compose(Function before) | 返回一个组合函数，首先将 before函数应用于其输入，然后 将此函数应用于结果（先执行before对象的apply，再执行当前对象的apply，将两个执行逻辑串起来）。 |
| static Function identity()                | 返回其输入参数的函数                                         |



##### 2.4.1 apply()

```java
 R apply(T t);
```

apply方法将T类型的泛型输入参数转换为R类型泛型输出值。

```java
Function<Integer,Integer> f1 = f -> f*2;
Function<Integer,Integer> f2 = f -> f*f;
Function<Integer,Integer> f3 = f -> f+10;

//apply
System.out.println(f1.apply(3));//执行f1.apply(3)方法，----> 3*2=6
List<String> names = Arrays.asList("Mal", "Wash", "Kaylee", "Inara", "Zoë", "Jayne", "Simon", "River", "Shepherd Book");
List<Integer> list = names.stream()
           .map(s -> {
                  return  s.length();//apply的传入参数为s，Function的apply方法的实现为输出s的长度，返回s的长度
             })
            .collect(Collectors.toList());
System.out.println("list:"+list);
```

map方法传入的对象为Function类型，apply方法的实现为将流中的元素通过lambda表达式将每个元素都转化为元素的长度，然后返回。



##### 2.4.2 andThen()

```java
default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
    Objects.requireNonNull(after);
    return (T t) -> after.apply(apply(t));
}
```

andThen()生成一个组合函数, 该函数首先将当前Function作为其输入, 然后将after函数应用于输入后产生的结果。

返回一个使用当前Function作为输入, 再应用after参数生成结果的Function

andThen()方法主要用于组装两个函数生成一个新的函数, 这个函数为原有函数的结果提供更强的快捷的拓展的运算的方法.

```java
//andThen()
//先执行 3*2=6，再执行 6*6=36
System.out.println(f1.andThen(f2).apply(3));
System.out.println(f2.apply(f1.apply(3)));

//先执行 3*2=6，再执行 6*6=36 ，最后执行 36+10=46
System.out.println(f1.andThen(f2).andThen(f3).apply(3));
System.out.println(f3.apply  ( f2.apply(  f1.apply(3)   ) ) );
```

执行f1.andThen(f2).apply(3)相当于执行f2.apply(f1.apply(3))，f2.apply(f1.apply(3))先执行括号里面的result=f1.apply(3),再将该结果作为传入参数，执行f2.apply(result)。使用andThen()可以避免无限嵌套，比如：f3.apply(f2.apply(f1.apply(3)))。

##### 2.4.3 compose()

```java
default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
    Objects.requireNonNull(before);
    return (V v) -> apply(before.apply(v));
}
```

| 参数类型                          | 参数名称 | 参数描述                        |
| --------------------------------- | -------- | ------------------------------- |
| `类型参数`                        | T        | 输入到函数的类型                |
| `类型参数`                        | R        | 从函数返回的结果的类型          |
| `类型参数`                        | V        | before 函数和组合函数的输入类型 |
| `Function<? super V,? extends T>` | before   | 应用当前函数后再应用的函数      |

生成一个组合函数, 该函数首先将 before 函数应用于当前函数的输入, 然后将当前函数应用于结果.

返回一个先将输入应用于 before 函数, 再将产生的结果应用于当前函数的 `Function`.

`compose()` 方法主要用于组装两个函数生成一个新的函数, 组合的新函数实际上是, before函数为当前 Function 做了前置的处理, 再调用当前 Function.

```java
//先执行 3*3 = 9，再执行 9*2=18
System.out.println(f1.compose(f2).apply(3));
System.out.println(f1.apply(f2.apply(3)));

//先执行 10+3=13，再执行 13*13=169，最后执行 169*2 = 338
System.out.println(f1.compose(f2).compose(f3).apply(3));
System.out.println(f1.apply(f2.apply(f3.apply(3))));
```

##### 2.4.4 identity()

```java
static <T> Function<T, T> identity() {
    return t -> t;
}
```

| 参数类型   | 参数名称 | 参数描述                   |
| ---------- | -------- | -------------------------- |
| `类型参数` | T        | 函数的输入和输出对象的类型 |

生成一个总是返回输入的函数

identity()就是Function接口的一个静态方法。Function.identity()返回一个输出跟输入一样的Lambda表达式对象，等价于形如`t -> t`形式的Lambda表达式。

```java
Map<String, Integer> map = Arrays.asList("a","abc","love")
        .stream()
        .map(Function.identity())
        .map(str -> str)
        .collect(  Collectors.toMap(Function.identity(),
                str -> str.length())  );
System.out.println(map);

Function<String, String> idHello = Function.identity();
String param = "Jack";
System.out.println("是否总是相等? " + idHello.apply(param).equals(param));
param = "猫猫";
System.out.println("是否总是相等? " + idHello.apply(param).equals(param));
```



##### 2.4.5 其他Function接口

| 接口                 | 单一抽象方法                     |
| -------------------- | -------------------------------- |
| IntFunction          | R apply(int value)               |
| DoubleFunction       | R apply(double value)            |
| LongFunction         | R apply(long value)              |
| ToIntFunction        | int applyAsInt(T value)          |
| ToDoubleFunction     | double applyAsDouble(T value)    |
| ToLongFunction       | long applyAsLong(T value)        |
| DoubleToIntFunction  | int applyAsInt(double value)     |
| DoubleToLongFunction | long applyAsLong(double value)   |
| IntToDoubleFunction  | double applyAsDouble(int value)  |
| IntToLongFunction    | long applyAsLong(int value)      |
| LongToDoubleFunction | double applyAsDouble(long value) |
| LongToIntFunction    | int applyAsInt(long value)       |
| BiFunction           | R apply(T t, U u)                |

IntFunction：R apply(int value)  ，apply()接收的参数的int，返回的值为泛型参数

```java
//IntFunction
IntFunction<String> i1 = i -> i+"*4="+i*4;
System.out.println(i1.apply(6));
```

ToIntFunction：int applyAsInt(T value) ，apply()接收的参数为泛型参数，返回的值为int型

```java
//ToIntFunction
ToIntFunction<String> i2 = i -> i.length();
System.out.println("字符串长度为："+i2.applyAsInt("abcdefg"));
```

DoubleToIntFunction：int applyAsInt(double value) ，apply()接收的参数为double型，返回的参数为int型

```java
//DoubleToIntFunction
DoubleToIntFunction i3 = i -> (int) (i*5);
System.out.println(i3.applyAsInt(3.5));
```

BiFunction：R apply(T t, U u);

`BiFunction`是一个函数式接口，它和`Function`函数接口十分相似，它可以接受两个不同类型的参数（泛型T类型和泛型U类型），然后返回一个其他类型的值（泛型R类型）。

```java
//BiFunction
BiFunction<String,String,Integer> b1 = (s1,s2) ->  s1.length()+s2.length();
System.out.println("长度之和为："+b1.apply("abc", "efghj"));
```

其他BiFunction接口：

| 接口               | 单一抽象方法                   |
| ------------------ | ------------------------------ |
| ToIntBiFunction    | int applyAsInt(T t, U u)       |
| ToDoubleBiFunction | double applyAsDouble(T t, U u) |
| ToLongBiFunction   | long applyAsLong(T t, U u)     |

ToIntBiFunction：int applyAsInt(T t, U u)，接收两个不同类型的参数，返回值为int型

```java
//ToIntBiFunction
ToIntBiFunction<String,String> b2 = (s1,s2) ->  s1.length()+s2.length();
System.out.println("长度之和为："+b2.applyAsInt("abc", "efghj"));
```





总结：

1. 

lambda表达式属于函数式接口中单一抽象方法的实现，可以将lambda表达式视为实现接口的匿名 内部类的主体，这就是lambda表达式必须与抽象方法兼容的原因，其参数类型和返回类型必须匹配该方 法的签名。

2. 四大接口：

| 接口              | 单一抽象方法      |
| ----------------- | ----------------- |
| Consumer(消费型)  | void accept(T t)  |
| Supplier(供给型)  | T get()           |
| Predicate(断言型) | boolean test(T t) |
| Function(函数型)  | R apply(T t）     |


