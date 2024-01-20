# Java文件IO


在 Java 中提供了许多关于文件的操作，如 File 类，该类用来封装一个路径，这个路径可以是绝对路径，也可以是相对路径

<!--more-->

# 文件IO

## 1 File 类

在 Java 中提供了许多关于文件的操作，如 File 类，该类用来封装一个路径，这个路径可以是绝对路径，也可以是相对路径。这个路径可以是一个文件，也可以是一个目录，通过 **File** 类封装完成后，可以对这些文件或目录进行一些常规操作。

### 1.1 构造方法

| 方法                              | 功能                                                         |
| --------------------------------- | ------------------------------------------------------------ |
| File(String pathname)             | 通过指定的一个字符串类型的文件路径来创建一个新的 File 对象   |
| File(String parent, String child) | 根据指定的一个字符串类型的父路径和一个字符串类型的子路径（包括文件名称）创建一个 File 对象 |

### 1.2 常用方法

| 方法                                 | 功能                                                         |
| ------------------------------------ | ------------------------------------------------------------ |
| boolean exists()                     | 判断 File 对象对应的文件或目录是否存在，若存在则返回 true，否则返回 false |
| boolean delete()                     | 删除 File 对象对应的文件或目录，若删除成功则返回 true，否则返回 false |
| boolean createNewFile()              | 当对象对应的文件不存在时，该方法将新建一个此 File 对象所指定的新文件，若创建成功则返回 true， 否则返回 false |
| String getName()                     | 返回 File 对象表示的文件或文件夹的名称                       |
| String getPath()                     | 返回 File 对象对应的路径                                     |
| String getAbsolutePath()             | 返回 File 对象对应的绝对路径                                 |
| String getParent()                   | 返回 File 对象对应目录的父目录                               |
| boolean canRead()                    | 判断 File 对象对应的文件或目录是否可读，若可读则返回true，反之返回false |
| boolean canWrite()                   | 判断 File 对象对应的文件或目录是否可写，若可写则返回true，反之返回false |
| boolean isFile()                     | 判断 File 对象对应的是否是文件（不是目录），若是文件则返回true，反之返回false |
| boolean isDirectory()                | 判断 File 对象对应的是否是目录（不是文件），若是目录则返回true，反之返回false |
| boolean isAbsolute()                 | 判断 File 对象对应的文件或目录是否是绝对路径                 |
| long lastModified()                  | 返回1970年1月1日0时0分0秒到文件最后修改时间的毫秒值          |
| long length()                        | 返回文件内容的长度                                           |
| String[] list()                      | 列出指定目录的全部内容，只是列出名称                         |
| String[] list(FilenameFilter filter) | 接收一个 FilenameFilter 参数，通过该参数可以只列出符合条件的文件 |
| File[] listFiles()                   | 返回一个包含了 File 对象所有子文件和子目录的 File 数组       |

示例：

```java
        File file = new File("E:\\TestIO\\file\\file.txt");
        System.out.println("文件名称:" + file.getName());
        System.out.println("文件的路径:" + file.getPath());
        System.out.println("文件的绝对路径:" + file.getAbsolutePath());
        System.out.println("文件的父路径:" + file.getParent());
        System.out.println(file.canRead() ? "文件可读" : "文件不可读");
        System.out.println(file.canWrite() ? "文件可写" : "文件不可写");
        System.out.println(file.isFile() ?  "是一个文件" : "不是一个文件");
        System.out.println(file.isDirectory() ? "是一个目录" :"不是一个目录");
        System.out.println(file.isAbsolute() ? "是绝对路径" : "不是绝对路径");
        System.out.println("最后修改时间为:" + file.lastModified());
        System.out.println("文件大小为:" + file.length() + " bytes");
//        System.out.println("是否成功删除文件"+file.delete());
```

### 1.3 遍历目录下的文件

使用 File 类中的 list() 方法遍历某个指定目录下的所有文件名称。

```java
    public static void forEachFileInDir() {
        File file = new File("E:\\TestIO\\file");
        if (file.isDirectory()) {
            String[] fileNames = file.list();
            Arrays.stream(fileNames)
                    .forEach(System.out::println);
        }
    }
```

```java
    public static void forEachFileInDir2() {
        File file = new File("E:\\TestIO\\file");
        List<String> fileList = new ArrayList<>();
        process(file, fileList);
        System.out.println(fileList.toString());
    }
    private static void process(File file, List<String> list) {
        File[] listFiles = file.listFiles();
        for (File f : listFiles) {
            list.add(f.getName());
            if (f.isDirectory()) {
                process(f, list);
            }
        }
    }
```

### 1.4 删除文件及目录

在操作文件时，经常需要删除一个目录下的某个文件或者整个文件夹，这时可以使用File类的delete()方法来实现，在使用该方法时需要判断当前目录下是否存在文件，如果存在则需要先删除内部文件，然后再删除空的文件夹。

