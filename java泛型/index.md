# java泛型


泛型在java中有很重要的地位，在面向对象编程及各种设计模式中有非常广泛的应用..

<!--more-->

# java 泛型详解

# 1. 概述

泛型在java中有很重要的地位，在面向对象编程及各种设计模式中有非常广泛的应用。

泛型是在java 1.5 引入的，随着java 8 的兴起，变得更为复杂。

什么是泛型？为什么要使用泛型？

泛型实质就是一个类型占位符

```
泛型，即“参数化类型”。最熟悉的就是定义方法时有形参，然后调用此方法时传递实参。
那么参数化类型怎么理解呢？
顾名思义，就是将类型由原来的具体的类型换成变量类型--参数化，类似于方法中的变量参数，此时类型也定义成参数形式（可以称之为类型形参），然后在使用/调用时传入具体的类型（类型实参）。

泛型的本质是为了参数化类型（在不创建新的类型的情况下，通过泛型指定的不同类型来控制形参具体限制的类型）。也就是说在泛型使用过程中，操作的数据类型被指定为一个参数，这种参数类型可以用在类、接口和方法中，分别被称为泛型类、泛型接口、泛型方法。
```

# 2. 一个实例及泛型属性



我们每天都在使用泛型！

泛型只能是引用类型！

```java
List<String> arrayList = new ArrayList<String>();
List<Integer> arrayList1 = new ArrayList<Integer>();
...
//实例化时，只能接收一具体类型--引用类型
    
//arrayList.add(100); 在编译阶段，编译器就会报错
    
 
```

注： 引入泛型的这个运算符<>称之为 （diamond operator）--钻石运算符



泛型只在编译阶段有效。看下面的代码： 

```java
List<String> stringArrayList = new ArrayList<String>();
List<Integer> integerArrayList = new ArrayList<Integer>();

Class classStringArrayList = stringArrayList.getClass();
Class classIntegerArrayList = integerArrayList.getClass();

if(classStringArrayList.equals(classIntegerArrayList)){
    Log.d("泛型测试","类型相同");
}     

```

<span style ="color:blue;">zzs注：以上其实是一个类对象！具有相同的属性以及方法签名</span>！

输出结果：`D/泛型测试: 类型相同`。

通过上面的例子可以证明，在编译之后程序会采取去泛型化的措施。也就是说Java中的泛型，只在编译阶段有效。在编译过程中，正确检验泛型结果后，会将泛型的相关信息擦除，并且在对象进入和离开方法的边界处添加类型检查和类型转换的方法。也就是说，泛型信息不会进入到运行时阶段。

**Java 泛型擦除**（类型擦除）是指在编译器处理带泛型定义的类、接口或方法时，会在字节码指令集里抹去全部泛型类型信息，泛型被擦除后在字节码里只保留泛型的原始类型（raw type）。

**原始类型**是指抹去泛型信息后的类型，在 Java 语言中，它必须是一个引用类型（非基本数据类型），一般而言，它对应的是泛型的定义上界。

示例：<T> 中的 T 对应的原始泛型是 Object，<T extends String> 对应的原始类型就是 String。

# 3. 泛型的使用

泛型有三种使用方式，分别为：泛型类、泛型接口、泛型方法

## 3.1 泛型类

泛型类型用于类的定义中，被称为泛型类。通过泛型可以完成对一组类的操作对外开放相同的接口。最典型的就是各种容器类，如：List、Set、Map。

泛型类的最基本写法（下面的例子中详解）：

```java
class 类名称 <泛型标识：可以随便写任意标识号，标识指定的泛型的类型>{
  private 泛型标识 /*（成员变量类型）*/ var; 
  .....

  }
}
```

一个最普通的泛型类：

