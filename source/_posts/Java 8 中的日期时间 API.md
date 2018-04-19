---
title: Java 8 中的日期时间 API
date: 2017-09-11 09:52:02
tags:
- Java
categories:
- Java
---

> 本文内容基本是对《Java 8实战》这本书中关于新日期时间API的讲述的再整理。

# Java 8之前的日期、时间类的缺陷

使用过java8之前的日期和时间类（`Date`、`Calendar`、`DateFormat`）的同学一定对其易用性的缺失表示遗憾，我本人平时使用时，对它们有以下几点不爽：

1. `Date`和`Calendar`类的`Month`值是从0开始计算的，有点反人类。
2. `Date`和`Calendar`类是可变对象，给代码可维护性造成了困难。
3. `DateFormat`类的方法不是线程安全的，导致无法在线程间共享单例。只能每次使用时去new个新对象或是使用`ThreadLocal`机制避开多线程访问的隐患。
4. 无法完成有关时间段的计算：如计算两个日期之前相隔x年y月z天这种需求。

在寻找上述第4条中描述的需求的实现方案时，一位同事找到了优秀的类库*JodaTime*，而后我们也得知Java 8中吸取了*JodaTime*中的精髓，提供了新的日期时间API，它们在`java.time.*`包中。

# Java 8对日期和时间的建模方法

相关的类包括：`LocalDate`、`LocalTime`、`LocalDateTime`、`Instant`。这几个类在Java 8中分别表示日期（年月日）、时间（时分秒）、日期时间（年月日时分秒）、时间偏移量。

## 使用LocalDate
通过如下方法获取一个LocalDate对象并获取日期有关的字段值。

    LocalDate date = LocalDate.now();          //对系统当前日期建模
    LocalDate date1 = LocalDate.of(2017,9,9);  //指定年月日建模
    date.get(ChronoField.DAY_OF_WEEK);         //获取日期是星期几
    date1.get(ChronoField.DAY_OF_YEAR);        //获取日期是一年中的第几天

##使用LocalTime
通过类似方法创建LocalTime对象，并获取字段值。

    LocalTime time = LocalTime.now();                     //为当前时间建模
    LocalTime time2 = LocalTime.of(12,14,56);             //指定时分秒
    int hourOfDay = time.get(ChronoField.HOUR_OF_DAY);    //获取当前时间在一天内是第几个小时，0-23
    int SecondOfDay = time2.get(ChronoField.SECOND_OF_DAY); //获取当前时间在一天内是第几秒

##使用LocalDateTime
大家可把`LocalDateTime`类看成是`LocalDate`和`LocalTime`两者功能的合集，事实上我们可以通过组合日期和时间对象的方式构建`LocalDateTime`对象:

    LocalDateTime dt = LocalDateTime.now();           //获取系统当前日期时间
    LocalDateTime dt2 = LocalDateTime.of(date,time);  //用`LocalDate`和`LocalTime`对象拼装一个`LocalDateTime`对象
    LocalDateTime dt3 = LocalDateTime.of(2017,9,10,20,51,24); //对`2017年9月10日20点51分24秒`建模
    dt.get(ChronoField.DAY_OF_MONTH);                 //获取一月当中的第几天这个字段值

##使用Instant

Instant是以Unix元年时间（UTC时间1970年1月1日午夜时分）开始所经历的秒数进行建模的。使用方法：
    
    Instant instant1 = Instant.now();    //对当前时间建模
    Instant instant2 = Instant.ofEpochSecond(1505049200);    //用秒数构建一个Instant对象
    Instant instant3 = Instant.ofEpochSecond(1505049200,1_111_000); //用秒数+纳秒数构建一个对象
    instant1.get(ChronoField.MILLI_OF_SECOND);    //获取对象中的毫秒部分值
    instant2.getLong(ChronoField.INSTANT_SECONDS);    //获取对象中的秒数值
    instant2.getLong(ChronoField.NANO_OF_SECOND);    //获取对象中的纳秒部分的值

> _TIP_：单一的Instant无法转换为日期时间，必须加上相应的**时区**信息才可以，反之亦然。具体请参考下文中关于**时区**的小节。
> _TIP_：单一的Instant无法转换为日期时间，必须加上相应的**时区**信息才可以，反之亦然。具体请参考下文中关于**时区**的小节。

