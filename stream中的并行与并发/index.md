# Stream中的并行与并发


并发的实质是同一个物理CPU上在若干个程序（线程）之间的多路复用，是强制有限的物理资源行使多用户共享来提高效率。而并行则是让多道程序（线程）在同一时刻不同的CPU上运行.

<!--more-->

# 并行与并发

#### 并行与并发

​		并发与并行是两个相似又不同的概念：

​		并发：多个任务可以在重叠的时间段内运行。

​		并行：多个任务可以同时运行。

​		并发的实质是同一个物理CPU上在若干个程序（线程）之间的多路复用，是强制有限的物理资源行使多用户共享来提高效率。而并行则是让多道程序（线程）在同一时刻不同的CPU上运行。从宏观上看，它们都使得线程可以同时运行，但实际上只有并行是真正的同时运行的。

​		因此，并行、并发与多线程的关系就是：

​		并行需要两个或两个以上的线程跑在不同的处理器上，并发可以跑在⼀个处理器上通过时间片进行切换。

​		也就是说，我们的处理器只要不是单核的，就具有并行运行的能力。

#### ParallelStream并行流的创建

​		JAVA8中引入了lamda表达式和Stream接口。其丰富的API及强大的表达能力极大的简化代码，提升了效率，同时还通过parallelStream提供并发操作的支持，本文探讨parallelStream方法的使用。

​		ParadellelStream可以帮助我们创建并行流，这些并行流默认是基于Fork/Join框架运行的，也就是说，它们运行在ForkJoin线程池中。Java 8为ForkJoinPool添加了一个通用线程池，这个线程池用来处理那些没有被显式提交到任何线程池的任务。在多数场景以及默认情况使用时，我们一般直接使用 ForkJoinPool 中的common池。

##### 并行流的创建

​		我们可以使用parallel()方法将一个流转换为并行流，也可以直接使用parallelStream将一个数据集合转换为并行流。

​		parallel方法是BaseStream接口实现的，而继承这个接口的有LongStream、IntStream、DoubleStream、Stream等。parallelStream是Collection接口的方法，用于把实现了Collection接口的集合转换为并行流。

​		在以下的代码中，我们使用BaseStream的另一个方法isParallel来判断一个流是否为并行流。注意，这个方法并不是终止操作，它只是于判断流的状态的一个方法。

```java
@Test
public void parallelStreamOf(){
    System.out.println(Stream.of(3,1,4,1,5,9).parallel().isParallel());
}

@Test
public void parallelStreamIterater(){
    System.out.println(Stream.iterate(1,n->n+1).parallel().isParallel());

}

    @Test
    public void parallelStreamInt(){
        System.out.println(IntStream.rangeClosed(0,250).parallel().isParallel());
    }

@Test
public void parallelStreamMethodOnCollection(){
    List<Integer> numbers = Arrays.asList(1,2,3,4,5,6,7);
    System.out.println(numbers.parallelStream().isParallel());
}

    @Test
    public void parallelStreamMethodOnCollection(){
        List<Integer> numbers = Arrays.asList(1,2,3,4,5,6,7,1,4,3);
        System.out.println(numbers.parallelStream().isParallel());

        Set<Integer> set = numbers.stream().collect(Collectors.toSet());
        System.out.println(set.parallelStream().isParallel());
        set.parallelStream().forEach(System.out::println);
    }
```

​		从上面的代码中我们可以注意到，我们使用了parallel()方法将一个顺序流转换成了一个并行流。那么我们也可以调用BaseStream接口的sequential()方法将一个并行流转换为顺序流。

```java
@Test
public void parallelStreamThensequential(){
    List<Integer> numbers = Arrays.asList(1,2,3,4,5,6,7);
    System.out.println(numbers.parallelStream().sequential().isParallel());

}
```

​		那么，我们可以不可以在同一个流中既运行并行流，又运行顺序流呢？

```java

//我们发现这段程序的所有流任务都是在main线程中运行的
@Test
public void parallelStreamThensequential1(){
    List<Integer> numbers = Arrays.asList(1,2,3,4,5,6,7);
    numbers.parallelStream().map(m -> {
        System.out.println(Thread.currentThread().getName());
        return m;
    }).sequential().sorted().collect(Collectors.toList());

}


//打印结果为不同的线程
    @Test
    public void parallelStreamThensequential12(){
        List<Integer> numbers = Arrays.asList(1,2,3,4,5,6,7);
        numbers.parallelStream().map(m -> {
            System.out.println(Thread.currentThread().getName());
            return m;
        }).sequential().sorted().parallel().collect(Collectors.toList());

    }
```

​		从上面的代码的运行结果来看，答案是否定的。也就是说，一个流要么是并行流，要么是顺序流。这是因为流在到达终止操作之前不会进行任何操作，只有当到达终止操作之后才会评估流的状态。如果在终止操作前最后调用的是sequential()方法，那么这个流将是一个顺序流。同理，如果是parallel()方法最后调用，那么这个流将是一个并行流。当我们需要同时在一段操作任务中使用到并行流和顺序流时，除了使用两个流，暂时没有更好的解决办法。

```java
@Test
public void parallelStreamThensequential2(){
    List<Integer> numbers = Arrays.asList(1,2,3,4,5,6,7);
    numbers.parallelStream()
            .map(m -> {
        System.out.println(Thread.currentThread().getName());
        return m;
    }).collect(Collectors.toList())
            .stream()
            .sequential()
            .sorted()
            .map(m -> {
        System.out.println(Thread.currentThread().getName());
        return m;
    }).collect(Collectors.toList());
}
```

##### 并行流的运行特点

###### 并行流对于中间操作和终止操作运行结果的影响

![](image\2021072810040213.png)

​		由于CPU核心的运行速度有差异，并行流对中间操作可能存在执行结时在先后顺序上的影响，比如说输出的顺序和源数据的顺序差异。并行流对于ForEach终止操作也有类似的影响。但对于收集操作，例如collect()、max()等，并行流并未对结果造成影响。对于短路操作而言，可能在运行时间上有细微差异，其它的影响也几乎没有。

​		并行流将一个大任务分成多个小任务，这些小任务并行执行，最终到达终止操作，等所有任务到达终止操作之后，返回结果。理论上会加快运行速度。

```java
    @Test
    public void parallelStreamsetForkJoinPooltwo(){
        Instant before = Instant.now();

//        int total = IntStream.of(3,1,4,1,5,9).map(ParallelStreamTest::doubleIt).sum();
//        int total = IntStream.of(3,1,4,1,5,9).parallel().map(ParallelStreamTest::doubleIt).sum();
//        int total = IntStream.rangeClosed(0,250).parallel().map(ParallelStreamTest::doubleIt).sum();

        Instant after = Instant.now();
        Duration duration = Duration.between(before,after);
        System.out.println("time= "+duration.toMillis() + "  ms");
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        int poolSize = ForkJoinPool.commonPool().getPoolSize();
        System.out.println(poolSize);
    }

    public static int doubleIt(int n){
        try{
            Thread.sleep(100);
        }catch (InterruptedException ignore){}
        return n*2;
    }
```

此外，由于每个线程运行的速度不一样，所以小任务执行完毕的先后也不一致。

```java
    @Test
    public void parallelStreamsetForkJoinPool0(){
        List<Integer> numbers = Arrays.asList(8,2,3,4,3,7,1,10,5,6,1,8,1,86,15,0);
//        IntStream.rangeClosed(0,250).parallel().forEach( n -> {
//            System.out.println("number -> "+n);
//        });


        //排序失败
//        IntStream.rangeClosed(0,250).parallel().sorted().forEach( n -> {
//            System.out.println("number -> "+n);
//        });
//        numbers.parallelStream().sorted().forEach( n -> {
//           System.out.println("number -> "+n);
//        });
        //排序成功
//        List<Integer> list = Stream.of(1,6,4,8,6,7).parallel().sorted().collect(Collectors.toList());
//        System.out.println(list);


        //skip成功
//        IntStream.rangeClosed(0,10).parallel().skip(5).forEach(n-> System.out.println(n));
        //skip成功
//        List<Integer> list = IntStream.rangeClosed(0,10).parallel().skip(5).mapToObj(n -> n).collect(Collectors.toList());
//        System.out.println(list);


//        //limit成功
//        List<Integer> list = numbers.parallelStream().limit(3).collect(Collectors.toList());
//        System.out.println(list);

        //limit成功，但顺序出现问题
        numbers.parallelStream().limit(5).forEach(System.out::println);

    }


    @Test		//使用并行流
    public void parallelStreamsetForkJoinPooltwo(){
        Instant before = Instant.now();

        int total = IntStream.of(3,1,4,1,5,9).map(ParallelStreamTest::doubleIt).sum();
//        int total = IntStream.of(3,1,4,1,5,9).parallel().map(ParallelStreamTest::doubleIt).sum();
//        int total = IntStream.rangeClosed(0,250).parallel().map(ParallelStreamTest::doubleIt).sum();

        Instant after = Instant.now();
        Duration duration = Duration.between(before,after);
        System.out.println("time= "+duration.toMillis() + "  ms");
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        int poolSize = ForkJoinPool.commonPool().getPoolSize();
        System.out.println(poolSize);
    }
```