```java
    public static void testDeleteDir() {
        File file = new File("E:\\TestIO\\fileDelete");
        deleteDir(file);
    }
    private static void deleteDir(File file) {
        File[] listFiles = file.listFiles();
        for (File f : listFiles) {
            if (f.isDirectory()) {
                deleteDir(f);
            }
            f.delete();
        }
        file.delete();
    }
```



##  2 Paths 类

Paths 类是一个工具类，它的作用是将一个路径封装成 Path 对象，以便于在 Files 工具类中使用。

通常使用 **Path.get()** 方法完成对一个路径的封装，示例：

```java
Path path = Paths.get("E:\\TestIO\\Paths\\Path.txt");
```

### 2.1 常用方法

| 方法                           | 功能                       |
| ------------------------------ | -------------------------- |
| boolean isAbsolute()           | 该路径是否是绝对路径       |
| boolean endsWith(String other) | 该路径是否以给定字符串结束 |



## 3 Files 类

Files 类提供了**处理文件和目录，以及读取文件和写入文件的静态方法**。可以用它**创建和删除路径、复制文件、检查路径是否存在等**。此外，Files 类还拥有**创建流对象**的方法。

### 3.1 常用方法

| 方法                                                         | 功能                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| long size(Path path)                                         | 返回文件大小                                                 |
| boolean isDirectory(Path path)                               | 是否是文件夹                                                 |
| boolean isExecutable(Path path)                              | 是否是可执行文件                                             |
| boolean isHidden(Path path)                                  | 是否是隐藏的                                                 |
| boolean exists(Path path)                                    | 该文件/文件夹是否存在                                        |
| boolean notExists(Path path)                                 | 该文件/文件夹是否不存在                                      |
| boolean isReadable(Path path)                                | 是否可读                                                     |
| boolean isWritable(Path path)                                | 是否可写                                                     |
| createFile(Path filePath)                                    | 创建文件，只能是文件，不能是文件夹。如果已存在同名文件，会报错 |
| createDirectory(Path dirPath)                                | 创建文件夹。如果已存在同名文件夹，会报错                     |
| createTempFile(String prefix, String suffix)                 | 在OS的临时文件夹中创建一个临时文件                           |
| createTempFile(Path dir, String prefix, String suffix)       | 在指定的目录下创建一个临时文件。prefix是文件名前缀，suffix是文件名后缀，一般是扩展名，比如“.zip”。中间会使用系统生成的一个随机数。返回该临时文件的Path对象（绝对路径）。文件名：前缀 + 随机数 + 后缀 |
| createTempDirectory(String prefix)                           | 在OS的临时文件夹中创建一个临时文件夹                         |
| createTempDirectory(Path dir, String prefix)                 | 在指定的目录下创建一个临时文件夹。文件夹名：前缀+一个系统生成的随机数 |
| Files.copy(Path src, Path target)                            | 复制文件，如果存在同名的目标文件，会报错                     |
| Files.copy(Path src, Path target , StandardCopyOption.REPLACE_EXISTING) | 如果存在同名的目标文件，会替换。只能是文件，不能是文件夹，要复制文件夹需要递归复制子文件、子文件夹。目标文件名可与原文件名不同。 |
| Files.move(Path src, Path target)                            | 剪切，如果目标已存在，会报错                                 |
| Files.move(Path src, Path target , StandardCopyOption.REPLACE_EXISTING) | 如果目标已存在，会替换。可以是文件、文件夹。                 |
|                                                              | 在copy()、move()操作中：如果Path的中间部分路径有不存在的，会报错，并不会自动创建。比如复制一张图片，目标是"D:\test\1.png" ，如果test不存在，会报错。剪切是一种特殊的复制，先复制，复制完成后再删除原文件/文件夹。 |
| Files.delete(Path path)                                      | 删除文件、空目录。如果不存在，会报错                         |
| Files.deleteIfExists(Path path)                              | 存在才删除，不存在时不会报错。                               |
|                                                              | 只能删除文件、空目录。如果该文件夹下包含子文件、子目录，即便子目录是空的，也会报错。要删除有内容的文件夹，需要递归删除子文件、子文件夹。 |

### 3.3 写文件

使用 **Files.write()** 方法向文件中写入数据。

```java
        Path path = Paths.get("E:\\TestIO\\files\\a.txt");
        List<String> list = new ArrayList<>();
        list.add("写文件!");
		Files.write(path, list, StandardOpenOption.APPEND, StandardOpenOption.CREATE);
		// StandardOpenOption.APPEND 追加写文件
		// StandardOpenOption.CREATE 如果文件不存在，则创建文件
```

### 3.4 读文件

1. 使用 **Files.readAllLines()** 方法从文本中读取数据。

   ```java
           Path path = Paths.get("E:\\TestIO\\files\\a.txt");
           List<String> list = Files.readAllLines(path);
           System.out.println(list);
   ```

2. 使用 **Files.lines()** 方法

   ```java
   		Path path = Paths.get("E:\\TestIO\\files\\a.txt");
   		Stream<String> lines = Files.lines(path);
           lines.forEach(System.out::println);
   ```