## 建模总结
这几个模型类有一些共性，总结如下：
* 都可以通过`now()`方法获得当前对象，或通过`of()`方法拼装对象
* 都可通过`get(TemporalField field)`方法获取对应的字段值。一般只需传入`ChronoField`类中的预定义的字段枚举即可。
  但是要注意：不同类支持的字段是不一样的。如：不能尝试获取`LocalTime`类的`DAY_OF_YEAR`字段，或尝试获取`LocalDate`类的`HOUR_OF_DAY`字段。否则你将收获异常-_-
* 这四个类的对象都是**不可变对象**，这意味着下文中对它们的修改事实上是创建了该对象的一个副本。

# 如何对日期、时间操作
回想一下，我们在工程中最经常做的关于日期和时间的计算是：
1. 在已有对象上进行字段**修改**或**加减**从而获得计算结果。
2. 把一个字符串解析成日期或时间对象；或将对象格式化输出为字符串。

本章对`LocalDate`、`LocalTime`、`LocalDateTime`、`Instant`这四个类的操作方法进行总结。

## 修改操作

### with方法

with方法的作用是通过重新设置一个字段的值来获取一个新的对象，如：

    LocalDate date = LocalDate.now();    //获取当前日期
    LocalDate date2 = date.withDayOfMonth(3);  //将日期改为3日
    LocalDate date3 = date2.withMonth(2);  //继续将月份改为2月

还有更通用的with方法的重载版本，它接受一个TemporalField作为要修改的字段：

    LocalDate date4 = date3.with(ChronoField.DAY_OF_WEEK,4);  //获取当周的星期四的日期对象

### plus、minus方法
这两个方法顾名思义，在原对象上增加或减少一个时间量，从而获取计算结果对象。
> 注意：此类方法有多个重载版本。因此，这个时间量可以是某个时间单位上的值，也可以是实现了`TemporalAmount`接口的对象，比如下文介绍的`Duration`和`Period`。

## 使用TemporalAdjuster
如果上述操作仍满足不了你对日期时间计算的需求，那么你可以使用`with()`方法的另一个重载版本，它接受一个`TemporalAdjuster`对象。该接口定义如下：

    @FunctionalInterface
    public interface TemporalAdjuster {
      Temporal adjustInto(Temporal temporal);
    }
也就是说你可以自己实现自己的计算过程。这还不算完，JDK中提供了工具类`TemporalAdjusters`，里面的静态方法实现了多个实用的TemporalAdjuster实现。如：

![image.png](http://upload-images.jianshu.io/upload_images/7838950-8baf0ec77e6405ac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 格式化和字符串解析

格式化方面，主要涉及的类是`DateTimeFormatter`，它是线程安全的。基本使用方法如下：

    DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd/MM/yyyy");
    LocalDate date1 = LocalDate.of(2014, 3, 18);
    String formattedDate = date1.format(formatter);
    LocalDate date2 = LocalDate.parse(formattedDate, formatter);

另外它还能通过ofPattern()的重载方法创建一个与Locale相关的Formatter。

DateTimeFormatterBuilder可以用来自主构建一个格式器。

# Java 8中的时间量（Duration、Period）

`Duration`用来记录与时间单位相关的时间量，例如"1天1小时30分20秒"。
`Period`用来记录日期单位相关的时间量，例如“两年3个月零10天”。
这两个类用来对“一段时间的长短”进行建模，因此它们很相似，也提供了一些共同的方法：

![image.png](http://upload-images.jianshu.io/upload_images/7838950-73930363d4c03199.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# Java 8中的时区

在Java 8中，`java.time.ZoneId`类用来表示一个时区。你可以通过这样的方式获取到`ZoneId`对象：

     ZoneId romeZone = ZoneId.of("Europe/Rome");
     ZoneId zoneId = TimeZone.getDefault().toZoneId();  //获取系统默认时区

当获取到了时区对象后，可以用它来：
1. 将`Instant`对象和`LocalDatetime`互相转换
2. 将`LocalDate`、`LocalTime`、`Instant`对象转换为`ZonedDateTime`对象，后者是日期时间加上时区信息后的模型。

