---
title: Java8新特性（三） 之 Stream API
date: 2018-12-25 22:50:10
categories:
  - Java
tags:
  - Java8
  - Lambda
---
## Stream API

本文继续介绍Java 8的另一个新特性——Stream API。Stream API是对Java中集合操作的增强，可以利用它进行各种过滤、排序、分组、聚合等操作。Stream API配合Lambda表达式可以加大的提高代码可读性和编码效率，Stream API也支持并行操作，我们不用再花费很多精力来编写容易出错的多线程代码了，Stream API已经替我们做好了，并且充分利用多核CPU的优势。借助Stream API和Lambda，开发人员可以很容易的编写出高性能的并发处理程序。

### 简介

Stream API是Java 8中加入的一套新的API，主要用于处理集合操作，不过它的处理方式与传统的方式不同，称为“数据流处理”。流（Stream）类似于关系数据库的查询操作，是一种声明式操作。比如要从数据库中获取所有年龄大于20岁的用户的名称，并按照用户的创建时间进行排序，用一条SQL语句就可以搞定，不过使用Java程序实现就会显得有些繁琐，这时候可以使用流：

```Java
List<String> userNames =
        users.stream()
        .filter(user -> user.getAge() > 20)
        .sorted(comparing(User::getCreationDate))
        .map(User::getUserName)
        .collect(toList());
```

在这个大数据的时代，数据变得越来越多样化，很多时候我们会面对海量数据，并对其做一些复杂的操作（比如统计，分组），依照传统的遍历方式`（for-each）`，每次只能处理集合中的一个元素，并且是按顺序处理，这种方法是极其低效的。你也许会想到并行处理，但是编写多线程代码并非易事，很容易出错并且维护困难。不过在Java 8之后，你可以使用Stream API来解决这一问题。

