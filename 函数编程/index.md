# 函数式编程和Lambda表达式


函数式编程和Lambda表达式...

<!--more-->

#  函数式编程/Lambda表达式

不用关心实现细节，调用系统API实现相关需求。

## Lambda表达式

eg:

```java
public static void main(String[] args) {
    Runnable ok = new Runnable() {
        @Override
        public void run() {
            System.out.println("ok");
        }
    };
    new Thread(ok).start();
    //jdk lambda
    new Thread(()-> System.out.println("ok")).start();
    //等于
    Runnable runnable = ()-> System.out.println("ok"); //箭头左边方法参数，右边方法执行体
    Runnable runnable2 = ()-> System.out.println("ok");
    System.out.println(runnable == runnable2);//false,每次返回新的对象实例
    new Thread(runnable).start();

    //
      Object o = (Runnable)()-> System.out.println("ok");//lambda表达式需要你告诉它实现的函数接口类型
      Runnable runnable3 = ()-> System.out.println("ok");

      //总结：lambda表达式返回一个指定接口类型的对象实例。
}
```

demo:

```java
@FunctionalInterface
interface Interface1{
    int doubleNum(int i);
    default int add(int x,int y){
        return x+y;
    }
}
public class LambdaDemo1 {
    public static void main(String[] args) {
        //写法
        Interface1 i1 = (i)-> i*2;
        Interface1 i2 = i -> i*2;             //1行默认有个return
        Interface1 i3 = (int i) -> i*2;
        Interface1 i4 = (i) ->{
            System.out.println("多行写法");
            return i*2;
        };
        i1.add(3,5);         //默认方法可以使用。
    }
```

总结：

```Java
//接口里只有一个需要实现的方法
//JDK8中新增了接口特性
// 新增一个@FunctionalInterface,限制了当前接口是一个函数接口，意思是只能含有一个未实现方法（单一职责）
// 接口里可以含有一个default方法，(当接口新增默认实现default的时候不用去修改已经实行此接口的类)
```

### 函数接口：

```Java
interface FormatMoney{
    String formatMoney(int i);
}

class MyMoney{
    int money;
    public MyMoney(int money){
        this.money = money;
    }
    public void printMoney(FormatMoney forM){
        System.out.println("我的存款："+forM.formatMoney(money));
    }
    public void printMoney2(Function<Integer,String> function){     
        //JDK8新提供的函数接口，只需告诉入参和出参，不用自己再去定义一个接口2.函数接口可支持链式操作
        System.out.println("我的存款2："+function.apply(money));
    }
}

public class MoneyDemo {
    public static void main(String[] args) {
       MyMoney money = new MyMoney(99999);
       money.printMoney(i -> new DecimalFormat("#,###").format(i));

       money.printMoney2(i -> new DecimalFormat("#,###").format(i));

       //函数接口链式操作
       Function<Integer,String> function = i -> new DecimalFormat("#,###").format(i);
       money.printMoney2(function.andThen(s -> "加在前"+s));
    }
```

可以看出我们这里使用函数接口就不用再自己定义一个接口。（这里只需要完成一个int类型的入参和一个String类型的返回值）

JDK8提供的函数接口以下：

| 接口                  | 输入参数 | 返回类型 | 说明                       |
| --------------------- | -------- | -------- | -------------------------- |
| **Predicate**<T>      | T        | boolean  | 断言                       |
| **Consumer<T>**       | T        | /        | 消费一个数据               |
| **Function<T,R>**     | T        | R        | 输入T输出R的函数           |
| **Supplier<T>**       | /        | T        | 提供一个数据               |
| **UnaryOperator<T>**  | T        | T        | 一元函数（输入出类型相同） |
| **BiFunction<T,U,R>** | (T,U)    | R        | 2个输入的函数              |
| **BinaryOperator<T>** | <T,T>    | T        | 二元函数（输入出类型相同） |

eg：

```Java
Predicate<Integer> predicate = (i) -> i>0;
System.out.println(predicate.test(-1));

Consumer<String> consumer = (s) -> System.out.println(s);
consumer.accept("yangeryiyi");
//JDK里有些函数接口直接带了类型，不需要使用泛型类
//eg:
IntConsumer intConsumer = i -> System.out.println("带类型的函数接口"+i);
intConsumer.accept(9);
```