```java
//此处T可以随便写为任意标识，常见的如T、E、K、V等形式的参数常用于表示泛型
//在实例化泛型类时，必须指定T的具体类型
public class Generic<T>{ 
    //key这个成员变量的类型为T,T的类型由外部指定  
    private T key;

    public Generic(T key) { //泛型构造方法形参key的类型也为T，T的类型由外部指定
        this.key = key;
    }

    public T getKey(){ //泛型方法getKey的返回值类型为T，T的类型由外部指定
        return key;
    }
}
```

 

```java
//泛型的类型参数只能是引用类型（包括自定义类），不能是简单类型
//传入的实参类型需与泛型的类型参数类型相同，即为Integer.
Generic<Integer> genericInteger = new Generic<Integer>(123456);

//传入的实参类型需与泛型的类型参数类型相同，即为String.
Generic<String> genericString = new Generic<String>("key_vlaue");
Log.d("泛型测试","key is " + genericInteger.getKey());
Log.d("泛型测试","key is " + genericString.getKey());
```

 

```java
12-27 09:20:04.432 13063-13063/? D/泛型测试: key is 123456
12-27 09:20:04.432 13063-13063/? D/泛型测试: key is key_vlaue
```

定义的泛型类，就一定要传入泛型类型实参么？

并不是这样，在使用泛型的时候如果传入泛型实参，则会根据传入特定的泛型实参做相应的限制，此时泛型才会起到本应起到的限制作用。

如果不传入泛型类型实参的话，在泛型类中使用泛型的方法或成员变量定义的类型可以为任何的类型。但很少有使用价值，不建议！

看一个例子：

```java
Generic generic = new Generic("111111");
Generic generic1 = new Generic(4444);
Generic generic2 = new Generic(55.55);
Generic generic3 = new Generic(false);

Log.d("泛型测试","key is " + generic.getKey());
Log.d("泛型测试","key is " + generic1.getKey());
Log.d("泛型测试","key is " + generic2.getKey());
Log.d("泛型测试","key is " + generic3.getKey());
```



```java
D/泛型测试: key is 111111
D/泛型测试: key is 4444
D/泛型测试: key is 55.55
D/泛型测试: key is false
```

 

**注意：**

- 泛型的类型参数只能是引用类型，不能是简单类型。（8种数据类型）.Java以后考虑加入!但也是进行封装自动装拆箱的操作！
- 不能对不确切的泛型类型使用 instanceof 操作。如下面的操作是非法的，编译时会出错。

　　if(ex_num instanceof Generic<Number>){ }

## 3.2 泛型接口

泛型接口与泛型类的定义及使用基本相同。泛型接口常被用在各种类的生产器中，可以看一个例子：

```java
//定义一个泛型接口
public interface Generator<T> {
    public T next();
}
```



当实现泛型接口的类，未传入泛型实参时：

此时与泛型类的定义相同，在声明类的时候，需将泛型的声明也一起加到类中。

```java
/**
 * 未传入泛型实参时，与泛型类的定义相同，在声明类的时候，需将泛型的声明也一起加到类中
 * 即：class FruitGenerator<T> implements Generator<T>{
 * 如果不声明泛型，如：class FruitGenerator implements Generator<T>，编译器会报错："Unknown class"
 */
class FruitGenerator<T> implements Generator<T>{
    @Override
    public T next() {
        return null;
    }
}
```

当实现泛型接口的类，传入泛型实参时：

在实现类实现泛型接口时，如已将泛型类型传入实参类型，则所有使用泛型的地方都要替换成传入的实参类型



```java
/**
 * 传入泛型实参时：
 * 定义一个生产器实现这个接口,虽然我们只创建了一个泛型接口Generator<T>
 * 但是我们可以为T传入无数个实参，形成无数种类型的Generator接口。
 * 在实现类实现泛型接口时，如已将泛型类型传入实参类型，则所有使用泛型的地方都要替换成传入的实参类型
 * 即：Generator<T>，public T next();中的的T都要替换成传入的String类型。
 */
public class FruitGenerator implements Generator<String> {

    private String[] fruits = new String[]{"Apple", "Banana", "Pear"};

    @Override
    public String next() {
        Random rand = new Random();
        return fruits[rand.nextInt(3)];
    }
}
```





