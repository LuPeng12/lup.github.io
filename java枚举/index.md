# Java枚举


枚举类型（enum type）是指由一组常量组成合法值的类型...

<!--more-->

## Java枚举



- [x] 枚举类型（enum type）是指由一组常量组成合法值的类型；
- [x] java枚举与其它语言的枚举不同，是Java语言的一个特性和优势；



**何时使用枚举**：

每当需要一组**固定常量**，并且在**编译时**就知道其成员的时候，就应该使用枚举！





### 枚举模式（enum pattern）



```java
//The int enum pattern
public static final int APPLE = 0;
public static final int BANANA = 1;
public static final int PEAR = 2;
public static final int PEACH = 3;    
public static final int ORANGE = 4;
```

其值可以在任一需要的地方调用！

也可以通过反射获取其变量名称

```java
System.out.println(EnumPatternDemo.APPLE);
System.out.println(EnumPatternDemo.BANANA);

for (Field field : EnumPatternDemo.class.getDeclaredFields())
    System.out.println(field.getName() + field);
```

也通过反射可以获取其全部值：

```java

for (Field field : EnumPatternDemo.class.getDeclaredFields()) {
    try {
        System.out.println(field.getName() + "-"+field.getInt(field));
    } catch (IllegalAccessException e) {
        e.printStackTrace();
    }
}
```



- 这种模式没有描述性；（譬如apple中的特性完全没有，想给apple赋予某种属性无法办到，想打印一个字符串也不行！）
- 不具有类型安全性；(譬如你将apple传到需要orange的方法中，是不会报错的，不好纠错！)
- 如果与apple相关联的值发生变化，必须重新编译；
- 把以上的int改成String也不会解决这些问题；



于是Java对枚举做了很大的扩展！

成为Java的特色和优势之一！



### 枚举类型（enum type）

```java
//enum type
public enum Fruits{APPLE,BANANA,PEAR,PEACH,ORANGE}
```



- [x] **Java的枚举类型是功能十分齐全的类**！
- [x] **其功能比其它语言的对应的类强大得多！**
- [x] **Java枚举类型允许添加任意的方法和属性！**
- [x] **Java枚举类型允许实现任意接口！**
- [x] **提供了Object所有方法的实现**
- [x] **实现了Comparable和serializable接口**



Java枚举类型基本思想：



- **通过共有的静态final属性为每个枚举常量导出一个实例！**
- **枚举类型没有可访问的构造器！是真正的final类！**
- **客户端不能创建枚举实例，也不能对它进行扩展，所以不存在任何客户端实例（--不能多态、继承）！**
- **只存在声明了的常量实例--枚举常量！**



- [ ] **枚举类型保证了编译时的安全！！譬如fruits, 它保证了在引用fruit的地方，只能传递以上五个常量实例之一，其它任何值都非法！**---编译时会报错！



可以通过向枚举类型添加方法或属性，将数据与常量关联起来：



```Java
//enum type
public enum Fruits{
    APPLE,
    BANANA,
    PEAR,
    PEACH,
    ORANGE
        
    private final String color;
    private final String smell;
    private final String taste;
    
    //constructor
    Fruit(String color,String smell,String taste){
        this.color = color;
        this.smell = smell;
        this.taste = taste;
    }
}
```

带有构造参数的枚举：

```java
import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

/**
 * @description: Todo
 * @author: ZZS
 * @date: 2022/9/14 17:12
 * @version: 1.0.0
 */

public enum FruitsOne {
    APPLE("Red","Fragrance","Sweet"),
    BANANA("Yellow","None","Sweet"),
    PEAR("Green","None","Sweet"),
    PEACH("Pink","Fragrance","Sweet"),
    ORANGE("Orange","Fragrance","Sweet");

    private final String color;
    private final String smell;
    private final String taste;

    //constructor
    FruitsOne(String color,String smell,String taste){
        this.color = color;
        this.smell = smell;
        this.taste = taste;
    }

    public double getPrice(){
        switch (this){
            case APPLE:
                return 12.45d;
            case BANANA:
                return 23.45d;
            case PEAR:
                return 16.56d;
            case PEACH:
                return 16.67d;
            case ORANGE:
                return 19.45d;
            default:
                return 0.0d;
        }
    }

   
}

```