#### 方法引用：

```Java
//Consumer<String> consumer = (s) -> System.out.println(s);
//当参数s和方法体里的参数调用相同时，这里可修改为：
Consumer<String> consumer2 = System.out::println;
consumer2.accept("输出");
```

静态方法的引用：

**类名::静态方法名**

eg:

```java
class Dog{
    private String name ="狗";
    public static void bark(Dog dog){
        System.out.println(dog.name+"叫");
    }
}
public class demo2 {
    public static void main(String[] args) {
        Consumer<Dog> consumer = Dog::bark; //只有一个输入参数，选择Consumer消费者
        Dog dog = new Dog();
        consumer.accept(dog);
    }
```

非静态方法：

**实例名::方法名**

构造方法：

**类名::new**

eg：

```Java
class Dog{
    private String name ="狗";
    private int food = 10;
    public int eat(Dog this,int num){         //默认在第一个参数会有一个this对象
        System.out.println("吃了"+num+"斤狗粮");
        this.food -= num;
        return this.food;
    }
}
public class demo2 {
    public static void main(String[] args) {
        Dog dog = new Dog();
        
        //非静态方法，使用对象实例的方法引用
        //分析eat方法需要一个输入int，返回一个int,可以使用Function<T,R>
        Function<Integer,Integer> function = dog::eat;
        System.out.println("还剩"+function.apply(8));

        //以上eat方法可以发现输入输出是一样的都是int，可以改成一元函数接口UnaryOperator
        //这里因为是基本的Integer类型，JDK都有对应的函数接口，不用写泛型
        //IntUnaryOperator u = dog::eat;  u.applyAsInt(8)
        UnaryOperator<Integer> unaryOperator = dog::eat;
        System.out.println("还剩"+unaryOperator.apply(8));

        /* 实际上非静态方法也可以通过类名去使用方法引用
         * JDK默认会把当前实例传入到非静态方法，参数名为this，位置是第一个
         *所以eat方法这里可以看成两个入参一个出参
         */
        BiFunction<Dog,Integer,Integer> biFunction = Dog::eat;
        System.out.println("还剩"+biFunction.apply(dog,2));

        //构造函数的方法引用 Dog::new
        //首先无参构造没有输入，但会返回一个实例，所以它是一个提供者
        //如果有参构造我们就使用Function<T,R>就完事了
        Supplier<Dog> supplier = Dog::new;
        System.out.println("创建了新对象"+supplier.get());
    }
}
```

#### 类型推断：

```java
interface IMath{
    int add(int x,int y);
}
interface IMath2{
    int add(int x,int y);
}
public class TypeInference {
    public static void main(String[] args) {
        //普通
        IMath lambda = (x,y) -> x+y;
        //数组
        IMath[] lambdas = {(x,y) -> x+y};
        //强转
        Object lam =(IMath)(x,y) -> x+y;
        //当有二义性时，使用强转对应的接口解决
        TypeInference typeInference = new TypeInference();
        typeInference.test((IMath) (x,y)-> x+y);
    }
    public void test(IMath iMath){}
    public void test(IMath2 iMath){}
}
```

#### 变量引用：

```java
public class VarDemo {
    public static void main(String[] args) {
        //JDK8之前匿名内部类引用外面的变量必须申明为final，之后已经默认final了
        //Java中参数传递是值传递，而不是传引用，如果不声明为final在匿名内部类中修改当前该变量会导致二义性
        String str = "等闲变却故人心";
        Consumer<String> consumer = s -> System.out.println(s);
        consumer.accept(str);
    }
}
//值传递复制了一份list2到表达式里，是指这里两个list2其实都指向new ArrayList<>()；而传引用咋代表表达式里的list2其实是指向第一个list2.
List<String> list2 = new ArrayList<>();
Consumer<String> consumer = s -> System.out.println(s+list2);
```

#### 级联表达式和柯里化：

