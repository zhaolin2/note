# Java

## 流操作



| 操作      | 类型 | 返回类型   | 使用的类型/函数式接口 | 函数描述符      |
| :-------- | :--- | :--------- | :-------------------- | :-------------- |
| filter    | 中间 | `Stream`   | `Predicate`           | `T -> boolean`  |
| distinct  | 中间 | `Stream`   |                       |                 |
| skip      | 中间 | `Stream`   | `long`                |                 |
| limit     | 中间 | `Stream`   | `long`                |                 |
| map       | 中间 | `Stream`   | `Function`            | `T -> R`        |
| flatMap   | 中间 | `Stream`   | `Function>`           | `T -> Stream`   |
| sorted    | 中间 | `Stream`   | `Comparator`          | `(T, T) -> int` |
| anyMatch  | 终端 | `boolean`  | `Predicate`           | `T -> boolean`  |
| noneMatch | 终端 | `boolean`  | `Predicate`           | `T -> boolean`  |
| allMatch  | 终端 | `boolean`  | `Predicate`           | `T -> boolean`  |
| findAny   | 终端 | `Optional` |                       |                 |
| findFirst | 终端 | `Optional` |                       |                 |
| forEach   | 终端 | `void`     | `Consumer`            | `T -> void`     |
| collect   | 终端 | `R`        | `Collector`           |                 |
| reduce    | 终端 | `Optional` | `BinaryOperator`      | `(T, T) -> T`   |
| count     | 终端 | `long`     |                       |                 |

## filter

Streams接口支持·filter方法，该方法接收一个Predicate<T>，函数描述符为T -> boolean，用于对集合进行筛选，返回所有满足的元素：
``` java
list.stream()
    .filter(s -> s.contains("#"))
    .forEach(System.out::println);
```
结果输出C#。

## distinct
distinct方法用于排除流中重复的元素，类似于SQL中的distinct操作。比如筛选中集合中所有的偶数，并排除重复的结果：
``` java
List<Integer> numbers = Arrays.asList(1, 2, 1, 3, 3, 2, 4);
numbers.stream()
       .filter(i -> i % 2 == 0)
       .distinct()
       .forEach(System.out::println);
```
结果输出2 4。

## skip
skip(n)方法用于跳过流中的前n个元素，如果集合元素小于n，则返回空流。比如筛选出以J开头的元素，并排除第一个：

```java
list.stream()
    .filter(s -> s.startsWith("J"))
    .skip(1)
    .forEach(System.out::println);
```

结果输出JavaScript。

## limit

limit(n)方法返回一个长度不超过n的流，比如下面的例子将输出Java JavaScript python：
``` java
list.stream()
    .limit(3)
    .forEach(System.out::println);
```

## map
map方法接收一个函数作为参数。这个函数会被应用到每个元素上，并将其映射成一个新的元素。如：
``` java
list.stream()
    .map(String::length)
    .forEach(System.out::println);
```
结果输出4 10 6 3 2 6 5 3 4。

map还支持将流特化为指定原始类型的流，如通过mapToInt，mapToDouble和mapToLong方法，可以将流转换为IntStream，DoubleStream和LongStream。特化后的流支持sum，min和max方法来对流中的元素进行计算。比如：
``` java
List<Integer> numbers = Arrays.asList(1, 2, 1, 3, 3, 2, 4);
IntStream intStream = numbers.stream().mapToInt(a -> a);
System.out.println(intStream.sum()); // 16
```
也可以通过下面的方法，将IntStream转换为Stream：
``` java
Stream<Integer> s = intStream.boxed();
```
## flatMap
flatMap用于将多个流合并成一个流，俗称流的扁平化。这么说有点抽象，举个例子，比如现在需要将list中的各个元素拆分为一个个字母，并过滤掉重复的结果，你可能会这样做：

``` java
list.stream()
   .map(s -> s.split(""))
   .distinct()
   .forEach(System.out::println);
```
输出如下：
``` java
[Ljava.lang.String;@e9e54c2
[Ljava.lang.String;@65ab7765
[Ljava.lang.String;@1b28cdfa
[Ljava.lang.String;@eed1f14
[Ljava.lang.String;@7229724f
[Ljava.lang.String;@4c873330
[Ljava.lang.String;@119d7047
[Ljava.lang.String;@776ec8df
[Ljava.lang.String;@4eec7777
```
这明显不符合我们的预期。实际上在map(s -> s.split(""))操作后，返回了一个Stream<String[]>类型的流，所以输出结果为每个数组对象的句柄，而我们真正想要的结果是Stream<String>！

在Stream中，可以使用Arrays.stream()方法来将数组转换为流，改造上面的方法：
``` java
list.stream()
    .map(s -> s.split(""))
    .map(Arrays::stream)
    .distinct()
    .forEach(System.out::println);
```
输出如下：
``` java
java.util.stream.ReferencePipeline$Head@eed1f14
java.util.stream.ReferencePipeline$Head@7229724f
java.util.stream.ReferencePipeline$Head@4c873330
java.util.stream.ReferencePipeline$Head@119d7047
java.util.stream.ReferencePipeline$Head@776ec8df
java.util.stream.ReferencePipeline$Head@4eec7777
java.util.stream.ReferencePipeline$Head@3b07d329
java.util.stream.ReferencePipeline$Head@41629346
java.util.stream.ReferencePipeline$Head@404b9385
```
因为上面的流经过map(Arrays::stream)处理后，将每个数组变成了一个新的流，返回结果为流的数组Stream<String>[]，所以输出是各个流的句柄。我们还需将这些新的流连接成一个流，使用flatMap来改写上面的例子：
``` java
list.stream()
    .map(s -> s.split(""))
    .flatMap(Arrays::stream)
    .distinct()
    .forEach(s -> System.out.print(s + " "));
```
输出如下：
``` java
J a v S c r i p t y h o n P H C # G l g w f + R u b
```
和map类似，flatMap方法也有相应的原始类型特化方法，如flatMapToInt等。

