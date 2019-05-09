---
title: Java8新特性（三） 之 Stream API
date: 2018-12-25 22:50:10
categories:
  - Java
tags:
  - Java8
  - Lambda
top: 1
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

Stream API将迭代操作封装到了内部，它会自动的选择最优的迭代方式，并且使用并行方式处理时，将集合分成多段，每一段分别使用不同的线程处理，最后将处理结果合并输出。

需要注意的是，**流只能遍历一次**，遍历结束后，这个流就被关闭掉了。如果要重新遍历，可以从数据源（集合）中重新获取一个流。如果你对一个流遍历两次，就会抛出`java.lang.IllegalStateException异常`：
```Java
List<String> list = Arrays.asList("A", "B", "C", "D");
Stream<String> stream = list.stream();
stream.forEach(System.out::println);
stream.forEach(System.out::println); // 这里会抛出java.lang.IllegalStateException异常，因为流已经被关闭
```
流通常由三部分构成：
![stream_1](/images/markdown-img-paste-20181220151915981.png)
1. 数据源：数据源一般用于流的获取，比如本文开头那个过滤用户的例子中users.stream()方法。
2. 中间处理：中间处理包括对流中元素的一系列处理，如：过滤（filter()），映射（map()），排序（sorted()）。
3. 终端处理：终端处理会生成结果，结果可以是任何不是流值，如List<String>；也可以不返回结果，如stream.forEach(System.out::println)就是将结果打印到控制台中，并没有返回。
![](/images/markdown-img-paste-20181220152359823.png)
Stream中的操作可以分为两大类：中间操作与结束操作，中间操作只是对操作进行了记录，只有结束操作才会触发实际的计算（即惰性求值），这也是Stream在迭代大集合时高效的原因之一。
中间操作又可以分为无状态（Stateless）操作与有状态（Stateful）操作，前者是指元素的处理不受之前元素的影响；后者是指该操作只有拿到所有元素之后才能继续下去。结束操作又可以分为短路与非短路操作，这个应该很好理解，前者是指遇到某些符合条件的元素就可以得到最终结果；而后者是指必须处理所有元素才能得到最终结果。

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

#### 求和(规约)
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
List<HKCsm> unique = list.stream()
            .collect(Collectors
            .collectingAndThen(Collectors
            .toCollection(() -> new TreeSet<>(Comparator.comparing(o -> o.getIDVUNIC()))), ArrayList::new));
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
```
更多参考：[Java8：Lambda表达式增强版Comparator和排序 \- ImportNew](http://www.importnew.com/15259.html)

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
        .sorted(Comparator.comparing(HKCsm::getPSNREFC))
        .collect(Collectors.groupingBy(HKCsm::getIDVUNIC))
        .forEach((k, v) -> {
            Optional<HKCsm> csm = v.stream().reduce((v1, v2) -> {
                v1.setPSNNAME_OGDIS(v1.getPSNNAME_OGDIS() + "、" + v2.getPSNNAME_OGDIS());
                list.remove(v2);
                return v1;
            });
        });

// 或者
holderList.stream().collect(Collectors.groupingBy(HkMjshhshgStu::getShhnameOgdis))
        .forEach((k, v) -> {
            Optional<HkMjshhshgStu> sum = v.stream().reduce((v1, v2) -> {	//合并
                v1.setNumrts(v1.getNumrts() + v2.getNumrts());
                v1.setPctishg(v1.getPctishg() + v2.getPctishg());
                return v1;
            });
            if (sum.isPresent()) {
                items.add(sum.get());
            }
        });
```
- ##### 自定义分组
```Java
Map<String, List<OrgHoldingData>> tt  = orgHoldingDatas.stream()
        .collect(Collectors.groupingBy((Function<OrgHoldingData, String>) org -> {
            String key;
            if (org.getOrgTypeEncode().equals("1")) {
                key = "苹果";
            } else if (org.getOrgTypeEncode().equals("2")) {
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
Map<Boolean, List<Student>> map4 = students.stream()
              .collect(Collectors.partitioningBy(student -> student.getScore() > 90));
```

#### 巧用flatMap
```Java
// 应该使用flatMap . flatMap()的作用在于打平
List<String> reList = list.stream().map(item -> item.split(" ")).flatMap(Arrays::stream).distinct()
        .collect(Collectors.toList());
```

#### 规约
规约是什么？规约就是我们想要处理后要进行计算的东西，如上面我们进行的 求和、分组、分区、

#### 流统计
- DoubleSummaryStatistics
- LongSummaryStatistics
- IntSummaryStatistics
### 并行流
并行流使用集合的`parallelStream()`方法可以获取一个并行流。Java内部会将流的内容分割成若干个子部分，然后将它们交给多个线程并行处理，这样就将工作的负担交给多核CPU的其他内核处理。

在并行流中，我们关注最多的大概就是性能问题。在观察发现，在选择合适的数据结构和处理后，并行流确实可以优于平时的for循环。

## 坑
- 在Stream.of创建的流中，对于流的使用只能操作一次，操作后会有个标志位被置为true，如果再次对此`流`进行操作，会报错。但是对于集合的流形式，如list.stream()不会有问题，可多次操作。
- parallelStream 的多线程并发写list会发生线程安全问题，list数据少了，导致数组越界。建议使用线程安全的集合类。
[深入浅出parallelStream \- 梦铃之境的专栏 \- CSDN博客](https://blog.csdn.net/u011001723/article/details/52794455)
[记一次java8 parallelStream使用不当引发的血案 \- AbeJeffrey的个人空间 \- 开源中国](https://my.oschina.net/7001/blog/1475500)
[java8的ParallelStream踩坑记录\-云栖社区\-阿里云](https://yq.aliyun.com/articles/652718)

#### 性能测试
[Java Stream API性能测试 \- CarpenterLee \- 博客园](https://www.cnblogs.com/CarpenterLee/p/6675568.html)

#### 参考
[Java8 新特性之流式数据处理 \- 深蓝至尊 \- 博客园](https://www.cnblogs.com/shenlanzhizun/p/6027042.html)
[Java 8新特性（二）：Stream API \| 一书生VOID的博客](https://lw900925.github.io/java/java8-stream-api.html)