## 3.3 泛型方法

在java中,泛型类的定义非常简单，但是泛型方法却略为复杂。

```
尤其是我们见到的大多数泛型类中的成员方法也都使用了泛型，有的甚至泛型类中也包含着泛型方法，这样在初学者中非常容易将泛型方法理解错了。
```

泛型类，是在实例化类的时候指明泛型的具体类型；

泛型方法，是在调用方法的时候指明泛型的具体类型 。

```java
/**
 * 泛型方法的基本介绍
 * @param tClass 传入的泛型实参
 * @return T 返回值为T类型
 * 说明：
 *     1）public 与 返回值中间<T>是必要的，可以理解为：声明此方法为泛型方法。
 *     2）只有使用钻石运算符，声明了<T>的方法才是泛型方法，泛型类中的使用了泛型的成员方法并不是泛型方法。
 *     3）<T>表明该方法将使用泛型类型T，此时才可以在方法中使用泛型类型T。
 *     4）与泛型类的定义一样，此处T可以随便写为任意标识，常见的如T、E、K、V等形式的参数常用于表示泛型。
 */
public <T> T genericMethod(Class<T> tClass) throws InstantiationException ,
  IllegalAccessException{
        T instance = tClass.newInstance();
        return instance;
}
```

具体调用：

```java
Object obj = genericMethod(Class.forName("com.test.test"));
```



如何理解泛型方法的签名格式：

一个泛型方法定义实例：

```java
static <T extends Object & Comparable<? super T>> T max(Collection<? extends T> collection)

```

T的理解：

1、T是有界的。 它既Object或是是Object的子类，又需要实现了Comparable接口。Comparable定义为T或T的任何父类。（Object是类，必须放于首位）！

2、max方法的参数为一个集合Collection实例，它的元素需要是T或T的任何子类的Collection.

3、该方法返回T类型。

譬如Integer， String都是可以的。

因为第一条显然满足。

 第二条：List、 Set等都是Collection, List<Integer>, Set<String>等都满足条件。List<Integer>, Set<String>的实例对象都可以作为参数传入。

### 3.3.1 泛型方法的基本用法

我们再通过一个例子，把泛型方法再总结一下:



```java
public class GenericTest {
   //这个类是个泛型类，在上面已经介绍过
   public class Generic<T>{     
        private T key;

        public Generic(T key) {
            this.key = key;
        }

        //其实这个，虽然在方法中使用了泛型，但是这并不是一个泛型方法。
       //这只是类中一个普通的成员方法，只不过他的返回值是在声明泛型类已经声明过的泛型。
        //所以在这个方法中才可以继续使用 T 这个泛型。
       
        public T getKey(){
            return key;
        }

        /**
         * 下面这个方法显然是有问题的，在编译器会给我们提示这样的错误信息"cannot reslove symbol E"
         * 因为在类的声明中并未声明泛型E，所以在使用E做形参和返回值类型时，编译器会无法识别。
        public E setKey(E key){
             this.key = keu
        }
        */
       
    }

    /** 
     * 这才是一个真正的泛型方法。
     * 首先在public与返回值之间的<T>必不可少，这表明这是一个泛型方法，并且声明了一个泛型T
     * 这个T可以出现在这个泛型方法的任意位置.
     * 泛型的数量也可以为任意多个 
     *    如：public <T,K> K showKeyName(Generic<T> container){
     *        ...
     *        }
     */
    
    public <T> T showKeyName(Generic<T> container){
        System.out.println("container key :" + container.getKey());
        //当然这个例子举的不太合适，只是为了说明泛型方法的特性。
        T test = container.getKey();
        return test;
    }

    //这也不是一个泛型方法，这就是一个普通的方法，只是使用了Generic<Number>这个泛型类做形参而已。
    public void showKeyValue1(Generic<Number> obj){
        Log.d("泛型测试","key value is " + obj.getKey());
    }

    //这也不是一个泛型方法，这也是一个普通的方法，只不过使用了泛型通配符?
    //同时这也印证了泛型通配符章节所描述的，?是一种类型实参，可以看做为Number等所有类的父类
    public void showKeyValue2(Generic<?> obj){
        Log.d("泛型测试","key value is " + obj.getKey());
    }

     /**
     * 这个方法是有问题的，编译器会为我们提示错误信息："UnKnown class 'E' "
     * 虽然我们声明了<T>,也表明了这是一个可以处理泛型的类型的泛型方法。
     * 但是只声明了泛型类型T，并未声明泛型类型E，因此编译器并不知道该如何处理E这个类型。
    public <T> T showKeyName(Generic<E> container){
        ...
    }  
    */

    /**
     * 这个方法也是有问题的，编译器会为我们提示错误信息："UnKnown class 'T' "
     * 对于编译器来说T这个类型并未项目中声明过，因此编译也不知道该如何编译这个类。
     * 所以这也不是一个正确的泛型方法声明。
    public void showkey(T genericObj){

    }
    */

}
```

