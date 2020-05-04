---
title: Java8新特性（五）之 Date Time API
date: 2019-01-16 23:06:43
categories:
  - Java
tags:
  - Java8
---
Java8新增的就是LocalDate、LocalTime、LocalDateTime、Instant、Duration、Period几个大类。

之前用joda能做到的，现在用Java8里面的api几乎都能做到，语法也很相似。

## LocalDateTime的一些用法
```
// 取时间戳
long localDateTime = LocalDateTime.of(LocalDate.now().minusDays(1), LocalTime.MIN).toInstant(ZoneOffset.UTC).toEpochMilli();

// 获取当天零点时间
LocalDateTime today_start = LocalDateTime.of(LocalDate.now(), LocalTime.MIN);//当天零点
String td_st_str =
today_start.format(DateTimeFormatter.ofPattern("yyyyMMddHHmmss"));


// 获取当天结束时间
LocalDateTime today_end = LocalDateTime.of(LocalDate.now(), LocalTime.MAX);//当天零点


// 设置2018年1月1日
LocalDateTime.of(2018, 1, 1, 0, 0).toLocalDate();
// 获取2018年1月1日时间戳
LocalDateTime.of(2018, 1, 1, 0, 0).toInstant(ZoneOffset.of("+8")).toEpochMilli();
// 此时减一年时间戳1
LocalDateTime.of(LocalDate.now().minusYears(1), LocalTime.MIN).toInstant(ZoneOffset.of("+8")).toEpochMilli();
// 此时减一年时间戳2
long lastYear = LocalDateTime.now().minusYears(1).withHour(0).withMinute(0).withSecond(0).withNano(0).toInstant(ZoneOffset.of("+8")).toEpochMilli();
```