3. 使用 **Files.newBufferedReader().lines()** 方法

   ```java
   		Path path = Paths.get("E:\\TestIO\\files\\a.txt");
   		BufferedReader br = Files.newBufferedReader(path);
           br.lines().forEach(System.out::println);
   ```

### 3.5 list()、walk()、find()的使用

Java8中，为 java.nio.file.Files 类添加了新的静态方法。这些方法返回的都是流接口 Stream。

| 方法  | 返回类型       |
| ----- | -------------- |
| lines | Stream<String> |
| list  | Stream<Path>   |
| walk  | Stream<Path>   |
| find  | Stream<Path>   |



```java
Files.list(Paths.get("E:\\TestIO\\books"))
                .forEach(System.out::println);
```

**Files.list()** 方法是非递归的，意味着使用该方法只能访问一个目录下的文件夹或者文件，而不能访问子目录。

要实现递归的访问，可以使用 **Files.walk()** 方法，该方法可以递归的访问给定的目录，返回一个Path流

```java
Files.walk(Paths.get("E:\\TestIO\\books"))
                .forEach(System.out::println);
```

**Files.find()** 方法在遍历文件树的时候遇到的每个文件都会通过 **BiPredicate** 去判断，如果 **BiPredicate** 为 返回true，则相应的 Path 对象将包含在返回的流中。

```java
Files.find(Paths.get("E:\\TestIO\\books"),
                Integer.MAX_VALUE,
                (path, attr) -> {
                    return attr.isRegularFile() && path.toString().endsWith(".txt");
                })
                .forEach(System.out::println);
```

### 3.6 递归删除文件及文件夹

1. 使用 **Files.walk()** 方法进行递归的删除文件夹 filesDelete 以及它子文件夹、文件。

   ```java
           Path path = Paths.get("E:\\TestIO\\filesDelete");
           try {
               Files.walk(path)
                       .sorted(Comparator.reverseOrder())
                       .map(Path::toFile)
                       .forEach(File::delete);
           } catch (IOException e) {
               e.printStackTrace();
           }
   ```

2. 使用 **Files.find()** 方法进行递归删除名字含 b 的文件和文件夹。

   ```java
           Path path = Paths.get("E:\\TestIO\\filesDelete");
           try {
               Files.find(path,
                       Integer.MAX_VALUE,
                       (p, attr) -> {
                           return p.toFile().getName().contains("b");
                       })
                       .sorted(Comparator.reverseOrder())
                       .map(Path::toFile)
                       .forEach(File::delete);
           } catch (IOException e) {
               e.printStackTrace();
           }
   ```



## 使用 try-with-resources

Java的一大特性就是 JVM 会对内部资源实现自动回收，但是 JVM 对外部资源（调用了底层操作系统的资源）的引用却无法自动回收，例如数据库连接、网络连接以及输入输出 IO 流等。这些连接就需要我们手动去关闭，不然会导致外部资源泄露以及文件被异常占用等。

传统的手动释放外部资源一般放在 **try-catch-finally** 代码块中，因为 finally 代码块中的语句肯定是会被执行的，这就保证了外部资源最后一定会被释放。同时考虑到 finally 代码块中也可能出现异常，finally 代码块中也有一个 try-catch。传统释放外部资源方法写法如下：

```java
        FileInputStream fis = null;
        try {
            fis = new FileInputStream(new File("E:\\TestIO\\lines.txt"));
            System.out.println(fis.read());
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                fis.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
```

JDK1.7 之后有了 try-with-resource 处理机制。首先被关闭的资源需要实现 **Closeable** 或者 **AutoCloseable** 接口。写法为：

```java
try(被关闭的资源) {
    
} catch() {
    
}
```

try-with-resources 代码块中可以声明一个或多个资源。程序执行完后会自动释放该资源。实现了 **java.lang.AutoCloseabe** 或 **java.io.Closeable** 的所有对象都可以声明。

使用 try-with-resources 改写上面的示例：

```java
        try (
                Stream<String> lines = Files.lines(Paths.get("E:\\TestIO\\lines.txt"))
        ) {
            lines.forEach(System.out::println);
        }
```

```java
        try (
                Stream<String> lines = Files.newBufferedReader(Paths.get("E:\\TestIO\\lines.txt")).lines()
        ) {
            lines.forEach(System.out::println);
        }
```

```java
        try (
                FileInputStream fis = new FileInputStream(new File("E:\\TestIO\\lines.txt"))
        ) {
            System.out.println(fis.read());
        } catch (IOException e) {
            e.printStackTrace();
        }
```

```java
        try (
                FileInputStream fis = new FileInputStream(new File("E:\\TestIO\\lines.txt"));
                FileOutputStream fos = new FileOutputStream(new File("E:\\TestIO\\linesCopy.txt"))
        ) {
            byte[] buffer = new byte[1024];
            int len;
            while ((len = fis.read(buffer)) != -1) {
                fos.write(buffer, 0, len);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
```