### 3.3.2 类中的泛型方法

   泛型方法可以出现杂任何地方和任何场景中使用。但是有一种情况是非常特殊的，当泛型方法出现在泛型类中时，我们再通过一个例子看一下

```java
public class GenericFruit {
    class Fruit{
        @Override
        public String toString() {
            return "fruit";
        }
    }

    class Apple extends Fruit{
        @Override
        public String toString() {
            return "apple";
        }
    }

    class Person{
        @Override
        public String toString() {
            return "Person";
        }
    }
    
    class GenerateTest<T>{
        public void show_1(T t){
            System.out.println(t.toString());
        }

        //在泛型类中声明了一个泛型方法，使用泛型E，这种泛型E可以为任意类型。可以类型与T相同，也可以不同。
        //由于泛型方法在声明的时候会声明泛型<E>，因此即使在泛型类中并未声明泛型，编译器也能够正确识别泛型方法中识别的泛型。
        public <E> void show_3(E e){
            System.out.println(e.toString());
        }

        //在泛型类中声明了一个泛型方法，使用泛型T，注意这个T是一种全新的类型，可以与泛型类中声明的T不是同一种类型。
        public <T> void show_2(T t){
            System.out.println(t.toString());
        }
    }

    public static void main(String[] args) {
        Apple apple = new Apple();
        Person person = new Person();

        GenerateTest<Fruit> generateTest = new GenerateTest<Fruit>();
        //apple是Fruit的子类，所以这里可以
        generateTest.show_1(apple);
        
        //编译器会报错，因为泛型类型实参指定的是Fruit，而传入的实参类是Person
        generateTest.show_1(person);

        //使用这两个方法都可以成功
        generateTest.show_2(apple);
        generateTest.show_2(person);

        //使用这两个方法也都可以成功
        generateTest.show_3(apple);
        generateTest.show_3(person);
    }
}
```



### 3.3.3 泛型方法与可变参数

再看一个泛型方法和可变参数的例子：

```java
  public <T> void printMsg( T... args){
        for(T t : args){
            System.out.println( "泛型测试 ---->t is " + t );
            System.out.println( (t instanceof String) ); //zzs 注：看是否字符串
            
        }
    }

printMsg("111",222,"aaaa","2323.4",55.55);
```

### 3.3.4 静态方法与泛型

静态方法有一种情况需要注意一下，那就是在类中的静态方法使用泛型：静态方法无法访问类上定义的泛型，因为实例化后才会给泛型赋具体的类型；

如果静态方法操作的引用数据类型不确定的时候，必须要将泛型定义在方法上。

即：**如果静态方法要使用泛型的话，必须将静态方法也定义成泛型方法 。**



```java
public class StaticGenerator<T> {
    ....
    ....
    /**
     * 如果在类中定义使用泛型的静态方法，需要添加额外的泛型声明（将这个方法定义成泛型方法）
     * 即使静态方法要使用泛型类中已经声明过的泛型也不可以。
     * 如：public static void show(T t){..},此时编译器会提示错误信息：
          "StaticGenerator cannot be refrenced from static context"
     */
    public static <T> void show(T t){

    }
}
```