```Java
/**
 * 级联表达式和柯里化
 * 柯里化：把多个参数的函数转换为只有一个参数的函数
 * 柯里化目的:函数标准化
* */
        Function<Integer,Function<Integer,Integer>> function = x-> y -> x+y;
        System.out.println(function.apply(2).apply(3));
        int[] nums = {2,3};
        Function f = function;
        for (int i = 0; i < nums.length; i++) {
            if(f instanceof Function){
                Object obj = f.apply(nums[i]);
                if(obj instanceof Function){
                    f = (Function) obj;
                }else{
                    System.out.println("Result"+obj);
                }
            }
        }
```

## Stream流编程

eg:

```java
public class StreamDemo1 {
    public static void main(String[] args) {
        int[] nums = {1,2,3};
        //外部迭代
        int sum = 0;
        for(int i:nums){
            sum += i;
    }
        System.out.println("The Result is "+sum);
        //使用Stream的内部迭代;使用这个数据量大时相关并行操作等实现方便
        //map就是中间操作（返回stream的操作）
        //sum就是终止操作
        int sum2 = IntStream.of(nums).map(i -> i*2).sum();
        System.out.println("The Result is "+sum2);
        //惰性求值
        //这里会输出三次这里乘2，但我们直接IntStream.of(nums).map(StreamDemo1::doubleNum)而不去sum（）求和操作，
        // 那么map(StreamDemo1::doubleNum)也不会执行。
        //惰性求值就是终止操作没有调用的情况下，中间操作不会执行
        int sum3 = IntStream.of(nums).map(StreamDemo1::doubleNum).sum();
        System.out.println("The Result is "+sum3);
    }
    public static int doubleNum(int i){
        System.out.println("这里乘2");
        i = i*2;
        return i;
    }
}
```

### 创建

|            | 相关方法                               |
| ---------- | -------------------------------------- |
| 集合       | Collection.stream/parallerStream       |
| 数组       | Arrays.stream                          |
| 数字Stream | IntStream/LongStream.range/rangeClosed |
|            | Random.ints/longs/doubles              |
| 自己创建   | Stream.generate/iterate                |

```java
public class create {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        //从集合创建
        list.stream();
        list.parallelStream();
        //从数组创建
        Arrays.stream(new int[]{1, 2, 3});
        //创建数字流
        IntStream.of(1, 2, 3);
        IntStream.rangeClosed(1,10);
        //使用random创建一个无限流(需配套一个短路相关操作类似limit)
        new Random().ints().limit(10);
        //自己创建
        Random random = new Random();
        Stream.generate(()-> random.nextInt()).limit(20);
    }
}
```

### 中间操作

<table style="text-align: center;">
     <tr>
        <td></td>
        <td>相关方法</td>
    </tr>
    <tr>
        <td rowspan="6">无状态操作</td>
    </tr>
    <tr>
        <td>map/mapToXxx</td>
    </tr>
      <tr>
        <td>flatMap/flatMapToXxx</td>
    </tr>
      <tr>
        <td>filter</td>
    </tr>
      <tr>
        <td>peek</td>
    </tr>
      <tr>
        <td>unordered</td>
    </tr>
    <tr>
        <td rowspan="4">有状态操作</td>
    </tr>
    <tr>
        <td>distinct</td>
    </tr>
     <tr>
        <td>sorted</td>
    </tr>
     <tr>
        <td>limit/skip</td>
    </tr>
</table>

eg：（有两个参数有状态，一个参数无状态）

```Java
public static void main(String[] args) {
    String str ="却道 故人 心 易变啊";
    Stream.of(str.split(" ")).map(s -> s.length()).forEach(System.out::println); //输出2 2 1 2
    Stream.of(str.split(" ")).filter(s -> s.length()>=2).map(s -> s.length()).forEach(System.out::println);//输出2 2 2

    //flatMap A元素下有B属性，B属性是个集合，最终得到所有的A元素里面的所有B属性集合
    //intStream/longStream并不是Stream的子类，所以要进行装箱boxed
    Stream.of(str.split(" ")).flatMap(s -> s.chars().boxed()).forEach(i -> System.out.println((char)i.intValue()));

    //peek 用于debug，是个中间操作，forEach是终止操作括号里放（System.out::println）=(s -> System.out.println(s))
    //结果输出每个空格拆分的字打印两次
    Stream.of(str.split(" ")).peek(s -> System.out.println(s)).forEach(s -> System.out.println(s));

    //limit使用，主要用于无限流
    //new Random().ints().forEach(System.out::println); //将会无限去打印产生的随机数
    new Random().ints().filter(i -> i>10&&i<30).limit(10).forEach(System.out::println);
}
```