## 终端操作
## anyMatch
anyMatch方法用于判断流中是否有符合判断条件的元素，返回值为boolean类型。比如判断list中是否含有SQL元素：
``` java
list.stream()
    .anyMatch(s -> "SQL".equals(s)); // false
```
## allMatch
allMatch方法用于判断流中是否所有元素都满足给定的判断条件，返回值为boolean类型。比如判断list中是否所有元素长度都不大于10：
``` java
list.stream()
    .allMatch(s -> s.length() <= 10); // true
```
## noneMatch
noneMatch方法用于判断流中是否所有元素都不满足给定的判断条件，返回值为boolean类型。比如判断list中不存在长度大于10的元素：
``` java
list.stream()
    .noneMatch(s -> s.length() > 10); // true
```
## findAny
findAny方法用于返回流中的任意元素的Optional类型，例如筛选出list中任意一个以J开头的元素，如果存在，则输出它：
``` java
list.stream()
    .filter(s -> s.startsWith("J"))
    .findAny()
    .ifPresent(System.out::println); // Java
```
## findFirst
findFirst方法用于返回流中的第一个元素的Optional类型，例如筛选出list中长度大于5的元素，如果存在，则输出第一个：
``` java
list.stream()
    .filter(s -> s.length() > 5)
    .findFirst()
    .ifPresent(System.out::println); // JavaScript
```
reduce
reduce函数从字面上来看就是压缩，缩减的意思，它可以用于数字类型的流的求和，求最大值和最小值。如对numbers中的元素求和：
``` java
List<Integer> numbers = Arrays.asList(1, 2, 1, 3, 3, 2, 4);
numbers.stream()
       .reduce(0, Integer::sum); // 16
```
reduce函数也可以不指定初始值，但这时候将返回一个Optional对象，比如求最大值和最小值：
``` java
numbers.stream()
       .reduce(Integer::max)
       .ifPresent(System.out::println); // 4

numbers.stream()
       .reduce(Integer::min)
       .ifPresent(System.out::println); // 1
```
## forEach
forEach用于迭代流中的每个元素，最为常见的就是迭代输出，如：
``` java
list.stream().forEach(System.out::println);
```
## count
count方法用于统计流中元素的个数，比如：
``` java
list.stream().count(); // 9
```
## collect
collect方法用于收集流中的元素，并放到不同类型的结果中，比如List、Set或者Map。举个例子：
``` java
List<String> filterList = list.stream()
        .filter(s -> s.startsWith("J")).collect(Collectors.toList());
```
如果需要以Set来替代List，只需要使用Collectors.toSet()就好了。

## 流的构建
除了使用集合对象的stream方法构建流之外，我们可以手动构建一些流。

数值范围构建
IntStream和LongStream对象支持range和rangeClosed方法来构建数值流。这两个方法都是第一个参数接受起始值，第二个参数接受结束值。但range是不包含结束值的，而rangeClosed则包含结束值。比如对1到100的整数求和：
``` java
IntStream.rangeClosed(1, 100).sum(); // 5050
```
## 由值构建
静态方法Stream.of可以显式值创建一个流。它可以接受任意数量的参数。例如，以下代码直接使用Stream.of创建了一个字符串流:

Stream<String> s = Stream.of("Java", "JavaScript", "C++", "Ruby");
也可以使用Stream.empty()构建一个空流：
``` java
Stream<Object> emptyStream = Stream.empty();
```
## 由数组构建
静态方法Arrays.stream可以通过数组创建一个流。它接受一个数组作为参数。例如：

``` java
int[] arr = {1, 2, 3, 4, 5};
IntStream intStream = Arrays.stream(arr);
```
## 由文件生成流
java.nio.file.Files中的很多静态方法都会返回一个流。例如Files.lines方法会返回一个由指定文件中的各行构成的字符串流。比如统计一个文件中共有多少个字：
``` java
long wordCout = 0L;
try (Stream<String> lines = Files.lines(Paths.get("file.txt"), Charset.defaultCharset())) {
    wordCout = lines.map(l -> l.split(""))
                    .flatMap(Arrays::stream)
                    .count();
} catch (Exception ignore) {}

```
由函数构造
Stream API提供了两个静态方法来从函数生成流：Stream.iterate和Stream.generate。这两个操作可以创建所谓的无限流。比如下面的例子构建了10个偶数：
``` java
Stream.iterate(0, n -> n + 2)
      .limit(10).forEach(System.out::println);
```
iterate方法接受一个初始值（在这里是0），还有一个依次应用在每个产生的新值上的Lambda（UnaryOperator类型）。这里，我们使用Lambda n -> n + 2，返回的是前一个元 素加上2。因此，iterate方法生成了一个所有正偶数的流：流的第一个元素是初始值0。然后加上2来生成新的值2，再加上2来得到新的值4，以此类推。

与iterate方法类似，generate方法也可让你按需生成一个无限流。但generate不是依次对每个新生成的值应用函数，比如下面的例子生成了5个0到1之间的随机双精度数：

```java
Stream.generate(Math::random)
      .limit(5)
      .forEach(System.out::println);输出结果如下：
```

``` java
0.6334646850587863
0.4190147641834009
0.4361968394515475
0.6911796456838655
0.08156838267267075

```