### 3.3.5 泛型方法总结

泛型方法能使方法独立于类而产生变化，以下是一个基本的指导原则：

```
无论何时，如果你能做到，你就该尽量使用泛型方法。也就是说，如果使用泛型方法将整个类泛型化，那么就应该使用泛型方法。另外对于一个static的方法而已，无法访问泛型类型的参数。所以如果static方法要使用泛型能力，就必须使其成为泛型方法。
```



## 3.4 泛型通配符与PECS



通配符是一种使用（？）的类型参量, 可能存在或也可能不存在上界或下界！

在使用泛型的时候，可以为传入的泛型类型实参进行上下边界的限制，如：类型实参只准传入某种类型的父类或某种类型的子类。

为泛型添加上边界，即传入的类型实参必须是指定类型的子类型

```java
public void showKeyValue1(Generic<? extends Number> obj){
    Log.d("泛型测试","key value is " + obj.getKey());
}
```



```java
Generic<String> generic1 = new Generic<String>("11111");
Generic<Integer> generic2 = new Generic<Integer>(2222);
Generic<Float> generic3 = new Generic<Float>(2.4f);
Generic<Double> generic4 = new Generic<Double>(2.56);

//这一行代码编译器会提示错误，因为String类型并不是Number类型的子类
//showKeyValue1(generic1);

showKeyValue1(generic2);
showKeyValue1(generic3);
showKeyValue1(generic4);
```



如果我们把泛型类的定义也改一下:

就可以限定泛型类实例化的类型了。如：

```java
public class Generic<T extends Number>{
    private T key;

    public Generic(T key) {
        this.key = key;
    }

    public T getKey(){
        return key;
    }
}
```

 

1、无界通配符具有局限性，譬如：

List<?> list = new ArrayList<>();

无法添加元素。但可以读去元素。

最大用途：无界List  所有传入List<?>作为参数的方法（Consummer）会在调用时接收任意列表！如：



```java
private static void printList(List<?> list){

	System.out,println(list);

}
```

 

2、问号是设置类型边界的利器，用法可以灵活多样化！



上界通配符（upper bounded wildcard）：父类定死了，所以称为上界！

使用关键词 extends, 如：

```java
List<? extends Comparable> 
    //表示实参只能接收实现了comparable接口的类或其子类！
```

注意：extends 之后可以是接口！

类型通配符一般是使用？代替具体的类型实参，注意了，此处’？’是类型实参，而不是类型形参 。是一种真实的类型。如：

```java
List<? extends Number> numbers = new ArrayList<>();
//numbrs.add(3);
//是无法添加值的！原因是编译器并不知道其具体类型！
//但可以作为方法定义。使用时再设置具体的类型！

```

我们通过定义上界指定变量必须符合的类型，以便方法实现能够正常工作。譬如：以上需要对数字求和时，只有Number及子类才具有求和的方法！



下界通配符（lower bounded wildcard）：子类定死了，所以称为下界！

使用关键词 super, 如：

```java
private static int mylist(List<? super Number>) 
    //可以是Number，Object
    
```

List<? super Integer> 中， 
    List<Integer>、List<Number>、List<Object>都符合要求。

使用以上下界通配符一位着列表用于存储整数，但方法允许在任何超类中的列表中使用引用。



PECS原则：（Producer Extends，Consumer Super）

如果泛型（参数化类型）代表的是生产者（producer）则使用extends，如果代表消费者（consumer）则使用super！如果同时代表两者，则无需使用通配符！

- 从泛型代表的数据体对象获取（输出）值时，使用extends；

- 向泛型代表的数据体对象写入（输入）值时，使用super；

- 需要同时读取和写入，使用显式类型；

- 例子：