[LocalDate、LocalDateTime与timestamp、Date的转换 \- 简书](https://www.jianshu.com/p/b4629857fc6f)
## Date和LocalDate互转
Date对象表示特定的日期和时间，而LocalDate(Java8)对象只包含没有任何时间信息的日期。
因此，如果我们只关心日期而不是时间信息，则可以在Date和LocalDate之间进行转换。
### Date转LocalDate
```
Date date = new Date();
Instant instant = date.toInstant();
ZoneId zoneId = ZoneId.systemDefault();

// atZone()方法返回在指定时区从此Instant生成的ZonedDateTime。
LocalDate localDate = instant.atZone(zoneId).toLocalDate();
System.out.println("Date = " + date);
System.out.println("LocalDate = " + localDate);

```

### LocalDate转Date
```
ZoneId zoneId = ZoneId.systemDefault();
LocalDate localDate = LocalDate.now();
ZonedDateTime zdt = localDate.atStartOfDay(zoneId);

Date date = Date.from(zdt.toInstant());

System.out.println("LocalDate = " + localDate);
System.out.println("Date = " + date);
```

## LocalDate和String互转

### LocalDate转String
```
LocalDate localDate = LocalDate.now();
String dateStr = localDate.format(DateTimeFormatter.ofPattern("yyyy-MM-dd"));
```

### LocalDateTime转String
```
LocalDateTime dateTime = LocalDateTime.now();

//使用预定义实例来转换
DateTimeFormatter fmt = DateTimeFormatter.ISO_LOCAL_DATE;
String dateStr = dateTime.format(fmt);
System.out.println("LocalDateTime转String[预定义]:"+dateStr);

//使用pattern来转换
//12小时制与24小时制输出由hh的大小写决定
DateTimeFormatter fmt12 = DateTimeFormatter.ofPattern("yyyy-MM-dd hh:mm:ss:SSS");
String dateStr12 = dateTime.format(fmt12);
System.out.println("LocalDateTime转String[pattern](12小时制):"+dateStr12);

DateTimeFormatter fmt24 = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss:SSS");
String dateStr24 = dateTime.format(fmt24);
System.out.println("LocalDateTime转String[pattern](24小时制):"+dateStr24);

//如果想要给12小时制时间加上am/pm,这样子做：
fmt12 = DateTimeFormatter.ofPattern("yyyy-MM-dd hh:mm:ss:SSS a");
dateStr12 = dateTime.format(fmt12);
System.out.println("LocalDateTime转String[pattern](12小时制带am/pm):"+dateStr12);
```

### String转LocalDate和LocalDateTime
```
String str = "2017-11-21 14:41:06:612";
DateTimeFormatter fmt = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss:SSS");
LocalDate date = LocalDate.parse(str, fmt);
LocalDateTime time = LocalDateTime.parse(str, fmt);
System.out.println("date:"+date);
System.out.println("time:"+time);
```

## 时区

Java 8中的时区操作被很大程度上简化了，新的时区类`java.time.ZoneId`是原有的`java.util.TimeZone`类的替代品。`ZoneId`对象可以通过`ZoneId.of()`方法创建，也可以通过`ZoneId.systemDefault()`获取系统默认时区：

```java
ZoneId shanghaiZoneId = ZoneId.of("Asia/Shanghai");
ZoneId systemZoneId = ZoneId.systemDefault();
```

`of()`方法接收一个“区域/城市”的字符串作为参数，你可以通过`getAvailableZoneIds()`方法获取所有合法的“区域/城市”字符串：

```java
Set<String> zoneIds = ZoneId.getAvailableZoneIds();
```

对于老的时区类`TimeZone`，Java 8也提供了转化方法：

```java
ZoneId oldToNewZoneId = TimeZone.getDefault().toZoneId();
```

有了`ZoneId`，我们就可以将一个`LocalDate`、`LocalTime`或`LocalDateTime`对象转化为`ZonedDateTime`对象：

```java
LocalDateTime localDateTime = LocalDateTime.now();
ZonedDateTime zonedDateTime = ZonedDateTime.of(localDateTime, shanghaiZoneId);
```

将`zonedDateTime`打印到控制台为：

```
2017-01-05T15:26:56.147+08:00[Asia/Shanghai]
```

`ZonedDateTime`对象由两部分构成，`LocalDateTime`和`ZoneId`，其中`2017-01-05T15:26:56.147`部分为`LocalDateTime`，`+08:00[Asia/Shanghai]`部分为`ZoneId`。

另一种表示时区的方式是使用`ZoneOffset`，它是以当前时间和**世界标准时间（UTC）/格林威治时间（GMT）**的偏差来计算，例如：

```java
ZoneOffset zoneOffset = ZoneOffset.of("+09:00");
LocalDateTime localDateTime = LocalDateTime.now();
OffsetDateTime offsetDateTime = OffsetDateTime.of(localDateTime, zoneOffset);
```


## DateTimeFormatter详解
DateTimeFormatter我们更多的是直接使用pattern来做转换，
其实这个类本身已经提供了一些预定义好的实例供我们使用。
下面把两者的具体释义和示例都贴出来供大家参考。

预定义
```
Predefined Formatters                       Formatter Description                                               Example
----------------------                      ----------------------                                              ------------
ofLocalizedDate(dateStyle)                  Formatter with date style from the locale                           '2011-12-03'
ofLocalizedTime(timeStyle)                  Formatter with time style from the locale                           '10:15:30'
ofLocalizedDateTime(dateTimeStyle)          Formatter with a style for date and time from the locale            '3 Jun 2008 11:05:30'
ofLocalizedDateTime(dateStyle,timeStyle)    Formatter with date and time styles from the locale                 '3 Jun 2008 11:05'
BASIC_ISO_DATE                              Basic ISO date                                                      '20111203'
ISO_LOCAL_DATE                              ISO Local Date                                                      '2011-12-03'
ISO_OFFSET_DATE                             ISO Date with offset                                                '2011-12-03+01:00'
ISO_DATE                                    ISO Date with or without offset                                     '2011-12-03+01:00'; '2011-12-03'
ISO_LOCAL_TIME                              Time without offset                                                 '10:15:30'
ISO_OFFSET_TIME                             Time with offset                                                    '10:15:30+01:00'
ISO_TIME                                    Time with or without offset                                         '10:15:30+01:00'; '10:15:30'
ISO_LOCAL_DATE_TIME                         ISO Local Date and Time                                             '2011-12-03T10:15:30'
ISO_OFFSET_DATE_TIME                        Date Time with Offset                                               '2011-12-03T10:15:30+01:00'
ISO_ZONED_DATE_TIME                         Zoned Date Time                                                     '2011-12-03T10:15:30+01:00[Europe/Paris]'
ISO_DATE_TIME                               Date and time with ZoneId                                           '2011-12-03T10:15:30+01:00[Europe/Paris]'
ISO_ORDINAL_DATE                            Year and day of year                                                '2012-337'
ISO_WEEK_DATE                               Year and Week                                                       '2012-W48-6'
ISO_INSTANT                                 Date and Time of an Instant                                         '2011-12-03T10:15:30Z'
RFC_1123_DATE_TIME                          RFC 1123 / RFC 822                                                  'Tue, 3 Jun 2008 11:05:30 GMT'
```

Pattern
```
All letters 'A' to 'Z' and 'a' to 'z' are reserved as pattern letters. The following pattern letters are defined:

  Symbol  Meaning                     Presentation      Examples
  ------  -------                     ------------      -------
   G       era                         text              AD; Anno Domini; A
   u       year                        year              2004; 04
   y       year-of-era                 year              2004; 04
   D       day-of-year                 number            189
   M/L     month-of-year               number/text       7; 07; Jul; July; J
   d       day-of-month                number            10

   Q/q     quarter-of-year             number/text       3; 03; Q3; 3rd quarter
   Y       week-based-year             year              1996; 96
   w       week-of-week-based-year     number            27
   W       week-of-month               number            4
   E       day-of-week                 text              Tue; Tuesday; T
   e/c     localized day-of-week       number/text       2; 02; Tue; Tuesday; T
   F       week-of-month               number            3

   a       am-pm-of-day                text              PM
   h       clock-hour-of-am-pm (1-12)  number            12
   K       hour-of-am-pm (0-11)        number            0
   k       clock-hour-of-am-pm (1-24)  number            0

   H       hour-of-day (0-23)          number            0
   m       minute-of-hour              number            30
   s       second-of-minute            number            55
   S       fraction-of-second          fraction          978
   A       milli-of-day                number            1234
   n       nano-of-second              number            987654321
   N       nano-of-day                 number            1234000000

   V       time-zone ID                zone-id           America/Los_Angeles; Z; -08:30
   z       time-zone name              zone-name         Pacific Standard Time; PST
   O       localized zone-offset       offset-O          GMT+8; GMT+08:00; UTC-08:00;
   X       zone-offset 'Z' for zero    offset-X          Z; -08; -0830; -08:30; -083015; -08:30:15;
   x       zone-offset                 offset-x          +0000; -08; -0830; -08:30; -083015; -08:30:15;
   Z       zone-offset                 offset-Z          +0000; -0800; -08:00;

   p       pad next                    pad modifier      1

   '       escape for text             delimiter
   ''      single quote                literal           '
   [       optional section start
   ]       optional section end
   #       reserved for future use
   {       reserved for future use
   }       reserved for future use
```