![Stream-Package](https://image-hosting-lan.oss-cn-beijing.aliyuncs.com/Stream-Package.jpg)

可以看到`Stream API`里的参数大多都是函数式接口的各种形态。

还不知道函数式接口的同学快来看看这两篇

 [Java8的Lambda表达式你了解吗？](https://mp.weixin.qq.com/s/1IvdjfOws3HxuqKSxRHgLg)	

[Java8的函数式接口你真的了解吗？](https://mp.weixin.qq.com/s/-XxROpgjtCNxyxXc4Lu7JQ)

Stream API将迭代操作封装到了内部，它会自动的选择最优的迭代方式，并且使用并行方式处理时，将集合分成多段，每一段分别使用不同的线程处理，最后将处理结果合并输出。

需要注意的是，**流只能遍历一次**，遍历结束后，这个流就被关闭掉了。如果要重新遍历，可以从数据源（集合）中重新获取一个流。如果你对一个流遍历两次，就会抛出`java.lang.IllegalStateException异常`：

```Java
List<String> list = Arrays.asList("A", "B", "C", "D");
Stream<String> stream = list.stream();
stream.forEach(System.out::println);
stream.forEach(System.out::println); // 这里会抛出java.lang.IllegalStateException异常，因为流已经被关闭
```

流通常由三部分构成：

![stream-api-1](https://image-hosting-lan.oss-cn-beijing.aliyuncs.com/stream-api-1.png)

1. 数据源：数据源一般用于流的获取，比如本文开头那个过滤用户的例子中users.stream()方法。
2. 中间处理：中间处理包括对流中元素的一系列处理，如：过滤（filter()），映射（map()），排序（sorted()）。
3. 终端处理：终端处理会生成结果，结果可以是任何不是流值，如List<String>；也可以不返回结果，如stream.forEach(System.out::println)就是将结果打印到控制台中，并没有返回。

![stream-api-2](https://image-hosting-lan.oss-cn-beijing.aliyuncs.com/stream-api-2.png)

中间操作也称为惰性操作。
终端操作也称为急切操作。
惰性操作不处理元素，直到在流上调用热切操作。
流上的中间操作产生另一流。
Streams链接操作以创建流管道。

### 创建流

创建流的方式有很多，具体可以划分为以下几种：

#### 由值创建流

使用静态方法`Stream.of()`创建流，该方法接收一个变长参数：

```Java
Stream<Stream> stream = Stream.of("A", "B", "C", "D");
```

也可以使用静态方法`Stream.empty()`创建一个空的流：

```Java
Stream<Stream> stream = Stream.empty();
```

#### 由数组创建流

使用静态方法`Arrays.stream()`从数组创建一个流，该方法接收一个数组参数：

```Java
String[] strs = {"A", "B", "C", "D"};
Stream<Stream> stream = Arrays.stream(strs);
```

#### 通过文件生成流

使用`java.nio.file.Files`类中的很多静态方法都可以获取流，比如`Files.lines()`方法，该方法接收一个`java.nio.file.Path`对象，返回一个由文件行构成的字符串流：

```Java
Stream<String> stream = Files.lines(Paths.get("text.txt"), Charset.defaultCharset());
```

#### 通过函数创建流

`java.util.stream.Stream`中有两个静态方法用于从函数生成流，他们分别是`Stream<T> generate(Supplier<T> s)`和`Stream<T> iterate(final T seed, final UnaryOperator<T> f)`：

```Java
// iteartor
Stream.iterate(0, n -> n + 2).limit(51).forEach(System.out::println);
// generate
Stream.generate(() -> "Hello Man!").limit(10).forEach(System.out::println);
```

第一个方法会打印100以内的所有偶数，第二个方法打印10个`Hello Man!`。**值得注意的是，这两个方法生成的流都是无限流，没有固定大小，可以无穷的计算下去**，在上面的代码中我们使用了`limit()`来避免打印无穷个值。

一般来说，`iterate()`用于生成一系列值，比如生成以当前时间开始之后的10天的日期：

```Java
Stream.iterate(LocalDate.now(), date -> date.plusDays(1)).limit(10).forEach(System.out::println);
```

`generate()`方法用于生成一些随机数，比如生成10个UUID：

```Java
Stream.generate(() -> UUID.randomUUID().toString()).limit(10).forEach(System.out::println);
```

### 使用流

`Stream`接口中包含许多对流操作的方法，这些方法分别为：

- `filter()`：对流的元素过滤
- `map()`：将流的元素映射成另一个类型
- `distinct()`：去除流中重复的元素
- `sorted()`：对流的元素排序
- `forEach()`：对流中的每个元素执行某个操作
- `peek()`：与`forEach()`方法效果类似，不同的是，该方法会返回一个新的流，而`forEach()`无返回
- `limit()`：截取流中前面几个元素
- `skip()`：跳过流中前面几个元素
- `toArray()`：将流转换为数组
- `reduce()`：对流中的元素归约操作，将每个元素合起来形成一个新的值
- `collect()`：对流的汇总操作，比如输出成`List`集合
- `anyMatch()`：匹配流中的元素，类似的操作还有`allMatch()`和`noneMatch()`方法
- `findFirst()`：查找第一个元素，类似的还有`findAny()`方法
- `max()`：求最大值
- `min()`：求最小值
- `count()`：求总数

下面逐一介绍这些方法的用法。

**简单栗子**：

```Java
Stream.of(1, 8, 5, 2, 1, 0, 9, 2, 0, 4, 8)
    .filter(n -> n > 2)     // 对元素过滤，保留大于2的元素
    .distinct()             // 去重，类似于SQL语句中的DISTINCT
    .skip(1)                // 跳过前面1个元素
    .limit(2)               // 返回开头2个元素，类似于SQL语句中的SELECT TOP
    .sorted()               // 对结果排序
    .forEach(System.out::println);
```

#### 过滤

```Java
List<Apple> filterList = appleList.stream().filter(a -> a.getName().equals("香蕉")).collect(Collectors.toList());
```

#### 求和(归约)

归约操作就是将流中的元素进行合并，形成一个新的值，常见的归约操作包括求和，求最大值或最小值。归约操作一般使用`reduce()`方法，与`map()`方法搭配使用，可以处理一些很复杂的归约操作。)

```Java
//计算 总金额
// map -> reduce
BigDecimal totalMoney = appleList.stream().map(Apple::getMoney).reduce(BigDecimal.ZERO, BigDecimal::add);
//计算 数量
int sum = appleList.stream().mapToInt(Apple::getNum).sum();
```

#### 提取Bean某一属性

```Java
// 取Bean某一属性
stuList.stream()
		.map(Student::getId).distinct()
		.collect(Collectors.toList());
```

#### 去重

- ##### 一般去重

```Java
List<Integer> distinctNumbers = numbers.stream().distinct().collect(Collectors.toList());
```

- ##### 条件去重

```Java
// 根据Bean的某种属性去重
// 首先定义一个过滤器
public static <T> Predicate<T> distinctByKey(Function<? super T, Object> keyExtractor) {
    Map<Object, Boolean> seen = new ConcurrentHashMap<>();
    return object -> seen.putIfAbsent(keyExtractor.apply(object), Boolean.TRUE) == null;
}

List<User> distinctUsers = users.stream()
        .filter(distinctByKey(User::getName))
        .collect(Collectors.toList());

这种去重也可以对多个Key进行，将多个Key拼接成一个Key。
private String getGroupingByKey(Person p){
    return p.getAge()+"-"+p.getName();
}
下面的分组也是用了这个思想。

// 或者
List<User> unique = list.stream()
            .collect(Collectors
            .collectingAndThen(Collectors
            .toCollection(() -> new TreeSet<>(Comparator.comparing(o -> o.getName()))), ArrayList::new));
            
// 最终版
list.stream().collect(Collectors.collectingAndThen(
                Collectors.toCollection(() -> new TreeSet<>(
                        Comparator.comparing(o -> o.getAge() + ";" + o.getName()))), ArrayList::new)).forEach(u -> println(u));
```

#### 排序

**排序需重新赋值，内部操作循环的foreach的话就不用赋值新变量**  
`list = list.stream().sorted(byNumber).collect(Collectors.toList());`

多属性先后顺序排序：  
自己重写comparator方法

```
@Test
public void testSort_with_multipleComparator() throws Exception {

   ArrayList<Human> humans = Lists.newArrayList(
           new Human("tomy", 22),
           new Human("li", 25)
   );

   Comparator<Human> comparator = (h1, h2) -> {

       if (h1.getName().equals(h2.getName())) {
           return Integer.compare(h1.getAge(), h2.getAge());
       }
       return h1.getName().compareTo(h2.getName());
   };
   humans.sort(comparator.reversed());
   Assert.assertThat("tomy", equalTo(humans.get(0).getName()));

}
```

```Java
// 一般排序
// 定义一个比较器，用于排序
Comparator<Rule> byNumber = Comparator.comparingInt(Rule::getNumber);

// 获取排序后的list，先通过filter筛选，然后在排序
List<Rule> rule = lstRule.stream().filter(s -> s.getCode() == 2).sorted(byNumber).collect(Collectors.toList());

// 联合排序
Comparator<Rule> byNumber = Comparator.comparingInt(Rule::getNumber);
Comparator<Rule> byCode = Comparator.comparingInt(Rule::getCode);
Comparator<Rule> byNumberAndCode = byNumber.thenComparing(byCode);
// byNumberAndCode是一个联合排序的比较器
List<Rule> rule = lstRule.stream().filter(s -> s.getCode() == 2).sorted(byNumberAndCode).collect(Collectors.toList());

// 排序时包括null
List<User> nList = list.stream()
    .sorted(Comparator.comparing(User::getCode, Comparator.nullsFirst(String::compareTo)))
    .collect(Collectors.toList());

List<User> list = minPriceList.stream()
    .sorted(Comparator.comparing(l -> l.getCreateDate(), Comparator.nullsLast(Date::compareTo)))
    .collect(Collectors.toList());
    
eg: 多字段排序例子，有个坑是后排序的字段需要先写
List<Food> list = new ArrayList<>();
list.add(new Food(3, "aa", 2));
list.add(new Food(3, "bb", null));
list.add(new Food(2, "cc", 1));
list.add(new Food(2, "dd", 15));

List<Food> sortedList = list.stream()
        .sorted(Comparator.comparing(Food::getPrice, Comparator.nullsLast(Integer::compareTo)).reversed())
        .sorted(Comparator.comparing(Food::getId, Comparator.nullsFirst(Integer::compareTo)))
        .collect(Collectors.toList());
```

#### 分组

此方法类似Mysql的group by，但是可能比sql的复杂语句来得简单些。

> 辅助POJO

    static class Person {
    
        private String name;
        private int age;
        private long salary;
    
        Person(String name, int age, long salary) {
    
            this.name = name;
            this.age = age;
            this.salary = salary;
        }
    
        @Override
        public String toString() {
            return String.format("Person{name='%s', age=%d, salary=%d}", name, age, salary);
        }
    }

- ##### 针对单个属性分组

```Java
// 一般分组后的结构是Map，key是分组的属性，value是组成员
Map<Integer, List<Person>> peopleByAge = people.stream().collect(Collectors.groupingBy(Person::getAge));

Map<Integer, List<Person>> peopleByAge = people.stream().collect(Collectors.groupingBy(Person::getAge, Collectors.toList()));

Map<Integer, List<Person>> peopleByAge = people.stream().collect(Collectors.groupingBy(p -> p.age, Collectors.mapping((Person p) -> p, toList())));

上面三种方法返回均相同，分组后的Map里的value也可以根据Collectors.groupingBy方法的第二个参数设置不同，Collectors里还有更多的方法，如求和、最值、均值、拼接。
```

- ##### 针对多个属性分组

```
方式一：
Map<String, Map<Integer, List<Person>>> map = people.stream()
    .collect(Collectors.groupingBy(Person::getName,
        Collectors.groupingBy(Person::getAge));
map.get("Fred").get(18);

方式二：
定义一个表示分组的类：
//静态内部类
class Person {
    public static class NameAge {
        public NameAge(String name, int age) {
            ...
        }

        // 注意 重写方法 must implement equals and hash function
    }

    public NameAge getNameAge() {
        return new NameAge(name, age);
    }
}


Map<NameAge, List<Person>> map = people.collect(Collectors.groupingBy(Person::getNameAge));
map.get(new NameAge("Fred", 18));

不定义分组类也可以使用如apache commons pair如果您使用这些库之一。
Map<Pair<String, Integer>, List<Person>> map =
    people.collect(Collectors.groupingBy(p -> Pair.of(p.getName(), p.getAge())));
map.get(Pair.of("Fred", 18));


最终方式：
将多个字段拼接成一个新字段，在使用Java8的groupBy进行分组。上面的去重里我也是沿用了这个思想。
这个方法虽然看起来简单笨拙，但是却是最有效的解决了我的问题的，多个字段如果大于2个字段，上面的Pair就不是很好用，并且分组出来的结构也复杂。
Map<String, List<Person>> peopleBySomeKey = people
                .collect(Collectors.groupingBy(p -> getGroupingByKey(p), Collectors.mapping((Person p) -> p, toList())));

//write getGroupingByKey() function
private String getGroupingByKey(Person p){
    return p.getAge()+"-"+p.getName();
}
```

- ##### 分组求和

```Java
//分组求和，key是分组属性名
Map<String, Long> tt = orgHoldingDatas.stream()
        .collect(Collectors.groupingBy(OrgHoldingData::getOrgTypeCode, Collectors.summingLong(OrgHoldingData::getHeldNum)));
```

- ##### 分组相加

```Java
list.stream()
  .sorted(Comparator.comparing(User::getAge))
  .collect(Collectors.groupingBy(User::getId))
  .forEach((k, v) -> {
    Optional<User> csm = v.stream().reduce((v1, v2) -> {
      v1.setName(v1.getNameS() + "、" + v2.getName());
      list.remove(v2);
      return v1;
    });
  });

// 或者
list.stream().collect(Collectors.groupingBy(User::getName))
  .forEach((k, v) -> {
    Optional<User> sum = v.stream().reduce((v1, v2) -> {	//合并
      v1.setNum(v1.getNum() + v2.getNum());
      v1.setPct(v1.getPct() + v2.getPct());
      return v1;
    });
    if (sum.isPresent()) {
      items.add(sum.get());
    }
  });
```

- ##### 自定义分组

```Java
Map<String, List<Fruit>> map = list.stream()
        .collect(Collectors.groupingBy((Function<Fruit, String>) fruit -> {
            String key;
            if (fruit.getType().equals("1")) {
                key = "苹果";
            } else if (fruit.getType().equals("2")) {
                key = "香蕉";
            } else {
                key = "其他";
            }
            return key;
        }, Collectors.toList()));
```

#### 分区

分区的话就结果就只有两个部分part这种。要么是这一个区要么是另一个。

```Java
Map<Boolean, List<Student>> map = students.stream()
              .collect(Collectors.partitioningBy(student -> student.getScore() > 90));
```

#### 巧用flatMap

```Java
// 应该使用flatMap . flatMap()的作用在于打平
List<String> reList = list.stream().map(item -> item.split(",")).flatMap(Arrays::stream).distinct()
        .collect(Collectors.toList());
```

#### 归约

归约操作就是将流中的元素进行合并，形成一个新的值，常见的归约操作包括求和，求最大值或最小值。归约操作一般使用`reduce()`方法，与`map()`方法搭配使用，可以处理一些很复杂的归约操作。有点儿类似大数据里的Map-Reduce。

#### 流统计

- DoubleSummaryStatistics
- LongSummaryStatistics
- IntSummaryStatistics

### 并行流

并行流使用集合的`parallelStream()`方法可以获取一个并行流。Java内部会将流的内容分割成若干个子部分，然后将它们交给多个线程并行处理，这样就将工作的负担交给多核CPU的其他内核处理。

在并行流中，我们关注最多的大概就是性能问题。在观察发现，在选择合适的数据结构和处理后，并行流确实可以优于平时的for循环。

`parallelStream()`本质上基于java7的Fork-Join框架实现，其默认的线程数为宿主机的内核数。

启动并行流式处理虽然简单，只需要将`stream()`替换成`parallelStream()`即可，但既然是并行，就会涉及到多线程安全问题，所以在启用之前要先确认并行是否值得（并行的效率不一定高于顺序执行），另外就是要保证线程安全。此两项无法保证，那么并行毫无意义，毕竟结果比速度更加重要。

更深入的`parallelStream`详见：[深入浅出parallelStream](https://blog.csdn.net/u011001723/article/details/52794455)  

### 坑

- 在`Stream.of`创建的流中，对于流的使用只能操作一次，操作后会有个标志位被置为true，如果再次对此`流`进行操作，会报错。但是对于集合的流形式，如list.stream()不会有问题，可多次操作。
- `parallelStream`的多线程并发写list会发生线程安全问题，list数据少了，导致数组越界。建议使用线程安全的集合类。  

[记一次java8 parallelStream使用不当引发的血案](https://my.oschina.net/7001/blog/1475500)  
[Java8的ParallelStream踩坑记录\-云栖社区\-阿里云](https://yq.aliyun.com/articles/652718) 

### 性能测试

[Java Stream API性能测试 \- CarpenterLee \- 博客园](https://www.cnblogs.com/CarpenterLee/p/6675568.html)

**参考**

[Java8 新特性之流式数据处理 \- 深蓝至尊 \- 博客园](https://www.cnblogs.com/shenlanzhizun/p/6027042.html)
[Java 8新特性（二）：Stream API\_Java\_一书生VOID\-CSDN博客](https://blog.csdn.net/lw900925/article/details/78921657)

---

<div style="text-align:center">求关注、分享、在看！！！
  你的支持是我创作最大的动力。</div>

![](https://image-hosting-lan.oss-cn-beijing.aliyuncs.com/qrcode_for_hbh.jpg)