### 终止操作

<table style="text-align: center;">
     <tr>
        <td></td>
        <td>相关方法</td>
    </tr>
    <tr>
        <td rowspan="5">非短路操作</td>
    </tr>
    <tr>
        <td>forEach/forEachOrdered</td>
    </tr>
      <tr>
        <td>collect/toArray</td>
    </tr>
      <tr>
        <td>reduce</td>
    </tr>
      <tr>
        <td>min/max/count</td>
    </tr>
    <tr>
        <td rowspan="3">短路操作</td>
    </tr>
    <tr>
        <td>findFirst/fandAny</td>
    </tr>
     <tr>
        <td>allMatch/anyMatch/noneMatch</td>
    </tr>
</table>

eg:

```java
public static void main(String[] args) {
    String str = "my name is 007";

    //关于带有ordered一般关于并行流
    //m mis 070na ye为打印的结果，顺序是乱的
    str.chars().parallel().forEach(i -> System.out.print((char)i));
    //保证了输出顺序my name is 007
    str.chars().parallel().forEachOrdered(i -> System.out.print((char)i));

    //收集到list, Collectors收集器还可以toset等等
    List<String> list = Stream.of(str.split(" ")).collect(Collectors.toList());

    //使用reduce拼接字符串
    Optional<String> reduce = Stream.of(str.split(" ")).reduce((s1, s2) -> s1 + "|" + s2);
    System.out.println(reduce.orElse(""));//如果reduce为空就返回空置

    //带初始值的reduce,可以直接返回一个String，因为不用去做非空判断
    String reduce1 = Stream.of(str.split(" ")).reduce("", (s1, s2) -> s1 + "|" + s2);
    System.out.println(reduce1);

    //计算所有单词的总长度
    Integer reduce2 = Stream.of(str.split(" ")).map(s -> s.length()).reduce(0, (s1, s2) -> s1 + s2);
    System.out.println(reduce2);

    //max操作,返回最长的单词name
    Optional<String> max = Stream.of(str.split(" ")).max((s1, s2) -> s1.length() - s2.length());
    System.out.println(max.get());

    //短路操作findFirst使用,中断无限流肯定会有第一个值
    OptionalInt first = new Random().ints().findFirst();
    System.out.println(first.getAsInt());
}
```

### 并行流

```java
public class ParallelStream {
    public static void main(String[] args) throws InterruptedException {
        //1.调用paraller产生一个并行流
        IntStream.range(1,100).parallel().peek(ParallelStream::debug).count();

        //要实现先并行再串行，
        //实际上多次调用parallel | parallel，以最后一次调用为准，这里就是sequential串行
//        IntStream.range(1,100)
//                .parallel().peek(ParallelStream::debug)
//                .sequential().peek(ParallelStream::debug)
//                .count();

        /**
         * 并行流默认使用的线程池：ForkJoinPool.commonPool
         * 默认的线程数是 当前机器的cpu个数
         * 使用System.setProperty("java.util.concurrent.ForkJoinPool.commin.parallelism","20");修改默认线程数
         * */

        //使用自己的线程池，不适用默认线程池，防止任务被阻塞
        //线程名字ForkJoinPool-1
        ForkJoinPool pool = new ForkJoinPool(20);
        pool.submit(()->IntStream.range(1,100).parallel().peek(ParallelStream::debug).count());
        pool.shutdown();

        //因为这里是再main函数里执行， 主函数已经退出了，而现在的线程池相当于一个守护线程，所以自动也退出了
        //让主线程等待
        synchronized (pool){
            pool.wait();
        }
    }

    public static void debug(int i){
      //  System.setProperty("java.util.concurrent.ForkJoinPool.commin.parallelism","20");
        System.out.println(Thread.currentThread().getName()+"debug"+i);
        try {
            TimeUnit.SECONDS.sleep(3);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
//打印结果
// maindebug65
//ForkJoinPool.commonPool-worker-9debug32
//ForkJoinPool.commonPool-worker-10debug94
    //。。。
}
```

### 收集器

把流处理后的数据收集起来，到一些集合类，或者把处理后的数据再处理，例如求和等

