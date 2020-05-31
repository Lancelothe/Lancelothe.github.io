---
title: Java8新特性（一） 之 Lambda表达式
date: 2018-12-15 23:30:53
categories:
  - Java
tags:
  - Java8
  - Lambda
---

## Lambda表达式是什么

Lambda表达式（lambda expression）是一个匿名函数，由数学中的λ演算而得名。在Java 8中可以把Lambda表达式理解为匿名函数，它没有名称，但是有参数列表、函数主体、返回类型等。


Lambda表达式的语法如下：
> (parameters) -> { statements; }

为什么要使用Lambda表达式？前面你也看到了，在Java中使用内部类显得十分冗长，要编写很多样板代码，Lambda表达式正是为了简化这些步骤出现的，它使代码变得清晰易懂。

## 如何使用Lambda表达式

Lambda表达式是为了简化内部类的，你可以把它当成是内部类的一种简写方式，只要是有内部类的代码块，都可以转化成Lambda表达式：


```Java
// Comparator排序
List<Integer> list = Arrays.asList(3, 1, 4, 5, 2);
list.sort(new Comparator<Integer>() {
    @Override
    public int compare(Integer o1, Integer o2) {
        return o1.compareTo(o2);
    }
});

// 使用Lambda表达式简化
list.sort((o1, o2) -> o1.compareTo(o2));
```

```Java
// Runnable代码块
Thread thread = new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello Man!");
    }
});

// 使用Lambda表达式简化
Thread thread = new Thread(() -> System.out.println("Hello Man!"));
```

可以看出，只要是内部类的代码块，就可以使用Lambda表达式简化，并且简化后的代码清晰易懂。

### 方法引用

甚至，Comparator排序的Lambda表达式还可以进一步简化：
> list.sort(Integer::compareTo);

这种写法被称为 **方法引用**，方法引用是Lambda表达式的简便写法。如果你的Lambda表达式只是调用这个方法，最好使用名称调用，而不是描述如何调用，这样可以提高代码的可读性。

方法引用使用`::`分隔符，分隔符的前半部分表示引用类型，后面半部分表示引用的方法名称。例如：`Integer::compareTo`表示引用类型为`Integer`，引用名称为`compareTo`的方法。

对于 Lambda 表达式到方法引用的简化，我们提供以下规则：

Lambda 表达式 | 方法引用
---|---
(args) -> ClassName.staticMethod(args)	   | ClassName::staticMethod
(arg0, ...) -> arg0.instanceMethod(...)	   | ClassName::instanceMethod
(args) -> expression.instanceMethod(args)  | expression::instanceMethod

特别的，对于构造函数的方法引用：  `ClassName::new`
类似使用方法引用的例子还有打印集合中的元素到控制台中：
`list.forEach(System.out::println);`

---

<div style="text-align:center">求关注、分享、在看！！！
  你的支持是我创作最大的动力。</div>

![](https://image-hosting-lan.oss-cn-beijing.aliyuncs.com/qrcode_for_hbh.jpg)