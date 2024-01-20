# java.time包


[java.time 包](http://download.java.net/jdk8/docs/api/java/time/package-summary.html)是在JDK8新引入的，提供了用于日期、时间和实例的主要API。使用instants, durations, dates, times, time-zones 等类定义的工厂方法。Date-Time API中的所有类均生成不可变的实例，所有实例都是不可变的

<!--more-->

# Java 8 之 java.time 包

### 概述

[java.time 包](http://download.java.net/jdk8/docs/api/java/time/package-summary.html)是在JDK8新引入的，提供了用于日期、时间和实例的主要API。使用instants, durations, dates, times, time-zones 等类定义的工厂方法。Date-Time API中的所有类均生成不可变的实例，所有实例都是不可变的。由于这些类不提供公共构造函数，需要采用工厂方法加以实例化。（of,now两种静态工厂方法）

### 一些类的介绍

- LocalDateTime:存储了日期和时间，如：`2013-10-15T14:43:14.539`。
- LocalDate:存储了日期，如：`2013-10-15`。
- LocalTime:存储了时间，如：`14:43:14.539`。
- **Instant类**
  Instant类对时间轴上的单一瞬时点建模，可以用于记录应用程序中的事件时间戳
- **Duration类**
  Duration类表示秒或纳秒时间间隔，适合处理较短的时间，需要更高的精确性。
- **Period类**
  Period类表示一段时间的年、月、日。
- **ZonedDateTime类**
  ZonedDateTime是具有时区的日期时间的不可变表示，此类存储所有日期和时间字段，精度为纳秒，时区为区域偏移量，用于处理本地日期时间。

上面的类可以由下面的类组合来：

- [Year](http://download.java.net/jdk8/docs/api/java/time/Year.html)：表示年份。
- [Month](http://download.java.net/jdk8/docs/api/java/time/Month.html)：表示月份。
- [YearMonth](http://download.java.net/jdk8/docs/api/java/time/YearMonth.html)：表示年月。
- [MonthDay](http://download.java.net/jdk8/docs/api/java/time/MonthDay.html)：表示月日。
- [DayOfWeek](http://download.java.net/jdk8/docs/api/java/time/DayOfWeek.html)：存储星期的一天。

用now创建实例

```
import java.time.*;
public class One {
    public static void main(String[] args) {
        System.out.println("Instant.now()"+ Instant.now());
        System.out.println("LocalDate.now()"+ LocalDate.now());
        System.out.println("LocalTime.now()"+ LocalTime.now());
        System.out.println("LocalDateTime.now()"+ LocalDateTime.now());
        System.out.println("ZonedDateTime.now()"+ ZonedDateTime.now());
    }
}

结果是：
Instant.now()2022-10-03T06:36:24.599Z
LocalDate.now()2022-10-03
LocalTime.now()14:36:24.662
LocalDateTime.now()2022-10-03T14:36:24.662
ZonedDateTime.now()2022-10-03T14:36:24.662+08:00[Asia/Shanghai]
```

在英国伦敦（Europe/London）它的偏移量是Z，代表+00:00偏移量，属于0时区、0偏移量地区，格林威治是世界的“时间中心”。格林尼治标准时间（旧译[格林威治](https://baike.baidu.com/item/格林威治?fromModule=lemma_inlink)平均时间或[格林威治标准时间](https://baike.baidu.com/item/格林威治标准时间/4895434?fromModule=lemma_inlink)；英语：GreenwichMeanTime，[GMT](https://baike.baidu.com/item/GMT?fromModule=lemma_inlink)）是指位于[英国伦敦](https://baike.baidu.com/item/英国伦敦/1291205?fromModule=lemma_inlink)郊区的皇家格林尼治天文台的标准时间，因为[本初子午线](https://baike.baidu.com/item/本初子午线?fromModule=lemma_inlink)被定义在通过那里的[经线](https://baike.baidu.com/item/经线/1695723?fromModule=lemma_inlink)。理论上来说，格林尼治标准时间的正午是指当太阳横穿[格林尼治子午线](https://baike.baidu.com/item/格林尼治子午线/5906804?fromModule=lemma_inlink)时（也就是在格林尼治时）的时间。由于地球在它的[椭圆轨道](https://baike.baidu.com/item/椭圆轨道/9150757?fromModule=lemma_inlink)里的[运动速度](https://baike.baidu.com/item/运动速度/3655127?fromModule=lemma_inlink)不均匀，这个时刻可能和实际的[太阳时](https://baike.baidu.com/item/太阳时/3389449?fromModule=lemma_inlink)相差16分钟。地球每天的自转是有些不规则的，而且正在缓慢减速。所以，格林尼治时间已经不再被作为标准时间使用。用标准时间──[协调世界时](https://baike.baidu.com/item/协调世界时?fromModule=lemma_inlink)（[UTC](https://baike.baidu.com/item/UTC?fromModule=lemma_inlink)）──由原子钟来代替。

下面列出世界主要城市时区ID对应的UTC偏移量：

| **时区ID**        | **UTC偏移** |
| ----------------- | ----------- |
| Asia/Shanghai     | +08:00      |
| Asia/Chongqing    | +08:00      |
| America/New_York  | -05:00      |
| **Europe/London** | Z           |
| Europe/Paris      | +01:00      |
| Europe/Moscow     | +03:00      |
| Asia/Tokyo        | +09:00      |
| Asia/Dubai        | +04:00      |
| Asia/Seoul        | +09:00      |
| Asia/Bangkok      | +07:00      |
| Asia/Jakarta      | +07:00      |





上面的例子所有输出值均使用ISO 8601标准格式。日期的基本格式为yyyy-MM-dd，而时间的基本格式为hh:mm:ss.sss。LocalDateTime类将两种格式合并，中间用大写字母T隔开。ZonedDateTime类用于显示包含时区信息的日期和时间，在后面添加了一个UTC偏移量以及地区名。Instant类定义的toString方法将输出以祖鲁时间（Zulu time）显示的当前时间。

静态方法of用于生成新的值。对于LocalDate类而言，of方法的参数为年、月（枚举或整形）、日。of方法可以根据给定的参数生成对应的日期/时间对象，基本上每个基本类都有of方法用于生成的对应的对象。

给你提供了静态的构造方法，不能new,它们都是private

![image-20221009154911429](C:\Users\86182\AppData\Roaming\Typora\typora-user-images\image-20221009154911429.png)

```
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.LocalTime;
import java.time.Month;

public class Armstrong {
    public static void main(String[] args) {
        System.out.println("First landing on the Moon:");
        LocalDate moonLandingDate=LocalDate.of(1969, Month.JULY,20);
        LocalTime moonLandingTime= LocalTime.of(20,18);
        System.out.println("Date:"+moonLandingDate);
        System.out.println("Time:"+moonLandingTime);
        System.out.println("Neil Armstrong steps onto the surface:");
        LocalTime walkTime=LocalTime.of(20,2,56,150_000_000);
        LocalDateTime walk=LocalDateTime.of(moonLandingDate,walkTime);
        System.out.println(walk);
    }
}

结果是：
First landing on the Moon:
Date:1969-07-20
Time:20:18
Neil Armstrong steps onto the surface:
1969-07-20T20:02:56.150
```





### 方法概览

Date-Time包中的API提供了大量相关的方法，这些方法一般有一致的方法前缀：

静态工厂：

- of：静态工厂方法。创建实例
- parse：静态工厂方法，解析输入字符串。

实例：

- format: 实例。生成格式化输出。
- get：返回对象状态的一部分。
- is：查询对象状态。
- with：通过修改现有对象的某个元素来创建新对象。
- plus：加一些量到某个对象来创建新对象。
- minus：从某个对象减去一些量来创建新对象。
- at：将对象与另一个对象组合起来，例如：`date.atTime(time)`。



**例子：用at方法为本地日期和时间添加时区。**

```
import java.time.LocalDateTime;
import java.time.Month;
import java.time.ZoneId;
import java.time.ZonedDateTime;

public class eightfour {
    public static void main(String[] args) {
        LocalDateTime dateTime= LocalDateTime.of(2017, Month.JULY,4,13,20,10);
        ZonedDateTime nyc=dateTime.atZone(ZoneId.of("America/New_York"));
        System.out.println(nyc);
        ZonedDateTime london=nyc.withZoneSameInstant(ZoneId.of("Europe/London"));//**withZoneSameInstant方法传入一个ZonedId,并查找另一个时区的日期和时间。**
        System.out.println(london);
    }
}

结果为：
2017-07-04T13:20:10-04:00[America/New_York]
2017-07-04T18:20:10+01:00[Europe/London]
```

**withZoneSameInstant方法传入一个ZonedId,并查找另一个时区的日期和时间。**

toEpochSecond() 方法用于将此 LocalTime 转换为自 1970-01-01T00:00:00Z 纪元以来的秒数。该方法将此本地时间与作为参数传递的指定日期和偏移量相结合，以计算纪元秒值，即从 1970-01-01T00:00:00Z 开始经过的秒数。纪元之后时间轴上的瞬间为正，更早为负。另外，java-time包中引入了Month和DayOfWeek两种枚举。Month包括标准日历中12个月份的常量（从JANUARY到DECEMBER）

DayOfWeek枚举包括7个工作日的常量（从Monday到Sunday）

```java
import java.time.DayOfWeek;
import java.time.Month;
import java.time.MonthDay;

public class monthenum {
    public static void main(String[] args) {
        System.out.println("Days in Feb in a leap year:"+ Month.FEBRUARY.length(true));
        System.out.println("Days of year for first day of Aug(leap year):"+Month.AUGUST.firstDayOfYear(true));
        System.out.println("Month.of(1):"+Month.of(1));
        System.out.println("dayofweek"+ DayOfWeek.of(7));
        System.out.println("Adding two months:"+Month.JANUARY.plus(2));
        System.out.println("Subtracting a month:"+Month.MARCH.minus(1));
    }
}
结果为：
Days in Feb in a leap year:29
Days of year for first day of Aug(leap year):214
Month.of(1):JANUARY
dayofweekSUNDAY
Adding two months:MARCH
Subtracting a month:FEBRUARY

Process finished with exit code 0

```

**java.time包中的类是不可变的，如果实例方法（如plus、minus或with）试图修改某个类，将生成一个新的实例。**



## 根据现有实例创建日期和时间

用户希望修改date-time API中的某个类现有实例

加减用plus或minus，其余用with

以LocalDate为例，它定义了多种对日期进行增减操作的方法，包括：

LocalDate plusDay(long days) 增加天数
LocalDate plusWeeks(long weeks) 增加周数
LocalDate plusMonths(long months) 增加月数
LocalDate plusYears(long years) 增加年数

以上方法均返回一个新的localDate，它是当前日期的副本，并添加了指定的值

我们要进行格式化显示，就要使用`DateTimeFormatter`。`DateTimeFormatter`可以只创建一个实例，到处引用。

创建`DateTimeFormatter`时，我们仍然通过传入格式化字符串实现：

```
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss:sss");
```

另一种创建`DateTimeFormatter`的方法是，传入格式化字符串时，同时指定`Locale`：

```
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("E, yyyy-MMMM-dd HH:mm", Locale.US);
```

这种方式可以按照`Locale`默认习惯格式化。我们来看实际效果：

```
import java.time.*;
import java.time.format.*;
import java.util.Locale;
public class Main {
    public static void main(String[] args) {
        ZonedDateTime zdt = ZonedDateTime.now();
        var formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd'T'HH:mm ZZZZ");
        System.out.println(formatter.format(zdt));

        var zhFormatter = DateTimeFormatter.ofPattern("yyyy MMM dd EE HH:mm", Locale.CHINA);
        System.out.println(zhFormatter.format(zdt));

        var usFormatter = DateTimeFormatter.ofPattern("E, MMMM/dd/yyyy HH:mm", Locale.US);
        System.out.println(usFormatter.format(zdt));
    }
}

结果是：
2022-10-08T15:34 GMT+08:00
2022 十月 08 星期六 15:34
Sat, October/08/2022 15:34
```

在格式化字符串中，如果需要输出固定字符，可以用`'zzz'`表示。

当我们直接调用`System.out.println()`对一个`ZonedDateTime`或者`LocalDateTime`实例进行打印的时候，实际上，调用的是它们的`toString()`方法，默认的`toString()`方法显示的字符串就是按照`ISO 8601`格式显示的，我们可以通过`DateTimeFormatter`预定义的几个静态变量来引用：

```
var ldt = LocalDateTime.now();
System.out.println(DateTimeFormatter.ISO_DATE.format(ldt));
System.out.println(DateTimeFormatter.ISO_DATE_TIME.format(ldt));
```

得到的输出和`toString()`类似：

```
2019-09-15
2019-09-15T23:16:51.56217
```

### 小结

对`ZonedDateTime`或`LocalDateTime`进行格式化，需要使用`DateTimeFormatter`类；

`DateTimeFormatter`可以通过格式化字符串和`Locale`对日期和时间进行定制输出。

```
import org.junit.Test;
import java.time.LocalDate;
import java.time.LocalTime;
import java.time.Month;
import java.time.format.DateTimeFormatter;
import static org.junit.Assert.assertEquals;
public class eightsix {
    @Test
    public void localDatePlus() throws Exception{
        DateTimeFormatter formatter=DateTimeFormatter.ofPattern("yyyy-MM-dd");
        LocalDate start=LocalDate.of(2017, Month.FEBRUARY,2);
        LocalDate end=start.plusDays(3);
        assertEquals("2017-02-05",end.format(formatter));//assertequals断言，去判断写入的值与它是否相等，相等则不报错。
        System.out.println(end.format(formatter));
        end=start.plusWeeks(5);
        assertEquals("2017-03-09",end.format(formatter));
        System.out.println(end.format(formatter));
        end=start.plusMonths(7);
        assertEquals("2017-09-02",end.format(formatter));
        System.out.println(end.format(formatter));
        end=start.plusYears(2);
        assertEquals("2019-02-02",end.format(formatter));
        System.out.println(end.format(formatter));
    }
    @Test
    public void localTimePlus() throws Exception{
        DateTimeFormatter formatter=DateTimeFormatter.ISO_LOCAL_TIME;
        LocalTime start=LocalTime.of(11,30,0,0);
        LocalTime end=start.plusNanos(1000000);
        assertEquals("11:30:00.001",end.format(formatter));
        System.out.println(end.format(formatter));
        end=start.plusSeconds(20);
        assertEquals("11:30:20",end.format(formatter));
        System.out.println(end.format(formatter));
        end=start.plusMinutes(45);
        assertEquals("12:15:00",end.format(formatter));
        System.out.println(end.format(formatter));
        end=start.plusHours(5);
        assertEquals("16:30:00",end.format(formatter));
        System.out.println(end.format(formatter));
    }
}

结果是：
2017-02-05
2017-03-09
2017-09-02
2019-02-02
11:30:00.001
11:30:20
12:15:00
16:30:00
```

assertequals断言，去判断写入的值与它是否相等，相等则不报错。

**with将时间修改成指定值。**

每种类都定义了一系列with方法，可以一次修改一个字段。

with方法用于处理日期和时间并提供了很多种修改时间的方式
LocalDateTime withNano(int i) 修改纳秒
LocalDateTime withSecond(int i) 修改秒
LocalDateTime withMinute(int i) 修改分钟
LocalDateTime withHour(int i) 修改小时
LocalDateTime withDayOfMonth(int i) 修改日
LocalDateTime withMonth(int i) 修改月
LocalDateTime withYear(int i) 修改年

```java
import org.junit.Test;
import java.time.DateTimeException;
import java.time.LocalDateTime;
import java.time.Month;
import java.time.format.DateTimeFormatter;
import static org.junit.Assert.assertEquals;
public class eighteight {
    @Test
    public void with() throws Exception{
        LocalDateTime start=LocalDateTime.of(2017, Month.FEBRUARY,2,11,30);
        LocalDateTime end=start.withMinute(45);
        assertEquals("2017-02-02T11:45:00",end.format(DateTimeFormatter.ISO_DATE_TIME));
        System.out.println(end.format(DateTimeFormatter.ISO_DATE_TIME));

        end=start.withHour(16);
        assertEquals("2017-02-02T16:30:00",end.format(DateTimeFormatter.ISO_DATE_TIME));
        System.out.println(end.format(DateTimeFormatter.ISO_DATE_TIME));

        end=start.withDayOfMonth(28);
        assertEquals("2017-02-28T11:30:00",end.format(DateTimeFormatter.ISO_DATE_TIME));
        System.out.println(end.format(DateTimeFormatter.ISO_DATE_TIME));

        end=start.withDayOfYear(300);
        assertEquals("2017-10-27T11:30:00",end.format(DateTimeFormatter.ISO_DATE_TIME));
        System.out.println(end.format(DateTimeFormatter.ISO_DATE_TIME));

        end=start.withYear(2020);
        assertEquals("2020-02-02T11:30:00",end.format(DateTimeFormatter.ISO_DATE_TIME));
        System.out.println(end.format(DateTimeFormatter.ISO_DATE_TIME));

    }
    @Test()//在括号里写expected=DateTimeException.class,很意外
    public void withInvalidDate() throws Exception{
        LocalDateTime start=LocalDateTime.of(2017,Month.FEBRUARY,2,11,30);
        start.withDayOfMonth(29);
    }
}

结果是：
2017-02-02T11:45:00
2017-02-02T16:30:00
2017-02-28T11:30:00
2017-10-27T11:30:00
2020-02-02T11:30:00
java.time.DateTimeException: Invalid date 'February 29' as '2017' is not a leap year

	at java.time.LocalDate.create(LocalDate.java:429)
	at java.time.LocalDate.of(LocalDate.java:269)
	at java.time.LocalDate.withDayOfMonth(LocalDate.java:1099)
	at java.time.LocalDateTime.withDayOfMonth(LocalDateTime.java:1023)
	at eighteight.withInvalidDate(eighteight.java:38)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.junit.runners.model.FrameworkMethod$1.runReflectiveCall(FrameworkMethod.java:50)
	at org.junit.internal.runners.model.ReflectiveCallable.run(ReflectiveCallable.java:12)
	at org.junit.runners.model.FrameworkMethod.invokeExplosively(FrameworkMethod.java:47)
	at org.junit.internal.runners.statements.InvokeMethod.evaluate(InvokeMethod.java:17)
	at org.junit.runners.ParentRunner.runLeaf(ParentRunner.java:325)
	at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:78)
	at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:57)
	at org.junit.runners.ParentRunner$3.run(ParentRunner.java:290)
	at org.junit.runners.ParentRunner$1.schedule(ParentRunner.java:71)
	at org.junit.runners.ParentRunner.runChildren(ParentRunner.java:288)
	at org.junit.runners.ParentRunner.access$000(ParentRunner.java:58)
	at org.junit.runners.ParentRunner$2.evaluate(ParentRunner.java:268)
	at org.junit.runners.ParentRunner.run(ParentRunner.java:363)
	at org.junit.runner.JUnitCore.run(JUnitCore.java:137)
	at com.intellij.junit4.JUnit4IdeaTestRunner.startRunnerWithArgs(JUnit4IdeaTestRunner.java:68)
	at com.intellij.rt.junit.IdeaTestRunner$Repeater.startRunnerWithArgs(IdeaTestRunner.java:33)
	at com.intellij.rt.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:230)
	at com.intellij.rt.junit.JUnitStarter.main(JUnitStarter.java:58)
```



## **调节器与查询**

给定时态值，用户希望自定义调整，或检索给定值的有关信息。

解决方式：创建TemporalAdjuster或规划TemporalQuery接口。

1.TemporalAdjuster接口定义了一个名为adjustInto的方法，它传入的Temporal值作为参数，并返回调整后的值。而TemporalAdjusters类包括一系列用作静态方法的调节器（adjuster）

static TemporalAdjuster firstDayOfNextMonth()

static TemporalAdjuster firstDayOfNextYear()

static TemporalAdjuster firstDayOfYear()

static TemporalAdjuster firstInMonth(DayOfWeek dayOfWeek)等

TemporalAdjusters类定义的部分静态方法

```java
import org.junit.Test;
import java.time.DayOfWeek;
import java.time.LocalDateTime;
import java.time.Month;
import java.time.temporal.TemporalAdjuster;
import java.time.temporal.TemporalAdjusters;
import static org.junit.Assert.assertEquals;
public class eightten {
    @Test
    public void adjusters() throws Exception{
        LocalDateTime start=LocalDateTime.of(2017, Month.FEBRUARY,2,11,30);
        LocalDateTime end=start.with(TemporalAdjusters.firstDayOfNextMonth());
        assertEquals("2017-03-01T11:30",end.toString());
        System.out.println(end);
        end=start.with(TemporalAdjusters.next(DayOfWeek.THURSDAY));//TemporalAdjusters类的next (DayOfWeek)方法用于返回下一个day-of-week TemporalAdjuster对象，该对象可用于获取新的Date对象，该对象是具有相同匹配DayOfWeek的下一个日期，该日期作为参数从任何Date对象上传递将应用此TempotralAdjuster。 
        assertEquals("2017-02-09T11:30",end.toString());
        System.out.println(end);
        end=start.with(TemporalAdjusters.previousOrSame(DayOfWeek.THURSDAY));//TemporalAdjusters类的previousOrSame (DayOfWeek)方法用于返回先前或相同的day-of-week TemporalAdjuster对象，该对象可用于获取新的Date对象，该对象是先前或相同日期的具有相同匹配的day-of-week作为参数从任何对象传递应用此TempotralAdjuster的Date对象。
        assertEquals("2017-02-02T11:30",end.toString());
        System.out.println(end);
    }
}

结果为：
2017-03-01T11:30
2017-02-09T11:30
2017-02-02T11:30
```



应用：假设员工在一个月中领取两次工资，且发薪日是每月15日和最后一天；如果某个发薪日为周末，则提前到周五。

```java
import org.junit.Test;
import java.time.DayOfWeek;
import java.time.LocalDate;
import java.time.Month;
import java.time.temporal.Temporal;
import java.time.temporal.TemporalAdjuster;
import java.time.temporal.TemporalAdjusters;
import java.util.stream.IntStream;
import static org.junit.Assert.assertEquals;

public class PayDayAdjuster implements TemporalAdjuster {
public Temporal adjustInto(Temporal input){//adjustInto方法已被添加到实现TemporalAdjuster接口的PaydayAdjuster类中
    LocalDate date= LocalDate.from(input);//from方法可以将任何时态对象转换为LocalDate
    int day;
    if(date.getDayOfMonth()<15){
        day=15;
    }else{
        day=date.with(TemporalAdjusters.lastDayOfMonth())
                .getDayOfMonth();
    }
    date=date.withDayOfMonth(day);
    if(date.getDayOfWeek()== DayOfWeek.SATURDAY||
    date.getDayOfWeek()==DayOfWeek.SUNDAY){
        date=date.with(TemporalAdjusters.previous(DayOfWeek.FRIDAY));
    }
    return input.with(date);
}

@Test
public void payDay() throws Exception{
    TemporalAdjuster adjuster=new PayDayAdjuster();
    IntStream.rangeClosed(1,14)
            .mapToObj(day->LocalDate.of(2017, Month.JULY,day))
            .forEach(date->
                    assertEquals(14,date.with(adjuster).getDayOfMonth()));
            IntStream.rangeClosed(15,31)
                    .mapToObj(day->LocalDate.of(2017,Month.JULY,day))
                    .forEach(date->
                            assertEquals(31,date.with(adjuster).getDayOfMonth()));

}
}
```

时态类对象（LocalDate类，LocalTime类）都有一个方法叫做query()，可以针对日期进行查询，R query(TemporalQuery query)这个方法是一个泛型方法，返回的数据就是传入的泛型类的类型，TemporalQuery是一个泛型接口，里面有一个抽象方法是R queryFrom(TemporalAccessor temporal)，TemporalAccessor是Temporal的父接口，实际上也就是LocalDate,LocalDateTime相关类的顶级父接口，这个**queryFrom()**的实现逻辑就是，传入一个日期/时间对象通过自定义逻辑返回数据。
如果要计划日期距离某一天特定天数差距多少天，可以自定义类实现TemporalQuery接口并且作为参数传到query方法中。

**TemporalQuery的应用**

TemporalQuery接口用作时态对象中query方法的参数。query方法调用TemporalQuery.queryFrom(TemporalAccessor)方法（传入this作为参数），并返回所需要的查询。TemporalAccessor接口定义的所有方法均可用于查询操作。Date-Time API还包括一个名为TemporalQueries的类，它定义了许多常见查询的常量：

static TemporalQuery<LocalDate> localDate()

static TemporalQuery<LocalTime> localTime()

static TemporalQuery<ZoneId> zone()

static TemporalQuery<ZoneId> zoneId()

```java
//计算当前时间距离下一个劳动节还有多少天？
import org.junit.Test;
import java.time.LocalDate;
import java.time.Month;
import java.time.temporal.ChronoUnit;
import java.time.temporal.TemporalAccessor;
import java.time.temporal.TemporalQuery;
public class UtilDayQueryImpl implements TemporalQuery<Long> {
    @Override
    public Long queryFrom(TemporalAccessor temporalAccessor) {
        // 1. TemporalAccessor是LocalDateTime的顶级父接口，相当LocalDate就是这个接口，将Temporal转换为LocalDate 进行使用
        LocalDate now = LocalDate.from(temporalAccessor);//from()方法用于根据给定的TemporalAccessor对象创建LocalDate的实例
        // 2.封装当年劳动节时间
        LocalDate laborDay = LocalDate.of(now.getYear(), Month.MAY, 1);
        // 3.判断当前时间是否已经超过了当年的劳动节
        if (now.isAfter(laborDay)) {
            laborDay = laborDay.plusYears(1);
        }
        // 4.通过ChronoUnit的between来计算两个时间差
        long days = ChronoUnit.DAYS.between(now, laborDay);//下例有
        return days;
    }
//测试
@Test
    public static void main(String[] args) {
        //获取今天日期，计算距离下一个劳动节还有多少天
        LocalDate now = LocalDate.now();
        //调用now的query方法，将我们自己实现的UtilDayQueryImpl作为参数传入
        Long day = now.query(new UtilDayQueryImpl());
         //2022-10-09
        System.out.println(now);
        //204
        System.out.println(day);
    }
}
```



## 解析与格式化

- [DateTimeFormatter](http://download.java.net/jdk8/docs/api/java/time/format/DateTimeFormatter.html)类用于创建日期/时间格式，可以在解析和格式化中使用。
- [ChronoUnit](http://download.java.net/jdk8/docs/api/java/time/temporal/ChronoUnit.html)：计算出两个时间点之间的时间距离，可按多种时间单位计算。
- [TemporalAdjuster](http://download.java.net/jdk8/docs/api/java/time/temporal/TemporalAdjuster.html)：各种日期计算功能。

DateTimeFormatter类提供大量预定义格式化器，包括常量（ISO_LOCAL_DATE)、模式字母（如uuuu-MMM-dd）以及本地化样式（ofLocalizedDate（dateStyle）） 在Date-time API中，所有的类均提供parse和format方法。



```
//对日期进行格式化
import java.sql.SQLOutput;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.Month;
import java.time.format.DateTimeFormatter;
import java.time.format.FormatStyle;
import java.util.Locale;

public class eightthirty {
    public static void main(String[] args) {
        LocalDateTime now = LocalDateTime.now();
        String text = now.format(DateTimeFormatter.ISO_DATE_TIME);//将LocalDateTime格式化为字符串
        LocalDateTime dateTime = LocalDateTime.parse(text);//将字符串解析为LocalDateTime
        LocalDate date = LocalDate.of(2017, Month.MARCH, 13);
        System.out.println("Full:" + date.format(DateTimeFormatter.ofLocalizedDate(FormatStyle.FULL)));
        System.out.println("long:" + date.format(DateTimeFormatter.ofLocalizedDate(FormatStyle.LONG)));
        System.out.println("Medium:" + date.format(DateTimeFormatter.ofLocalizedDate(FormatStyle.MEDIUM)));
        System.out.println("Short:" + date.format(DateTimeFormatter.ofLocalizedDate(FormatStyle.SHORT)));
        System.out.println("France:" + date.format(DateTimeFormatter.ofLocalizedDate(FormatStyle.FULL).withLocale(Locale.FRANCE)));
        Locale loc = new Locale.Builder().setLanguage("sr").setScript("Latn").setRegion("RS").build();
//setScript设置脚本        System.out.println("Serbian"+date.format(DateTimeFormatter.ofLocalizedDate(FormatStyle.FULL).withLocale(loc)));
    }
}

结果是：
Full:2017年3月13日 星期一
long:2017年3月13日
Medium:2017-3-13
Short:17-3-13
France:lundi 13 mars 2017
Serbianponedeljak, 13. mart 2017.
```

**查找具有非整数小时偏移量的时区**

大部分时区的UTC偏移量均为小时的整数，当然有些地区也存在UTC偏移量为半小时甚至四十五分钟的时区，该如何解决？

方案：获取每个时区的地区偏移量，并计算总秒数除以3600之后的剩余时间。通过ZoneOffset类查找每个时区ID相对于UTC的偏移量，并将其与3600秒（1小时）进行比较。

```java
import java.time.Instant;
import java.time.ZoneId;
import java.time.ZoneOffset;
import java.time.ZonedDateTime;
import java.time.format.DateTimeFormatter;
import java.time.format.FormatStyle;
import static java.util.Comparator.comparingInt;

public class FunnyOffsets {
    public static void main(String[] args) {
        Instant instant= Instant.now();
        ZonedDateTime current=instant.atZone(ZoneId.systemDefault());
        System.out.printf("Current time is %s%n%n",current);
        System.out.printf("%10s %20s %13s%n","Offset","ZoneId","Time");
        ZoneId.getAvailableZoneIds().stream()
                .map(ZoneId::of)//将地区ID映射到时区ID
                .filter(zoneId -> {
                    ZoneOffset offset=instant.atZone(zoneId).getOffset();//计算偏移量
                    return offset.getTotalSeconds()%(60*60)!=0;//仅返回偏移量无法被3600整除的时区
                })
                .sorted(comparingInt(zoneId -> instant.atZone(zoneId).getOffset().getTotalSeconds()))//按照offset偏移量排序
                .forEach(zoneId -> {
                    ZonedDateTime zdt=current.withZoneSameInstant(zoneId);
                    System.out.printf("%10s %25s %10s%n",
                            zdt.getOffset(),zoneId,
                            zdt.format(DateTimeFormatter.ofLocalizedTime(FormatStyle.SHORT)));//ofLocalizedTime和本地时间比较
                });
    }
}



结果为：
Current time is 2022-10-04T19:32:12.447+08:00[Asia/Shanghai]

    Offset               ZoneId          Time
    -09:30         Pacific/Marquesas     上午2:02
    -02:30          America/St_Johns     上午9:02
    -02:30       Canada/Newfoundland     上午9:02
    +03:30                      Iran     下午3:02
    +03:30               Asia/Tehran     下午3:02
    +04:30                Asia/Kabul     下午4:02
    +05:30              Asia/Kolkata     下午5:02
    +05:30              Asia/Colombo     下午5:02
    +05:30             Asia/Calcutta     下午5:02
    +05:45            Asia/Kathmandu     下午5:17
    +05:45             Asia/Katmandu     下午5:17
    +06:30               Asia/Yangon     下午6:02
    +06:30              Asia/Rangoon     下午6:02
    +06:30              Indian/Cocos     下午6:02
    +08:30            Asia/Pyongyang     下午8:02
    +08:45           Australia/Eucla     下午8:17
    +09:30           Australia/North     下午9:02
    +09:30          Australia/Darwin     下午9:02
    +10:30      Australia/Yancowinna    下午10:02
    +10:30        Australia/Adelaide    下午10:02
    +10:30     Australia/Broken_Hill    下午10:02
    +10:30           Australia/South    下午10:02
    +13:45                   NZ-CHAT     上午1:17
    +13:45           Pacific/Chatham     上午1:17

```





## **获取事件之间的时间**

将时间转换成人们可读的格式，使用时态类（temporal class）定义的between或until方法，或period类定义的between方法以生成period对象，若不需要转换时间格式，则使用以秒和纳秒为单位对时间量进行建模的duration类。在java.time.temporal包中，TempoalUnit接口由定义在同一个包中的ChronoUnit枚举实现。TemporalUnit接口定义了一个名为between的方法，它传入两个TemporalUnit实例并返回二者之间的时间量（long型数据）。

**计算当天到美国选举日之间的天数**

```java
import java.time.LocalDate;
import java.time.Month;
import java.time.temporal.ChronoUnit;
public class USchairman {
    public static void main(String[] args) {
        LocalDate electionary=LocalDate.of(2022, Month.NOVEMBER,3);
        LocalDate today=LocalDate.now();
        System.out.printf("%d day(s) to go ...%n", ChronoUnit.DAYS.between(today,electionary));
    }
}

结果是：
31 day(s) to go...
```

**用period类计算时间差**

主要是Period类方法getYears（），getMonths（）和getDays（）来计算.

```java
import java.time.LocalDate;
import java.time.Month;
import java.time.Period;
public class TestPeriod {
    public static void main(String[] args) {
        LocalDate today = LocalDate.now();
        System.out.println("Today : " + today);
        LocalDate birthDate = LocalDate.of(1993, Month.OCTOBER, 19);
        System.out.println("BirthDate : " + birthDate);
        Period p = Period.between(birthDate, today);
        System.out.printf("年龄 : %d 年 %d 月 %d 日", p.getYears(), p.getMonths(), p.getDays());
    }
}
结果是：
Today : 2022-10-08
BirthDate : 1993-10-19
年龄 : 28 年 11 月 19 日
```