eg;

```java
 class Student {
    private String name;
    private int age;
    private Gender gender;
    private Grade grade;
    public Student(String name, int age, Gender gender, Grade grade) {
        super();
        this.name = name;
        this.age = age;
        this.gender = gender;
        this.grade = grade;
    }

    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
    public Gender getGender() {
        return gender;
    }
    public void setGender(Gender gender) {
        this.gender = gender;
    }
    public Grade getGrade() {
        return grade;
    }
    public void setGrade(Grade grade) {
        this.grade = grade;
    }
    @Override
    public String toString() {
        return "CollectDemo{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", gender=" + gender +
                ", grade=" + grade +
                '}';
    }
    enum Gender{
        MALE,FEMALE
    }
    enum Grade{
        ONE,TWO,THREE,FOUR
    }

}
public class CollectDemo {
    public static void main(String[] args) {
        List<Student> students = Arrays.asList(
                new Student("张三",10, Student.Gender.MALE, Student.Grade.ONE),
                new Student("李四",20, Student.Gender.FEMALE, Student.Grade.TWO),
                new Student("王五",30, Student.Gender.MALE, Student.Grade.TWO)
        );
        //得到所有学生年龄列表
        //s -> s.getAge()  可以修改为 Student::getAge,不会多生成一个类似lambda$0这样的函数
        List<Integer> ages = students.stream().map(Student::getAge)
                .collect(Collectors.toList());
        System.out.println("所有学生年龄"+ages);
        //所有学生年龄[10, 20, 30]

        //统计汇总信息
        IntSummaryStatistics ageSummaryStatistics = students.stream()
                .collect(Collectors.summarizingInt(Student::getAge));
        System.out.println("年龄汇总信息"+ageSummaryStatistics);
        //年龄汇总信息IntSummaryStatistics{count=3, sum=60, min=10, average=20.000000, max=30}

        //分块,按男女（特殊的分组，只有两组）
        Map<Boolean, List<Student>> gender = students.stream()
                .collect(Collectors.partitioningBy(s -> s.getGender() == Student.Gender.MALE));
        System.out.println("男女列表"+gender);
       // 男女列表{false=[CollectDemo{name='李四', age=20, gender=FEMALE, grade=TWO}],
        //        true=[CollectDemo{name='张三', age=10, gender=MALE, grade=ONE}, CollectDemo{name='王五', age=30, gender=MALE, grade=TWO}]}

        //分组
        Map<Student.Grade, List<Student>> grade = students.stream()
                .collect(Collectors.groupingBy(Student::getGrade));
        System.out.println("班级分组"+grade);
        //班级分组{TWO=[CollectDemo{name='李四', age=20, gender=FEMALE, grade=TWO}, CollectDemo{name='王五', age=30, gender=MALE, grade=TWO}],
        // ONE=[CollectDemo{name='张三', age=10, gender=MALE, grade=ONE}]}

        //得到所有班级学生个数
        Map<Student.Grade, Long> num = students.stream()
                .collect(Collectors.groupingBy(Student::getGrade,Collectors.counting()));
        System.out.println("班级人数"+num);
        //班级人数{TWO=2, ONE=1}
    }
```

### 相关原理

eg：

```java
/**
 * 1.所有元素链式掉用，一个元素只迭代一次（交替执行，一个元素会走完中间操作再到下一个）
 * 2.每个中间操作返回一个新的流，流里面有一个属性sourceStage指向同一个地方，就是head
 * Head -> nextStage ->nextStage ->..->null
 * 3.有状态操作会把无状态操作截断
 * 4.并行环境下，有状态的中间操作不一定能并行操作
 * 5.paraller | sequetial 这两个操作也是中间操作（也就是会返回stream）但是他们不创建流，他们只修改Head的并行标志
 */
public class PrincipleStream {
    public static void main(String[] args) {
        Random random = new Random();
        //随机产生数据
     Stream<Integer> stream = Stream.generate(random::nextInt)
                .limit(200)
                //第一个无状态操作
                .peek(s -> System.out.println("peek操作"+s))
                //第二个无状态操作
                .filter(s -> {
                    System.out.println("filter操作"+s);
                    return s>10000;
                });

                //终止操作
                 stream.count();
    }
    //peek操作-1872745922
    //filter操作-1872745922
    //peek操作1728955806
    //filter操作1728955806
}
```