###### 影响并行流性能主要存在的几个因素

对于并行流，一定不要陷入一个误区：并行一定比串行快。并行在不同的情况下它不一定是比串行快的。影响并行 流性能主要存在5个因素：

1）数据大小：输入数据的大小，直接影响了并行处理的性能。因为在并行内部实现中涉及到了fork/join操作，它本身就存在性能上的开销（比如创建线程，如果为一个简单的任务创建一个新的线程，那么性能上的提高是得不偿失的）。因此只有当数据量很大，使用并行处理才有意义。

2）源数据结构：fork时会对源数据进行分割，数据源的特性直接影响了fork的性能。 ArrayList、数组或IntStream.range，可分解性最佳，因为他们都支持随机读取，因此可以被任意分割。 HashSet、TreeSet，可分解性一般，其虽然可被分解，但因为其内部数据结构，很难被平均分解。 LinkedList、Streams.iterate、BufferedReader.lines，可分解性极差，因为他们长度未知，无法确定在哪里进行 分割。

3）装箱拆箱 尽量使用基本数据类型，避免装箱拆箱。

4）CPU核数 fork的产生数量是与可用CPU核数相关，可用的核数越多，获取的性能提升就会越大。

5）单元处理开销 花在流中每个元素的时间越长，并行操作带来的性能提升就会越明显。
因此，我们在使用并行流时应当满足以下几个条件：

1.数据量较大

2.每个元素的处理较为耗时

3.数据源易于分解

4.操作是无状态且关联的，有一些操作**sort、distinct**看上去和filter、map差不多，他们接收一个流，再生成一个流，但是区别在于排序和去重复项**需要知道先前的历史**。

5.可以用基本数据类型时，就不要使用泛型流，尽量避免拆箱装箱

```java
    @Test
    public void parallelStreamsetForkJoinPooltwo1(){
        Instant before = Instant.now();


//        Stream.iterate(1L,i -> i+1).limit(3000000).reduce(0L,Long::sum);

//        Stream.iterate(1L,i -> i+1).limit(3000000).parallel().reduce(0L,Long::sum);

//        LongStream.rangeClosed(1,3000000).sum();

        LongStream.rangeClosed(1,3000000).parallel().sum();

        Instant after = Instant.now();
        Duration duration = Duration.between(before,after);
        System.out.println("time= "+duration.toMillis() + "  ms");
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        int poolSize = ForkJoinPool.commonPool().getPoolSize();
        System.out.println(poolSize);
    }

//如果已有一个包装类型的流，我们可以直接使用mapToInt装换等方法为基本类型进行操作,减少拆装箱.
    @Test
    public void parallelStreamsetForkJoinPooltwo11(){

        Stream<Long> longStream = Stream.iterate(1L,i -> i+1).limit(30);

        longStream.mapToLong(l -> l).parallel().forEach(System.out::println);
    }
```

此外，并行流可能引发线程安全问题。显而易见，stream.parallel.forEach()中执行的操作并非线程安全。如果需要线程安全，可以把集合转换为同步集合，如：Collections.synchronizedList(new ArrayList<>())。

```java
    @Test
    public void ma1() throws InterruptedException {
        List<Integer> listFor = new ArrayList<>();
        List<Integer> listParallel = new ArrayList<>();
//        List<Integer> listParallel = Collections.synchronizedList(new ArrayList<>());

        IntStream.range(0, 1000).forEach(listFor::add);
        IntStream.range(0, 1000).parallel().forEach(listParallel::add);

        System.out.println("listFor size :" + listFor.size());
        System.out.println("listParallel size :" + listParallel.size());
    }



    @Test
    public void ma2() throws InterruptedException {
        Set<Integer> setFor = new HashSet<>();
//        Set<Integer> setParallel = new HashSet<>();
        Set<Integer> setParallel = Collections.synchronizedSet(new HashSet<>());

        IntStream.range(0, 1000).forEach(setFor::add);
        IntStream.range(0, 1000).parallel().forEach(setParallel::add);

        System.out.println("listFor size :" + setFor.size());
        System.out.println("listParallel size :" + setParallel.size());
    }


    @Test
    public void ma3() {
        //无序输出
//        Stream.of(1, 3, 5, 7, 9, 2, 4, 6, 8, 10)
//                .parallel()
//                .sorted()
//                .forEach(System.out::println);

		//可以成功排序，这是因为sorted操作中在排序之前先会创建一个数组收集元素
        List<Integer> list = Stream.of(1, 3, 5, 7, 9, 2, 4, 6, 8, 10)
                .parallel()
                .sorted()
                .collect(Collectors.toList());
        System.out.println("list = " +list);
    }
```

##### 改变线程池的大小

​		在默认情况下，并行流的运行基于ForkJoinPool的通用线程池，这个线程池的大小等于JVM可用处理器数量。一般来讲，这个值由Runtime.getRuntime().availableProcessors()确定，如果我的电脑有16个逻辑处理单元，那么这个值就为16，然后线程池的大小为16-1 = 15，外加一个主线程。

​		有时候我们希望自定义通用线程的数量，我们可以采用以下方式：

1.在执行任务前添加如下代码：

```java
System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism", "20");
```

```java
       @Test
    public void corecount() throws InterruptedException {
        System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism", "16");       //限制的是ForkJoinPool最大可创建的线程数量
        System.out.println(Runtime.getRuntime().availableProcessors());
        Thread.sleep(1000);
        int poolSize = ForkJoinPool.commonPool().getPoolSize();
        System.out.println(poolSize);
    }

	@Test
    public void parallelStreamsetForkJoinPool2() {

//        System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism", "8");
//        System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism", "16");
        System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism", "32");
        Instant before = Instant.now();
        long total = LongStream.rangeClosed(1,3000000).parallel().sum();
        System.out.println("total= "+total);
        int poolSize = ForkJoinPool.commonPool().getPoolSize();
        Instant after = Instant.now();
        System.out.println(poolSize);
        Duration duration = Duration.between(before,after);
        System.out.println("time= "+duration.toMillis() + "  ms");

    }
```

后面的值可以根据自己需要进行设置，不一定为20。只不过，通用线程池的大小不宜设置得过大，如果计算机上只有16个逻辑处理单元，那么你的计算机就只能一次性执行16个线程。大于这个数目对性能提升并没有帮助，反而会因为创建新线程加大性能开销。

2.在命令行中输入以下命令：

```cmd
$ java -cp build/classes/main concurrency.CommonPoolSize

Pool size: 20


$ java currentThreadTest.ForkJoinPoolTest4
```

注意，编程设置会覆盖命令行设置。

为了满足实际的并发场景，我们也可以自定义ForkJoinPool。

```java
@Test
public void parallelStreamsetForkJoinPool3() {
    ForkJoinPool pool = new ForkJoinPool(15);

    Instant before = Instant.now();

    long total;
    ForkJoinTask<Long> task = pool.submit(
    () -> LongStream.rangeClosed(1, 3_000_000)
            .parallel()
            .sum());
    try {
        total = task.get();
    } catch (InterruptedException | ExecutionException e) {
        e.printStackTrace();
    } finally {
        pool.shutdown();
    }

}
```

#### 线程的创建

线程的创建有四种方式：

1）继承Thread类创建线程

```java
public class MyThread extends Thread{//继承Thread类

　　public void run(){
　　//重写run方法

　　}

}

public class Main {
　　public static void main(String[] args){
　　　　new MyThread().start();//创建并启动线程

　　}

}
```