- ```java
  map(Function<? super T, ? extends R>)
      //对流中每个元素(T)类型（输入参数---消费类型）应用mapper方法，将其转换成R类型（输出参数--生产类型）.
  ```

  注：<span style="color:blue;">从PECS原则看，Function接口定义或许令人困惑，似乎类型是反向的！不过只要记住FUnction<T，R>消费T并产生R,就容易理解为何super后跟T,extends后跟R!</span>

  

  

多重边界（multiple bound）" &"

接口作为边界的数量并无限制，但只能有一个类边界，且类边界必须居于首位！



//上边界 在读取T这个这个类型数据的时候，但不写入数据的时候i，使用上边界·

//下边界 需要写入数据的时候，但不需要读取数据的时候



## 3.6 关于泛型数组要提一下(参照)

看到了很多文章中都会提起泛型数组，经过查看sun的说明文档，在java中是”不能创建一个确切的泛型类型的数组”的。

也就是说下面的这个例子是不可以的：

```java
List<String>[] ls = new ArrayList<String>[10];  
```

而使用通配符创建泛型数组是可以的，如下面这个例子：

```java
List<?>[] ls = new ArrayList<?>[10]; 
```

这样也是可以的：

```java
List<String>[] ls = new ArrayList[10];
```

下面使用[Sun](http://docs.oracle.com/javase/tutorial/extra/generics/fineprint.html)[的一篇文档](http://docs.oracle.com/javase/tutorial/extra/generics/fineprint.html)的一个例子来说明这个问题：

```java
List<String>[] lsa = new List<String>[10]; // Not really allowed.    
Object o = lsa;    
Object[] oa = (Object[]) o;    
List<Integer> li = new ArrayList<Integer>();    
li.add(new Integer(3));    
oa[1] = li; // Unsound, but passes run time store check    
String s = lsa[1].get(0); // Run-time error: ClassCastException.
```



```
这种情况下，由于JVM泛型的擦除机制，在运行时JVM是不知道泛型信息的，所以可以给oa[1]赋上一个ArrayList而不会出现异常，但是在取出数据的时候却要做一次类型转换，所以就会出现ClassCastException，如果可以进行泛型数组的声明，上面说的这种情况在编译期将不会出现任何的警告和错误，只有在运行时才会出错。

而对泛型数组的声明进行限制，对于这样的情况，可以在编译期提示代码有类型安全问题，比没有任何提示要强很多。
```

 

下面采用通配符的方式是被允许的:数组的类型不可以是类型变量，除非是采用通配符的方式，因为对于通配符的方式，最后取出数据是要做显式的类型转换的。



```java
List<?>[] lsa = new List<?>[10]; // OK, array of unbounded wildcard type.    
Object o = lsa;    
Object[] oa = (Object[]) o;    
List<Integer> li = new ArrayList<Integer>();    
li.add(new Integer(3));    
oa[1] = li; // Correct.    
Integer i = (Integer) lsa[1].get(0); // OK 
```



# 4. 最后

本文中的例子主要是为了阐述泛型中的一些思想而简单举出的，并不一定有着实际的可用性。另外，一提到泛型，相信大家用到最多的就是在集合中，其实，在实际的编程过程中，自己可以使用泛型去简化开发，且能很好的保证代码质量。



另：

```java
public class Father {
    void test(Object o){}
}
class Son<T> extends Father{
     void test(T o){}//编译错误！
}
```



这段代码会报一个编译错误，both methods have same erasure, yet neither overrides the other。

这个错误的意思是，两个方法在类型擦除后，具有相同的原生类型参数列表，但是也不能覆盖另一个方法。

泛型类型在编译后，会做类型擦除，只剩下原生类型。如参数列表中的T类型会编译成Object，但是会有一个Signature。

尽管两个test方法具有相同的字节码，但是类型参数信息用 一个新的签名（signature） 属性记录在类模式中。JVM 在装载类时记录这个签名信息，并在运行时通过反射使它可用。

这就导致了这个方法既不能作为覆盖父类test方法的方法，也不能作为test方法的重载。