## JDK 9 Reactive Stream

异步编程常见的问题：

1. 超时、异常处理困难
2. 难以重构
3. 多个异步任务协同处理

为了解决异步编程过程中出现的种种难题，人们提出了各种各样方法来规避这些问题，这些方法称为`反应式编程(Reactive Programming)`，就像面向对象编程，函数式编程一样，反应式编程也是另一种编程范式。

反应式编程，本质上是对数据流或某种变化所作出的**反应**，但是这个变化什么时候发生是未知的，所以他是一种基于异步、回调的方式在处理问题。

对于Java程序员，Reactive Streams是一个API。Reactive Streams为我们提供了Java中的Reactive Programming的通用API。

`Reactive Streams API`中仅仅包含了如下四个接口：

```
Publisher<T>
```

```
Subscriber<T>
```

```
Subscription
```

```
Processor<T,R>
```

Reactive Streams的主要目标有这两个：

1. 管理跨异步边界的流数据交换 - 即将元素传递到另一个线程或线程池；
2. 确保接收方不会强制缓冲任意数量的数据，为了使线程之间的队列有界，引入了回压(Back Pressure)。

> 回压(Back Pressure)，可以动态控制线程间消息交换的速率，避免生产者产生过多的消息，消费者消费不完等类似问题。

Reactive Streams，是一套非阻塞背压的异步数据流的API

而异步特性，是体现在每一步的处理过程中的，每一步处理都是消息驱动的，不阻塞应用程序，被动获得结果后继续进行下一步。

背压（back-pressure）是一种重要的反馈机制，使得系统得以优雅地响应负载，而不是在负载下崩溃。相反，如果下游组件比较空闲，则可以向上游组件发出信号，请求获得更多的调用。

eg:

```java
public class demo1 {
    public static void main(String[] args) throws InterruptedException {
        //定义发布者，发布的数据类型是Integer
        //直接使用jdk自带的SubmissionPublisher,它实现了Publisher接口
        SubmissionPublisher<Integer> publisher = new SubmissionPublisher<Integer>();

        //2.定义订阅者
        Subscriber<Integer> sb = new Subscriber<Integer>() {

            private Flow.Subscription subscription;
            @Override
            public void onSubscribe(Flow.Subscription subscription) {
             //保存订阅关系，需要用它来给发布者响应
                this.subscription = subscription;
                //请求一个数据
                this.subscription.request(1);
                //

            }

            @Override
            public void onNext(Integer item) {
                //接受到一个数据处理
                System.out.println("接收到数据"+ item);

                //处理完调用request再请求一个数据
                 this.subscription.request(1);

                //或者已经达到目标，调用cancel告诉发布者不用接受数据了
                //this.subscription.cancel();
            }

            @Override
            public void onError(Throwable throwable) {
              //出现异常（处理数据时产生了异常）
                throwable.printStackTrace();

                //可以告诉发布者，不接受数据了
                this.subscription.cancel();
            }

            @Override
            public void onComplete() {
            //发布者关闭时调用
                System.out.println("处理完成");
            }
        };

        //3.发布者和订阅者建立订阅关系
        publisher.subscribe(sb);

        //4.生产数据，并发布
        //这里忽略数据生产过程
        int data=222;
        publisher.submit(data);
        publisher.submit(333);
        //5.结束后 关闭发布者
        //一般放在finally中关闭
        publisher.close();

        //主线程延迟停止，否者数据还没有消费就退出
        Thread.currentThread().join(1000);
    }
}
```

Processor