2）实现Runnable接口创建线程

```java
public class ThreadCreateTest {
    public static void main(String[] args) {
        Thread thread = new Thread(()-> System.out.println("hello"));
        thread.start();
    }
}
```

3）使用Callable和Future创建线程

```java
public class ThreadCreateTest1 {
    public static void main(String[] args) {
        FutureTask futureTask = new FutureTask(()-> "5");

        new Thread(futureTask).start();

        try {
            System.out.println(futureTask.get());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}
```

4）使用线程池例如用Executor框架

```java
public class currentThreadTest1 {


    public static void main(String[] args) {
        ExecutorService service = Executors.newCachedThreadPool();
        Future<String> future = service.submit(() -> {
                    Thread.sleep(10);
                    return "Hello World";
                }
        );
        System.out.println("More Processing...");
        while (!future.isDone()){
            System.out.println("Waiting...");
            System.out.println("a");
        }
        getIfNotCancelled(future);
        service.shutdown();     //如果没有明确关闭，jvm不会停止
    }


    public static void getIfNotCancelled(Future<String> future){
        try{
            if(!future.isCancelled()){
                System.out.println(future.get());
            }else {
                System.out.println("Cancelled");
            }
        }catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }
    }
    //程序最终会结束

}
```

##### 线程的底层创建方式Thread类

##### 实现Runnable接口与Callable接口

​		Runnable接口的run方法是没有返回值的。而有时候我们需要这个线程执行并且返回一个结果，我们无法通过Runnable接口做到（我们在使用Thread(Runnable runnable)的时候，传入的是匿名内部类，它无法访问主线程代码中的变量，虽然可以访问常量，但是常量是不可改变的，所以我们无法获取到runable里面的计算结果）。

​		因而我们需要使用Callable接口。

##### Future接口

​		前面我们看到，Future接口可以接收一个Callable接口，并且由此我们可以执行一个线程并且获得结果。

​		Future模式可以这样来描述：我有一个任务，提交给了Future，Future替我完成这个任务。期间我自己可以去做任何想做的事情。一段时间之后，我就便可以从Future那儿取出结果。就相当于下了一张订货单，一段时间后可以拿着提订单来提货，这期间可以干别的任何事情。简单来说就是：主线程在执行的过程中，需要用到一些额外的数据，这些数据需要经过一些计算才能获得，如果把计算放在主线程中去执行将会大大消耗性能。为了提高性能，就将这些计算抛给另外一个线程去处理，这个过程中主线程可以完成其它的事情，当主线程需要用到数据的时候，则把另一个线程的计算结果获取到，然后继续执行主线程。

​		因此为了实现这种模式，我们至少需要调用future.get()方法，但是Thread的构造方法不能直接传入Future接口。这时，我们可以使用FutureTask接口，FutureTask接口实现了Future和Runable，这样我们可以通过Runnable接口来创建线程，也可以通过Future接口调用get方法。

​		Future接口一共有五个方法

​		get（）方法可以当任务结束后返回一个结果，如果调用时，工作还没有结束，则会阻塞线程，直到任务执行完毕

​		get（long timeout,TimeUnit unit）做多等待timeout的时间就会返回结果

​		cancel（boolean mayInterruptIfRunning）方法可以用来停止一个任务，如果任务可以停止（通过mayInterruptIfRunning来进行判断），则可以返回true,如果任务已经完成或者已经停止，或者这个任务无法停止，则会返回false.

​		isDone（）方法判断当前方法是否完成

​		isCancel（）方法判断当前方法是否取消


#### Executor接口与线程池

​		我们知道线程池就是线程的集合，线程池集中管理线程，以实现线程的重用，降低资源消耗，提高响应速度等。线程用于执行异步任务，单个的线程既是工作单元也是执行机制，从JDK1.5开始，为了把工作单元与执行机制分离开，Executor框架诞生了，他是一个用于统一创建与运行的接口。Executor框架实现的就是线程池的功能。

Executor框架包括3大部分

（1）任务。也就是工作单元，包括被执行任务需要实现的接口：Runnable接口或者Callable接口；

（2）任务的执行。也就是把任务分派给多个线程的执行机制，也就是线程池对线程的管理功能，包括Executor接口及继承自Executor接口的ExecutorService接口。

（3）异步计算的结果。包括Future接口及实现了Future接口的FutureTask类。

![](image\微信图片_20221008152014.png)

​		使用步骤：

​		（1）创建Runnable并重写run（）方法或者Callable对象并重写call（）方法

​		（2）创建Executor接口的实现类ThreadPoolExecutor类或者ScheduledThreadPoolExecutor类的对象，然后调用其execute（）方法或者submit（）方法把工作任务添加到线程中，如果方法调用有返回值则返回Future对象。其中，submit方法可以提交Callable<T>对象或者Runnable对象，而execute方法只能提交Runnable对象。

​		 （3）调用Future对象的get（）方法后的返回值，或者调用Future对象的cancel（）方法取消当前线程的执行。最后关闭线程池。

```java
public class currentThreadTest { 
public static void main(String[] args) {
        ExecutorService service = Executors.newCachedThreadPool();
//        ExecutorService service = Executors.newFixedThreadPool(3);
        Future<String> future = service.submit(new Callable<String>(){
            @Override
            public String call() throws Exception {
                Thread.sleep(100);
                return "Hello World";
            }
        });

//        Future<String> future = service.submit(() -> {
//                Thread.sleep(100);
//                return "Hello World";
//            }
//        );

        System.out.println("More Processing...");
//        future.cancel(true);
        getIfNotCancelled(future);
//        future.cancel(true);
        System.out.println("结束");
    	
    }

    public static void getIfNotCancelled(Future<String> future){
        try{
            if(!future.isCancelled()){
                System.out.println(future.get());
            }else {
                System.out.println("Cancelled");
            }
        }catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }
    }
}
```

##### ThreadPoolExecutor

我们可以通过ThreadPoolExecutor来创建线程池（ThreadPoolExecutor是Executor框架中很重要的一个实现类）

```java
ThreadPoolExecutor tpe = new ThreadPoolExecutor(corePoolSize, maximumPoolSize, keepAliveTime, timeUnit, runnalbleTaskQueue, handler);
```

可以看到初始化的参数很多，但是记清楚了工作流程应该就很容易记得了：

（1）corePoolSize：核心线程池大小。如果调用了prestartAllCoreThread（）方法，那么线程池会提前创建并启动所有基本线程。

（2）maximumPoolSize：线程池大小。一个线程池中最多能容纳的线程数量。

（3）keepAliveTime：线程空闲后，线程存活时间。如果任务多，任务周期短，可以调大keepAliveTime，提高线程利用率。

（4）timeUnit：存活时间的单位，有天（DAYS）、小时(HOURS)、分（MINUTES）、秒（SECONDS）、毫秒（MILLISECONDS）、微秒（MICROSECONDS）、纳秒（NANOSECONDS）

（5）runnalbleTaskQueue：阻塞队列。可以使用ArrayBlockingQueue、LinkedBlockingQueue、SynchronousQueue、PriorityBlockingQueue

静态工厂方法Executors.newFixedThreadPool(  )使用了LinkedBlockingQueue；

静态工厂方法Executors.newCachedThreadPool(  )使用了SynchronousQueue；

（6）handler：饱和策略的句柄，当线程池满了的时候，任务无法得到处理，这时候需要饱和策略来处理无法完成的任务，饱和策略中有4种处理策略：

AbortPolicy：这是默认的策略，直接抛出异常；

CallerRunsPolicy：只是用调用者所在线程来运行任务；

DiscardOldestPolicy：丢弃队列中最老的任务，并执行当前任务；

DiscardPolicy：不处理，直接把当前任务丢弃；

当然也可以自定义饱和处理策略，需要实现RejectedExecutionHandler接口，比如记录日志或者持久化不能存储的任务等。