枚举时无法访问到构造方法和通过反射调用方法的：

```java
 public static void main(String[] args) {
        System.out.println(FruitsOne.APPLE.getPrice());
        System.out.println(FruitsOne.BANANA.color);

        Class<FruitsOne> fruitsOneClass = FruitsOne.class;
        Class<? extends FruitsOne> aClass = FruitsOne.ORANGE.getClass();

         Class<FruitEnum1> fruitEnum1Class = FruitEnum1.class;

        Constructor<?>[] declaredConstructors = fruitEnum1Class.getDeclaredConstructors();

        for (Constructor<?> constructor : declaredConstructors) {
            constructor.setAccessible(true);
            try {
                constructor.newInstance("33","","");
            } catch (InstantiationException e) {
                e.printStackTrace();
            } catch (IllegalAccessException e) {
                e.printStackTrace();
            } catch (InvocationTargetException e) {
                e.printStackTrace();
            }
        }
     /////不允许通过构造方法创建美剧对象！！

        Method[] declaredMethods = aClass.getDeclaredMethods();

        for (Method declaredMethod : declaredMethods) {
            declaredMethod.setAccessible(true);
            if (declaredMethod.getName().equals("getPrice"))
            try {
                declaredMethod.invoke(ORANGE);
            } catch (IllegalAccessException e) {
                e.printStackTrace();
            } catch (InvocationTargetException e) {
                e.printStackTrace();
            }
        }

    }
```





枚举的values（）方法：



```java
   for (FruitsOne one:FruitsOne.values()) {
            System.out.println(one.smell);
   }
```





**枚举的方法添加**：

```java
/**
 * @description: Todo
 * @author: ZZS
 * @date: 2022/9/15 9:33
 * @version: 1.0.0
 */

public enum FruitTwo {
    APPLE{
        public void eatMethod(){
            System.out.println("Use mouth;");
        }
        public String Place(){
            return "I am in Sichuan";
        }
    },
    BANANA{
        public void eatMethod(){
        System.out.println("Use ear");
        }
        public String Place(){
            return "I am in Sichuan";
        }},
    PEAR{@Override public void eatMethod(){
        System.out.println("Use Hands");
    }

        public String Place(){
            return "I am in Sichuan";
        }},
    PEACH{@Override public void eatMethod(){
        System.out.println("Use feet");
        }
        public String Place(){
            return "I am in Sichuan";
        }
    },
    ORANGE{@Override public void eatMethod(){
        System.out.println("Use Other");
        }
        public String Place(){
            return "I am in Sichuan";
        }
    };



    public double amount(double weight){
        switch (this) {
            case APPLE:
                return weight*1.34;
            case BANANA:
                return weight*2.34;
            case PEAR:
            case PEACH:
                return weight*3.34;
            case ORANGE:
                return weight*4.34;
            default:
                return 0;
        }

    }

    public abstract void eatMethod();
    public abstract String Place();

}

```







**枚举的valueOf(String)方**法的含义与用法**：

```java
  public static void main(String[] args) {

        FruitTwo.APPLE.eatMethod();

        for (FruitTwo two : FruitTwo.values()) {
            two.eatMethod();
        }

        FruitTwo.valueOf("APPLE").eatMethod();
    }
//FruitTwo.valueOf("APPLE").eatMethod();  用法演示
```



**枚举不能通过继承实现**：





**实现接口类型的枚举****：



```java
/**
 * @description: Todo
 * @author: ZZS
 * @date: 2022/9/15 11:34
 * @version: 1.0.0
 */

public enum FruitThree  implements Apply,Apply2 {
    APPLE{public void eat(){
        System.out.println("I am eating Apple!");
    }},
    BANANA{public void eat(){
        System.out.println("I am eating BANANA!");
    }},
    PEAR{public void eat(){
        System.out.println("I am eating PEAR!");
    }},
    PEACH{public void eat(){
        System.out.println("I am eating PEACH!");
    }},
    ORANGE{public void eat(){
        System.out.println("I am eating ORANGE!");
    }};

    @Override
    public void watch(){
        System.out.println("I am watching");
    }


    public static void main(String[] args) {
        FruitThree.APPLE.eat();

        for (FruitThree three : FruitThree.values()) {
            three.eat();
            three.watch();
        }
    }
}

```
