```java
/**
 * Processor,需要继承SubmissionPublisher并实现Processor接口
 * 输入源数据integer，过滤掉小于0的，然后转换成字符串发布出去
 */
class MyProcessor extends SubmissionPublisher<String> implements Flow.Processor<Integer,String> {

    private Flow.Subscription subscription;
    @Override
    public void onSubscribe(Flow.Subscription subscription) {
        //保存订阅关系，需要用它来给发布者响应
        this.subscription = subscription;
        //请求一个数据
        this.subscription.request(1);
    }

    @Override
    public void onNext(Integer item) {
        System.out.println("处理器接受到的数据"+item);

        //过滤小于0的，发布出去
        if (item > 0){
            this.submit("转换后"+item);
        }

        //处理完调用request再请求一个数据
        this.subscription.request(1);

        //或达到目标，调用cancel告诉发布者不再接受数据
        //this.subscription.cancel();
    }

    @Override
    public void onError(Throwable throwable) {

    }

    @Override
    public void onComplete() {

    }
}
public class demo2 {
    public static void main(String[] args) throws InterruptedException {
        //1.定义发布者，发布的数据类型是Integer
        //直接使用jdk自带的SubmissionPublisher,它实现了Publisher接口
        SubmissionPublisher<Integer> publisher = new SubmissionPublisher<Integer>();

       //2.定义处理器，对数据过滤，转换为String
        MyProcessor myProcessor = new MyProcessor();

        //3.发布者和处理器建立订阅关系
        publisher.subscribe(myProcessor);

        //4.定义最终订阅者，消费String类型数据
        Flow.Subscriber<String> sb = new Flow.Subscriber<String>() {

            private Flow.Subscription subscription;
            @Override
            public void onSubscribe(Flow.Subscription subscription) {
                //保存订阅关系，需要用它来给发布者响应
                this.subscription = subscription;
                //请求一个数据
                this.subscription.request(1);
                //

            }

            @Override
            public void onNext(String item) {
                //接受到一个数据处理
                System.out.println("接收到数据"+ item);

                //处理完调用request再请求一个数据
                this.subscription.request(1);

                //或者已经达到目标，调用cancel告诉发布者不用接受数据了
                //this.subscription.cancel();
            }

            @Override
            public void onError(Throwable throwable) {
                //出现异常（处理数据时产生了异常）
                throwable.printStackTrace();

                //可以告诉发布者，不接受数据了
                this.subscription.cancel();
            }

            @Override
            public void onComplete() {
                //发布者关闭时调用
                System.out.println("处理完成");
            }
        };

        //5.处理器和最终订阅者建立订阅关系
        myProcessor.subscribe(sb);

        //6.生产数据，并发布
        //这里忽略数据生产过程
       // for (int i = 0; i < 1000; i++) {//这个写法可以看出在消费者收到256条数据后如果没有即使消费这里的submit会阻塞
        //     publisher.submit(i);
       // }
        int data=222;
        publisher.submit(data);
        publisher.submit(-333);

        //7.结束后 关闭发布者
        //一般放在finally中关闭
        publisher.close();

        //主线程延迟停止，否者数据还没有消费就退出
        Thread.currentThread().join(1000);
    }
}
/**
 * 响应式流里面最关键的就是一个反馈，通过一个反馈可以调节发布者创建数据的速度
 * 关键在于publisher.submit（）是一个阻塞方法，当发布者发布数据时如果订阅者的缓冲池满了（存在一个array数组，有发布者过来的数据），
 * 那么submit就会被阻塞，就不会再生产数据,(等消费者消费一条再生产一条)
*
* */
```

## Spring WebFlux

Spring WebFlux 是 Spring Framework 5.0中引入的新的响应式web框架。与Spring MVC不同，它不需要Servlet API，是完全异步且非阻塞的，并且通过Reactor项目实现了Reactive Streams规范。

Spring WebFlux 用于创建基于事件循环执行模型的完全异步且非阻塞的应用程序。

Reactive Streams是一套用于构建高吞吐量、低延迟应用的规范。而Reactor项目是基于这套规范的实现，它是一个完全非阻塞的基础，且支持背压。

Spring WebFlux基于Reactor实现了完全异步非阻塞的一套web框架，是一套响应式堆栈。

【spring-webmvc + Servlet + Tomcat】命令式的、同步阻塞的

【spring-webflux + Reactor + Netty】响应式的、异步非阻塞的





### 异步servlet

同步servlet阻塞了什么？

其实是阻塞了Tomcat 容器的Servlet线程。

异步：

不会阻塞tomcat线程，把耗时的业务操作放在独立的线程池里，那么servlet线程就可以立刻返回，去处理

其他请求

代码：

1.开启异步上下文环境

2.将耗时的业务代码放入独立的线程池中去完成

3.处理完返回

eg.