```java
public class currentThreadTest3 {
//    final static ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(2,5,10L, TimeUnit.SECONDS
//            ,new ArrayBlockingQueue<>(8));
//
//    public static void main(String[] args) {
////        for(int i = 0;i<8;i++){
////        for(int i = 0;i<13;i++){
//        for(int i = 0;i<14;i++){
//            threadPoolExecutor.execute(() ->{
//                try {
//                    Thread.sleep(2000);
//                }catch (InterruptedException e){
//                    e.printStackTrace();
//                }
//                System.out.println(Thread.currentThread().getName());
//            });
//        }
//    }

//    final static ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(2,5,2L, TimeUnit.SECONDS
//            ,new ArrayBlockingQueue<>(8));
//
//
//    public static void main(String[] args) {
//        try{
//            for(int i = 0;i<13;i++){
//            threadPoolExecutor.execute(() ->{
//                try {
//                    Thread.sleep(2000);
//                }catch (InterruptedException e){
//                    e.printStackTrace();
//                }
//                System.out.println(Thread.currentThread().getName());
//            });
//        }
//        Thread.sleep(15000);        //可能需要打个断点检查
//        System.out.println("main_end");
//        }catch (InterruptedException e){
//            e.printStackTrace();
//        }
//
//    }



//    final static ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(2,5,2L, TimeUnit.SECONDS
//            ,new ArrayBlockingQueue<>(8));
//
//
//    public static void main(String[] args) {
//        try{
//            for(int i = 0;i<13;i++){
//                final int j = i;
//                threadPoolExecutor.execute(() ->{
//                    try {
//                        Thread.sleep(2000);
//                    }catch (InterruptedException e){
//                        e.printStackTrace();
//                    }
//                    System.out.println(Thread.currentThread().getName());
//                    System.out.println(Thread.currentThread().getName() + "     任务"+j);
////                System.out.println("任务"+j);
////                System.out.println("========================");
//                });
//            }
//            Thread.sleep(15000);        //可能需要打个断点检查
//            System.out.println("main_end");
//        }catch (InterruptedException e){
//            e.printStackTrace();
//        }
//
//    }


    //CallerRunsPolicy
//    final static ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(2,5,2L, TimeUnit.SECONDS
//            ,new ArrayBlockingQueue<>(8),new ThreadPoolExecutor.CallerRunsPolicy());
//
//
//    public static void main(String[] args) {
//        try{
//            for(int i = 0;i<14;i++){
//                final int j = i;
//                threadPoolExecutor.execute(() ->{
//                    try {
//                        Thread.sleep(2000);
//                    }catch (InterruptedException e){
//                        e.printStackTrace();
//                    }
//                    System.out.println(Thread.currentThread().getName());
//                    System.out.println(Thread.currentThread().getName() + "     任务"+j);
////                System.out.println("任务"+j);
////                System.out.println("========================");
//                });
//            }
//            Thread.sleep(15000);        //可能需要打个断点检查
//            System.out.println("main_end");
//        }catch (InterruptedException e){
//            e.printStackTrace();
//        }
//
//    }



    //自定义handler
//    final static ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(2,5,2L, TimeUnit.SECONDS
//            ,new ArrayBlockingQueue<>(8),((runnable, executor) -> {
//            System.out.println(executor);
//    }));
//
//
//    public static void main(String[] args) {
//        try{
//            for(int i = 0;i<14;i++){
//                final int j = i;
//                threadPoolExecutor.execute(() ->{
//                    try {
//                        Thread.sleep(2000);
//                    }catch (InterruptedException e){
//                        e.printStackTrace();
//                    }
//                    System.out.println(Thread.currentThread().getName());
//                    System.out.println(Thread.currentThread().getName() + "     任务"+j);
////                System.out.println("任务"+j);
////                System.out.println("========================");
//                });
//            }
//            Thread.sleep(15000);        //可能需要打个断点检查
//            System.out.println("main_end");
//        }catch (InterruptedException e){
//            e.printStackTrace();
//        }
//
//    }

//    //自定义handler
//    final static ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(2,5,2L, TimeUnit.SECONDS
//            ,new ArrayBlockingQueue<>(8),((runnable, executor) -> {
//            System.out.println(executor);
//    }));
//
//
//    public static void main(String[] args) {
//        try{
//            for(int i = 0;i<14;i++){
//                final int j = i;
//                threadPoolExecutor.execute(() ->{
//                    try {
//                        Thread.sleep(2000);
//                    }catch (InterruptedException e){
//                        e.printStackTrace();
//                    }
//                    System.out.println(Thread.currentThread().getName());
//                    System.out.println(Thread.currentThread().getName() + "     任务"+j);
////                System.out.println("任务"+j);
////                System.out.println("========================");
//                });
//            }
//            Thread.sleep(15000);        //可能需要打个断点检查
//            System.out.println("main_end");
//        }catch (InterruptedException e){
//            e.printStackTrace();
//        }
//
//    }


    //自定义ThreadFactory
    final static ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(2, 5, 2L, TimeUnit.SECONDS
            , new ArrayBlockingQueue<>(8), new ThreadFactory() {
        @Override
        public Thread newThread(Runnable r) {

            Thread t = Executors.defaultThreadFactory().newThread(r);
            t.setDaemon(true);
            return t;
        }
    }, ((runnable, executor) -> {
        System.out.println(executor);
    }));


    public static void main(String[] args) {
        try{
            for(int i = 0;i<14;i++){
                final int j = i;
                threadPoolExecutor.execute(() ->{
                    try {
                        Thread.sleep(2000);
                    }catch (InterruptedException e){
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName());
                    System.out.println(Thread.currentThread().getName() + "     任务"+j);
//                System.out.println("任务"+j);
//                System.out.println("========================");
                });
            }
            Thread.sleep(15000);        //可能需要打个断点检查
            System.out.println("main_end");
        }catch (InterruptedException e){
            e.printStackTrace();
        }

    }



}
```

```java
public class currentThreadTest4 {
    public static void main(String[] args) {
        ExecutorService service = Executors.newCachedThreadPool();
//        ExecutorService service = Executors.newFixedThreadPool(3);
        Future<String> future = service.submit(new Callable<String>(){
            @Override
            public String call() throws Exception {
                Thread.sleep(1000);
                return "Hello World";
            }
        });

//        Future<String> future = service.submit(() -> {
//                Thread.sleep(100);
//                return "Hello World";
//            }
//        );

        System.out.println("More Processing...");
        future.cancel(true);
        getIfNotCancelled(future);
//        future.cancel(true);
        System.out.println("结束");
    }

    public static void getIfNotCancelled(Future<String> future){
        try{
            if(!future.isCancelled()){
                System.out.println(future.get());
            }else {
                System.out.println("Cancelled");
            }
        }catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }
    }
}
```

###### 提交任务：execute() 或者 submit()

execute用于提交没有返回值的任务，所以无法判断任务是否被线程执行过。

submit用于提交需要返回值的对象。返回一个future对象，可以通过future对象判断任务是否被线程执行；可以通过future对象的get（）方法获取返回的值，get（）方法会阻塞当前线程直到future对象被返回，也可以调用get（long timeout, TimeUnit unit）来实现由等待时间的获取返回值，如果超时仍没有返回值，那么立刻返回，这时候任务可能没有执行完成。

###### 关闭线程池：可以通过shutdown或者shutDownNow来关闭线程池

（1）shutDown:把线程池的状态设置为SHUTDOWN，然后**中断所有没有正在执行任务的线程，而已经在执行任务的线程继续执行直到任务执行完毕**；

（2）shutDownNow：把当前线程池状态设为STOP，尝试**停止所有的正在执行或者暂停的线程**，并返回等待执行的任务的列表；



###### 线程池的五种状态

线程池的5种状态：Running、ShutDown、Stop、Tidying、Terminated。

1.RUNNING
状态说明：线程池处在RUNNING状态时，能够接收新任务，以及对已添加的任务进行处理。
状态切换：线程池的初始化状态是RUNNING。换句话说，线程池被一旦被创建，就处于RUNNING状态，并且线程池中的任务数为0！
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));

2.SHUTDOWN

状态说明：线程池处在SHUTDOWN状态时，不接收新任务，但能处理已添加的任务。
状态切换：调用线程池的shutdown()接口时，线程池由RUNNING -> SHUTDOWN。

3.STOP

状态说明：线程池处在STOP状态时，不接收新任务，不处理已添加的任务，并且会中断正在处理的任务。
状态切换：调用线程池的shutdownNow()接口时，线程池由(RUNNING or SHUTDOWN ) -> STOP。

4.TIDYING

状态说明：当所有的任务已终止，ctl记录的”任务数量”为0，线程池会变为TIDYING状态。当线程池变为TIDYING状态时，会执行钩子函数terminated()。terminated()在ThreadPoolExecutor类中是空的，若用户想在线程池变为TIDYING时，进行相应的处理；可以通过重载terminated()函数来实现。
状态切换：当线程池在SHUTDOWN状态下，阻塞队列为空并且线程池中执行的任务也为空时，就会由 SHUTDOWN -> TIDYING。
当线程池在STOP状态下，线程池中执行的任务为空时，就会由STOP -> TIDYING。

5.TERMINATED

状态说明：线程池彻底终止，就变成TERMINATED状态。
状态切换：线程池处在TIDYING状态时，执行完terminated()之后，就会由 TIDYING -> TERMINATED。

![](image\微信图片_20221008200033.png)

###### ThreadPoolExecutor的几个工厂方法

FixedThreadPool、SingleThreadExecutor、CachedThreadPool，它们的底层都是ThreadPoolExecutor的构造方法。

FixedThreadPool是线程数量固定的线程池。适用于为了满足资源管理的需求，而需要适当限制当前线程数量的情景，适用于负载比较重的服务器。

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>(),
                                      threadFactory);
    }
```

SingleThreadExecutor是只有一个线程的线程池，常用于需要让线程顺序执行，并且在任意时间，只能有一个任务被执行，而不能有多个线程同时执行的场景。

```java
public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
 
    public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory) {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>(),
                                    threadFactory));
    }
```

CachedThreadPool适用于并发量变化较大的情形，或者是负载较轻的服务器

CachedThreadPool使用SynchronizedQueue作为阻塞队列，SynchronizedQueue是不存储元素的阻塞队列，实现“一对一的交付”，也就是说，每次向队列中put一个任务必须等有线程来take这个任务，否则就会一直阻塞该任务，如果一个线程要take一个任务就要一直阻塞直到有任务被put进阻塞队列。

简单来说，就是每有一个新的任务被提交给线程池，就会有空线程来获取并执行它，若没有空线程，就立刻新建一个线程来执行它。

```java
public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
public static ExecutorService newCachedThreadPool(ThreadFactory threadFactory) {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>(),
                                  threadFactory);
}
```

###### ScheduledThreadPoolExecutor

ScheduledThreadPoolExecutor可以创建n个可延迟执行的线程池。延迟执行需要调用schedule方法。

```java
ScheduledExecutorService stp = Executors.newScheduledThreadPool(int threadNums);
ScheduledExecutorService stp = Executors.newScheduledThreadPool(int threadNums, ThreadFactory threadFactory);
```

```java
public class currentThreadTest5 {
    public static void main(String[] args) {
        ScheduledExecutorService threadPool = Executors.newScheduledThreadPool(5);
        System.out.println("添加时间：" + new Date());
        threadPool.schedule(()->{
            System.out.println("执行时间：" + new Date());
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },1,TimeUnit.SECONDS);
    }
}
```



##### ForkJoinPool

​		同ThreadPoolExecutor一样，ForkJoinPool也实现了Executor和ExecutorService接口。它使用了一个无限队列来保存需要执行的任务，而线程的数量则是通过构造函数传入，如果没有向构造函数中传入希望的线程数量，那么当前计算机可用的CPU数量会被设置为线程数量作为默认值。

```java
public class ForkJoinPoolTest0 {
    static ForkJoinPool forkJoinPool = new ForkJoinPool();
//    static ForkJoinPool forkJoinPool = new ForkJoinPool(10);
    
    //static ForkJoinPool forkJoinPool = new ForkJoinPool(10,ForkJoinPool.defaultForkJoinWorkerThreadFactory,(thread,ex)->{
//    System.out.println(thread.getName());
//},false,5,10,1,null,100L, TimeUnit.SECONDS);
    public static void main(String[] args) {

        IntStream.range(0,30).forEach(n ->{
            forkJoinPool.submit(() ->{
                try {
                    Thread.sleep(200);
                }catch (InterruptedException e){
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName());
            });
        });
        System.out.println("主线程");

        try {
            Thread.sleep(1200);
            System.out.println(forkJoinPool.getPoolSize());
        }catch (InterruptedException e){
            e.printStackTrace();
        }


    }
}
```

###### 工作窃取与拆分子任务

​		使用ForkJoinPool的核心思想就是任务拆分，把任务拆分成多个子任务（调用fork方法），然后多个子任务同时运行，最后把结果收集起来（join方法），从而达到节省性能的目的。

![](image\微信图片_20221009204428.png)

​		work-stealing（工作窃取），ForkJoinPool提供了一个更有效的利用线程的机制，当ThreadPoolExecutor还在用单个队列存放任务时，ForkJoinPool已经分配了与线程数相等的队列，当有任务加入线程池时，会被平均分配到对应的队列上，各线程进行正常工作，当有线程提前完成时，会从队列的末端“窃取”其他线程未执行完的任务，当任务量特别大时，CPU多的计算机会表现出更好的性能。

​		当有一个新任务被提交的时候，ForkJoinPool可能会创建一个新的线程。然后会有一个等待队列与这个线程对应。当我们给任务拆分（添加）子任务的时候，新加的任务会被添加到一些已有的队列里，当其他线程的任务执行完毕后，可以从这个线程对应的等待队列里获取任务并执行。

![](image\微信图片_20221009204434.png)

​		

​		ForkJoinPool可以调用submit和execute方法来提交任务，因为它也继承了AbstractExecutorService类。此外，它还有invoke方法，我们可以使用invoke方法并在该方法中传入一个ForkJoinTask的实现对象来进行拆分子任务的操作。

​		具体操作是：

​		1.我们调用invoke方法的时候，要对传入的ForkJoinTask类进行继承。主要是要重写三个方法 getRawResult()，setRawResult(Long value)，exec()。这时我们还没有拆分子任务。

​		其中，我们在exec方法中写拆分子任务的代码（当然也可以写自己要执行的任务）。然后通过getRawResult返回结果。因为这个ForkJoinTask一旦被执行，首先执行的就是它的exec方法，而exec方法的返回值为boolean，所以，只有当exec执行完毕并返回true时，才会执行getRawResult。这个结果，需要通过taskObj.join方法才能拿到，而在invoke方法中，最后返回的就是taskObj.join。因此invoke和taskObj.join一样，是一个阻塞的方法。

```java
public class ForkJoinPoolTest1 {

    static ForkJoinPool forkJoinPool = new ForkJoinPool();

    public static void main(String[] args) {
        Long invoke = forkJoinPool.invoke(new ForkJoinTask<Long>() {
            @Override
            public Long getRawResult() {
                System.out.println("get");
                return 10L;
            }

            @Override
            protected void setRawResult(Long value) {
                System.out.println("set");
            }

            @Override
            protected boolean exec() {
                System.out.println(Thread.currentThread().getName());
                return true;
            }
        });
        System.out.println(invoke);

    }
}
```

​		2.我们在exec中创建新的 ForkJoinTask对象，并重写这个类的三个方法。然后调用这个ForkJoinTask对象的fork方法提交任务，并用join方法返回结果。这样，我们就拆分出了一个个子任务。

```java
public class ForkJoinPoolTest2 {

    static ForkJoinPool forkJoinPool = new ForkJoinPool();

    public static void main(String[] args) {
        Long invoke = forkJoinPool.invoke(new ForkJoinTask<Long>() {

            private Long rt = 0L;

            @Override
            public Long getRawResult() {
                System.out.println("get");
                return rt ;
            }

            @Override
            protected void setRawResult(Long value) {
                System.out.println("set");
            }

            @Override
            protected boolean exec() {
                System.out.println(Thread.currentThread().getName());
                ForkJoinTask<Long> task1 = new ForkJoinTask<Long>() {
                    @Override
                    public Long getRawResult() {
                        return 55L + rt;
                    }

                    @Override
                    protected void setRawResult(Long value) {

                    }

                    @Override
                    protected boolean exec() {
                        System.out.println(Thread.currentThread().getName());
//                        return false;
                        return true;
                    }
                };
                ForkJoinTask<Long> task2 = new ForkJoinTask<Long>() {
                    @Override
                    public Long getRawResult() {
                        return 55L + rt;
                    }

                    @Override
                    protected void setRawResult(Long value) {

                    }

                    @Override
                    protected boolean exec() {
                        System.out.println(Thread.currentThread().getName());
//                        return false;
                        return true;
                    }
                };

                task1.fork();
                task2.fork();
//                try {
//                    Thread.sleep(500);
//                } catch (InterruptedException e) {
//                    e.printStackTrace();
//                }
                rt = task1.join()+task2.join();

                return true;
            }
        });
        System.out.println(invoke);

    }
}
```

我们还可以通过继承ForkJoinTask的子类RecursiveTask来实现子任务拆分

```java
public class ForkJoinPoolTest3 {