```java
@WebServlet(name = "helloServlet", value = "/hello-servlet",asyncSupported = true)
public class HelloServlet extends HttpServlet {
    private String message;

    public void init() {
        message = "Hello World!";
    }
    public void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
        long t1 = System.currentTimeMillis();
        //开启异步
        AsyncContext asyncContext = request.startAsync();
        //执行业务代码,开启一个新线程去执行，不阻塞当前servlet线程
       CompletableFuture.runAsync(()-> {
           try {
              doSome(asyncContext,asyncContext.getRequest(),asyncContext.getResponse());
           } catch (InterruptedException e) {
               e.printStackTrace();
           } catch (IOException e) {
                e.printStackTrace();
           }
        });
        System.out.println("用时：");
        System.out.println(System.currentTimeMillis()-t1);  
        }
    }

    private void doSome(AsyncContext asyncContext, ServletRequest request, ServletResponse response) throws IOException, InterruptedException {
        TimeUnit.SECONDS.sleep(5);
        response.getWriter().append("done");

        //业务代码处理完毕
        asyncContext.complete();
    }
}
```



//webflux

```
Reactor = stream + reactive stream
```

```java
public class ReactorDemo {
    public static void main(String[] args) {
        //Mono 0-1个元素
        //Flux 0-N个元素
        String[] str = {"1","2","3"};

        Subscriber<Integer> sb = new Subscriber<Integer>() {

            private Subscription subscription;
            @Override
            public void onSubscribe(Subscription subscription) {
                //保存订阅关系，需要用它来给发布者响应
                this.subscription = subscription;
                //请求一个数据
                this.subscription.request(2);


            }

            @Override
            public void onNext(Integer item) {
                //接受到一个数据处理
                System.out.println("接收到数据"+ item);

                //处理完调用request再请求一个数据
              //  this.subscription.request(2);

                //或者已经达到目标，调用cancel告诉发布者不用接受数据了
             //   this.subscription.cancel();
            }

            @Override
            public void onError(Throwable throwable) {
                //出现异常（处理数据时产生了异常）
                throwable.printStackTrace();

                //可以告诉发布者，不接受数据了
                this.subscription.cancel();
            }

            @Override
            public void onComplete() {
                //发布者关闭时调用
                System.out.println("处理完成");
            }
        };

         //JDK8的stream
        Flux.fromArray(str).map(s -> Integer.parseInt(s))
                //最终操作
                //JDK9的reactive stream
                .subscribe(sb);
    }
}
```



eg.

```java
@RestController
@Slf4j
public class TestController {

    @GetMapping("/1")
    private String get() {
        log.info("get1--start");
        String a = createStr();
        log.info("get1--end");
        return a;
    }

    @GetMapping("/2")
    private Mono<String> get2() {
        log.info("get2--start");
        Mono<String> mo = Mono.fromSupplier(()-> createStr());
        log.info("get2--end");
        return mo;
    }

    private String createStr() {
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "等闲变却";
    }
}
```

在第一个例子中请求的线程会阻塞5秒，而第二个例子直接执行后面的代码

```
//Mono 0-1个元素
//Flux 0-N个元素
```

Flux的使用（会像流一样输出flux-data---1-5）

```java
@GetMapping(value = "/3", produces = "text/event-stream")
private Flux<String> get3() {
    Flux<String> result = Flux.fromStream(IntStream.range(1,5).mapToObj(i -> {
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "flux-data---"+i;
    }));
    return result;
}
```



SSE

```java
   public void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
       //这两个头一定要设置
        response.setContentType("text/event-stream");
        response.setCharacterEncoding("UTF-8");
       
        for (int i = 0; i < 5; i++) {
            //指定格式:data: + 数据 +2个回车
            response.getWriter().write("data:" +i +"\n\n");
            //另一种写法②
            response.getWriter().write("event:me\n");
            response.getWriter().flush();

            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
```

前端写法：

```js
//依赖H5
var sse = new EventSource("HelloServlet");
sse.onmessage =function(e){
    console.log("message",e.data,e)
}
//或②
sse.addEventListener("me",function(e){
    console.log("me message",e.data,e)
})

//关闭自动重连
//在函数体里写sse.close();
if(e.data ==3){
    sse.close();
}
```

 