    private static ForkJoinPool forkJoinPool = new ForkJoinPool();

    private static class SumTask extends RecursiveTask<Long>{
        private long[] numbers;
        private int from;
        private int to;
        private long total = 0;

        public SumTask(long[] numbers,int from,int to){
            this.numbers = numbers;
            this.from = from;
            this.to = to;
        }

        @Override
        protected Long compute() {
            if(to-from<10){
                IntStream.rangeClosed(from,to).forEach(n ->{
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    total += numbers[n];
                });
                return total;
            }else{
                int middle = (from+to)/2;
                SumTask task1 = new SumTask(numbers,from,middle);
                SumTask task2 = new SumTask(numbers,middle+1,to);
                task1.fork();
                task2.fork();
                return task1.join()+task2.join();
            }
        }
    }

    public static void main(String[] args) {

        long[] numbers = LongStream.rangeClosed(1,100).toArray();
        long startTime = System.currentTimeMillis();

        Long invoke = forkJoinPool.invoke(new SumTask(numbers,0,numbers.length-1));


//        Long invoke = 0L;
//        for (int i = 0;i< numbers.length;i++){
//            try {
//                Thread.sleep(100);
//            } catch (InterruptedException e) {
//                e.printStackTrace();
//            }
//            invoke += numbers[i];
//        }


        long endTime = System.currentTimeMillis();


        System.out.println(invoke);
        System.out.println("程序运行时间：" + (endTime - startTime) + "ms");

    }
}
```

#### CompletableFuture

​		Future的使用实际上就是获取线程的执行结果与执行状态，当需要进一步处理时，主线程需要调用get()方法。
​		但调用get()方法的时机很难把控，立即调用相当于串行了，很久之后调用那就白白浪费了等待的时间。因此Future的对异步处理的调用并不灵活。

​		在jdk1.8，java推出了CompletableFuture。在异步任务完成后，需要用到任务的返回结果时无需等待，直接通过thenAccept、thenApply、thenCompose等方法来对异步调用的结果进行处理。

##### 完成一个CompletableFuture

```java
public class CompletableFutureTest {    

	private static Map<Integer, Product> cache = new HashMap<>();


    private Logger logger = Logger.getLogger(this.getClass().getName());


    private Product getLocal(int id) {
        return cache.get(id);  }




    //创建一个新的Product
    private Product getRemote(int id) {
        try {
            Thread.sleep(100);
            System.out.println(Thread.currentThread().getName());
            if (id == 666) {
                throw new RuntimeException("Evil request");
            }
        } catch (InterruptedException ignored) {
        }
        return new Product(id, "name");
    }    


public static void main(String[] args) throws Exception {
        CompletableFutureTest completableFutureTest = new CompletableFutureTest();
        try{

//            Product p = completableFutureTest.getProduct(333).get();
            Product p = completableFutureTest.getProduct(666).get();
            System.out.println(p);
        }catch (ExecutionException e){
            System.out.println(e.getClass());
            System.out.println(e.getCause().getClass());
        }
//
//
//            Product p = completableFutureTest.getProduct(333).join();
//////            Product p = completableFutureTest.getProduct(666).get();
//            System.out.println(p);

    }
    
    


    public CompletableFuture<Product> getProduct(int id) {
        try {
            Product product = getLocal(id);
            if (product != null) {
                return CompletableFuture.completedFuture(product);
            } else {
                CompletableFuture<Product> future = new CompletableFuture<>();
                Product p = getRemote(id);
                cache.put(id, p);
                future.complete(p);
                return future;
            }
        } catch (Exception e) {
            CompletableFuture<Product> future = new CompletableFuture<>();
            future.completeExceptionally(e);
//            throw new RuntimeException("nonono");           //如果不返回future直接抛出异常，下面的代码则不会执行到catch，因为下面的catch抓的是ExecutionException
            return future;                            //如果返回future而不直接抛出异常，下面的代码则会执行到catch，因为get方法会抛出ExecutionException。
                                                        //那么为什么我们一定要使用这么一个ExecutionException对这个异常进行一个”包装“呢？
                                                        // 这其实就是CompletableFuture的意义，在实际的运行环境中，如果异步操作出现异常，get方法抛出来的ExecutionException作为一个受检异常，要求程序员必须显示处理这个异常
                                                        //  要么try/catch，要么throws。而其join()方法则是一个非受检异常，不会强制处理
        }
    }
    
    
        public CompletableFuture<Product> getProductAsync(int id) {
        try {
            Product product = getLocal(id);
            if (product != null) {
                logger.info("getLocal with id=" + id);
                return CompletableFuture.completedFuture(product);
            } else {
                logger.info("getRemote with id=" + id);
                                //CompletableFuture<Product> future = new CompletableFuture<>();
                //Product p = getRemote(id);
                //cache.put(id, p);
                //future.complete(p);
                //return future;
                return CompletableFuture.supplyAsync(() -> {
                    Product p = getRemote(id);
                    cache.put(id, p);
                    return p;
                });
            }
        } catch (Exception e) {
            logger.info("exception thrown");
            CompletableFuture<Product> future = new CompletableFuture<>();
            future.completeExceptionally(e);
            return future;
        }
    }
    
}
```

从上面的代码可以看出，我们可以往一个CompletableFuture里面放数据或者异常，然后用Get方法把它拿出来。

#####  runAsync 和 supplyAsync方法

​	CompletableFuture 提供了四个静态方法来创建一个异步操作。这些方法可以直接把执行结果或者异常放进CompletableFuture里面。并且，我们可以使用Join或者get方法等待这个异步操作执行完成并检索返回值。

```java
public static CompletableFuture<Void> runAsync(Runnable runnable)
public static CompletableFuture<Void> runAsync(Runnable runnable, Executor executor)
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier)
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier, Executor executor)
```

没有指定Executor的方法会使用ForkJoinPool.commonPool() 作为它的线程池执行异步代码。如果指定线程池，则使用指定的线程池运行。以下所有的方法都类同。

- runAsync方法不支持返回值。

- supplyAsync可以支持返回值。

  ```java
  public void runAsync() throws Exception {
      CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
          try {
              TimeUnit.SECONDS.sleep(1);
          } catch (InterruptedException e) {
          }
          System.out.println("run end ...");
      });
      
      future.get();
  }
  
  //有返回值
  public static void supplyAsync() throws Exception {         
      CompletableFuture<Long> future = CompletableFuture.supplyAsync(() -> {
          try {
              TimeUnit.SECONDS.sleep(1);
          } catch (InterruptedException e) {
          }
          System.out.println("run end ...");
          return System.currentTimeMillis();
      });
  
      long time = future.get();
      System.out.println("time = "+time);
  }
  ```

##### 计算结果完成时的回调方法

当CompletableFuture的计算结果完成，或者抛出异常的时候，可以执行特定的Action。主要是下面的方法：

```java
public CompletableFuture<T> whenComplete(BiConsumer<? super T,? super Throwable> action)
public CompletableFuture<T> whenCompleteAsync(BiConsumer<? super T,? super Throwable> action)
public CompletableFuture<T> whenCompleteAsync(BiConsumer<? super T,? super Throwable> action, Executor executor)
public CompletableFuture<T> exceptionally(Function<Throwable,? extends T> fn)
```

可以看到Action的类型是BiConsumer<? super T,? super Throwable>它可以处理正常的计算结果，或者异常情况。

whenComplete 和 whenCompleteAsync 的区别：
 whenComplete：是执行当前任务的线程执行继续执行 whenComplete 的任务。相当于讲两部分任务结合在一起一次性完成。
 whenCompleteAsync：是执行把 whenCompleteAsync 这个任务继续提交给线程池来进行执行。提交时有可能刚刚执行的线程会被其它任务抢占，这时候就需要在一个新线程中执行。

exceptionally：当执行时出现异常时，执行其传入的Function的方法。其返回值必须和调用其的CompletableFuture实例包裹的类型具有继承关系。

```java
public void whenComplete() throws Exception {
    CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
        }
        if(new Random().nextInt()%2>=0) {
            int i = 12/0;
        }
        System.out.println("run end ...");
    });
    
    future.whenComplete(new BiConsumer<Void, Throwable>() {
        @Override
        public void accept(Void t, Throwable action) {
            System.out.println("执行完成！");
        }
        
    });
    future.exceptionally(new Function<Throwable, Void>() {
        @Override
        public Void apply(Throwable t) {
            System.out.println("执行失败！"+t.getMessage());
            return null;
        }
    });
    
    TimeUnit.SECONDS.sleep(2);
}
```

##### thenApply 方法

当一个线程依赖另一个线程时，可以使用 thenApply 方法来把这两个线程串行化。

```java
public <U> CompletableFuture<U> thenApply(Function<? super T,? extends U> fn)
public <U> CompletableFuture<U> thenApplyAsync(Function<? super T,? extends U> fn)
public <U> CompletableFuture<U> thenApplyAsync(Function<? super T,? extends U> fn, Executor executor)
```

Function<? super T,? extends U>
T：上一个任务返回结果的类型
U：当前任务的返回值类型

```java
public void thenApply() throws Exception {
    CompletableFuture<Long> future = CompletableFuture.supplyAsync(new Supplier<Long>() {
        @Override
        public Long get() {
            long result = new Random().nextInt(100);
            System.out.println("result1="+result);
            return result;
        }
    }).thenApply(new Function<Long, Long>() {
        @Override
        public Long apply(Long t) {
            long result = t*5;
            System.out.println("result2="+result);
            return result;
        }
    });
    
    long result = future.get();
    System.out.println(result);
}
```

第二个任务依赖第一个任务的结果。

##### handle 方法

handle 是执行任务完成时对结果的处理。
handle 方法和 thenApply 方法处理方式基本一样。不同的是 handle 是在任务完成后再执行，还可以处理异常的任务（thenApplyAsync）。thenApply 只可以执行正常的任务，任务出现异常则不执行 thenApply 方法。

```java
public <U> CompletionStage<U> handle(BiFunction<? super T, Throwable, ? extends U> fn);
public <U> CompletionStage<U> handleAsync(BiFunction<? super T, Throwable, ? extends U> fn);
public <U> CompletionStage<U> handleAsync(BiFunction<? super T, Throwable, ? extends U> fn,Executor executor);
```

```java
public void handle() throws Exception{
    CompletableFuture<Integer> future = CompletableFuture.supplyAsync(new Supplier<Integer>() {

        @Override
        public Integer get() {
            int i= 10/0;
            return new Random().nextInt(10);
        }
    }).handle(new BiFunction<Integer, Throwable, Integer>() {
        @Override
        public Integer apply(Integer param, Throwable throwable) {
            int result = -1;
            if(throwable==null){
                result = param * 2;
            }else{
                System.out.println(throwable.getMessage());
            }
            return result;
        }
     });
    System.out.println(future.get());
}
```

从示例中可以看出，在 handle 中可以根据任务是否有异常来进行做相应的后续处理操作。而 thenApply 方法，如果上个任务出现错误，则不会执行 thenApply 方法。

##### thenAccept 方法

接收任务的处理结果，并消费处理，无返回结果。

```java
public CompletionStage<Void> thenAccept(Consumer<? super T> action);
public CompletionStage<Void> thenAcceptAsync(Consumer<? super T> action);
public CompletionStage<Void> thenAcceptAsync(Consumer<? super T> action,Executor executor);
```

```java
public void thenAccept() throws Exception{
    CompletableFuture<Void> future = CompletableFuture.supplyAsync(new Supplier<Integer>() {
        @Override
        public Integer get() {
            return new Random().nextInt(10);
        }
    }).thenAccept(integer -> {
        System.out.println(integer);
    });
    future.get();
}
```

##### thenRun 方法

该方法同 thenAccept 方法类似。

跟 thenAccept 方法不一样的是，不关心任务的处理结果。只要上面的任务执行完成，就开始执行 thenRun 。而上个任务处理完成后，并不会把计算的结果传给 thenRun 方法。

```java
public CompletionStage<Void> thenRun(Runnable action);
public CompletionStage<Void> thenRunAsync(Runnable action);
public CompletionStage<Void> thenRunAsync(Runnable action,Executor executor);
```

```java
public void thenRun() throws Exception{
    CompletableFuture<Void> future = CompletableFuture.supplyAsync(new Supplier<Integer>() {
        @Override
        public Integer get() {
            return new Random().nextInt(10);
        }
    }).thenRun(() -> {
        System.out.println("thenRun ...");
    });
    future.get();
}
```

##### thenCombine 合并任务

thenCombine 会把 两个 CompletionStage 的任务都执行完成后，把两个任务的结果一块交给 thenCombine 来处理。与其它方法不同的是，在不指定Executor的时候，thenCombine方法与thenCombineAsync方法的执行效果实际上差不多。thenCombine会在前两个任务执行的其中一个线程跟随着继续执行。

```java
public <U,V> CompletionStage<V> thenCombine(CompletionStage<? extends U> other,BiFunction<? super T,? super U,? extends V> fn);
public <U,V> CompletionStage<V> thenCombineAsync(CompletionStage<? extends U> other,BiFunction<? super T,? super U,? extends V> fn);
public <U,V> CompletionStage<V> thenCombineAsync(CompletionStage<? extends U> other,BiFunction<? super T,? super U,? extends V> fn,Executor executor);
```

```java
private void thenCombine() throws Exception {
    CompletableFuture<String> future1 = CompletableFuture.supplyAsync(new Supplier<String>() {
        @Override
        public String get() {
            return "hello";
        }
    });
    CompletableFuture<String> future2 = CompletableFuture.supplyAsync(new Supplier<String>() {
        @Override
        public String get() {
            return "hello";
        }
    });
    CompletableFuture<String> result = future1.thenCombine(future2, new BiFunction<String, String, String>() {
        @Override
        public String apply(String t, String u) {
            return t+" "+u;
        }
    });
    System.out.println(result.get());
}
```

##### thenAcceptBoth

当两个CompletionStage都执行完成后，把结果一块交给thenAcceptBoth来进行消耗。

```java
public <U> CompletionStage<Void> thenAcceptBoth(CompletionStage<? extends U> other,BiConsumer<? super T, ? super U> action);
public <U> CompletionStage<Void> thenAcceptBothAsync(CompletionStage<? extends U> other,BiConsumer<? super T, ? super U> action);
public <U> CompletionStage<Void> thenAcceptBothAsync(CompletionStage<? extends U> other,BiConsumer<? super T, ? super U> action,     Executor executor);
```

```java
public void thenAcceptBoth() throws Exception {
    CompletableFuture<Integer> f1 = CompletableFuture.supplyAsync(new Supplier<Integer>() {
        @Override
        public Integer get() {
            int t = new Random().nextInt(3);
            try {
                TimeUnit.SECONDS.sleep(t);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("f1="+t);
            return t;
        }
    });
        
    CompletableFuture<Integer> f2 = CompletableFuture.supplyAsync(new Supplier<Integer>() {
        @Override
        public Integer get() {
            int t = new Random().nextInt(3);
            try {
                TimeUnit.SECONDS.sleep(t);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("f2="+t);
            return t;
        }
    });
    f1.thenAcceptBoth(f2, new BiConsumer<Integer, Integer>() {
        @Override
        public void accept(Integer t, Integer u) {
            System.out.println("f1="+t+";f2="+u+";");
        }
    });
    Thread.sleep(5000);
}
```

##### applyToEither 方法

两个CompletionStage，谁执行返回的结果快，我就用那个CompletionStage的结果进行下一步的转化操作。

```java
public <U> CompletionStage<U> applyToEither(CompletionStage<? extends T> other,Function<? super T, U> fn);
public <U> CompletionStage<U> applyToEitherAsync(CompletionStage<? extends T> other,Function<? super T, U> fn);
public <U> CompletionStage<U> applyToEitherAsync(CompletionStage<? extends T> other,Function<? super T, U> fn,Executor executor);
```

```java
public void applyToEither() throws Exception {
    CompletableFuture<Integer> f1 = CompletableFuture.supplyAsync(new Supplier<Integer>() {
        @Override
        public Integer get() {
            int t = new Random().nextInt(3);
            try {
                TimeUnit.SECONDS.sleep(t);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("f1="+t);
            return t;
        }
    });
    CompletableFuture<Integer> f2 = CompletableFuture.supplyAsync(new Supplier<Integer>() {
        @Override
        public Integer get() {
            int t = new Random().nextInt(3);
            try {
                TimeUnit.SECONDS.sleep(t);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("f2="+t);
            return t;
        }
    });
    
    CompletableFuture<Integer> result = f1.applyToEither(f2, new Function<Integer, Integer>() {
        @Override
        public Integer apply(Integer t) {
            System.out.println(t);
            return t * 2;
        }
    });

    System.out.println(result.get());
    Thread.sleep(5000);
}
```

##### acceptEither 方法

两个CompletionStage，谁执行返回的结果快，我就用那个CompletionStage的结果进行下一步的消耗操作。

```java
public CompletionStage<Void> acceptEither(CompletionStage<? extends T> other,Consumer<? super T> action);
public CompletionStage<Void> acceptEitherAsync(CompletionStage<? extends T> other,Consumer<? super T> action);
public CompletionStage<Void> acceptEitherAsync(CompletionStage<? extends T> other,Consumer<? super T> action,Executor executor);
```

```java
public void acceptEither() throws Exception {
    CompletableFuture<Integer> f1 = CompletableFuture.supplyAsync(new Supplier<Integer>() {
        @Override
        public Integer get() {
            int t = new Random().nextInt(3);
            try {
                TimeUnit.SECONDS.sleep(t);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("f1="+t);
            return t;
        }
    });
        
    CompletableFuture<Integer> f2 = CompletableFuture.supplyAsync(new Supplier<Integer>() {
        @Override
        public Integer get() {
            int t = new Random().nextInt(3);
            try {
                TimeUnit.SECONDS.sleep(t);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("f2="+t);
            return t;
        }
    });
    f1.acceptEither(f2, new Consumer<Integer>() {
        @Override
        public void accept(Integer t) {
            System.out.println(t);
        }
    });
}
```

##### runAfterEither 方法

两个CompletionStage，任何一个完成了都会执行下一步的操作（Runnable）

```java
public CompletionStage<Void> runAfterEither(CompletionStage<?> other,Runnable action);
public CompletionStage<Void> runAfterEitherAsync(CompletionStage<?> other,Runnable action);
public CompletionStage<Void> runAfterEitherAsync(CompletionStage<?> other,Runnable action,Executor executor);
```

```java
public void runAfterEither() throws Exception {
    CompletableFuture<Integer> f1 = CompletableFuture.supplyAsync(new Supplier<Integer>() {
        @Override
        public Integer get() {
            int t = new Random().nextInt(3);
            try {
                TimeUnit.SECONDS.sleep(t);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("f1="+t);
            return t;
        }
    });
        
    CompletableFuture<Integer> f2 = CompletableFuture.supplyAsync(new Supplier<Integer>() {
        @Override
        public Integer get() {
            int t = new Random().nextInt(3);
            try {
                TimeUnit.SECONDS.sleep(t);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("f2="+t);
            return t;
        }
    });
    f1.runAfterEither(f2, new Runnable() {
        
        @Override
        public void run() {
            System.out.println("上面有一个已经完成了。");
        }
    });
}
```

##### runAfterBoth

两个CompletionStage，都完成了计算才会执行下一步的操作（Runnable）。

```java
public CompletionStage<Void> runAfterBoth(CompletionStage<?> other,Runnable action);
public CompletionStage<Void> runAfterBothAsync(CompletionStage<?> other,Runnable action);
public CompletionStage<Void> runAfterBothAsync(CompletionStage<?> other,Runnable action,Executor executor);
```

```java
public void runAfterBoth() throws Exception {
    CompletableFuture<Integer> f1 = CompletableFuture.supplyAsync(new Supplier<Integer>() {
        @Override
        public Integer get() {
            int t = new Random().nextInt(3);
            try {
                TimeUnit.SECONDS.sleep(t);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("f1="+t);
            return t;
        }
    });
        
    CompletableFuture<Integer> f2 = CompletableFuture.supplyAsync(new Supplier<Integer>() {
        @Override
        public Integer get() {
            int t = new Random().nextInt(3);
            try {
                TimeUnit.SECONDS.sleep(t);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("f2="+t);
            return t;
        }
    });
    f1.runAfterBoth(f2, new Runnable() {
        
        @Override
        public void run() {
            System.out.println("上面两个任务都执行完成了。");
        }
    });
}
```

##### thenCompose 方法

thenCompose 方法允许你对两个 CompletionStage 进行流水线操作，第一个操作完成时，将其结果作为参数传递给第二个操作。

```java
public <U> CompletableFuture<U> thenCompose(Function<? super T, ? extends CompletionStage<U>> fn);
public <U> CompletableFuture<U> thenComposeAsync(Function<? super T, ? extends CompletionStage<U>> fn) ;
public <U> CompletableFuture<U> thenComposeAsync(Function<? super T, ? extends CompletionStage<U>> fn, Executor executor) ;
```

```java
public void thenCompose() throws Exception {
        CompletableFuture<Integer> f = CompletableFuture.supplyAsync(new Supplier<Integer>() {
            @Override
            public Integer get() {
                int t = new Random().nextInt(3);
                System.out.println("t1="+t);
                return t;
            }
        }).thenCompose(new Function<Integer, CompletionStage<Integer>>() {
            @Override
            public CompletionStage<Integer> apply(Integer param) {
                return CompletableFuture.supplyAsync(new Supplier<Integer>() {
                    @Override
                    public Integer get() {
                        int t = param *2;
                        System.out.println("t2="+t);
                        return t;
                    }
                });
            }
            
        });
        System.out.println("thenCompose result : "+f.get());
    }
```

示例：

```java
public List<Map<String, Object>> R2DBCTest1(List<Integer> integerList){
    String sql = "SELECT NAME,score FROM table1 WHERE id = :id";
    List<Map<String, Object>> list = Flux.fromIterable(integerList).flatMap( i -> databaseClient.execute(sql).bind("id",i)
                    .fetch()
                    .first())
                    .collect(Collectors.toList())
                    .block();
    System.out.println(list);
    return list;
}

    private void saveResultList(List<Map<String,Object>> results) {
        results.parallelStream().forEach(this::saveResultToFile);
    }
    public void saveResultToFile(Map result) {
       
        try {
            File file = new File("");
            System.out.println("文件已修改");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public OptionalInt getMaxScore(List<Map<String,Object>> results) {
        return results.stream()
                .mapToInt(m -> Integer.parseInt(m.get("score").toString()))
                .max();
    }

    public Optional<String> getMaxName(List<Map<String,Object>> results) {
        return results.stream()
                .max((m1,m2) ->{
                    if(Integer.parseInt(m1.get("score").toString())>Integer.parseInt(m2.get("score").toString()))
                                    return 1;
                    else if(Integer.parseInt(m1.get("score").toString())<Integer.parseInt(m2.get("score").toString()))
                        return -1;
                    else return 0;
                }).map(m -> m.get("name").toString());
    }

    @Test
    public void printGames() {
        CompletableFuture<List<Map<String,Object>>> future =
                CompletableFuture.supplyAsync(() ->{
                    try {
                        Thread.sleep(1000);
                    }catch (InterruptedException e){
                     e.printStackTrace();
                    }
                    return Stream.of(1,2,3,4,5,6).collect(Collectors.toList());
                }).thenApply( this::R2DBCTest1);
        CompletableFuture<Void> futureWrite =
                future.thenAcceptAsync(this::saveResultList)
                .exceptionally(ex -> {
                        System.err.println(ex.getMessage());
                        return null;
                });
        CompletableFuture<OptionalInt> futureMaxScore =
                future.thenApplyAsync(this::getMaxScore);
        CompletableFuture<Optional<String>> futureMaxGame =
                future.thenApplyAsync(this::getMaxName);
        CompletableFuture<String> futureMax =
                futureMaxScore.thenCombineAsync(futureMaxGame,
        (score, result) ->
                String.format("Highest score: %d, Max Game: %s",
                        score.orElse(0), result.orElse(" ")));
        CompletableFuture.allOf(futureWrite, futureMax).join();
        future.join().forEach(System.out::println);
        System.out.println(futureMax.join());
    }
```

